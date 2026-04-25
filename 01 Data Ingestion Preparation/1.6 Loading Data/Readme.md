# Loading Data

# 1. Title
SnowPro Advanced: Bulk & Streaming Data Loading Architecture

# 2. Overview
- **What it does**: Defines the mechanics, execution paths, error routing, and optimization strategies for ingesting structured, semi-structured, and delimited files into Snowflake via `COPY INTO` and Snowpipe.
- **Why it exists**: Load operations are the highest-leverage point for pipeline cost, latency, and data quality. Misconfigured file formats, improper parallelism, or incorrect error handling cause silent data loss, warehouse saturation, or unbounded compute spend. Deterministic loading ensures predictable ingestion, idempotent execution, and traceable audit trails.
- **Where it fits**: Operates as the entry point between external storage/cloud providers and Snowflake's internal storage layer. Precedes validation, transformation, and curation stages.
- **Intended consumer**: Data engineers, platform architects, ingestion pipeline developers, FinOps analysts, and SnowPro Advanced candidates evaluating load parallelism, Snowpipe behavior, error routing, and file format parsing mechanics.

# 3. SQL Object Summary
| Field | Value |
|-------|-------|
| Object Scope | Data Loading & Ingestion Pipeline |
| Type | `COPY INTO` + `Snowpipe` + `FILE_FORMAT` + Stage Configuration |
| Purpose | Move raw files into Snowflake tables with deterministic parsing, error isolation, and execution tracking |
| Source Objects | External stages (S3, Azure, GCS), internal stages, `PUT` CLI uploads, partner network drops |
| Output Object | Raw landing tables, error quarantine tables, `COPY_HISTORY` logs, Snowpipe load queues |
| Execution Mode | Batch (`COPY INTO`), Event-Driven (`Snowpipe Auto-Ingest`), API-Driven (`Snowpipe REST`), Manual (`PUT` + `COPY`) |

# 4. Architecture
Loading in Snowflake separates file staging from execution. Stages register metadata; compute engines (warehouse or serverless) parse and write rows. Error routing and history tracking operate alongside the main load path to preserve auditability without halting valid ingestion.

```mermaid
graph TD
  A[External Storage / CLI PUT] -->|File Transfer| B[Internal / External Stage]
  B -->|Metadata Registration| C[File Manifest]
  C -->|Trigger| D{Load Mechanism}
  D -->|Batch| E[COPY INTO (Warehouse)]
  D -->|Event/API| F[Snowpipe (Serverless/Standard)]
  E -->|Parsing & Validation| G[Row Insertion]
  F -->|Parsing & Validation| G
  G -->|Success| H[Raw Landing Table]
  G -->|Failure| I[Error Table / COPY_HISTORY]
  H -->|Downstream Validation| J[Transformation Layer]
  I -->|Alert/Retry| K[Quarantine & Recovery]
```

# 5. Data Flow / Process Flow
| Step | Input | Transformation | Output | Purpose |
|------|-------|----------------|--------|---------|
| 1. File Staging | Local/cloud files | `PUT`, cloud sync, or provider drop | Registered stage files with `METADATA$FILENAME`, `SIZE`, `MD5` | Establish deterministic file manifest for load tracking |
| 2. Format Resolution | `FILE_FORMAT` definition | Delimiter, encoding, compression, type mapping applied | Parsing configuration bound to `COPY`/`PIPE` | Standardize row splitting, null handling, and type coercion |
| 3. Load Execution | Stage + format + target table | Parallel parsing across warehouse nodes or serverless workers | Inserted rows, error logs, load history entries | Move data into Snowflake storage with minimal compute overhead |
| 4. Error Routing | Malformed rows, type mismatches | `ON_ERROR` routing (`ABORT`, `CONTINUE`, `SKIP_FILE`) | Partial load + error table or pipeline halt | Isolate bad data without losing valid records |
| 5. History & Audit | Load completion state | `COPY_HISTORY` insertion, `QUERY_ID` generation | Load traceability, row counts, file-to-table mapping | Enable reconciliation, debugging, and idempotency checks |

