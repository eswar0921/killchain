# KILLCHAIN AI Deployment Guide

## Local test setup (Windows)

Prerequisites: Python 3.11 and Node.js/npm. The checked-in `.venv` and
`frontend/node_modules` may already be available; do not reinstall dependencies
unless the environment is missing them.

```powershell
Set-Location C:\Users\rishi\OneDrive\Desktop\hack
.\.venv\Scripts\python.exe -m uvicorn api.main:app --host 127.0.0.1 --port 8000
```

In another terminal:

```powershell
Set-Location C:\Users\rishi\OneDrive\Desktop\hack\frontend
npm run dev
```

The UI is at `http://localhost:5173`, the API at `http://127.0.0.1:8000`, and
OpenAPI docs at `/docs`.

## Configuration and persistence

- `KILLCHAIN_DB`: optional DuckDB file path. Default:
  `data/processed/killchain.duckdb`.
- `CORS_ORIGINS`: comma-separated allowed browser origins. Default permits
  local Vite on port 5173.
- `config/correlation_rules.yaml`: FR-4 event sequences, time windows, and
  clock-skew settings.
- `config/risk_weights.yaml`: FR-7 risk weights; supported factors must sum
  to 100.
- `config/sigma_rules.yaml`: active Sigma-subset rules. Manage using API/UI
  rather than hand-editing while concurrent rule updates are in progress.
- Local ATT&CK/STIX data remains in `mitre/attack_data`.
- Raw uploads, Parquet output, and FR-1 checkpoints are stored under `data`.
- Uploaded dataset sources live under `data/uploads/<dataset_id>`; persistent
  dataset registry, upload chunk receipts, run history, and audit records share
  `data/processed/fr1_checkpoints.sqlite`.
- Uploads use 32 MiB chunks. The UI can resume by reselecting the original
  file; pausing and cross-browser ownership are not implemented.
- Dataset processing admits at most four queued/running runs and executes at
  most two analyses concurrently. The worker pool and in-flight chunks remain
  bounded.

Back up the DuckDB file and the YAML configuration files together. Keep upload,
database, and checkpoint directories on reliable local storage with sufficient
free disk space; avoid synchronizing active DuckDB/SQLite files through a
cloud-sync client.

## Operational checks

- `GET /health` checks basic API reachability.
- `GET /api/v1/system/health` reports database counts, process RSS/CPU sample,
  active job counts, and the most recent live update ID.
- `GET /api/v1/live/checkpoint` reports processed-event checkpoint and
  pending/failed live records.
- `GET /api/v1/alerts/stats` reports raw/correlated/uncorrelated alert counts.
- Run `.\.venv\Scripts\python.exe -m pytest -q` and
  `Set-Location frontend; npm run build` before a test deployment.

## Security and production limitations

The API currently has no authentication, authorization, tenant isolation,
rate limiting, or built-in TLS. Bind to loopback for local testing. Before
shared-network use, place it behind an authenticated reverse proxy, restrict
origins and network access, and establish an access-control model for asset
compromise confirmation and rule changes. Do not expose `/docs` or write
access to evidence/rule endpoints publicly.

Live ingestion accepts normalized JSON only; there is no raw log tailer or
external message broker. Event and incident processing is synchronous per API
request, with batch size capped at 500 and entity candidates capped at 500.
Live retry is persisted but explicit, not an automatic startup worker.
Background upload/detection/correlation job registries are process-local.
Large incident snapshots, concurrent writes, and production-scale retention
require deployment load testing and operational monitoring.

The Dataset Manager performs validation, FR-1 ingestion/normalization,
FR-2 detection/fusion, and FR-3 mapping. It does not automatically invoke
FR-4 correlation or FR-5/FR-6 investigation. Optional duplicate validation
uses a temporary disk-backed index and may need substantial scratch storage.
PDF/JSON/CSV dataset exports contain stored metadata and run summaries, not
raw source contents.

Do not configure production secrets in source control. Sarvam credentials and
provider behavior were not used or changed by this implementation.
