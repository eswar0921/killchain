# Post-Cleanup Test Report

## Cleanup summary

- Removed 10 approved, regenerable files without the OneDrive reparse-point attribute: **503,809 bytes (about 0.48 MiB)**.
- Created and verified a backup archive before deletion: `C:\Users\rishi\.copilot\session-state\87e209ae-b0ca-427d-b8e5-29e69d6e869e\cleanup-backup-generated-safe-20261101.zip`.
- Did not delete source, dependencies, datasets, uploads, databases, checkpoints, environments, or ambiguous content.
- Retained generated files carrying OneDrive reparse-point attributes.

## Validation performed

| Check | Result | Notes |
|---|---|---|
| Backend test suite | **Passed: 53 tests** | `python -m pytest -q -p no:cacheprovider`; used a temporary `KILLCHAIN_DB` and disabled Python bytecode writes to avoid touching the live DB or recreating caches |
| Frontend type-check | **Passed** | `npm exec -- tsc --noEmit --incremental false` |
| Frontend startup | **Passed** | Vite dev server returned HTTP 200 on local port 5176; the exact temporary server PID was stopped |
| API health | **Passed** | Existing API returned `status: ok`; it was left running |
| Cleanup scope verification | **Passed** | All 10 approved ordinary generated files were absent; no reparse-point path was deleted |
| Backup verification | **Passed** | 10 archive entries validated against source lengths and SHA-256 hashes before deletion |

The backend test run emitted one existing `StarletteDeprecationWarning` about using `httpx` with `starlette.testclient`; all tests passed.

## Not run / limitations

- A production Vite build was not run because its default output cleanup could replace/delete the retained OneDrive reparse-point assets in `frontend/dist`. A type-check and dev-server HTTP smoke test were used instead.
- No destructive dataset upload, database migration, or analysis run was started. User data and the live database/WAL were protected.
- Individual feature areas were validated through the existing test suite; separate interactive tests were not run for every UI workflow (upload, live monitoring, analysis timer, timeline, and next-hop interface).
- A running application updated the database WAL during the audit. Cleanup did not modify the database or WAL.

## Overall

The approved generated-file cleanup completed. The backend suite, frontend type-check, frontend startup smoke test, API health check, and backup verification passed. No source changes, dependency changes, or file moves were made.
