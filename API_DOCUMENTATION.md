# API documentation

Base URL: `http://localhost:8000`

| Method | Path | Purpose |
|---|---|---|
| GET | `/health` | Service and event count |
| GET | `/api/v1/dashboard` | Dashboard metrics |
| GET | `/api/v1/events?limit=100&search=host` | Paginated normalized events |
| GET | `/api/v1/detections` | Persisted rule/ML detections |
| POST | `/api/v1/detections/run` | Start hybrid rule, ML, behavioral, Sigma and IOC detection |
| GET | `/api/v1/detections/jobs/{job_id}` | Get detection job status and detector metrics |
| GET | `/api/v1/detections/{detection_id}` | Retrieve alert evidence |
| GET | `/api/v1/detections/stats` | Detector and severity totals |
| GET | `/api/v1/detection-rules` | List configured detection rules |
| PUT | `/api/v1/detection-rules/{rule_id}?enabled=true` | Enable or disable a rule |
| GET | `/api/v1/incidents` | Correlated incidents and risk scores |
| GET | `/api/v1/incidents?severity=HIGH&status=new&search=host` | Filter incidents by severity, status, or entity |
| POST | `/api/v1/correlation/run` | Start asynchronous evidence-backed incident correlation |
| GET | `/api/v1/correlation/jobs/{job_id}` | Get correlation job status |
| GET | `/api/v1/correlation/rules` | List YAML-configured correlation rules |
| PUT | `/api/v1/correlation/rules/{rule_id}?enabled=false` | Enable or disable a correlation rule |
| GET | `/api/v1/correlation/suggestions?limit=100&minimum_similarity=0.25` | Retrieve unconfirmed ML-ranked alert similarities |
| GET | `/api/v1/incidents/{incident_id}` | Retrieve incident timeline, graph and mappings |
| GET | `/api/v1/incidents/{incident_id}/events` | Retrieve related normalized events |
| GET | `/api/v1/incidents/{incident_id}/alerts` | Retrieve related raw detections |
| GET | `/api/v1/incidents/{incident_id}/entities` | Retrieve involved hosts, users, accounts, IPs, and processes |
| GET | `/api/v1/incidents/{incident_id}/timeline` | Retrieve chronological incident evidence |
| GET | `/api/v1/incidents/{incident_id}/relationships` | Retrieve typed relationship evidence |
| GET | `/api/v1/incidents/{incident_id}/risk` | Retrieve explainable incident risk contributors |
| GET | `/api/v1/incidents/{incident_id}/progression` | Retrieve observed activity and explicit compromise evidence |
| GET | `/api/v1/incidents/{incident_id}/reachability` | Retrieve evidence-backed potential asset paths |
| GET | `/api/v1/incidents/{incident_id}/report?format=json` | Download deterministic evidence report as JSON |
| GET | `/api/v1/incidents/{incident_id}/report?format=pdf` | Download deterministic evidence report as PDF |
| PUT | `/api/v1/incidents/{incident_id}/status?status=investigating` | Update investigation status |
| GET | `/api/v1/assets?search=host` | Search evidence-backed asset inventory |
| POST | `/api/v1/assets` | Register/update an asset; requires `evidence_references` |
| PUT | `/api/v1/assets/{asset_id}/compromise-status` | Record/reset compromise state with explicit evidence |
| GET | `/api/v1/topology` | Retrieve assets, directional network relationships, and permissions |
| POST | `/api/v1/topology/relationships` | Register an evidence-backed network relationship |
| POST | `/api/v1/topology/permissions` | Register an evidence-backed principal permission |
| GET | `/api/v1/topology/relationships` | List topology relationships |
| GET | `/api/v1/topology/permissions` | List asset permissions |
| GET | `/api/v1/alerts/stats` | Raw, correlated, and uncorrelated alert lineage counts |
| GET | `/api/v1/sigma-rules` | List supported, disabled, and unsupported Sigma rules |
| POST | `/api/v1/sigma-rules/validate` | Validate a rule without saving it |
| POST | `/api/v1/sigma-rules` | Create/version a YAML Sigma-subset rule |
| PUT | `/api/v1/sigma-rules/{rule_id}` | Update/version a rule |
| DELETE | `/api/v1/sigma-rules/{rule_id}` | Delete an active rule and record the audit entry |
| GET | `/api/v1/sigma-rules/{rule_id}/history` | Retrieve rule version/audit history |
| POST | `/api/v1/live/events` | Ingest one normalized event idempotently |
| POST | `/api/v1/live/events/batch` | Ingest up to 500 normalized events |
| GET | `/api/v1/live/stream?after=0` | Reconnectable SSE stream of event/incident updates |
| GET | `/api/v1/live/checkpoint` | Retrieve persisted processing checkpoint/recovery counts |
| POST | `/api/v1/live/recover?limit=100` | Retry failed/interrupted events |
| GET | `/api/v1/system/health` | Live service, DuckDB, process, and job metrics |
| GET | `/api/v1/incidents/{incident_id}/attack-graph?limit=250&offset=0` | Retrieve a bounded, filtered incident graph |
| GET | `/api/v1/incidents/{incident_id}/attack-graph/nodes?limit=250&offset=0` | Page through graph nodes |
| GET | `/api/v1/incidents/{incident_id}/attack-graph/edges?limit=250&offset=0` | Page through graph edges |
| GET | `/api/v1/incidents/{incident_id}/attack-graph/node-evidence?node_id=event%3A...` | Retrieve node and supporting event/alert evidence |
| GET | `/api/v1/incidents/{incident_id}/attack-graph/edge-evidence?edge_id=edge%3A...` | Retrieve edge and supporting event/alert evidence |
| GET | `/api/v1/incidents/{incident_id}/attack-paths?event_limit=1000&max_paths=50` | Reconstruct and rank bounded candidate paths |
| GET | `/api/v1/graph` | Observed graph nodes and edges |
| GET | `/api/v1/progression` | Observed versus confirmed progression |
| GET | `/api/v1/reachability` | Potential next targets |
| GET | `/api/v1/mitre` | Local ATT&CK technique index |
| GET | `/api/v1/mitre/version` | ATT&CK catalog provenance and version |
| GET | `/api/v1/mitre/tactics` | ATT&CK tactic catalog |
| GET | `/api/v1/mitre/techniques?search=PowerShell&tactic=Execution` | Search validated techniques |
| GET | `/api/v1/mitre/techniques/{technique_id}` | Retrieve technique details |
| POST | `/api/v1/mitre/map-detection/{detection_id}` | Validate and persist detection mapping |
| GET | `/api/v1/mitre/mappings?detection_id=...&technique_id=...` | Retrieve evidence mappings |
| POST | `/api/v1/process?path=...` | Process a file inside `data` |
| POST | `/api/v1/datasets/upload` | Upload and process JSON/JSONL/CSV/LOG/Windows Event XML exports |
| POST | `/api/v1/datasets/local/register` | Register a local file or recursively scan a folder without copying; folder results are individual datasets |
| POST | `/api/v1/datasets/init?filename=events.jsonl&size=1000&total_chunks=1&chunk_size=33554432` | Create persistent chunk upload session |
| PUT | `/api/v1/datasets/{dataset_id}/chunks/{chunk_number}` | Upload and acknowledge one verified, retry-safe chunk |
| POST | `/api/v1/datasets/{dataset_id}/complete` | Assemble source, verify byte count and SHA-256 |
| GET | `/api/v1/datasets/history?limit=100&offset=0&search=events` | Search/page dataset library; includes `source_available` (chunk receipts are returned by the per-dataset route) |
| GET | `/api/v1/datasets/{dataset_id}` | Retrieve upload/analysis progress, received chunk numbers, source availability, and validation metadata |
| POST | `/api/v1/datasets/{dataset_id}/select` | Reuse an existing source and set it active |
| GET | `/api/v1/datasets/active/current` | Retrieve the persistent active dataset |
| DELETE | `/api/v1/datasets/{dataset_id}?confirm=true` | Delete source after explicit confirmation; retain run/audit history |
| POST | `/api/v1/datasets/{dataset_id}/validate?check_duplicates=false` | Stream-validate format, UTF-8, records, timestamps and checksum; optional disk-backed duplicate scan |
| GET | `/api/v1/datasets/{dataset_id}/validation` | Retrieve persisted validation results |
| GET | `/api/v1/datasets/{dataset_id}/validation/report?format=json` | Download validation JSON or CSV |
| GET | `/api/v1/processing/profiles` | List built-in and persisted analysis profiles |
| POST | `/api/v1/processing/profiles` | Create a versioned processing profile |
| PUT | `/api/v1/processing/profiles/{profile_id}` | Update a custom profile; existing analysis runs keep their snapshots |
| POST | `/api/v1/processing/profiles/{profile_id}/duplicate` | Duplicate a profile with a new name |
| DELETE | `/api/v1/processing/profiles/{profile_id}` | Delete a custom profile; built-ins are read-only |
| GET | `/api/v1/processing/system` | Retrieve current process/worker memory, system RAM, CPU, and queued-byte metrics |
| GET | `/api/v1/datasets/{dataset_id}/processing-recommendation?profile_id=...` | Inspect dataset characteristics and retrieve a profile-aware, unbenchmarked recommendation |
| POST | `/api/v1/datasets/{dataset_id}/analyze` | Queue bounded FR-1, FR-2 and FR-3 processing as a distinct run; accepts optional `profile_id` and/or one-run `configuration` |
| GET | `/api/v1/datasets/{dataset_id}/metrics` | Check chunks, throughput, RAM and CPU |
| GET | `/api/v1/datasets/{dataset_id}/runs` | List persistent analysis runs for a dataset |
| GET | `/api/v1/analysis-runs?limit=100&offset=0` | Page persistent analysis history |
| GET | `/api/v1/analysis-runs/{run_id}` | Retrieve run stages, status, results and errors |
| GET | `/api/v1/analysis-runs/compare?left_run_id=...&right_run_id=...` | Compare compatible stored run metrics |
| GET | `/api/v1/datasets/{dataset_id}/audit` | Retrieve dataset upload, selection, validation and deletion audit events |
| GET | `/api/v1/datasets/{dataset_id}/report?format=json` | Download dataset and analysis history as JSON, CSV, or PDF |
| GET | `/api/v1/storage` | Retrieve raw, processed, temporary, checkpoint and disk usage |
| GET | `/api/v1/events?dataset_id=...` | Retrieve normalized events for one dataset |
| POST | `/api/v1/investigate` | Local evidence-bounded investigation summary |