# 6. Logical Breakdown of the SQL
| Component | Responsibility | Inputs | Outputs | Dependencies | Failure Modes / Risks |
|-----------|----------------|--------|---------|--------------|-----------------------|
| `CREATE STAGE` | File location & credential binding | Cloud URL, storage integration, IAM/role | Stage metadata object | Network access, valid credentials | Expired tokens, path typos, missing integration |
| `CREATE FILE_FORMAT` | Parsing rule definition | `TYPE`, `FIELD_DELIMITER`, `RECORD_DELIMITER`, `COMPRESSION`, `NULL_IF`, `SKIP_HEADER` | Reusable format object | Target data structure | Mismatched delimiters cause row collapse, wrong encoding corrupts strings |
| `COPY INTO TABLE` | Bulk load execution | Stage, format, `PATTERN`, `ON_ERROR`, `VALIDATION_MODE` | Loaded rows, `COPY_HISTORY`, error table | Correct schema mapping, warehouse availability | Silent truncation on `ENFORCE_LENGTH=FALSE`, type mismatch halts load on `ABORT` |
| `CREATE PIPE` + `AUTO_INGEST=TRUE` | Event-driven micro-batch ingestion | Stage, cloud notification queue (SNS/PubSub/Event Grid) | Continuous row inserts | Cloud event bridge, pipe permissions | Missed notifications cause lag, duplicate delivery on retry triggers idempotent skip |
| `MATCH_BY_COLUMN_NAME` | Column alignment without positional mapping | Source column names, target schema | Column-matched insert | Exact name match, case sensitivity | Fails on extra/missing columns if `CASE_INSENSITIVE=FALSE`, drops unmapped cols |
| `VALIDATION_MODE` | Dry-run load testing | `COPY` syntax, `RETURN_N_ROWS`, `RETURN_ERRORS` | Error preview without modifying target | File availability, format alignment | Returns partial errors; does not validate downstream schema compatibility |
| `COPY_HISTORY` | Load audit & reconciliation | `QUERY_ID`, `FILE_NAME`, `ROWS_LOADED`, `FIRST_ERROR_MESSAGE` | Time-series load metadata | `MONITOR` role, account retention (14 days) | Historical queries expire; long-term audit requires external archival |

# 7. Data Model
| Entity | Role | Important Fields | Grain | Relationships | Keys | Null Handling |
|--------|------|------------------|-------|---------------|------|---------------|
| `RAW_LANDING` | Immutable append-only ingest target | `METADATA$FILENAME`, `METADATA$FILE_ROW_NUMBER`, `LOAD_TIMESTAMP`, parsed columns | 1 row = 1 parsed record from source file | Input for validation, transformation, quarantine | `METADATA$FILE_ROW_NUMBER` + `FILENAME` for dedup | Preserved as-is; nulls mapped per `FILE_FORMAT` rules |
| `COPY_HISTORY` | Load execution audit | `QUERY_ID`, `FILE_NAME`, `STAGE_LOCATION`, `ROW_COUNT`, `FIRST_ERROR_MESSAGE`, `STATUS` | 1 row = 1 file load attempt | Maps to `QUERY_HISTORY`, `WAREHOUSE_METERING_HISTORY` | `QUERY_ID` + `FILE_NAME` | `FIRST_ERROR_MESSAGE` null on success |
| `LOAD_ERROR_TABLE` | Isolated bad records | `RECORD`, `ERROR`, `FILENAME`, `LINE_NUMBER`, `CHARACTER_POSITION`, `CATEGORY` | 1 row = 1 parsing/validation failure | References `RAW_LANDING`, feeds retry pipeline | `ERROR_ID` (surrogate) | `RECORD` stores raw payload; `ERROR` contains deterministic message |
| `STAGE_MANIFEST` | File registration tracking | `FILENAME`, `SIZE`, `MD5`, `LAST_MODIFIED`, `IS_LOADED` | 1 row = 1 staged file | Drives `PATTERN` filtering, idempotency checks | `FILENAME` + `MD5` | `IS_LOADED` boolean; null before first successful `COPY` |

