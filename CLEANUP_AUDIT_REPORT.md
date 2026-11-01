# Cleanup Audit Report

## Scope and safety

- Audited folder: `C:\Users\rishi\OneDrive\Desktop\hack`
- Initial recursive inventory: **16,073 files** across **1,290 folders**, approximately **88.55 GiB**.
- Final inventory after creating the requested reports: **16,068 files** across **1,290 folders**, approximately **88.913 GiB**. The project size increased while the running application updated its database WAL; that user data was not modified by cleanup.
- The project root has no `.git` directory, so there is no version-controlled recovery point.
- No application source or user data was deleted, moved, or modified. After approval, 10 generated files were removed and the requested audit reports were created.
- Large datasets, databases, upload chunks, the virtual environment, and installed frontend dependencies were treated as protected content.

The inventory includes vendored/installed package trees and user data. Source-reference review was targeted to entry-point/configuration areas and the empty-file candidates; it was not a semantic review of every source file. No non-empty source module or dependency was verified safe to remove.

## Classification

| File or folder | Current purpose | Classification | Reason | Recommended action |
|---|---|---|---|---|
| `api/`, `engine/`, `config/`, `storage/` | Backend API, processing, detections/correlation/graph logic, configuration, and persistence | ESSENTIAL | README and project structure identify these as application runtime modules | Keep |
| `frontend/src/`, `frontend/index.html`, `frontend/package.json`, `frontend/package-lock.json`, `frontend/tsconfig.json`, `frontend/vite.config.ts` | React/Vite application, source, and reproducible frontend setup | ESSENTIAL / REQUIRED FOR DEVELOPMENT | Frontend entry points and build configuration are present and active | Keep |
| `tests/`, `evaluation/` | Test suite, performance tooling, and evaluation fixtures | REQUIRED FOR DEVELOPMENT / REQUIRED FOR HACKATHON | Includes active tests and benchmark/evaluation content | Keep |
| Root documentation (`README.md`, `API_DOCUMENTATION.md`, `ARCHITECTURE.md`, `DEPLOYMENT_GUIDE.md`, `USER_GUIDE.md`, `EVALUATION_REPORT.md`, `PERFORMANCE_REPORT.md`, `PROJECT_PROGRESS.md`, `DEVELOPMENT_LOG.md`) | Setup, architecture, API, project history, evaluation, and submission evidence | REQUIRED FOR HACKATHON | Documentation is explicitly protected by the request; age alone is not evidence of obsolescence | Keep |
| `config/`, `mitre/`, `integrations/` | Detection/correlation rules, ATT&CK data, and optional integration code | ESSENTIAL / REQUIRED FOR HACKATHON | Runtime configuration and intelligence data are referenced by the project; integrations are not proven abandoned | Keep |
| `data/firex-30G.log` and `data/uploads/` | Source dataset copies and resumable upload sessions | UNCERTAIN / PROTECTED USER DATA | Includes multiple same-sized log files and 1,047 `.part` files totaling about 32.72 GiB; same size is not proof of duplicate content or stale upload state | Keep; do not deduplicate or remove without explicit approval |
| `data/processed/` | DuckDB database, SQLite checkpoints/history, and a live WAL | ESSENTIAL / PROTECTED USER DATA | Contains persisted analysis state; the database/WAL was locked by a running process during the audit | Keep; do not inspect by destructive or offline repair operations |
| `.venv/` | Project Python environment | REQUIRED FOR DEVELOPMENT / PROTECTED ENVIRONMENT | Contains 8,821 files; explicitly protected by the request | Keep |
| `frontend/node_modules/` | Installed frontend dependencies | REQUIRED FOR DEVELOPMENT | 5,968 files (about 94.21 MiB); reproducible but not a disposable cache, and useful to the current working setup | Keep |
| `.pytest_cache/`, project `__pycache__/` directories, project `.pyc` files | Test and Python runtime caches | GENERATED | Re-creatable; many entries were OneDrive reparse points | Removed only 8 ordinary cache/bytecode files (about 0.45 MiB); retained all reparse-point files and directories |
| `frontend/dist/` | Vite production build output | GENERATED | Re-creatable by the configured frontend build; files included OneDrive reparse points | Removed only ordinary `index.html` (169 bytes); retained the two reparse-point assets |
| `frontend/tsconfig.tsbuildinfo` | TypeScript incremental-build state | GENERATED | Re-creatable build metadata (67 bytes) | Removed (67 bytes) |
| `frontend/src/dashboard-polish.css` | Empty CSS source file | UNCERTAIN | Zero bytes and no reference found in `frontend/src`; it may be an abandoned placeholder, but source ownership is unclear | Do not remove without approval |
| `tests/test_event_api.py` | Empty test module | UNCERTAIN | Zero bytes; the pytest cache contains stale node IDs for tests formerly associated with this path, so deleting it could conceal missing test coverage | Keep pending review; do not remove |
| `New folder/`, `reports/`, and empty `data/` subfolders | Empty directories | UNCERTAIN | Could be intentional project layout or future data destinations | Keep; do not remove |
| Python/frontend dependency declarations | Runtime and build dependencies | REQUIRED FOR DEVELOPMENT | Some Python requirements have no direct import match in the searched source, but may be CLI, plugin, test, transitive, or optional-runtime dependencies | No dependency removal proposed; perform deeper validation before any change |

## Duplicate and obsolete findings

- A byte-hash scan of 93 small source/config/document files found one identical-content group: the two **zero-byte** files `frontend/src/dashboard-polish.css` and `tests/test_event_api.py`.
- This is not treated as a confirmed removable duplicate: the files have distinct roles, and the test path has stale cached test identifiers.
- No duplicate non-empty source modules were verified.
- No non-empty abandoned implementation was verified obsolete.
- Three large `firex-30G.log` files have matching names/sizes in different data locations. They were not content-hashed because they are large user datasets; they remain protected and are **not** counted as confirmed duplicates.

## Cleanup outcome

- Deleted **10 ordinary generated files**, totaling **503,809 bytes (about 0.48 MiB)**, after approval and backup.
- The backup archive was created and verified before deletion at `C:\Users\rishi\.copilot\session-state\87e209ae-b0ca-427d-b8e5-29e69d6e869e\cleanup-backup-generated-safe-20261101.zip`.
- **85 generated files with OneDrive reparse-point attributes were retained** (about 1.55 MiB); cache/build directories were not removed.
- No source, dependency, user dataset, upload, database, checkpoint, or environment was deleted or moved.
- The remaining possible benefit from the unremoved generated files was not taken because their OneDrive reparse-point status could affect synchronized content.

## Validation status

- Backend: **53 tests passed**, one existing Starlette/httpx deprecation warning.
- Frontend TypeScript check: passed.
- Frontend Vite dev server: returned HTTP 200; the temporary process and logs were removed.
- Existing API health endpoint: responded successfully.
- See `POST_CLEANUP_TEST_REPORT.md` for exact commands and limitations.