Processing configurations support `mode` (`automatic` or `manual`),
`chunk_size_bytes`, `max_individual_chunk_bytes`, `max_records_per_chunk`,
`memory_budget_bytes`, `max_queue_chunks`, `max_queue_bytes`,
`backpressure_threshold`, `worker_count` (zero means automatic),
`max_concurrent_tasks`, and `detection_engines`. The current FR-1 run requires
all five integrated core engines (rule-based, ML anomaly, behavioral, Sigma,
and threat intelligence); unsupported optional engines are rejected rather than
silently recorded as enabled.

Example request:

```json
{
  "profile_id": "mitigata-standard",
  "configuration": {
    "mode": "manual",
    "chunk_size_bytes": 4194304,
    "max_individual_chunk_bytes": 16777216,
    "max_records_per_chunk": 5000,
    "memory_budget_bytes": 1073741824,
    "max_queue_chunks": 2,
    "max_queue_bytes": 16777216,
    "backpressure_threshold": 0.8,
    "worker_count": 2,
    "max_concurrent_tasks": 2,
    "detection_engines": ["rule_based", "ml_anomaly", "behavioral", "sigma", "threat_intelligence"]
  }
}
```

In automatic mode the selected profile and dataset-aware recommendation
determine safe operational values. Every run stores its effective configuration,
profile version, start/completion times, software version, and performance
metrics. Changing chunk framing produces a separate checkpoint namespace.

