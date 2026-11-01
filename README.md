# KILLCHAIN AI

Evidence-first attack detection and investigation for the Kill Chain and Attack Path Analysis challenge.

## What is implemented

- Streaming JSON, JSONL, CSV, FortiGate key-value LOG, and Windows Event XML
  exports into DuckDB (binary EVTX is not supported).
- Structured event normalization with source references and malformed-record accounting.
- YAML-configured threshold, sequence and pattern detections.
- Explainable statistical anomaly detection.
- Detection fusion, MITRE ATT&CK STIX enrichment, incident correlation and risk scoring.
- Hybrid FR-2 detection with configurable rules, behavioral/statistical
  baselines, reproducible anomaly scoring, a tested Sigma-compatible subset,
  optional local IOC matching, evidence-preserving fusion, and alert APIs.
- Evidence-first FR-3 ATT&CK tactic, technique and sub-technique mapping with
  separate Cyber Kill Chain stage reasoning.
- FR-4 evidence-scored correlation with typed entities, clock-skew-aware
  windows, configurable sequence rules, incident persistence, relationship
  evidence, risk scoring, and investigation status APIs.
- FR-5 incident-scoped temporal graphs, UTC-ordered evidence timelines,
  explainably ranked attack-path candidates, graph evidence APIs, and an
  interactive investigation view.
- FR-6 evidence-backed progression states, analyst-evidence compromise
  confirmation, asset/topology/permission inventory, and potential reachability
  paths distinct from observed movement.
- FR-7 incident evidence reports (deterministic JSON/PDF), configurable
  explainable risk, duplicate-safe raw-alert lineage, versioned Sigma-subset
  management, incremental live-event processing, checkpoint recovery, SSE
  replay, and connected investigation dashboard pages.
- Local Sarvam adapter that does not make remote calls by default.
- Responsive React/Vite SOC console backed by the REST API.
- Bounded FR-1 workers with atomic per-chunk Parquet output, SQLite recovery
  checkpoints, and psutil resource metrics.

Synthetic files under `data/samples` are clearly labelled development data; they are not official hackathon ground truth.

### FR-1 large-file behavior

The ingestion path reads JSON arrays incrementally with `JSONDecoder.raw_decode`
and reads JSONL, CSV, XML, and line-oriented logs record by record. The Data
Manager's **Processing Configuration** supports automatic recommendations or
manual chunk-byte/record, memory, queue-byte/count, and worker limits. Queue
admission is bounded by both bytes and in-flight chunk count; the processing
engine applies backpressure when either limit is reached. The default parser
read buffer is 1 MiB, and individual source records are capped at 64 MiB by
default.

Analysis profiles persist requirement versions and resource settings in
`data/processed/fr1_checkpoints.sqlite`. Each run stores a configuration
snapshot, profile version, selected required detection engines, recommendation,
checkpoint compatibility note, and measured FR-1/detection stage metrics.
Chunk-boundary settings are fingerprinted into checkpoint namespaces, so a
changed chunk configuration cannot reuse incompatible completed chunks.

Uploaded dataset analysis uses a bounded worker pool and writes completed chunks under
`data/processed/parquet`, and records chunk state in
`data/processed/fr1_checkpoints.sqlite`. Event batches are loaded into DuckDB
through Arrow tables, with UTC timestamps normalized before persistence.
FortiGate key-value traffic, UTM, and authentication records are normalized
with their source references retained. Query the live metrics while a job runs
with `GET /api/v1/datasets/{dataset_id}/metrics`. Completed chunks are
idempotent and can be skipped after an interrupted run.

Dataset Manager provides **Datasets**, **Events**, **Processing**, **History**,
and **Storage** views backed by the API. Uploads use persistent 32 MiB chunk
sessions, per-chunk SHA-256 integrity, server-acknowledged progress, final
source SHA-256 verification, and resume by selecting the exact original file.
Dataset metadata, active selection, analysis runs, and deletion audit entries
persist in `data/processed/fr1_checkpoints.sqlite`. Reuse selects the stored
source without copying it. Explicit deletion requires confirmation, refuses
active processing, removes only the dataset-owned upload directory, and
retains analysis/audit history.

