# Feature 1: Project Scaffolding

## Scope

Prerequisite infrastructure for every feature that follows — initializes the Expo/TypeScript project, test tooling, and the feature-based folder convention. No app behavior of its own; nothing here is timer-specific.

## Tasks

- Initialize an Expo + TypeScript project at the repo root, with `strict: true` in `tsconfig.json` — decided upfront since turning strict mode on later, after code exists, is much more painful.
- Pin the toolchain: `.nvmrc` for the Node version (current LTS), `engines` field in `package.json`, and npm as the package manager (matches Expo's own default `npx create-expo-app` flow) — avoids "works on my machine" drift across the three platform tracks.
- Create the `features/` directory convention (self-contained modules; no importing between feature folders; shared code, if any, goes in a dedicated `shared/` folder — see `specs/02-core-timer.md` for the first consumer, `features/timer/`).
- Install and configure Jest (with a mockable/fake clock, since `features/timer/timer.ts` will need to test timestamp-based logic without real `sleep`).
- Set up linting via Expo's official tooling (`npx expo lint`, which installs `eslint-config-expo`) rather than a hand-rolled ESLint config.
- Set up Prettier for formatting, with `eslint-config-prettier` to disable ESLint's own style rules so the two don't conflict — ESLint stays responsible for code correctness, Prettier for formatting.
- Set up a pre-commit hook (`husky` + `lint-staged`) running ESLint and Prettier against staged files, so lint/format issues are caught before they land in a commit rather than only in CI or on demand.
- Basic project hygiene: `.gitignore` for Expo/RN/Node artifacts, `package.json` scripts for running, testing, linting, and formatting.

## Acceptance criteria

- [ ] `npx expo start` runs successfully from a clean checkout with no app-specific code yet (default template screen is fine).
- [ ] `tsconfig.json` has `strict: true`.
- [ ] `.nvmrc` and `package.json`'s `engines` field both specify the same Node version; a clean `npm install` on that version succeeds.
- [ ] `npm test` (or equivalent) runs Jest successfully against a trivial placeholder test.
- [ ] `npm run lint` (or equivalent) runs ESLint successfully with no errors against the scaffolded project.
- [ ] `npm run format` (or equivalent) runs Prettier successfully against the scaffolded project, with no conflicting rules between ESLint and Prettier.
- [ ] Committing a file with a deliberate lint/format issue is blocked by the pre-commit hook until fixed.
- [ ] `features/` directory exists and is documented (e.g. in this file or the README) as the folder convention for all subsequent features.
- [ ] `.gitignore` excludes `node_modules/`, Expo/Metro caches, and platform build artifacts.

## Out of scope (deferred)

- Any timer logic or UI — see Feature 2 (core timer logic) and Feature 3 (Android UI)
- Zustand setup — not needed until a feature requires cross-screen shared state
- Navigation (`expo-router`) — not needed until there's more than one screen
- CLAUDE.md — per project decision, written once this feature ships and there's real build/test/convention content to document
