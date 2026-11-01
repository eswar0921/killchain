# KILLCHAIN AI User Guide

## Start the local application

From `C:\Users\rishi\OneDrive\Desktop\hack`, start the backend:

```powershell
# Optional: add permitted local folders using semicolon-separated full paths.
$env:KILLCHAIN_LOCAL_DATA_ROOTS = "C:\Users\you\Downloads;D:\SecurityLogs"
.\.venv\Scripts\python.exe -m uvicorn api.main:app --host 127.0.0.1 --port 8000
```

In another PowerShell terminal:

```powershell
Set-Location frontend
npm run dev
```

Open `http://localhost:5173`. FastAPI's interactive API documentation is at
`http://127.0.0.1:8000/docs`.

The dashboard has two workspaces. **Command center** contains Overview,
Datasets, and Reports. The Live monitor page has been removed; live-event APIs
remain available. **Detection** contains Events, Incidents,
Investigation, Rules, and Threat & risk. Dataset manager tools are under
**Command center → Datasets**; the live elapsed analysis timer is under
**Command center → Datasets → Processing**. Event membership links in
**Detection → Events** come from incident event IDs returned by the correlation
API. Under **Detection → Investigation**, choose an incident and filter its
graph/timeline by host, user, IP, and time. Shared entities alone are not
presented as confirmed incident links. The standalone ATT&CK catalog is hidden;
evidence-backed mappings remain visible in incident investigations.

## Investigation workflow

1. In **Command center → Datasets**, upload supported JSON, JSONL, CSV, LOG,
   or Windows XML event exports by file selection or drag-and-drop. The upload progress bar
   counts acknowledged bytes only; bytes still being transferred are shown
   separately. After an interrupted upload, select **Resume** and choose the
   exact same file.
   To process data already on the API machine, enter its path in **Register an
   existing file or folder**. Approved roots default to the project `data`
   directory; set `KILLCHAIN_LOCAL_DATA_ROOTS` before backend startup to permit
   other folders. Folder registration makes each supported file a separate
   dataset. Local files are not copied, and deleting their registry entry does
   not delete the source.
2. Select **Validate** to review streaming format, encoding, checksum,
   malformed-record and recognized timestamp results. The optional
   disk-backed duplicate scan can take additional time and scratch space.
3. Use **Reuse** to select a stored source without copying it, **Analyze** to
   start a separate analysis run, or **Delete** to remove a confirmed source
   file while retaining run and audit history. Open **Command center → Datasets
   → Processing** for acknowledged upload and record-based analysis progress;
   analysis continues while navigating dashboard sections.
4. Open **Command center → Datasets → History** to review prior runs and
   compare two runs, or **Storage** to inspect source, processed, checkpoint,
   temporary, and filesystem usage.
   Dataset reports are available as JSON, CSV, and PDF.
5. **Detection → Events** displays paginated normalized records and marks
   incident membership by exact event ID for a selected incident. Selecting a
   row opens its normalized evidence.
6. Use **Detection → Threat & risk** to view risk-ranked incidents, evidence
   scores, persisted detections, configured local IOC matches, and severity pie
   charts. Exploit-labelled alerts count only explicit exploit/CVE wording in
   detection metadata; they do not prove exploitability. No CVE enrichment or
   vulnerability feed is available.
7. Use **Detection → Incidents** to search and filter correlated incidents,
   review alert lineage/risk, and update the investigation status.
8. Open **Detection → Investigation** to review a compact host/user attack
   path and a separate chronological timeline. The path only draws an observed
   authentication link when a suspicious incident event explicitly supports
   it; other observed entities remain unconnected unless direct evidence
   supports a relationship. Select a host or user for its related events and
   source references. Select a timeline event to highlight its related entity
   and open the original event evidence. Expand the secondary path hypotheses,
   filters, and performance details only when needed.
