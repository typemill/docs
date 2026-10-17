Integrate a test folder and setup playwright for api and e2e tests. Use this as your initial setup:
- tests/unit/ — PHPUnit (implement later)
- tests/api/ — Playwright API tests
- tests/e2e/ — Playwright browser tests
- tests/fixtures/ — shared test support
- Root-level phpunit.xml and playwright.config.js
- Separate Composer and npm scripts

Migrate the old cypress tests from /cypress to playwright e2e tests if possible (tests are very old and might fail).

---

# Review & Implementation Plan (2026-10-12)

## 1. Review of the current state

Findings from exploring the codebase:

- **No implementation plan existed** in this task — created below.
- **Environment**: App is live at `http://localhost/typemill/` (nginx → Apache, PHP 8.5, Docker-style setup). Node v25 / npm 11 available for Playwright.
- **Old Cypress suite** (`/cypress`, Cypress v9-era, plain JS):
  - 6 specs: `01-system-setup-signup`, `02-initial-frontend`, `03-system-settings`, `04-theme-settings`, `05-blox-editor`, `99-login`.
  - Cypress is **not even in devDependencies**; npm scripts `e2e` / `install-and-e2e` assume a global install. `fs-extra` is only used by `cypress/plugins/index.js`.
  - Specs are **stateful and order-dependent** (shared cookies via `Cypress.Cookies.preserveOnce`, sequential spec numbering).
  - `01-system-setup-signup` uses a **destructive** `resetSetup` task that deletes `settings/users/` and `settings/settings.yaml` of the *live* instance — must not run unguarded against a real installation.
  - `99-login.spec.js` line 64 contains a syntax bug (`passwordhave.value`) — confirms tests cannot be ported blindly.
  - Hard-coded credentials `trendschau` / `password` in specs and `support/commands.js`.
- **API surface**: `system/typemill/routes/api.php` exposes `/api/v1/*` behind `ApiAuthentication` (HTTP Basic Auth, or session via `X-Session-Auth` header; optional trusted IP/host restrictions). Public guest endpoints exist (`/api/v1/navigation`, `/api/v1/article/*`). Well suited for Playwright API tests with a `request` context.
- **composer.json**: no `scripts`, no `require-dev`, custom vendor-dir `system/vendor`. PHPUnit itself belongs to the separate task `todo-php-unit-setup.md` (currently a stub) — this task only scaffolds `tests/unit/` + `phpunit.xml`.
- **.gitignore**: `cypress` and `ai/tasks` are ignored; a new `tests/` folder will be tracked (desired). Playwright artifacts must be added to `.gitignore`.
- **.dockerignore**: whitelists only runtime dirs — `tests/` and `playwright.config.js` are excluded from the image automatically. Verify, no change expected.

## 2. Clarifications / improvements to the task description

- `tests/fix shared test support` → interpreted as **`tests/fixtures/`** for shared helpers, seeds, and test data.
- Playwright tests will be **plain JavaScript** (matches old suite and codebase style; no TS build step).
- The ambiguous "Separate Composer and npm scripts" → npm scripts drive Playwright (`test:api`, `test:e2e`); a Composer script `test:unit` is added as a documented stub that becomes functional once `todo-php-unit-setup.md` installs PHPUnit.
- Cypress removal is part of this task, executed **only after** the Playwright suite is green.

## 3. Implementation plan

### Phase 1 — Scaffolding
1. `npm i -D @playwright/test` and `npx playwright install chromium` (+ `--with-deps` if the environment lacks browser libs).
2. Create `playwright.config.js` (root):
   - `baseURL` from env `TYPEMILL_BASE_URL`, default `http://localhost/typemill/`.
   - Two projects: `api` (testDir `tests/api`, no browser) and `chromium` (testDir `tests/e2e`).
   - `workers: 1`, `fullyParallel: false` (legacy suites are stateful; relax later per spec).
   - `trace`/`screenshot`/`video` retain-on-failure; reporters `list` + `html`.
   - No `webServer` block (app is served by the existing stack); add a lightweight `globalSetup` that pings `baseURL` and fails with a clear message if unreachable.
