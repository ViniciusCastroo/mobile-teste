---
name: joker-detection-vocabulary
description: "Name-detection needs the ibge_names table loaded, or every tokenize with tokenize_name fails"
metadata: 
  node_type: memory
  type: project
  originSessionId: ded12615-321c-45b9-9265-2e8e2ba763d3
  modified: 2026-09-03T14:18:48.598Z
---

Tokenizing a file with `tokenize_name=true` (shares upload, drive migration, scan)
fails with **`DETECTION_VOCABULARY_UNAVAILABLE`** when the `ibge_names` table is empty.
The surname ranking + "não-nomes" denylists are CSVs baked into the image; the ~128k
IBGE **first names** live in a DB table that must be bulk-loaded:

```
make load-names          # = docker compose exec backend python manage.py load_detection_data --force
make prod-load-names
```

Reads a bundled CSV (`backend/core/detection/data/nomes-censo-2022.csv`), no network
needed. Idempotent without `--force`. The prod entrypoint is meant to run it on every
start. Verify: `docker compose exec -T backend python manage.py shell -c "from apps.processing.models import IbgeName; print(IbgeName.objects.count())"` → ~128458.

A fresh DB volume / `docker compose down -v` wipes it — reload after.

Guard lives in `backend/apps/processing/detection_service.py:get_name_detector`; it
raises before caching, so a reload takes effect on the next task without a worker
restart. Downstream, `apps/shares/tasks.py` maps it to a `FAILED` file with that
error_code, and the share's readiness then reports "espere todos os arquivos ficarem
prontos" until every file is `READY`.

Related: [[neotoken-secure-sharing-frontend]].