Upload chunk size defaults to 32 MiB. Successful chunk acknowledgements, byte
counts, chunk SHA-256 digests, and session metadata persist in the existing
SQLite state file. Retrying an identical acknowledged chunk is idempotent;
different bytes at an acknowledged position are rejected. The browser reports
acknowledged progress separately from bytes currently in flight. Resume by
choosing the exact original file again. Upload pause, cancellation, and
cross-browser session ownership are not implemented.

Finalization assembles the original source beneath `data/uploads/{dataset_id}`,
verifies its declared size and stores a SHA-256 checksum. Dataset metadata,
analysis runs, active selection, and audit history are persisted in
`data/processed/fr1_checkpoints.sqlite`. Deleting a dataset removes only its
unique upload directory; metadata, analysis-run summaries, and audit history
are retained.
Incomplete uploads cannot be selected as active sources. The history API
reports `source_available=false` until the assembled source is present; if a
previously stored source has gone missing, selection returns HTTP 409 with
instructions to upload the original file again.

## Local dataset registration

For a local deployment, `POST /api/v1/datasets/local/register` accepts
`{"path":"C:\\Users\\you\\Documents\\security.jsonl","source_os":"windows"}`.
`source_os` is analyst-provided metadata (`unknown`, `windows`, `macos`,
`android`, `linux`, or `network`); the service does not auto-detect it. A
directory is scanned recursively for JSON, JSONL, CSV, LOG, and Windows Event
XML-export files (up to 500); each file is registered as an individual dataset
and remains at its original path. Repeating an unchanged registration reuses
the existing dataset record. A changed file produces a new dataset version;
the previous entry is marked unavailable by size/mtime metadata checks.
Deleting a local dataset removes only its registry record/source status and
does not delete the file.

