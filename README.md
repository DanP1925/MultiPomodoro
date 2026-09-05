# MultiPomodoro

A Pomodoro timer app built to test and learn different technologies — not intended for production use.

## Goal

Single codebase (Expo / React Native) targeting Android, iOS, and Web. A backend may be added later; for now this is frontend-only.

## Status

Feature 1 (project scaffolding) is done. See `specs/` for the feature roadmap.

## Development

- Node version is pinned via `.nvmrc` / `engines` in `package.json`; use that version and npm as the package manager.
- `npm start` — run the Expo dev server.
- `npm test` — run Jest.
- `npm run lint` — run ESLint (`expo lint`).
- `npm run format` / `npm run format:check` — Prettier write / check.
- A pre-commit hook (husky + lint-staged) runs ESLint and Prettier on staged files.
- `features/` holds one self-contained module per feature; feature folders don't import from each other. Shared code goes in a `shared/` folder once more than one feature needs it.

## Release plan

Versioning follows semver; `1.0.0` means Android, iOS, and Web are all deployed. Roadmap:

1. Project scaffolding
2. Core timer logic (platform-agnostic)
3. Android UI (local)
4. iOS UI (local)
5. Web UI (local)
6. Deploy Android
7. Deploy iOS
8. Deploy Web

**The first available version of the app will be Android** (features 1 → 2 → 3 → 6). After that, iOS (4 → 7) and Web (5 → 8) are built in parallel.
