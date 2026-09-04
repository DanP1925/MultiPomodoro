# Architecture

## Scope

One codebase, one architecture, three targets: Android, iOS, Web — via Expo / React Native (react-native-web for web). No separate native UIs per platform.

Frontend only for now. A backend may be added later behind an interface, so it can replace local logic without reworking the UI.

## v1 scope (current)

Deliberately minimal:

- No persistence
- No session-complete notification
- No navigation (single screen)

## Structure

- `core/timer.ts` — pure TypeScript state machine, no React/React Native imports. States: `idle`, `running`, `paused`. Tracks remaining time via a start timestamp + duration (not a decrementing counter), so it stays correct across re-renders or the app losing focus.
- `App.tsx` — single screen. A `useTimer()` hook re-renders off `core/timer.ts` on an interval (for display only, not for timekeeping), with start/pause/reset controls.

## Deferred (not in v1, may come later)

- Persistence (e.g. `AsyncStorage`) behind a repository interface
- Session-complete notifications (e.g. `expo-notifications`)
- Navigation (e.g. `expo-router`) once there's more than one screen
- Backend/API, once added, replacing or backing the local repository interface