For safety, filesystem access is restricted to configured roots. The default
is the project `data` directory. Add roots using `KILLCHAIN_LOCAL_DATA_ROOTS`
with platform path separators (semicolon-delimited on Windows), then restart
the API. This endpoint is for a backend running on the same machine as the
files; a remote server cannot access a browser user's local filesystem. The API
currently has no authentication, so keep it bound to loopback and do not
expose it to untrusted networks.

`GET /api/v1/events` returns normalized event pages (up to 1,000 per request)
and accepts `dataset_id`, `search`, `limit`, and `offset`. Raw source bytes are
not stored in DuckDB; event details include normalized fields and the exact
source record reference, while the registered local file remains the raw
evidence source.

Supported structured input is JSON, JSONL, CSV, LOG, and Windows Event XML
exports. Binary EVTX, macOS Unified Log, Android Logcat-specific, and arbitrary
XML formats are not currently parsed. An `.xml` file without recognized
Windows Event `System/EventID` records is rejected by validation; partially
recognized exports produce a warning for unrecognized records. Do not treat
file-extension acceptance as a claim of parser support.

Analysis runs use the bounded FR-1 parser/normalizer and existing FR-2
detection/FR-3 mapping, with dataset-scoped event IDs and detector queries.
Checkpointed FR-1 chunks can be reused after interruption. Progress uses a
record denominator from streaming validation and reaches 100% only after the
implemented stages complete. The current dataset pipeline does not
automatically execute FR-4 correlation, FR-5 attack-path reconstruction, or
FR-6 reachability; those features remain available through their existing
investigation APIs. Multiple simultaneous analyses are bounded to two workers
with a four-job admission limit.

