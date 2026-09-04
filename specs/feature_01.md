# Feature 01: Basic Pomodoro Timer

## Scope

One codebase, one architecture, three targets: Android, iOS, Web — via Expo / React Native (react-native-web for web). No separate native UIs per platform.

Frontend only for now. A backend may be added later behind an interface, so it can replace local logic without reworking the UI.

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

## Timekeeping contract (`core/timer.ts`)

Pure TypeScript, no React/React Native imports. Poll-only — no subscriptions or callbacks; the UI is responsible for re-reading on its own interval.

```ts
createTimer(durationMs: number): Timer

Timer {
  start(): void
  pause(): void
  reset(): void
  getRemainingMs(): number
  getState(): 'idle' | 'running' | 'paused' | 'completed'
}
```

Behavioral contract (this is what implementations must satisfy, not how to satisfy it):

- No internal interval/timer is kept — state and remaining time are computed fresh from wall-clock time on every call, not decremented by a background tick.
- `getRemainingMs()` never returns a negative number (clamp at 0), and reflects the frozen value while `idle`/`paused`.
- `getState()` must reach `completed` on its own the moment it (or `getRemainingMs()`) is queried after the deadline has passed — no external trigger, missed tick, or interval alignment required to detect it.
- Clock source: `Date.now()`. A manual system clock change or NTP resync mid-countdown can cause the display to jump or complete early/late. Accepted as a known v1 risk — not handled.

<details>
<summary>Design hint (non-normative — one valid way to implement the contract above)</summary>

Track two fields: `remainingAtLastActionMs` (remaining time as of the last Start/Pause/Reset) and `runningSinceTimestamp` (timestamp the current run started, or `null` if not running). Derive `getRemainingMs()` as `remainingAtLastActionMs` when not running, or `Math.max(0, remainingAtLastActionMs - (now - runningSinceTimestamp))` when running. Derive `getState()` from the same two fields plus whether `getRemainingMs()` is `0`. This is a suggestion, not a requirement — any implementation satisfying the contract above is acceptable.

</details>

## Actions

- `start()`: transitions to `running` if currently `idle` or `paused`; no-op if already `running` or `completed`.
- `pause()`: transitions to `paused` if currently `running`; no-op otherwise.
- `reset()`: always returns to `idle` with the full duration remaining, regardless of current state.

## Backgrounding / tab throttling

v1 does not attempt to handle backgrounding specially — no `AppState`/`visibilitychange` listeners, no forced resync on foreground. Two distinct failure modes follow from that, both accepted for v1:

- **Process survives (e.g. a backgrounded but not reclaimed web tab):** the display may show a stale value until the app is foregrounded and something re-renders it — not bounded to "one interval," since OS-level throttling (especially Android Doze/App Standby) can defer timers far longer than that. It self-corrects once read again; no data is lost.
- **Process is killed (common on both Android and iOS under memory pressure while backgrounded):** the entire in-memory `Timer` — including a `running` or `paused` session — is lost. Since v1 has no persistence, relaunching gives a fresh timer at `idle`/25:00, with no indication to the user that a session was in progress. This is consistent with the "no persistence" scope decision above, not a new gap, but is called out explicitly here since it's easy to mistake for a bug rather than a known limitation.

## UI requirements (`App.tsx`)

- Single screen, time displayed as `mm:ss`, computed from `getRemainingMs()`.
- A visible state indicator (e.g. a text label) distinct for each of `idle` / `running` / `paused` / `completed` — state must be readable at a glance, not only inferable from which buttons are enabled.
- Three buttons: Start, Pause, Reset.
- Start disabled when state is `running` or `completed`.
- Pause disabled when state is not `running`.
- Reset always enabled.
- A `useTimer()` hook re-renders the display on an interval (for display refresh only — actual remaining time always comes from `core/timer.ts`, never tracked separately in the UI).

## Acceptance criteria

- [ ] Unit tests (Jest) on `core/timer.ts` (using fabricated/mocked timestamps, not real `sleep`) cover: idle→running→paused→running, reset from every state, and remaining time reaching exactly 0 → state becomes `completed`.
- [ ] `getRemainingMs()` never returns a negative number, verified by a Jest test that reads past the deadline.
- [ ] The app builds and launches from a clean checkout on all three targets: Android, iOS, and Web.
- [ ] Manual smoke test on each of Android, iOS, and Web: Start counts down, Pause freezes the display, Reset returns to 25:00, letting it run to completion shows `00:00` and stops.
- [ ] On each platform, `idle` / `running` / `paused` / `completed` are visually distinguishable from the state indicator alone, without reading button enabled/disabled states.

## Deferred (not in v1, may come later)

- Persistence (e.g. `AsyncStorage`) behind a repository interface — would also address the process-kill-while-backgrounded gap above
- Session-complete notifications (e.g. `expo-notifications`)
- Navigation (e.g. `expo-router`) once there's more than one screen
- Backend/API, once added, replacing or backing the local repository interface
- Proactive foreground resync (`AppState`/`visibilitychange`)