3. Create structure: `tests/api/`, `tests/e2e/`, `tests/fixtures/`, `tests/unit/` (`.gitkeep`), root `phpunit.xml` skeleton (testsuite → `tests/unit`, bootstrap → `system/vendor/autoload.php`).
4. `.gitignore`: add `playwright-report/`, `test-results/`, `blob-report/`, `tests/fixtures/.auth/`.

### Phase 2 — Shared fixtures (`tests/fixtures/`)
5. `credentials.js` — admin user/pass from env (`TYPEMILL_TEST_USER` / `TYPEMILL_TEST_PASSWORD`), defaults `trendschau` / `password`.
6. `instance.js` — plain Node helpers `backupSettings()` / `restoreSettings()` (snapshot `settings/settings.yaml` + `settings/users/`) used by settings-mutating suites via `beforeAll`/`afterAll`. The destructive Cypress `resetSetup` task is **not ported**.
7. `auth.js` — login once via UI, save `storageState` into `tests/fixtures/.auth/`, reused by e2e specs; helper to build a Playwright `request` context with HTTP Basic Auth for API tests.

### Phase 3 — Migrate Cypress → Playwright e2e (order: safe → fragile)
8. `99-login.spec.js` → `tests/e2e/login.spec.js` (fix the line-64 selector bug).
9. `02-initial-frontend.spec.js` → `tests/e2e/frontend.spec.js`.
10. `03-system-settings` + `04-theme-settings` → serial describes wrapped in settings backup/restore so the live instance is left untouched.
11. ~~`01-system-setup-signup`~~ → **NOT migrated** (destructive; decision: skip entirely).
12. `05-blox-editor.spec.js` → port last; if the Vue editor UI drifted too much, port what's stable and mark the rest `test.skip` with a note.

Mapping notes: `cy.get('form').submit()` → `locator.evaluate(f => f.submit())` (parity: bypasses HTML5 validation, exercises server-side validation); `cy.getCookie` → `context.cookies()`; `cy.url().should('include')` → `expect(page).toHaveURL()`; `cy.get().type()` → `locator.fill()`.

### Phase 4 — API tests (`tests/api/`)
13. First meaningful specs (read-only by default):
    - Public guest endpoint: `GET /api/v1/navigation` returns 200 + JSON.
    - Negative auth: protected endpoint (e.g. `GET /api/v1/settings`) returns 401 without credentials.
    - Basic auth: `GET /api/v1/settings` with admin credentials returns 200.
    - Keep mutating API tests out of scope unless they restore state.

### Phase 5 — Scripts & cleanup
14. `package.json`: replace `e2e` / `install-and-e2e` with `test` (all), `test:api`, `test:e2e`, `test:e2e:ui` (`--ui`); remove `fs-extra` devDep (only used by Cypress plugin file).
15. `composer.json`: add `"scripts": { "test:unit": "phpunit -c phpunit.xml" }` (stub; active after the PHPUnit task).
16. Delete `/cypress` and `cypress.json` **only after** the Playwright suite is green.

### Phase 6 — Verification
17. `npm run test:api` and `npm run test:e2e` green against `http://localhost/typemill/`.
18. Verify `settings/` and `settings/users/` are unchanged after settings-mutating suites ran (backup/restore works).

## 4. Decisions (confirmed 2026-10-12)

1. **Base URL**: `http://localhost/typemill/` as default, overridable via `TYPEMILL_BASE_URL` env. ✅
2. **Browsers**: Chromium only. ✅
3. **Destructive setup test**: **NOT migrated** — `01-system-setup-signup` is skipped entirely; the `resetSetup` task is not ported. ✅
4. **Cypress removal**: delete `/cypress`, `cypress.json`, old npm scripts and `fs-extra` after the Playwright suite is green. ✅
5. **Language**: plain JavaScript. ✅

## 5. Acceptance criteria

- `npx playwright test --list` discovers api + e2e specs; `npm run test:api` and `npm run test:e2e` pass.
- Settings-mutating suites leave the instance untouched (settings/users restored via backup/restore).
- `tests/`, `playwright.config.js`, `phpunit.xml` skeleton committed-ready; old Cypress artifacts removed; `.gitignore` covers Playwright output.