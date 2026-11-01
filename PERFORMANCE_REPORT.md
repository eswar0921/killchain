# FR-1/FR-6/FR-7 Performance Report

## Synthetic live-ingestion benchmark

Command: `.\.venv\Scripts\python.exe evaluation\fr67_live_benchmark.py`

Workload: 1,000 normalized synthetic events, no detector matches, submitted in
batches of up to 500. The benchmark uses in-memory DuckDB and calls the batch
handler directly; ASGI/network transport and frontend work are excluded.

| Measurement | Result |
|---|---:|
| Events requested / processed | 1,000 / 1,000 |
| Elapsed time | 4.551719 seconds |
| Throughput | 219.70 events/second |
| Peak traced Python allocation | 4,375,816 bytes (about 4.17 MiB) |
| Process RSS delta during run | 16,588,800 bytes (about 15.82 MiB) |
| Alerts created | 0 |
| Live updates retained | 1,000 |

These are results from one local synthetic run, not a production capacity
claim. In particular, the no-match workload avoids detector and incident
formation work, and in-memory DuckDB does not measure disk contention,
concurrent writers, or long-term retention costs. The RSS delta is process
wide and can include interpreter/runtime allocations; it is not an isolated
per-event memory measurement.

## Synthetic FR-1 FortiGate ingestion benchmark

Command: `.\.venv\Scripts\python.exe evaluation\fr1_benchmark.py --records 50000`

Workload: 50,000 synthetic FortiGate key-value records through the full FR-1
streaming, parsing, normalization, DuckDB and Parquet pipeline. Configuration
used four parser workers, 5,000 records per chunk, and at most four in-flight
chunks. This is a local temporary-database benchmark; upload, FR-2/FR-3
detection, and API transport are excluded.

| Measurement | Result |
|---|---:|
| Records normalized | 50,000 |
| Input size | 11,750,000 bytes |
| Elapsed time | 6.764 seconds |
| Throughput | 7,392.27 events/second |
| Input throughput | 1.66 MiB/second |
| Parent-process peak working set | 272,236,544 bytes |

The test ran on Windows with Python 3.11.15 and an AMD Ryzen 7 7445HS
(6 cores / 12 logical processors, 15.23 GiB RAM). The memory measurement is
for the parent process only and excludes worker-process RSS. The workload is
synthetic and does not establish 15/30/50 GB dataset capacity or production
throughput. On the same synthetic workload and configuration, Arrow-backed
bulk insertion reduced one earlier 50,000-record run from 48.525 seconds
(1,030 events/second) to 6.764 seconds (7,392 events/second), about 7.2x
faster; these are individual runs, not repeated-run statistics.

## Real-source prefix configuration comparison

Command: `.\.venv\Scripts\python.exe evaluation\fr1_config_benchmark.py --file data\firex-30G.log --suite --repeats 2`

The six configurations were compared twice each on the same first 64 MiB
record-aligned prefix of `firex-30G.log` (67,108,822 bytes; 126,829 normalized
records). Every run produced the same record count. Temporary DuckDB,
checkpoint, and Parquet outputs were removed after each trial. The actual file
is 19,621,285,387 bytes (about 18.27 GiB), so this sample is only about 0.34%
of the input and is not a full-dataset benchmark.

| Configuration | Workers | Chunk target | Median elapsed | Median input rate | Max observed RSS (workers included) |
|---|---:|---:|---:|---:|---:|
| Existing/default | 4 | 64 MiB / 5,000 records | 22.133 s | 2.91 MiB/s | 620.8 MiB |
| Small | 4 | 1 MiB | 48.823 s | 1.88 MiB/s | 566.0 MiB |
| Medium | 4 | 8 MiB | 18.226 s | 3.51 MiB/s | 654.5 MiB |
| Large | 4 | 16 MiB | 19.493 s | 3.30 MiB/s | 656.9 MiB |
| Automatic | 2 | 4 MiB | 19.105 s | 3.36 MiB/s | 513.9 MiB |
| Memory optimized | 2 | 4 MiB | 25.169 s | 2.74 MiB/s | 517.4 MiB |

On this prefix, the 8 MiB configuration had the best median throughput
(about 17.7% faster than the existing/default comparison) while automatic
used less peak memory. Small chunks were slower; the results do not support
one universal chunk size. For the 8 MiB runs, the largest accumulated stage
times were DuckDB writes (about 11.8 s), parser work across workers (about
7.4 s), and normalization work across workers (about 4.4 s). Worker parser
times are accumulated service times and can overlap; these values identify
where to investigate, not exclusive wall-clock contributions. FR-2 detector
and ML timings, FR-4 correlation, and FR-3 mapping are not part of this
FR-1-only benchmark.