**Output Grain**: Determined at `RAW_LANDING`. 1:1 with source records after row splitting and format parsing. Grain shifts if `COPY` applies `SPLIT_PART`, JSON flattening, or aggregation during load.

# 8. Business Logic
| Rule | Effect | Implementation Pattern | Edge Case |
|------|--------|------------------------|-----------|
| **File Selection** | Loads only matching files | `PATTERN = '.*2024-.*\\.gz$'` | Regex mismatch drops valid files; missing escape characters break parsing |
| **Error Routing** | Controls pipeline halt vs continuation | `ON_ERROR = CONTINUE` + `ERROR_ON_COUNT_THRESHOLD` | High error rates inflate storage; `ABORT_STATEMENT` halts entire batch on first failure |
| **Column Matching** | Aligns source to target by name | `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` | Extra source columns ignored if not in target; missing target columns filled with defaults/nulls |
| **Null Mapping** | Standardizes missing/empty values | `NULL_IF = ('\\N', '', 'NULL')`, `EMPTY_FIELD_AS_NULL = TRUE` | Whitespace-only fields treated as valid strings unless `TRIM_SPACE = TRUE` |
| **Type Coercion** | Converts strings to target types | Implicit casting per `FILE_FORMAT`, `ENFORCE_LENGTH` toggle | `ENFORCE_LENGTH=FALSE` silently truncates; `ENFORCE_LENGTH=TRUE` fails load |
| **Idempotency** | Prevents duplicate file ingestion | Snowpipe tracks `METADATA$FILENAME` + content hash | Manual `COPY` lacks built-in dedup; requires external tracking or `COPY` with `FORCE=FALSE` |

# 9. Transformations
| Source | Derived | Formula / Rule | Business Meaning | Impact |
|--------|---------|----------------|------------------|--------|
| Raw string columns | Typed target columns | Implicit `CAST` via `FILE_FORMAT`, `DATE_FORMAT`, `TIMESTAMP_FORMAT` | Aligns external types to Snowflake standards | Mismatched format causes `ON_ERROR` routing; explicit `TRY_CAST` in post-load recommended |
| Semi-structured payload | Flattened relational columns | `col:$.path::TYPE` during `COPY` | Schema-on-read resolution at ingestion | Reduces storage bloat, enables pruning; invalid JSON paths route to error table |
| Multiple delimited fields | Surrogate key generation | `MD5(CONCAT_WS('|', col1, col2))` in post-load SQL (load-time limited) | Stable join key across sources | Not natively supported in `COPY`; computed in staging layer |
| Timestamp strings | UTC-normalized datetime | `TO_TIMESTAMP_TZ(col, format) AT TIME ZONE 'UTC'` | Global time alignment | Eliminates timezone skew in aggregations; requires explicit format mapping |
| Raw file metadata | Load audit fields | `METADATA$FILENAME`, `METADATA$FILE_ROW_NUMBER`, `CURRENT_TIMESTAMP()` | Traceability & incremental boundary | Enables deterministic dedup, supports reconciliation queries |