9. **What Could the Attacker Reach Next?** lists evidence-referenced potential
   destinations separately from the confirmed path. Confidence is shown as
   unassessed when the backend does not calculate it. Potential reachability
   does not establish access, observed movement, or compromise. Progression and
   topology inventory details remain expandable; do not mark an asset
   compromised without an explicit source evidence reference.
10. Register real inventory records in the asset context before using
   reachability. Every network edge and permission must have one or more
   evidence references. Potential access is not observed movement.
11. Open **Command center → Reports** to search and filter stored incident and
    dataset reports. Select a result to preview its organized evidence summary,
    then download incident JSON/PDF or dataset JSON/CSV/PDF. The initial view
    uses an available stored record; if none exists, it shows a clearly
    identified report template rather than fabricated data.

The dataset-analysis job currently completes FR-1 ingestion/normalization,
FR-2 detection/fusion, and FR-3 mapping. FR-4 through FR-6 remain separately
available investigation features and are not automatically run by the Dataset
Manager. Upload pause and analysis cancellation are not supported.
Browser file selection cannot provide an arbitrary filesystem path to the
backend; local-path registration reads from the machine where the API runs.

No example incidents, benchmark values, or production topology are seeded by
the application. Add synthetic data only when it is clearly identified as
synthetic.

## Add evidence-backed topology

The **Assets & access** forms accept JSON bodies. An asset body resembles:

```json
{
  "asset_id": "asset-001",
  "asset_type": "host",
  "name": "workstation-01",
  "hostname": "workstation-01",
  "ip_addresses": ["10.0.0.12"],
  "network_segment": "corp-workstations",
  "criticality": "medium",
  "evidence_references": ["inventory:record-001"]
}
```

Create a directional network relationship only when the source has supporting
evidence:

```json
{
  "relationship_id": "route-001",
  "source_asset_id": "asset-001",
  "target_asset_id": "asset-002",
  "relationship_type": "network_connectivity",
  "evidence_references": ["firewall:policy-001"]
}
```

Permission records use `permission_id`, `principal_type`, `principal_id`,
`target_asset_id`, `permission`, and `evidence_references`.

To confirm compromise, select an asset and enter the forensic, event, or
ticket evidence reference in **Progression**. The local API does not
authenticate analysts; do not treat the recorded actor label as an identity
assertion.

## Manage supported Sigma rules

**Detection rules** provides create, edit, validate, delete, and audit history.
Rules are stored in `config/sigma_rules.yaml`. The implemented subset requires
a unique rule ID, title, supported severity, one exact-match selector, and
`condition: selection`. Only `event.category`, `event_type`, `hostname`,
`username`, `source_ip`, and `destination_ip` are accepted. Unsupported
conditions are reported; they do not silently degrade into different logic.

## Push normalized events and monitor updates

The live API accepts normalized events rather than raw log lines. Example
PowerShell request:

```powershell
$event = @{
  event_id = "live-001"
  timestamp = [DateTime]::UtcNow.ToString("o")
  source_type = "example-source"
  event_type = "network_connection"
  raw_event_reference = "source-file:record-001"
  category = "network_connection"
  hostname = "workstation-01"
  source_ip = "10.0.0.12"
  destination_ip = "203.0.113.15"
} | ConvertTo-Json

Invoke-RestMethod -Method Post `
  -Uri "http://127.0.0.1:8000/api/v1/live/events" `
  -ContentType "application/json" -Body $event
```

Batch requests may contain up to 500 events. Reuse of an event ID with
different content returns HTTP 409. The live-event API retains 10,000 updates and asks stale
clients to reload a snapshot. Check
`GET /api/v1/live/checkpoint`; retry failed/interrupted events with
`POST /api/v1/live/recover`.

## Risk and reports

Adjust evidence factor weights in `config/risk_weights.yaml`; all weights must
be non-negative and sum to 100. Returned scores include per-factor evidence
references and are not probabilities. JSON/PDF reports are derived from
persisted incident evidence and include explicit limitations.
