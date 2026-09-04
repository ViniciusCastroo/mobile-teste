---
name: playwright-verification-notes
description: "Technical gotchas for verifying darwin's frontend with real Playwright browser automation"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8c8c9ea6-135e-4fd3-9789-972d864f95ee
  modified: 2026-09-01T19:40:37.259Z
---

Two gotchas when writing Playwright scripts to verify UI changes in this repo (dev server at localhost:5173, dev login dev-preview@darwin.local):

1. **Auth is in-memory only** (`frontend/src/store/authStore.js` has no persist middleware, no localStorage). After logging in, do NOT `page.goto()` to a new route — a hard navigation reloads the SPA and drops the session, bouncing back to `/login`. Instead click through the app's own nav links (e.g. `page.getByRole('link', { name: 'Conhecimento' })`) so React Router does a client-side transition and keeps the in-memory auth state.

2. **Migrations must be applied manually to the persistent dev DB.** `makemigrations` inside the darwin-backend container only creates the migration file — it does not get applied automatically. pytest's own test DB gets migrated fresh on every test run, so the backend test suite can pass even when the dev DB itself is still missing the migration (surfaces as a `ProgrammingError`/500 when hitting the API through the real browser). Always run `python manage.py migrate <app>` in the dev container after adding a migration, before doing browser verification.

Also: after any Playwright test-data creation this session, the working agreement is to leave test data in place (not clean it up) — see the earlier "Recriar e deixar os dados de teste" decision.

3. **Dev DB is missing later migrations (2026-09-01).** Creating/saving a `User` or `Group` through the ORM in the dev container fails with `ProgrammingError: relation "core_systemsettings" does not exist` — a signal handler (default_user_group) queries that table. So you can't quickly seed a limited-permission user to preview role-gated screens without first running `manage.py migrate`. Raw SQL (`connection.cursor()`) bypasses the signal if you need to undo a half-created Group. The dev DB has no auth Groups at all; the "grupo demo" the user refers to lives in their staging/prod env.