# 10. Parameters / Variables / Macros
| Name | Type | Purpose | Allowed Format | Default | Usage | Effect on Output |
|------|------|---------|----------------|---------|-------|------------------|
| `ON_ERROR` | Enum | Load failure behavior | `ABORT_STATEMENT`, `CONTINUE`, `SKIP_FILE`, `SKIP_FILE_N` | `ABORT_STATEMENT` | `COPY INTO` | Determines pipeline halt vs silent skip; affects error table population |
| `PATTERN` | String | File selection regex | POSIX-compatible regex | `.*` (all files) | `COPY INTO` | Controls which files are ingested; misalignment drops data |
| `MATCH_BY_COLUMN_NAME` | Enum | Column alignment strategy | `CASE_SENSITIVE`, `CASE_INSENSITIVE`, `DISABLED` | `DISABLED` | `COPY INTO` | Enables name-based mapping; fails if target lacks source columns |
| `ENFORCE_LENGTH` | Boolean | String truncation control | `TRUE` / `FALSE` | `TRUE` | `COPY INTO`, `FILE_FORMAT` | `FALSE` silently truncates; `TRUE` halts load on overflow |
| `COMPRESSION` | Enum | Automatic decompression | `AUTO`, `GZIP`, `BZ2`, `BROTLI`, `ZSTD`, `DEFLATE`, `RAW_DEFLATE`, `NONE` | `AUTO` | `FILE_FORMAT` | Determines parsing speed, storage transfer cost, warehouse CPU usage |
| `VALIDATION_MODE` | Enum | Dry-run error preview | `RETURN_N_ROWS`, `RETURN_ERRORS`, `RETURN_ALL_ERRORS` | None | `COPY INTO` | Prevents target modification; returns parse errors only |
| `PURGE` | Boolean | Post-load file cleanup | `TRUE` / `FALSE` | `FALSE` | `COPY INTO` | Removes files from stage after success; irreversible if enabled |

# 11. APIs / Interfaces
| Interface | Invocation Method | Input Structure | Output Structure | Error Behavior | Consumers |
|-----------|-------------------|-----------------|------------------|----------------|-----------|
| `COPY INTO TABLE` | SQL | Stage, format, pattern, error mode | Loaded rows, `COPY_HISTORY` entries | Fails or skips based on `ON_ERROR`; returns partial success on `CONTINUE` | Batch ingestion pipelines, dbt `source` models |
| `CREATE PIPE` + `COPY INTO` | SQL | Stage, auto-ingest flag, notification config | Continuous inserts, `PIPE` metadata | Throttled on notification loss; retries missed events up to retention | Real-time landing layers, streaming ingestion |
| `PUT` / `GET` | CLI / SDK | Local file, stage path, compression flag | Uploaded file in stage, transfer status | Fails on permission/quota mismatch, network timeout | Pre-load staging, archival, local-to-cloud sync |
| `SYSTEM$COPY_FILE_STATUS` | SQL | Pipe name, file list | Load status per file (`LOAD_STARTED`, `LOADED`, `LOAD_FAILED`) | Returns partial status if file not tracked | Pipeline monitoring, idempotency checks |
| `INFORMATION_SCHEMA.COPY_HISTORY` | SQL | Date range, table/pipe filters | Load records, row counts, error messages | 14-day retention; requires `MONITOR` role | Audit, reconciliation, FinOps tracking |

