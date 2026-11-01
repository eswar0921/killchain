# FR-6/FR-7 Evaluation Report

## Scope

This report records local automated validation of the FR-6 progression and
reachability work and FR-7 risk, reporting, lineage, Sigma-rule, and live-event
capabilities. Test data is explicitly synthetic. No official evaluation
dataset, planted-attack ground truth, topology feed, or production workload was
available.

## Results

- Python regression suite: **38 passed**, with one upstream Starlette/httpx
  deprecation warning.
- Python compilation: `compileall` completed successfully for `api`, `engine`,
  `storage`, `tests`, and `evaluation`.
- Frontend production build: `tsc -b` and `vite build` completed successfully.
- Synthetic API coverage exercised evidence-required compromise confirmation,
  evidence-backed potential reachability, explainable risk, deterministic JSON
  and PDF reports, Sigma validation and versioned rule CRUD, alert
  deduplication, late/cross-chunk live correlation, event-ID idempotency,
  checkpoint recovery, API routes, and SSE delivery.
- FR-4/FR-5 regression tests in the same suite cover evidence-supported
  correlation and attack-path behavior, including the rule that username-only
  matches or temporal proximity do not establish an attack path.

## Accuracy and limitations

The automated tests verify expected behavior for their synthetic fixtures;
they are not a measured incident-formation accuracy, false-relationship rate,
attack-path ordering accuracy, or planted-attack recall benchmark. Those
metrics remain **not measured**. No official-dataset coverage or production
readiness is claimed.

The live benchmark measures only normalized, no-match events. It excludes ASGI
transport, frontend rendering, and external storage. The benchmark uses
in-memory DuckDB and is not representative of a persistent production
deployment. Manual browser interaction and end-to-end rendering latency were
not measured.

## Reproduction

From the project root:

```powershell
.\.venv\Scripts\python.exe -m pytest -q
.\.venv\Scripts\python.exe -m compileall -q api engine storage tests evaluation
Set-Location frontend; npm run build
Set-Location ..; .\.venv\Scripts\python.exe evaluation\fr67_live_benchmark.py
```

See [PERFORMANCE_REPORT.md](./PERFORMANCE_REPORT.md) for the recorded synthetic
throughput and memory results.
