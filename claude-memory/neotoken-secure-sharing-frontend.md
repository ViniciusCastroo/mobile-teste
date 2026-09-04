---
name: neotoken-secure-sharing-frontend
description: "New frontend for the S3-only \"secure sharing\" use case (apps/shares) — status and open decisions"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3df63951-d6a8-49c2-bb6c-52e9fa20e7c4
  modified: 2026-09-03T15:15:53.742Z
---

The user is building a dedicated frontend for the secure file-sharing / "neotoken" use
case, which uses only the S3 integration (`apps/shares` + `core/s3`, merged to main in
`962ab4f`). Distinct use case from scan/upload and database.

Design mockup: `~/Downloads/quadro-neo-token/` (Claude Design canvas, `.dc.html`) — 11
screens for `/compartilhar`, `/compartilhamentos`, `/compartilhar/<id>`, and the public
`/acesso/<token>` recipient portal.

**Architecture (updated 2026-09-03):** this shipped as a SEPARATE app, `frontend-s3/`
(own Vite project, `jokr-s3-frontend`), NOT a section of `frontend/`. It is S3-only —
the user is firm on that ("apenas no S3"). Runs as compose service `frontend-s3`
(host port via `S3_FRONTEND_PORT`, default 5175). Reuses the app's neutral gray +
dark-mode vocabulary but takes jokr blue `#1B7FB8` as its own primary identity
(the main `frontend/` keeps brand-green). Recipient portal `/acesso/<token>` still
intended as a public, unauthenticated surface.

**Design canvas (2026-09-03):** Claude Design canvas "Compartilhamento Seguro S3"
at https://claude.ai/code/artifact/ec32c44b-b31e-4907-82e1-2e959d6ed689 — static
mockups, 19 artboards, 6 pages (Operador mobile / Operador desktop / Upload e login /
Destinatário público / Tema escuro / Direções descartadas). Working files in
`frontend-s3/design-canvas/` (`*.dc.html` + `canvas.json`); re-seed with the `design`
skill helper (contract 0.1.31) to update. `Main.dc.html` = mobile share list.

**Chosen visual direction: "Google Drive / Material 3"** (2026-09-03, 3rd pass).
History: v1 generic → rejected "cara de IA"; v2 "Terminal/Mono" → rejected as
niche/confusing; **v3 = Material Design 3 / Google Drive look — accepted direction.**
System: `Roboto` 400/500/700 (deliberate, to match the Drive reference); white
surfaces with soft elevation (`0 1px 2px rgba(60,64,67,.3), 0 1px 3px 1px rgba(60,64,67,.15)`),
NOT heavy borders; radii 8px cards / 16–24px containers / full pills for chips+buttons;
Google blue `#0B57D0`, app bg `#F8FAFD`, text `#1F1F1F`/`#5F6368`, green `#188038`,
red `#C5221F`, amber banner `#FEF7E0`/`#E37400`. Dark (M3): bg `#131314`, surface
`#1E1F20`, blue `#A8C7FA`. Fonts via Google Fonts `<link>`.
**Key UX mapping to Drive patterns:** ShareList = Drive file list (type icon + tonal
status pill); **"Compartilhar" = Drive's Share sheet** — add people + a per-person
**policy dropdown** (mascarado / em claro) replacing Viewer/Editor; ShareDetail = the
Drive info "i" panel (Quem tem acesso / Detalhes / Atividade); audit = Activity feed
grouped by day, denials in red; upload = Drive upload toast (bottom card, progress →
green check). Mobile: FAB "Novo", bottom sheet for Compartilhar.
**Mobile-first** — mobile artboards primary, desktop as the expansion (nav rail + search).
**Brand is NeoToken only** — no "jokr"/"joker" anywhere (user was firm).

**IMPLEMENTED in React (2026-09-03), operator area** — `frontend-s3/src` reskinned to
this style, direction approved by user. Changes: `tailwind.config.js` Material palette
(`primary` blue ramp #0B57D0, `success`/`danger`/`warn`, `surface`/`outline` tints,
`shadow-e1`/`e2`, Roboto as `font-sans`); Roboto `<link>` in `index.html`;
`components/shares/styles.ts` reworked (pill buttons, elevation cards, tonal chips);
`SharesShell` = nav rail + search bar + account menu + mobile FAB/chips; new
`FileTypeIcon` + `Avatar`; `ShareList` = Drive row list; `ShareDetail` = elevation cards
+ right info rail; `AccessAuditTable` = day-grouped activity feed (was a wide table);
`ClearTypeWarning` = amber Material banner; `Dropzone`/`S3Upload` = Material dropzone +
bottom-right upload toast; `Login`/`Register` reskinned; `Logo.tsx` now single
`neotoken-light.svg` + `dark:invert` (dropped legacy `jokr-dark.svg` import). All 26
vitest pass, `tsc -b` + `vite build` + eslint clean (2 pre-existing context warnings).
Test contracts kept: ShareCreate CPF/Nome checkboxes carry `aria-label`; ShareList row
renders `typesLabel` as its own node ("CPF · NOME"); only one status badge on ShareDetail
(in the sidebar). Run tests: `docker compose exec -T frontend-s3 npx vitest run`.