# 12. Execution / Deployment
- **Manual vs Scheduled**: `COPY INTO` runs manually or via CI/CD orchestration. `Snowpipe` runs event-driven or via `INSERT_FILES` REST API.
- **Batch vs Incremental**: `COPY INTO` is batch-oriented. Snowpipe enables micro-batch streaming (~5-10s latency). Incremental tracking relies on `METADATA$FILENAME` or external file manifest.
- **Orchestration**: Airflow/Dagster/dbt Cloud trigger `COPY` via SQL operators. Snowpipe integrates with cloud notification services. Tasks coordinate file arrival, load execution, and post-load validation.
- **Upstream Dependencies**: File availability, stage credentials, network egress, warehouse resume state, cloud event bridge configuration.
- **Environment Behavior**: Dev/test often use smaller files, `ON_ERROR=CONTINUE`, and manual `PUT`. Prod enforces `PURGE=FALSE`, strict error thresholds, serverless Snowpipe, and integration-linked alerting.
- **Runtime Assumptions`: `COPY` distributes work across warehouse nodes (threads = node count × cores). Serverless Snowpipe scales automatically but incurs cold-start latency. `COPY_HISTORY` retains 14 days. Load commits are atomic per file; partial file loads are not supported.

# 13. Observability
| Metric | Implementation | Detection Method | Operational Threshold |
|--------|----------------|------------------|------------------------|
| Load success rate | `ROWS_LOADED / (ROWS_LOADED + ERROR_ROWS)` per file | `COPY_HISTORY` aggregation | <99.5% sustained = format drift or source corruption |
| File sizing impact | `AVG(SIZE) / THREAD_COUNT` vs optimal range (10-250MB) | `COPY_HISTORY` + warehouse metrics | <5MB = underutilized parallelism; >1GB = spill/timeout risk |
| Snowpipe latency | `CURRENT_TIMESTAMP - FILE_LAST_MODIFIED` for loaded files | `SYSTEM$COPY_FILE_STATUS` | >10 min = notification queue backlog or compute saturation |
| Error table growth | `COUNT(*) FROM LOAD_ERROR_TABLE WHERE TIMESTAMP > NOW() - INTERVAL 1 DAY` | Query + alerting | >1k rows/day = systemic source degradation; requires format review |
| Warehouse utilization | `AVG(AVG_RUN_TIME) / WAREHOUSE_SIZE` during load | `WAREHOUSE_METERING_HISTORY` | >90% queue time = undersized warehouse; consider serverless or scale up |

# 14. Failure Handling & Recovery
| Failure Scenario | What Breaks | Detection | Fallback Behavior | Recovery Approach |
|------------------|-------------|-----------|-------------------|-------------------|
| Parse error on malformed row | `ON_ERROR=ABORT` halts entire file load | `FIRST_ERROR_MESSAGE` in `COPY_HISTORY` | Zero rows inserted for that file | Switch to `CONTINUE`, isolate bad rows, fix source or update `FILE_FORMAT` |
| Duplicate file ingestion | Double-counted metrics, inflated row counts | `METADATA$FILENAME` appears twice in `COPY_HISTORY` | Snowpipe skips automatically; manual `COPY` requires `FORCE=FALSE` + tracking | Implement file manifest, use `PURGE=FALSE`, track loaded MD5 hashes |
| Warehouse saturation | Load timeouts, spill to disk, high latency | `QUERY_HISTORY.SPILLED_BYTES`, queue wait time | Load fails or drifts past schedule | Scale warehouse, enable serverless Snowpipe, optimize file sizes |
| Notification queue loss | Snowpipe misses file events, ingestion stalls | `SYSTEM$COPY_FILE_STATUS` shows `NOT_STARTED` for recent files | Pipeline halts silently | Manually trigger `ALTER PIPE ... REFRESH`, verify cloud event bridge, monitor queue retention |
| Schema mismatch | Target table lacks source columns, type drift | `COPY` fails with `column does not exist` or type error | Load halts, no data inserted | Alter target table, use `MATCH_BY_COLUMN_NAME`, enforce schema contracts pre-load |
| Network timeout during `PUT` | Partial file upload, corrupted stage | CLI/SDK returns error, stage shows incomplete file | Subsequent `COPY` fails with parse error | Retry `PUT`, verify checksum, use resumable uploads, implement pre-load validation |

# 15. Security & Access Control
| Control | Implementation | Effect |
|---------|----------------|--------|
| Storage Integration | IAM role binding, temporary STS tokens, no long-lived credentials | Decouples Snowflake from static keys, enables secure cross-account access |
| Stage Encryption | Server-side encryption (SSE-S3, SSE-KMS), client-side `PUT` encryption | Complies with data-at-rest requirements, prevents unauthorized stage access |
| Network Policy | `NETWORK POLICY` + `ALLOWLIST` for `PUT`/`GET` and external stage access | Restricts ingestion to approved CIDRs, blocks exfiltration |
| Role-Based Grants | `USAGE`/`READ` on stage, `INSERT`/`OWNERSHIP` on target table | Enforces least-privilege ingestion, prevents unauthorized table modification |
| Error Table Isolation | Separate schema for `LOAD_ERROR_TABLE`, restricted `SELECT` access | Protects raw payloads from downstream consumption, enables secure debugging |

# 16. Performance / Scalability Considerations
| Bottleneck | Cause | Tradeoff | Mitigation |
|------------|-------|----------|------------|
| Underutilized parallelism | Small files (<10MB compressed), low warehouse size | Warehouse nodes idle, high latency per GB | Consolidate files, increase warehouse size, use `AUTO` compression |
| Warehouse spill & timeout | Oversized files (>250MB), complex load-time transforms | Disk I/O bottleneck, query failure | Split files, move transforms to post-load staging, enable serverless Snowpipe |
| Snowpipe cold-start latency | Serverless compute initialization after inactivity | 10-30s delay for first micro-batch | Keep pipe active via dummy loads, schedule warm-up queries, accept tradeoff for burst workloads |
| Non-sargable `PATTERN` regex | Complex regex, missing anchors | Stage metadata scan overhead, delayed load start | Use prefix/suffix anchors (`^data_.*\.gz$`), limit scope, pre-filter externally |
| Late filtering in `COPY` | `WHERE` clause applied during load vs post-load | Increased parse cost, wasted compute on skipped rows | Use `PATTERN` for file-level filtering, apply `WHERE` in staging query |
| High compression overhead | `BROTLI`/`ZSTD` on large warehouses | CPU-bound parsing, slower throughput than `GZIP` | Match compression to warehouse capacity, test `AUTO` vs explicit, prioritize `GZIP` for speed |

# 17. Assumptions & Constraints
- **No concrete SQL provided**: Documentation reflects canonical loading patterns for SnowPro Advanced. Exact behavior depends on file format, stage type, warehouse configuration, and cloud provider integration.
- **Parallelism scales with warehouse size**: `COPY INTO` uses threads = node count × cores. X-Small = 8 threads, 4X-Large = 128 threads. Serverless Snowpipe auto-scales but lacks explicit thread control.
- **Idempotency is Snowpipe-native, not COPY-native**: Snowpipe skips duplicate files via metadata tracking. `COPY INTO` requires external tracking or `FORCE=FALSE` to prevent duplicates.
- **`COPY_HISTORY` retention is fixed at 14 days**: Historical load tracking expires automatically. Long-term audit requires exporting to external tables or data lake.
- **Load-time transforms are limited**: `COPY` supports basic JSON extraction, column matching, and null mapping. Complex logic (window functions, joins, aggregations) must run post-load.
- **Exam trap assumptions**: SnowPro Advanced tests `COPY` parallelism mechanics, `ON_ERROR` routing behavior, `MATCH_BY_COLUMN_NAME` constraints, `VALIDATION_MODE` output, Snowpipe idempotency guarantees, `SYSTEM$COPY_FILE_STATUS` usage, file sizing optimization, and `PUT` compression defaults. Memorize engine limits and default behaviors.

# 18. Future Enhancements
- **Automate file sizing validation**: Pre-load pipeline checks file size distribution, rejects outliers (<1MB or >500MB), and triggers consolidation/splitting jobs.
- **Implement dynamic error thresholds**: Replace static `ON_ERROR` routing with adaptive logic that switches to `CONTINUE` below threshold, `ABORT` above, and routes to quarantine with auto-alerting.
- **Standardize load-time schema contracts**: Enforce `STRICT=TRUE` in `FILE_FORMAT`, integrate `INFORMATION_SCHEMA.COLUMNS` validation pre-`COPY`, fail fast on drift.
- **Optimize Snowpipe notification routing**: Centralize cloud event queues, implement dead-letter queues for missed notifications, add retry logic with exponential backoff.
- **Integrate post-load transform staging**: Replace inline `COPY` transforms with dedicated staging layer using `STREAM` + `MERGE`. Enables idempotency, better caching, and easier debugging.
- **Harden load audit pipeline**: Automate `COPY_HISTORY` export to external storage, build reconciliation dashboards tracking `ROWS_LOADED` vs source manifest, alert on variance.