Validation is streaming. Optional exact duplicate detection uses a temporary
disk-backed SQLite index rather than retaining record hashes in RAM and may be
slow or require scratch disk proportional to record count. Timestamp checks
apply only to common recognized timestamp fields; security-log schemas vary.
Storage telemetry reports filesystem usage and does not automatically remove
source data.

Detection filtering supports `severity`, `host`, `user`, and `ip` query
parameters. Sigma support is intentionally limited to the documented
field-equality subset in `config/sigma_rules.yaml`; unsupported constructs are
reported by the Sigma detector rather than treated as matches. IOC matching
uses only entries explicitly supplied in `config/threat_intelligence.yaml`.

## FR-4 incident correlation

Correlation selects structured suspicious events, categories referenced by
enabled sequence rules, and events explicitly referenced by stored detections
rather than materializing every normalized record. It
preserves event IDs, timestamp order, entity source fields, IP source/destination
roles, relationship confidence, and raw alert references. Shared usernames and
account IDs are weak evidence and cannot create an edge alone. Benign events
sharing only one entity do not form an incident; suspicious evidence,
progression context, or multiple strong shared entities are required.

Rules in `config/correlation_rules.yaml` support `enabled`, `sequence`,
`event_conditions`, `required_entities`, `time_window_minutes`, `severity`, and
`evidence_requirements` (`shared_entity` and `ordered_events`). The default
correlation window is 30 minutes, incident duration is capped at 120 minutes,
and clock skew tolerance is 120 seconds. Rule edits can be made in YAML or
enabled/disabled through the API. `CorrelationState` accepts chunks, deduplicates
event IDs, handles late/out-of-order chunks, and exports a checkpoint;
`CorrelationCheckpointStore` persists and recovers those snapshots from SQLite.
The API job registry is process-local, so a restarted server reruns correlation
from durable DuckDB events rather than resuming the prior job ID.

Correlation jobs report candidate events processed, records in the database,
elapsed time, candidate throughput, and sampled peak process RSS. These are
runtime measurements, not benchmark claims. ML alert similarity results are
labelled `suggested_unconfirmed` and are never added to incidents or graphs
automatically. Risk output returns its weighted evidence contributors and
marks compromise confirmed only when an explicit confirmation is present.

## FR-5 attack-path reconstruction

The incident-scoped attack graph uses event, alert, incident, entity, and
ATT&CK-technique nodes. Edges retain type, evidence references, supporting
event IDs, timestamp, confidence, correlation method, and observed/inferred
status. Entity and event edges are created only from normalized event fields
or FR-4 evidence. ATT&CK mappings add context nodes, not attack transitions.
Successful network connections and logins are not treated as proof of
compromise.

The graph, timeline, and path endpoints accept the filters `host`, `user`,
`ip`, `technique`, `start`, and `end`. Time bounds are ISO-8601 timestamps.
Timestamps are normalized to UTC; timeline ties retain source order and
events without usable timestamps are placed last. Invalid or reversed time
bounds return HTTP 400. Graph event pagination defaults to 250 and is capped
at 1,000 per graph request; node and edge responses are also paginated.
Path reconstruction defaults to 1,000 events, accepts at most 5,000 per
request, and returns at most 100 candidates. Traversal is additionally bounded
to 12 events per path and 10,000 edge expansions. Truncated results identify
the applicable bound rather than implying completeness. The incident graph
cache holds at most eight serialized graph inputs per API process.

Path scores use an explainable evidence-strength, supported-transition,
detection-reliability, specificity, event-identity, timestamp, and
contradiction rubric. Scores rank candidates and are not calibrated
probabilities. Outputs distinguish directly observed edges,
evidence-supported inferences, and partial unverified paths, and include
evidence gaps plus a conservative progression handoff for FR-6. A
network connection or login alone never marks a host compromised.