For files already on the backend machine, Dataset Manager also supports
registering an existing local file or folder without uploading/copying it.
Folders produce one registered dataset per supported JSON, JSONL, CSV, LOG, or
Windows XML event-export file (up to 500; traversal is capped at 50,000
entries). For safety, the local API reads only from the project `data`
directory by default. Configure additional approved roots before starting the
backend, for example in PowerShell:

```powershell
$env:KILLCHAIN_LOCAL_DATA_ROOTS = "C:\Users\you\Downloads;D:\SecurityLogs"
.\.venv\Scripts\python.exe -m uvicorn api.main:app --host 127.0.0.1 --port 8000
```

In the UI, enter the full local path in **Register an existing file or
folder**. This accesses files on the API host, not arbitrary paths on a remote
browser client. The API has no authentication; keep it bound to loopback.
Deleting a locally registered dataset does not delete the original file.
Dataset Manager's **Events** view provides paginated normalized evidence and
source record references; raw source content remains in the original file.

Automatic recommendations use dataset size, sampled record size, available
memory, CPU count, and selected profile limits. They are conservative estimates,
not benchmark results; storage throughput and workload complexity are not
measured by the recommender. Required core engines (rule-based, ML anomaly,
behavioral, Sigma, and threat intelligence) remain enabled. Sequence and graph
analysis remain separate workflows and are not represented as enabled FR-1
engines.

For repeatable local measurements, run the configuration benchmark on a single
input with:

```powershell
.\.venv\Scripts\python.exe evaluation\fr1_config_benchmark.py --file data\firex-30G.log --suite --output $env:TEMP\fr1-benchmark.json
```

The six-run suite uses temporary databases and Parquet output, deletes those
after each trial, and reports FR-1 stage timings and worker memory. It does not
run FR-2, FR-4, or FR-3; do not treat a partial-file or synthetic result as a
full-dataset capacity claim.

Validation streams records, checks UTF-8/format/checksum and recognized
timestamps, and retains malformed-record references. Optional duplicate
scanning uses disk-backed temporary SQLite rather than an in-memory hash set.
Upload and analysis bars are independent: upload percentage reflects only
server-acknowledged bytes; analysis percentage uses validated record counts
and completes only after the actual FR-1/FR-2/FR-3 pipeline stages finish.
Separate analysis runs preserve results and checkpoint recovery; detector
queries and event IDs are dataset-scoped.

