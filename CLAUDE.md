# cumplo-authenticator

## Overview
Google Cloud Function (Gen2, Node.js 18) that uses Puppeteer to perform browser-based login on
Cumplo's website and return a session cookie to callers.

## Build & Run

Install dependencies:
```
npm install
```

Compile TypeScript:
```
npm run compile
```

Compile and run locally (requires `.env`):
```
npm run locally
```

Serve via Functions Framework (for manual end-to-end testing, requires `.env`):
```
npm start
```

## Lint & Format

ESLint (includes Prettier rules via `eslint-plugin-prettier`):
```
npx eslint src/
```

Auto-format with Prettier, then lint:
```
npx prettier --write src/ && npx eslint src/
```

## Architecture

- **Entry point**: `src/index.ts` — exports `getCumploAuthenticationCookie` as the Cloud Function handler.
- **Auth flow**: `src/integrations/cumplo.ts` launches a headless Chromium browser, fills the
  login form at `CUMPLO_LOGIN_URL`, waits for the session cookie response, and returns the cookie value.
- **Logging**: `winston` + `@google-cloud/logging-winston` for structured Cloud Logging output.
- **Deployment**: `cloudbuild.json` drives `gcloud functions deploy` (Gen2, `us-central1`).

## Environment Variables

Local dev requires a `.env` file (not committed):
```
EMAIL=...
PASSWORD=...
CUMPLO_LOGIN_URL=...
COOKIE_SETTER_URL=...
SESSION_COOKIE_NAME=...
ENVIRONMENT=DEVELOPMENT
```

Setting `ENVIRONMENT=DEVELOPMENT` triggers a local test run in `src/index.ts`; omit it in
production so only the exported Cloud Function handler is active.

## Gotchas

- **No test suite** — jest/vitest/mocha are not configured; tests are a known gap in this repo.
- **Puppeteer's Chromium binary** — the `gcp-build` script (`npm run build && node node_modules/puppeteer/install.js`)
  installs the browser binary as a Cloud Build step. Cloud Build does not bundle Chromium;
  skipping `gcp-build` causes the function to crash at runtime.
- **Dual lock files** — both `package-lock.json` and `yarn.lock` exist. The `package.json`
  scripts use `npm`; prefer `npm install` to stay consistent.
- **`build/` is generated output** — never commit it; it is excluded by `.eslintignore`.
- **PR title convention**: conventional commit subject starting with an uppercase letter,
  enforced by `.github/workflows/pr-title.yml` (e.g. `chore: Add CLAUDE.md`).

## Before Committing

After making changes, run `npx prettier --write src/ && npx eslint src/` and ensure they pass.
Check no hardcoded secrets — before committing.