The dashboard's Attack graph section uses these APIs for the graph, timeline,
path candidates, and selected-node/edge evidence. It supports text/type
filtering, pan/zoom, dragging, fullscreen, timeline playback, and JSON/PNG
export. Graph, timeline, and path requests are bounded independently; large
incidents may therefore require pagination to inspect later evidence. The
current cache and background-job registries are process-local. Graph builds
are incident-scoped rather than dataset-wide, but incident payload retrieval
still deserializes the stored incident JSON and should be load-tested against
large production incidents before deployment.

## FR-6 attacker progression and reachability

The global endpoints accept an optional `incident_id`; incident-scoped
endpoints are preferred for investigation. Progression records observed
events, asset states, timestamp order, evidence references, and evidence
gaps. A successful login or connection does not confirm compromise. To record
confirmation, submit:

```json
{
  "status": "confirmed_compromised",
  "evidence_references": ["forensic:case-123"],
  "notes": "Optional analyst context"
}
```

to `PUT /api/v1/assets/{asset_id}/compromise-status`. The current local API
does not authenticate callers, so confirmation records explicitly say that
the caller identity is unauthenticated.

Asset creation accepts `asset_id`, `asset_type`, `name`, optional `hostname`,
`ip_addresses`, `network_segment`, `criticality`, and one or more
`evidence_references`. Topology relationships support
`network_connectivity`, `network_segment`, `can_route_to`, and `connects_to`;
both endpoint assets must already exist and evidence references are required.
Permissions require a known target asset, principal identity/type, permission,
and evidence references. Reachability returns potential paths, their evidence,
and a `potential_access_not_observed_movement` classification. Missing
topology is returned as a limitation, not guessed connectivity.

## FR-7 risk, reports, alert lineage, rules, and live events

Risk weights are configured in `config/risk_weights.yaml` and must include all
supported factors and sum to 100. API output includes each factor's value,
weight, points, and evidence references. It is not a calibrated probability.
Report format must be `json` or `pdf`; JSON is deterministic and PDF uses a
local dependency-free writer. Both derive from persisted incident evidence.

The raw-alert ledger fingerprint uses `(engine_source, rule_id, sorted
event_ids)`; alerts without event IDs use their detection ID. This prevents
repeat processing from inflating counts while retaining the original alert
record. Existing alert rows are streamed into the ledger once during a schema
migration. Correlation updates the associated incident ID; use
`GET /api/v1/alerts/stats` to inspect counts.

Sigma rule endpoints modify `config/sigma_rules.yaml`, persist version numbers
and audit records in DuckDB, and report unsupported constructs. Supported
rules require a unique `id`, `title`, severity `level`, and a single exact
selector with `condition: selection`. Supported fields are `event.category`,
`event_type`, `hostname`, `username`, `source_ip`, and `destination_ip`.
Lists mean exact-match OR values within one field; selector fields are ANDed.
Wildcards, modifiers, aggregation, and compound conditions are not executed.

Live endpoints accept **normalized JSON events**, not arbitrary raw files.
Required fields are `event_id`, ISO timestamp, `source_type`,
`raw_event_reference`, and supported `category`. The batch endpoint accepts at
most 500 records. Duplicate event IDs with different content return HTTP 409.
Supported enabled Sigma rules run on arrival. Correlation queries a maximum
of 500 evidence-entity candidates, applies the configured FR-4 sequence
categories and time window, and is persisted in DuckDB. Failed/interrupted
events retain a retry payload; successful rows retain a payload hash and the
normalized event. `POST /api/v1/live/recover` retries persisted pending/failed
work; recovery is explicit rather than automatic on process startup.

SSE update IDs resume from the `Last-Event-ID` header or `after` query
parameter. The replay table retains the latest 10,000 updates; if a reconnect
falls behind that window, the API emits `stream_reset` with
`reload_snapshot: true`. `GET /api/v1/live/checkpoint` exposes the last
successfully processed event and pending/failed counts.

The local API does not provide authentication, authorization, TLS termination,
or per-tenant isolation. Keep it on a trusted host and place it behind an
authenticated reverse proxy before any shared-network deployment. No Sarvam
credential was read, changed, or used by these APIs.
