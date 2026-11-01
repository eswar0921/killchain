# Dependency Audit

## Result

**No dependencies were removed or changed.** The current Python requirements and npm lockfile are preserved. No package was proven safe to remove after accounting for CLI usage, test tooling, FastAPI upload support, optional integrations, transitive/runtime behavior, and build tooling.

## Python dependencies

Declared in `requirements.txt`:

`fastapi`, `uvicorn[standard]`, `pydantic`, `pydantic-settings`, `duckdb`, `networkx`, `scikit-learn`, `numpy`, `scipy`, `pyyaml`, `psutil`, `requests`, `httpx`, `pytest`, `python-multipart`, `pyarrow`, and `polars`.

The source search found no direct import-name matches for `pydantic_settings`, `scipy`, `requests`, `httpx`, or `polars` in the searched Python application/test directories. This is only a candidate signal: it does not rule out entry-point commands, indirect imports, optional code paths, plugins, or dependencies used by another package. These packages remain installed and declared pending deeper runtime/packaging verification.

Other packages have evident project roles: FastAPI/Pydantic for the API and validation, DuckDB for event storage, NetworkX for graph processing, scikit-learn/NumPy for ML, PyYAML for rules, psutil for resource metrics, PyArrow for ingestion writes, python-multipart for uploads, and pytest for tests. Uvicorn is an application server command rather than necessarily a Python import.

## Frontend dependencies

`frontend/package.json` and `frontend/package-lock.json` are retained. The app directly imports React, React DOM, and Lucide icons; Vite and its React plugin configure/build the app; TypeScript and the React type packages support the build and type-check. No frontend dependency removal is proposed.

## Locking and reproducibility

- `frontend/package-lock.json` is present and retained.
- No Python lockfile was found; `requirements.txt` uses version ranges.
- No lockfile was replaced or regenerated.
- No dependency installation or upgrade was performed.

## Validation

- Frontend TypeScript check passed with the installed dependencies.
- Backend tests passed with the existing `.venv`.
- This audit did not attempt destructive uninstall/reinstall experiments.