**Refinement (2026-09-03, per user):**
- **Strict 3-colour palette** in `tailwind.config.js`: neutral greys + primary blue
  `#0B57D0` (actions + all "good"/live states) + danger red `#C5221F` (destructive +
  all "bad" states + the clear-type consequence). No green, no amber — `success`/`warn`
  aliases now point at blue/red. Avatars are single neutral grey (was multi-hue). File
  tiles: PDF red tint, all others blue tint.
- **Less rounded**: radius scale — controls `rounded-lg` (8px), containers `rounded-xl`
  (12px), pills/status `rounded-md` (6px), avatars/icon-circles stay round. Dropped all
  `rounded-full` buttons, `rounded-2xl`/`3xl` cards.
- **Verified against live backend** (`:8002`): register→login→`/auth/me/`→
  `/files/supported-formats/`→`GET`+`POST /shares/` all 200/201; CORS preflight returns
  `allow-origin: http://192.168.130.55:5175` + credentials + PATCH/DELETE. Frontend API
  contracts match. Responsive structure checked (md/lg breakpoints, no fixed-width
  overflow traps); no in-browser pixel pass done here.

**Recipient portal `/acesso/<token>` — BUILT (2026-09-03).** Route added in `App.tsx`
(public, outside `RequireAuth`). New files: `src/api/access.ts` (own axios instance,
`baseURL ${API_BASE_URL}/access`, `withCredentials: true`, NO auth interceptors;
`toAccessError` flattens 410→unavailable / 401→code_invalid / 429 / 409 / network;
`runAccessDownload` = prepare→poll→blob-download), `src/components/CodeInput.tsx`
(6-box OTP, auto-advance + paste), `src/pages/Access.tsx` (Material/Drive shell,
phases loading→code→files→unavailable→error). Backend endpoints already existed at
`/api/v1/access/<token>/` (`AccessLinkView` etc. in `apps/shares/views/access.py`),
mounted only when `SHARES_ENABLED`. Session is an httponly `jokr_access` cookie
(SameSite=Lax, path=/) — works cross-port on the same host/IP.
Full chain verified against the running API: open link → "Seu código de acesso" email
→ verify (204 + cookie) → list files → prepare (202) → poll → download 200 with the
per-recipient masking applied (`CPF: 529.***.***-25`, name in clear). 26/26 vitest,
tsc + vite build + eslint clean.
Still a single bundle with the operator app (not the separate minimal bundle the
original plan wanted) — fine for now; split later if the public surface needs it.

**Delivered (2026-08-28), operator area, first slice:**
`frontend/src/api/shares.ts` (+ test), `components/shares/` (styles, format, SharesShell,
ShareStatusBadge, FileStatusBadge, ClearTypeWarning, FileManager, RecipientManager,
AccessAuditTable), `pages/ShareList|ShareCreate|ShareDetail.tsx` (+ tests), routes in
`App.tsx`. 24 new tests, full suite green (235), eslint clean. NB: `npm run build`
(`tsc -b`) has 2 PRE-EXISTING errors on main (Scan.tsx, FolderPicker.test.tsx) unrelated
to this work; `vite build` alone succeeds.

**Decisions made (2026-08-28):**
1. Recipient portal deploy: **own subdomain** (`acesso.<domain>`), separate minimal bundle
   — recommended a 2nd Vite entrypoint in `frontend/` (`frontend/access/`), shared tooling,
   isolated origin, deploy only `dist/access/`. Rationale: origin isolation for the
   unauthenticated surface, mirrors `access_urls.py`, small bundle for one-time visitors.
2. `SHARES_ENABLED` in frontend: **runtime, dedicated `GET /api/v1/config/`** (AllowAny),
   NOT `/health/` (liveness probe) and NOT a `VITE_` build var (drifts across envs when the
   same bundle is promoted). ~15-line backend add, not done yet.
3. ShareDetail layout: **1b — two columns with a sticky right sidebar** (status, deadline
   countdown, tokenized-type chips, counts, actions). Shipped: `ShareSidebar.tsx` +
   refactored `ShareDetail.tsx`. Full suite green (235).
4. Clear-type warning: **keep the inline red banner** (`ClearTypeWarning`, already shipped).

**Not built yet:** the `/api/v1/config/` endpoint + `useFeatures()` gating; recipient portal
(`/acesso/<token>`); "reenviar emails" (needs a new backend endpoint — `send/` refuses a
non-draft); audit CSV export; pagination on ShareList / audit table.
