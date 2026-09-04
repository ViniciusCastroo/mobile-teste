---
name: joker-ports-vs-darwin
description: "joker runs on default ports with darwin stopped; earlier port-remap approach is retired"
metadata:
  node_type: memory
  type: project
  originSessionId: ded12615-321c-45b9-9265-2e8e2ba763d3
  modified: 2026-09-03T14:49:59.017Z
---

**Current (2026-09-03, user's choice):** `darwin` is **stopped**; `joker` runs on the
**default ports** — backend `:8000`, frontend `:5173`, **frontend-s3 `:5175`**, db
`:5432`, redis `:6379`. `.env` has no `*_PORT` overrides. The local
`docker-compose.override.yml` (db→5434 pin) was **removed**. To start:
`docker compose -p darwin down` first if darwin is up, then `docker compose up -d`.

**Mailpit** (SMTP catcher) is NOT in `docker-compose.yml` — the dev `.env` points
`EMAIL_HOST=mailpit` at it, so "Enviar links" fails with "0 de N emails enviados"
unless it runs. It's added in the local, untracked `docker-compose.override.yml`
(image `axllent/mailpit:v1.20`, web inbox on `:8025`). Only loads with a plain
`docker compose up` (not the prod overlay). Mail is caught locally — for real delivery
point `EMAIL_HOST`/`PORT`/`USER`/`PASSWORD` at a relay. `.env` now:
`DEFAULT_FROM_EMAIL=NeoToken <noreply@neotel.com.br>` (domain guessed from the operator
account email — adjust if wrong) and `SHARES_PUBLIC_BASE_URL=http://192.168.130.55:5175`
(recipient-email access links resolve there — but `/acesso/<token>` isn't built yet, so
the link 404s / redirects to login for now).

**LAN access is configured** (user opens it from another device at `192.168.130.55`):
`.env` has `ALLOWED_HOSTS=localhost,127.0.0.1,192.168.130.55`,
`CORS_ALLOWED_ORIGINS` lists `http://192.168.130.55:5175` + `:5173` (+ localhost),
and both `VITE_API_BASE_URL` **and** `S3_VITE_API_BASE_URL` = `http://192.168.130.55:8000/api/v1`
(the frontend-s3 compose service reads `S3_VITE_API_BASE_URL`, the scan frontend reads
`VITE_API_BASE_URL`). Recreate backend + frontends after any of these change.

**History:** earlier this session joker was remapped (8002/5273/5175/6380, db 5434 via
an override file, LAN-IP `VITE_API_BASE_URL`) to coexist with darwin. The user then
reset `.env` to defaults (also adding real S3 + mailpit config) and chose to stop
darwin instead. If both stacks must run again, re-add `*_PORT` vars to `.env` and keep
`POSTGRES_PORT=5432` (it doubles as Django's DB port — remap only the db *host publish*
via an override file). `docker compose -p darwin down` can leave one container
running — check `docker ps | grep darwin` and `docker stop` any leftover.

Related: [[neotoken-secure-sharing-frontend]], [[joker-detection-vocabulary]].