The current analysis job executes validation, bounded FR-1 ingestion/parsing/
normalization, FR-2 detection/fusion, and FR-3 mapping. It does **not** run
FR-4 correlation, FR-5 attack-path reconstruction, or FR-6 reachability
automatically. Upload pause/cancel and analysis cancellation are not
implemented. JSON, CSV, and PDF dataset reports are generated from persisted
dataset and run metadata. See [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
for routes and limitations.

The local source OS label is analyst-provided metadata; OS detection is not
implemented. XML support is limited to Windows Event XML exports with
recognized `System/EventID` records; unsupported or unrecognized XML fails
validation. Binary EVTX,
macOS Unified Log, Android Logcat-specific, and arbitrary XML parsers are not
implemented. The **Threat signals** section currently shows persisted matches
from the configured local IOC feed only; there is no CVE enrichment or
asset-vulnerability feed. No exploit is executed.

The dashboard has two top-level workspaces. **Command center** contains
Overview, Datasets, and Reports. The Live monitor page has been removed from
the dashboard; normalized live-event APIs remain available. **Detection** contains Events,
Incidents, Investigation (attack graph, progression, reachability, and asset
context), Rules, and a combined Threat & risk view. Incident event links use
the correlation API's event IDs; graph filters support host, user, IP, and
time. Shared entity/time candidates are not shown as confirmed incident links.
Threat & risk includes severity pie charts for incidents and local IOC matches,
plus alerts explicitly labelled with exploit/CVE terms; those labels are not
proof of exploitability, and no CVE enrichment or vulnerability feed is present.
The standalone ATT&CK catalog page is hidden; evidence-backed technique
mappings remain in incident investigations. Dataset Processing shows a live
elapsed-time clock for the active analysis run. Investigation presents a
compact host/user path and a separate chronological timeline. It only connects
an observed authentication when a suspicious incident event explicitly
supports that link; other entities remain unconnected. Select entities or
timeline events to inspect source evidence. Alternative reconstructed paths,
filters, performance details, progression, and topology inventory are
expandable rather than permanently displayed.
Configurable run-level memory budgets and processing controls are not yet
exposed; the FR-1 worker and in-flight limits remain bounded by existing
defaults. Performance targets for 15/30/50 GB input remain unmeasured.

### FR-2 detection

Start detection with `POST /api/v1/detections/run`, then poll the returned job
with `GET /api/v1/detections/jobs/{job_id}`. Alerts are stored in DuckDB and
can be filtered through `/api/v1/detections`. Rule configuration is in
`config/detection_rules.yaml`; optional Sigma rules and threat-intelligence
entries are in `config/sigma_rules.yaml` and
`config/threat_intelligence.yaml`. Empty IOC configuration means IOC reputation
is unavailable, not benign.

### FR-3 ATT&CK mapping

The local catalog is loaded from `mitre/attack_data/enterprise-attack.json`,
which is populated from the official MITRE Enterprise ATT&CK STIX bundle. The
API reports its provenance and local content hash through
`GET /api/v1/mitre/version`.
Explicit rule references are validated against the catalog. Unsupported IDs
remain `unsupported` and are never presented as confirmed. Kill Chain stages
are separately configured in `config/kill_chain_mapping.yaml` and may be
`Unknown` when evidence is insufficient.

This design supports a 15 GB input on a 16 GB laptop in principle, subject to
the record size, DuckDB's configured 4 GB limit, available disk space, and the
cost of downstream analysis. It does not claim a benchmark until the target
hardware and dataset are measured.

### FR-4 correlation

Start correlation with `POST /api/v1/correlation/run` and poll
`GET /api/v1/correlation/jobs/{job_id}`. The API considers structured
suspicious events, categories configured in enabled event sequences, and events
referenced by detections, avoiding loading every normalized event row into
application memory. A shared username or account ID
alone is insufficient; entity provenance, source/destination IP roles,
timestamp tolerance, sequence context, and an incident duration cap are
preserved. Rules in `config/correlation_rules.yaml` include ordered sequences,
evidence requirements, per-rule windows, severity, and enabled status.

Incident APIs expose event, alert, entity, timeline, risk, and relationship
details. The dashboard supports incident search/filters, severity distribution,
investigation status, evidence inspection, and an interactive graph. ML
similarity is available at `/api/v1/correlation/suggestions`; suggestions are
unconfirmed and never create incident relationships automatically. Risk is a
weighted, explainable assessment, not an addition of detector scores.

Correlation jobs report records in the database, candidate events processed,
elapsed time, throughput, and sampled peak RSS. The following engine-only
measurement used synthetic input on the current Windows 10 / Python 3.11.15
environment (12 logical CPUs, 15.2 GiB RAM). Input event objects were created
before timing; this is not an ingestion-to-incident benchmark:

| Synthetic workload | Events | Incidents | Evidence edges | Time | Throughput | Peak traced Python allocation | Process RSS delta |
|---|---:|---:|---:|---:|---:|---:|---:|
| One suspicious same-host event chain | 20,000 | 1 | 19,999 | 6.896 s | 2,900 events/s | 26.05 MiB | 26.04 MiB |

This synthetic result does not establish million-event performance or
correlation accuracy on an official dataset. `CorrelationState` provides
cross-chunk deduplication, late/out-of-order event handling, and serializable
checkpoint/recovery, and `CorrelationCheckpointStore` persists state snapshots
in SQLite. The asynchronous API job registry itself is process-local; after a
server restart, rerun correlation against durable DuckDB data rather than
resuming its previous job ID. Larger production benchmarks remain
dataset/deployment validation work.

A separate labeled acceptance fixture contained 300 event pairs: 100 suspicious
same-host positives, 100 suspicious shared-username-only negatives, and 100
benign shared-host negatives. It produced 100 true-positive pairs, 0 false
positives, 0 false negatives (precision/recall 1.00 on this fixture), and the
expected 300 incidents. These deliberately simple cases are a regression
check, not a representative accuracy estimate.

### FR-5 attack-path reconstruction

The Attack graph view builds directed graphs from incident events, alerts,
typed entities, and FR-4 relationships. Every edge retains its evidence,
source event IDs, timestamp, relationship method, confidence, and
observed/inferred label. ATT&CK technique nodes provide detection context
only; mappings do not create attack transitions. Timeline timestamps are
normalized to UTC with stable ordering for ties and missing timestamps listed
last.

Candidate paths are time-respecting and only traverse supported event-to-event
edges. Ranking factors and evidence gaps are returned with each path; scores
are ordinal, not calibrated probabilities. Missing links remain partial
evidence, and a login or network connection alone never confirms compromise.
FR-5's progression data is only an evidence handoff; FR-6 progression and
potential reachability are implemented separately below.

The graph API defaults to 250 events (maximum 1,000 per request); path
reconstruction defaults to 1,000 and caps at 5,000 events, with traversal
bounded to 12 events per path and 10,000 expansions. Responses expose
pagination and truncation. The graph cache is process-local and retains at most
eight inputs. Incident-scoped graph generation avoids a dataset-wide graph,
but retrieving the current stored incident payload still deserializes its
incident JSON; very large incidents need production load testing and a
normalized incident-event store before claiming bounded retrieval.

The dashboard uses real incident graph, timeline, path, and evidence APIs.
The host/user view shows observed entities and only directly evidenced
suspicious authentication links; it does not interpret connections or
successful logins alone as compromise. Timeline selection focuses linked
entities and opens source event evidence. Alternative reconstructed paths are
clearly separated as hypotheses. The FR-6 Investigation view includes
**What Could the Attacker Reach Next?** based on evidence-backed topology and
explicit permission records. Candidates are
potential—not observed movement or compromise—and are ordered by explicit
permission evidence, reference count, path length, and supplied asset
criticality. This is an evidence ordering, not a probability or calibrated
risk score; the view lists missing validation such as current authentication,
effective privileges, and access-control restrictions. These are client-side visualization
features; the API does not claim sub-second performance without a measured
deployment benchmark. Synthetic test fixtures are regression cases and are
not official attack ground truth.

Synthetic FR-5 benchmark on the same Windows 10 / Python 3.11.15 environment
(12 logical CPUs, 15.2 GiB RAM). Input event objects were created before
timing. Memory values are traced Python allocations and process RSS delta,
not total deployment memory:

| Synthetic workload | Events | Graph nodes | Graph edges | Graph build | Path reconstruction | Peak traced Python allocation | Process RSS delta | Result |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| One shared-entity incident graph | 1,000 | 1,005 | 6,999 | 0.898 s (1,114 events/s) | 0.047 s | 15.10 MiB | 17.64 MiB | 12-event path, correctly marked truncated at the per-path limit |

A separate labeled ground-truth fixture had 12 planted four-event
failed-login → successful-login → process-execution → network-connection
sequences spanning two hosts. The exact ordered path was reconstructed in
12/12 scenarios. This small, deliberately connected fixture is not a
representative accuracy estimate or official-dataset result. On the four-event
browser fixture, the initial API-open measurement (graph, timeline, and path
requests together) was 128 ms and the client-render measurement was 45.6 ms;
these measurements do not establish the targets for larger incidents.

Validation on this environment: `pytest -q` passed all 30 tests in 34.93
seconds; `compileall` and the TypeScript/Vite production build succeeded. The
test run reports one upstream Starlette/httpx deprecation warning. Browser
validation loaded the clearly labelled synthetic incident's graph, ranked
path, timeline, and selected-edge evidence through the APIs.

### FR-6 progression and reachability

`GET /api/v1/incidents/{incident_id}/progression` returns chronological
evidence-backed activity, per-host observation state, first/latest suspicious
events, the furthest observed host, evidence references, and evidence gaps.
Observed logins and network activity do not confirm compromise. An analyst
must explicitly set an asset to `confirmed_compromised` and provide evidence
references through `PUT /api/v1/assets/{asset_id}/compromise-status`.

The asset inventory and network/permission records are maintained through
`/api/v1/assets`, `/api/v1/topology/relationships`, and
`/api/v1/topology/permissions`. Topology edges and permission grants require
evidence references; reachability reports potential access paths only and
labels them separately from observed attacker movement. Empty inventory or
unsupported relationship types are reported as limitations, not assumed
connectivity.

### FR-7 reports, risk, lineage, rules, and live processing

Risk factors and weights are in `config/risk_weights.yaml` and must sum to 100.
The output includes per-factor points and evidence references; severity is a
category, missing evidence contributes zero, and the score is not a
probability. Incident JSON/PDF exports are deterministic and use stored
events, alerts, relationships, mappings, and limitations; PDF generation uses
the standard library and adds no service dependency.

The raw-alert ledger deduplicates on detector source, rule ID, and sorted event
IDs while preserving the original alert record. For alerts without event IDs,
the detection ID is used instead, since duplicate identity cannot safely be
inferred. `GET /api/v1/alerts/stats` reports raw, correlated, and uncorrelated
counts. Existing detections are backfilled once into the ledger using a
streaming migration.

`config/sigma_rules.yaml` can be managed with the Sigma rule CRUD, validation,
and history APIs or the **Detection rules** dashboard. The supported syntax is
one selector with exact scalar values and `condition: selection`; wildcards,
modifiers, aggregation, and compound conditions are rejected/reported rather
than approximated. YAML edits are versioned and audited in DuckDB.

For incremental processing, submit normalized events to
`POST /api/v1/live/events` or batches of up to 500 to
`POST /api/v1/live/events/batch`. Event IDs are idempotent, event content is
hashed, active processing records retain retry payloads, and completed records
retain only their hash. Enabled supported Sigma rules execute on arrival.
Correlation checks at most 500 indexed entity candidates in the configured
time horizon, and event sequence stages use the configured FR-4 categories.
`GET /api/v1/live/stream` is an SSE stream with event IDs and replay; its
retention is bounded to 10,000 updates. Clients can recover after a missed
retention window by reloading a snapshot. `GET /api/v1/live/checkpoint` and
`POST /api/v1/live/recover` expose persisted checkpoints and retry failed or
interrupted events. Live input is already normalized JSON; there is no
continuous raw-file tailer. Recovery is an explicit API action, not an
automatic background retry.

The two dashboard workspaces group these features: Command center includes
dataset management and Reports; Detection includes Events,
Incidents, graph/progression/reachability/asset investigation, Rules, and
Threat & risk. `GET /api/v1/system/health` reports live DuckDB counts, process
RSS, CPU sample, and job counts. The local API currently has no authentication or
role-based authorization; do not expose it to untrusted networks. Analyst
identity fields are explicitly marked unauthenticated. Sarvam credentials and
provider behavior were not modified or used.

### FR-6/FR-7 validation scope

The automated FR-6/FR-7 tests use explicitly labeled synthetic events and
assets. They exercise explicit compromise confirmation, unsupported
relationship filtering, weighted risk, deterministic JSON/PDF output,
versioned rule CRUD, duplicate alert suppression, late cross-chunk event
correlation, event-ID idempotency, checkpoint recovery, API endpoints, and SSE
delivery. No official evaluation dataset or planted attack ground truth was
provided for this work; no official accuracy or large-scale throughput claim
is made. See [EVALUATION_REPORT.md](./EVALUATION_REPORT.md) and
[PERFORMANCE_REPORT.md](./PERFORMANCE_REPORT.md).

## Run

From `C:\Users\rishi\OneDrive\Desktop\hack`:

```powershell
.\.venv\Scripts\python.exe -m uvicorn api.main:app --reload --port 8000
```

In another terminal:

```powershell
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173`. API documentation is available at `http://localhost:8000/docs`.
By default, the API accepts frontend origins on `localhost` and `127.0.0.1`
on any port, so Vite's automatic port fallback continues to work. Set
`CORS_ORIGINS` to a comma-separated allowlist to override this for deployment.

To process one of the checked-in samples:

```powershell
Invoke-RestMethod -Method Post "http://localhost:8000/api/v1/process?path=C:\Users\rishi\OneDrive\Desktop\hack\data\samples\windows_events.json"
```

For large files, open **Dataset manager** in the console. Upload is resumable in
32 MB chunks (four concurrent chunks), then choose **Analyze dataset** after
the upload reaches `UPLOADED`. The source file is assembled on disk and parsed
in bounded streaming batches; it is not loaded into RAM. Throughput is limited
by the browser-to-server network and server disk, so a 50 GB upload can still
take significant time. Keep the backend storage on a fast local SSD and avoid
OneDrive synchronization for the upload directory during processing.

## Tests

```powershell
.\.venv\Scripts\python.exe -m pytest -q
```

## Evidence and limitations

The application never treats a successful login as proof of compromise, and graph topology is reported as potential reachability rather than observed movement. Permission-aware reachability requires permission/topology records in the dataset. MITRE mappings are marked as candidates when a technique is not present in the local STIX knowledge base. Sarvam is disabled unless explicitly configured and the development adapter sends no raw logs.
