# Feature 3: Android UI

## Scope

Wraps Feature 2's core timer logic (`features/timer/timer.ts`) in a single-screen UI, built and verified running locally on Android via Expo (Expo Go or a dev build). Not deployed to the Play Store yet — see Feature 6.

## Depends on

- Feature 2 (`specs/02-core-timer.md`): the timer's behavioral contract — `start()`, `pause()`, `reset()`, `getRemainingMs()`, `getState()`.

## UI requirements (`features/timer/TimerScreen.tsx`)

This is the shared UI baseline — Features 4 (iOS) and 5 (Web) reuse this same code and reference back here rather than redefining it.

- Single screen, time displayed as `mm:ss`, computed from `getRemainingMs()`.
- A visible state indicator (e.g. a text label) distinct for each of `idle` / `running` / `paused` / `completed` — state must be readable at a glance, not only inferable from which buttons are enabled.
- Three buttons: Start, Pause, Reset.
- Start disabled when state is `running` or `completed`.
- Pause disabled when state is not `running`.
- Reset always enabled.
- A `useTimer()` hook re-renders the display on an interval (for display refresh only — actual remaining time always comes from `features/timer/timer.ts`, never tracked separately in the UI).

## Android-specific behavior

- **If the process is killed while backgrounded** (common under memory pressure or Doze/App Standby): the in-memory timer is lost entirely. Since there's no persistence yet (deferred, see Feature 2), relaunching resets to `idle`/25:00 with no indication a session was in progress. This is an accepted v1 risk, not a bug — call it out during testing so it isn't mistaken for one.
- **If the process survives backgrounding** (briefly switched away, not reclaimed): the display may be stale until the app is foregrounded and something re-renders it. It self-corrects immediately on the next read, per Feature 2's timekeeping contract — no additional handling needed here.

## Acceptance criteria

- [ ] App builds and launches on Android from a clean checkout (Expo Go or a dev build).
- [ ] Manual smoke test: Start counts down, Pause freezes the display and Start resumes correctly, Reset returns to 25:00 from any state, letting it run to completion shows `00:00` and stops.
- [ ] `idle` / `running` / `paused` / `completed` are visually distinguishable via the state indicator alone, without reading button enabled/disabled states.
- [ ] Manually verify backgrounding: background the app mid-countdown long enough to risk process death (or force-stop it), relaunch, and confirm it resets to `idle` cleanly with no crash — matching the documented accepted risk above, not surprising anyone during testing.

## Out of scope (deferred)

- Deploying to the Play Store (Feature 6)
- Persistence, session-complete notifications, navigation (see Feature 2's deferred list)
- Proactive foreground resync (`AppState` listeners)
