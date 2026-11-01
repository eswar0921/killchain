# KILLCHAIN AI - Development Log

## Entry 001 - 2026-10-31: Environment & Project Inception
- **Action**: Inspected target environment on Windows host.
- **Path Discovery**: Workspace active at `C:\Users\rishi\Downloads\New folder`. Linked `C:\Users\rishi\OneDrive\Desktop\mitigata` via Windows NTFS Junction (`mklink /J`) to ensure transparent bidirectional synchronization.
- **Runtime**: Created Python 3.11 virtual environment using `uv`. Installed core packages:
  - `fastapi`, `uvicorn`: API server
  - `duckdb`: High-performance embedded columnar analytical database for large datasets
  - `networkx`: In-memory graph engine for attack path & reachability
  - `scikit-learn`, `numpy`, `scipy`: Machine learning & statistical anomaly detection
  - `pyyaml`: Rule configuration engine
  - `psutil`: Real-time system memory and CPU profiling
  - `pytest`, `httpx`: Automated testing harness
- **Module Structure**: Created directory tree under `engine/`, `mitre/`, `storage/`, `api/`, `integrations/`, `data/`, `evaluation/`, `tests/`.
- **Status**: Environment verified and ready for Phase 1.

## Entry 002 - 2026-11-01: Persistent Dataset Manager
- **Action**: Extended the existing FastAPI, FR-1 checkpoint, DuckDB, and React
  paths for a persistent dataset library and independent analysis runs.
- **Upload integrity**: Added resumable chunk receipts, exact acknowledged-byte
  progress, chunk SHA-256 verification, atomic writes, and streamed final
  assembly with source-size/SHA-256 verification. The frontend now streams raw
  chunk bodies; multipart remains supported for existing API clients.
- **Analysis and validation**: Added streaming format/timestamp validation,
  optional disk-backed duplicate detection, dataset-scoped event IDs and
  detector queries, run history, checkpoint recovery visibility, storage
  metrics, audit history, and JSON/CSV/PDF reports.
- **Interface and documentation**: Added Datasets, Processing, History, and
  Storage views; documented APIs, workflow, deployment considerations, and
  limitations.
- **Verification**: Full Python test suite passed (41 tests); Python
  `compileall` and frontend TypeScript/Vite production build passed. No
  representative large-file performance benchmark has been run.

## Follow-up: Local source registration and XML validation
- **Action**: Extended the dataset workflow for source files already present
  on the API host, Windows Event XML exports, paginated normalized-event
  inspection, and grouped console navigation.
- **Local sources**: Registration remains within configured allowed roots,
  does not copy or delete the original file, reports unavailable/changed
  sources, and bounds folder traversal to 50,000 entries and registration to
  500 supported files.
- **XML**: Streaming Windows Event XML export parsing is covered by tests.
  XML files without recognized `System/EventID` records are rejected during
  validation; binary EVTX, macOS Unified Log, Android-specific Logcat, and
  arbitrary XML parsing remain unsupported.
- **Verification**: Full Python suite passed (45 tests), `compileall`
  completed, and the frontend TypeScript/Vite production build passed. These
  checks do not measure large-dataset throughput or memory use.
