# Feature 2: Core Timer Logic

## Scope

The platform-agnostic timer engine (`features/timer/timer.ts`) that Features 3-5 (Android, iOS, Web UI) will each build a screen against. Pure TypeScript, no UI, no platform target of its own.

One codebase, one architecture, three targets overall: Android, iOS, Web — via Expo / React Native (react-native-web for web). No separate native UIs per platform. Frontend only for now; a backend may be added later behind an interface, so it can replace local logic without reworking the UI.

## v1 scope (current)

Deliberately minimal:

- No persistence
- No session-complete notification
- No navigation (single screen)
- Duration is hardcoded (25:00), not user-editable

## Functional requirements

- Default duration: 25:00 (minutes:seconds), fixed for v1.
- Start begins counting down from the current remaining time.
- Pause freezes the countdown at the current remaining time.
- Resume (Start again after Pause) continues from where it was frozen.
- Reset returns remaining time to the default duration and stops any countdown, regardless of current state.
- When remaining time reaches zero: state becomes `completed`, display shows `00:00`, and it stays there until Reset. No auto-transition, no notification (deferred).
- Start, Pause, and Reset are idempotent: calling one when it doesn't apply (e.g. Pause while already paused, Start while already running) is a no-op, not an error. Rapid repeated presses have no cumulative effect.

## Timekeeping contract (`features/timer/timer.ts`)

Pure TypeScript, no React/React Native imports. Poll-only — no subscriptions or callbacks; the UI is responsible for re-reading on its own interval.

```ts
const DURATION_MS = 1_500_000 // 25:00, hardcoded for v1 — see Functional requirements

createTimer(): Timer

Timer {
  start(): void
  pause(): void
  reset(): void
  getRemainingMs(): number
  getState(): 'idle' | 'running' | 'paused' | 'completed'
}
```

`createTimer()` takes no arguments — the 25:00 duration is a hardcoded internal constant, not a caller-supplied parameter. This matches "fixed for v1" in Functional requirements literally: there is no external input to validate or guard against (no negative/zero-duration case to handle), since nothing outside this module chooses the duration. Revisit this signature (e.g. `createTimer(durationMs: number)`) only if/when a later feature makes duration configurable.

Behavioral contract (this is what implementations must satisfy, not how to satisfy it):

- No internal interval/timer is kept — state and remaining time are computed fresh from wall-clock time on every call, not decremented by a background tick.
- `getRemainingMs()` never returns a negative number (clamp at 0), and reflects the frozen value while `idle`/`paused`.
- `getState()` must reach `completed` on its own the moment it (or `getRemainingMs()`) is queried after the deadline has passed — no external trigger, missed tick, or interval alignment required to detect it.
- Clock source: `Date.now()`. A manual system clock change or NTP resync mid-countdown can cause the display to jump or complete early/late. Accepted as a known v1 risk — not handled.

<details>
<summary>Design hint (non-normative — one valid way to implement the contract above)</summary>

Track two fields: `remainingAtLastActionMs` (remaining time as of the last Start/Pause/Reset, initialized to `DURATION_MS`) and `runningSinceTimestamp` (timestamp the current run started, or `null` if not running). Derive `getRemainingMs()` as `remainingAtLastActionMs` when not running, or `Math.max(0, remainingAtLastActionMs - (now - runningSinceTimestamp))` when running. Derive `getState()` from the same two fields plus whether `getRemainingMs()` is `0`. This is a suggestion, not a requirement — any implementation satisfying the contract above is acceptable.

</details>

## Actions

- `start()`: transitions to `running` if currently `idle` or `paused`; no-op if already `running` or `completed`.
- `pause()`: transitions to `paused` if currently `running`; no-op otherwise.
- `reset()`: always returns to `idle` with the full duration remaining, regardless of current state.

## Acceptance criteria

- [ ] Unit tests (Jest) on `features/timer/timer.ts` (using fabricated/mocked timestamps, not real `sleep`) cover: idle→running→paused→running, reset from every state, and remaining time reaching exactly 0 → state becomes `completed`.
- [ ] `getRemainingMs()` never returns a negative number, verified by a Jest test that reads past the deadline.

## Out of scope (deferred)

- UI of any kind — see Feature 3 (Android, first implementation), reused by Features 4 (iOS) and 5 (Web)
- Platform-specific backgrounding behavior (process death, tab throttling, etc.) — the contract above guarantees the core self-corrects with no missed-tick dependency, but what the _user_ experiences on each OS is documented per-platform in Features 3-5
- Persistence (e.g. `AsyncStorage`) behind a repository interface
- Session-complete notifications (e.g. `expo-notifications`)
- Navigation (e.g. `expo-router`) once there's more than one screen
- Backend/API, once added, replacing or backing the local repository interface
- Proactive foreground resync (`AppState`/`visibilitychange`) — platform UI concern, see Features 3-5