The previously supplied 18 GB / 4 min 34 sec / approximately 67 MiB/s figure
was not independently reproduced. A full six-configuration run against all
19.6 GB was not run: extrapolating the prefix rate suggests roughly 1.5 hours
for one best-case FR-1 pass, so a full comparison would consume many hours and
write substantial temporary output. The benchmark command above can repeat it
when a full-dataset maintenance window is approved; do not treat the prefix
estimate as a full-file timing.

## Not measured

- Million-event throughput or sustained ingestion.
- Persistent-database throughput and storage growth.
- Concurrent request throughput or contention behavior.
- Matched-alert and multi-stage sequence workload performance.
- Large-incident graph construction and path-reconstruction latency.
- Frontend graph rendering or investigation-panel opening latency.
- Peak production memory under a representative dataset.
- Full-file Dataset Manager upload and end-to-end processing throughput on
  multi-GB files; the measured real-source trial was only a 64 MiB prefix.
- Dataset analysis peak RAM, sustained storage growth, simultaneous upload
  contention, and behavior on the requested 15/30/50 GB dataset tiers.

The Dataset Manager's automated tests verify bounded chunk/request handling,
streaming validation and parsing, dataset-scoped detection, and persisted run
state. They do not establish large-file capacity or processing throughput.
Optional exact duplicate scanning builds a temporary disk-backed SQLite index;
its wall-clock time and peak scratch-space consumption have not been
benchmarked on representative datasets.

The deployment guide recommends local testing and load testing against the
intended persistent database and workload before shared or production use.

## Large-file profile and FR-1 hot-path optimization

The local history contained a 19,621,285,387-byte source (about 18.27 GiB)
and completed analysis runs. The run taking **4:34.839** recorded:

| Stage | Elapsed |
|---|---:|
| Dataset validation | 100.868 s |
| FR-1 ingestion stage | 173.721 s |
| Detection and fusion | 0.166 s |
| MITRE mapping | 0.013 s |
| Completion | 0.009 s |

This is **not a cold, full-processing baseline**: the saved FR-1 metrics report
`records_processed: 0`, and all 7,437 chunks were already checkpointed. It
measures a validation plus checkpoint-reuse scan, not 37 million records
parsed, normalized, inserted, and detected in that run. A second checkpoint
reuse run took 9:18.282 (109.514 s validation and 448.557 s ingestion), showing
large run-to-run variation. Neither run records filesystem cache state. The
provided 67 MiB/s figure therefore cannot be treated as a measured fresh-run
throughput or a reliable performance target.

In the earlier real-source 64 MiB prefix comparison, the 8 MiB configuration
had the largest measured FR-1 service-time components: DuckDB insertion
(about 11.8 s), parser work across workers (about 7.4 s), and normalization
(about 4.4 s). Parser and normalization times are accumulated worker service
times that overlap; they are not additive wall time. This profile identified
the storage and per-record transform paths for follow-up, but does not measure
the complete 18.27 GiB run.

### Change made

Chunk admission previously serialized each record once to check a byte
boundary, serialized it again when appending it, then serialized the record
again in the pipeline to account for queued bytes. Plain log lines were also
UTF-8 encoded twice for byte accounting and the record-size guard. The
streamer now computes the canonical queued-record byte size once, reuses it
for boundary and append decisions, reports the current chunk byte count to
the pipeline, and reuses a single line encoding for both counters. Record
contents, IDs, parser inputs, record ordering, malformed-record handling,
chunk framing limits, and checkpoints are unchanged.

### Post-change measurement

A paired CPU microbenchmark ran the old and new size-accounting operations
over the same first 50,000 non-empty records (26,369,075 source bytes) from
`firex-30G.log`, with three repeats:

| Measurement | Old accounting | Optimized accounting |
|---|---:|---:|
| Median time | 0.6684 s | 0.2734 s |
| Accounting-only change | — | 2.445x faster / 59.1% less time |

This is a **per-record accounting microbenchmark only**; it excludes disk
I/O, parsing, normalization, model validation, Parquet, DuckDB, and detection.
It is not an end-to-end FR-1 throughput result and must not be extrapolated to
claim the 18.27 GiB analysis is 59.1% faster. The 7 targeted FR-1 streaming
and bounded-pipeline regression tests passed after the change; the complete
backend suite then passed with 54 tests.

### Outstanding benchmark requirements and resource limits

- A fresh full-file run and the sequential/thread/process/hybrid comparisons
  have not been performed. The stored full-file runs reused checkpoints, so
  there is no trustworthy before/after full-processing result.
- At profiling time, the existing API process held several GiB of resident
  memory, system free memory was about 1 GiB, and the API health request timed
  out. No additional 18 GiB analysis or multi-configuration benchmark was
  started under that resource pressure.
- Do not claim an end-to-end improvement until a fresh, checkpoint-isolated
  full-file run is repeated on the same hardware, data, rules, configuration,
  and cache conditions. Record actual detection counts, malformed records,
  processed/failed records, RSS, CPU, disk use, and stage timings.
