# Updated Project Structure

## Current layout

No files or directories were moved during cleanup. This is the existing architecture, not a proposed migration.

```text
hack/
├── api/                    FastAPI application
├── config/                 Detection, correlation, kill-chain, risk, Sigma, TI rules
├── data/
│   ├── raw/                Registered raw-data location
│   ├── chunks/             Chunk-processing location
│   ├── normalized/         Normalized-data location
│   ├── uploads/            Resumable upload sessions and source copies
│   ├── samples/            Synthetic development fixtures
│   └── processed/          DuckDB, checkpoints/history, WAL, generated processing output
├── engine/
│   ├── ai/
│   ├── correlation/
│   ├── detection/
│   ├── events/
│   ├── fusion/
│   ├── graph/
│   ├── incidents/
│   ├── ingestion/
│   ├── ml/
│   ├── normalization/
│   ├── parsing/
│   ├── partitioning/
│   ├── progression/
│   ├── reachability/
│   ├── risk/
│   └── rules/
├── evaluation/             Benchmarks and evaluation tools
├── frontend/
│   ├── src/                React application and styles
│   ├── dist/               Generated production output (partially retained)
│   ├── node_modules/       Installed frontend dependencies
│   └── package-lock.json   Reproducible npm dependency resolution
├── integrations/           Optional integrations
├── mitre/                  ATT&CK data and mapping support
├── storage/                Persistence helpers
├── tests/                  Backend unit/integration tests and fixtures
├── .venv/                  Local Python environment
└── *.md                    Setup, user, API, architecture, evaluation, and project reports
```

## Cleanup notes

- The existing module layout is already organized by feature; no restructuring was justified by this audit.
- Empty data directories and other empty folders were retained because they may be expected paths.
- `data/uploads/`, `data/processed/`, `.venv/`, and `frontend/node_modules/` remain in place.
- The generated cleanup left directories and OneDrive reparse-point files intact.
