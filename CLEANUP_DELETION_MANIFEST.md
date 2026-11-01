# Cleanup Deletion Manifest

**Status: Partial generated-only cleanup completed with approval.**

The project has no Git recovery point. Before deletion, a dated backup of the exact files below was created and verified. Only ordinary files without the reparse-point attribute were removed. All OneDrive reparse-point files were retained. No directories were recursively deleted.

Backup: `C:\Users\rishi\.copilot\session-state\87e209ae-b0ca-427d-b8e5-29e69d6e869e\cleanup-backup-generated-safe-20261101.zip`

## Files deleted

| Deleted path | Purpose | Classification | Size | Recovery |
|---|---|---|---:|---|
| `.\.pytest_cache\v\cache\lastfailed` | Pytest cache | GENERATED | 401 bytes | Recreated by pytest |
| `.\.pytest_cache\v\cache\nodeids` | Pytest collection cache | GENERATED | 5,910 bytes | Recreated by pytest |
| `.\api\__pycache__\main.cpython-311.pyc` | Python bytecode | GENERATED | 308,374 bytes | Recreated by Python |
| `.\engine\ingestion\__pycache__\streamer.cpython-311.pyc` | Python bytecode | GENERATED | 23,739 bytes | Recreated by Python |
| `.\tests\__pycache__\test_dataset_manager.cpython-311-pytest-9.2.0.pyc` | Test bytecode | GENERATED | 105,972 bytes | Recreated by pytest |
| `.\tests\__pycache__\test_dataset_manager.cpython-311.pyc` | Python bytecode | GENERATED | 35,847 bytes | Recreated by Python |
| `.\tests\__pycache__\test_fr1_pipeline.cpython-311-pytest-9.2.0.pyc` | Test bytecode | GENERATED | 16,191 bytes | Recreated by pytest |
| `.\tests\__pycache__\test_fr1_pipeline.cpython-311.pyc` | Python bytecode | GENERATED | 7,139 bytes | Recreated by Python |
| `.\frontend\dist\index.html` | Vite production build entry | GENERATED | 169 bytes | Recreated by the frontend build |
| `.\frontend\tsconfig.tsbuildinfo` | TypeScript incremental compilation state | GENERATED | 67 bytes | Recreated by TypeScript |

**Total removed: 503,809 bytes (about 0.48 MiB).**

## Generated files retained

| Path or content | Reason retained |
|---|---|
| Remaining files carrying OneDrive reparse-point attributes in caches and build outputs | Not removed because deletion could affect synchronized content; 85 files total, about 1.55 MiB |
| Remaining two assets in `frontend/dist/` | Included in the 85 retained reparse-point files; not removed |
| The cache/build directories themselves | Not recursively removed |
| `.venv\` and `frontend\node_modules\` | Protected development environments/dependencies |
| Datasets, uploads, `.part` files, databases, WAL, checkpoints, and analysis history | Protected user data |
| Empty source/test files and empty project directories | Ambiguous; left untouched |
| Dependency declarations and lockfiles | No dependency was verified unnecessary |
| All other source, rules, tests, fixtures, reports, and documentation | Not part of approved generated-only cleanup |

## Verification

- Backup archive verified: 10 entries, content lengths and SHA-256 hashes checked.
- Removed paths verified absent.
- No reparse-point paths were deleted.
- Backend tests and frontend checks passed; see `POST_CLEANUP_TEST_REPORT.md`.
