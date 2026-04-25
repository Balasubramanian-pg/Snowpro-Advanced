# Data Modeling

# 1. Title
SnowPro Advanced: Data Modeling Architecture & Pattern Selection

# 2. Overview
- **What it does**: Defines logical schema design, physical storage alignment, incremental transformation patterns, and optimization attachment strategies tailored to Snowflake’s columnar, separated storage-compute architecture.
- **Why it exists**: Traditional RDBMS modeling rules (heavy normalization, strict indexing, row-store optimization) are inefficient or counterproductive in Snowflake. Misapplied normalization increases join overhead, while uncontrolled denormalization inflates storage and breaks incremental boundaries. Explicit modeling patterns align business grain with micro-partition mechanics, caching behavior, and query execution strategies.
- **Where it fits**: Sits between validated staging layers and downstream consumption (BI, ML, data sharing). Governs how entities are structured, how relationships are resolved, and how storage/compute tradeoffs are managed.
- **Intended consumer**: Data engineers, analytics engineers, platform architects, BI developers, and SnowPro Advanced candidates evaluating schema design, SCD implementation, incremental modeling, optimization strategies, and exam-relevant anti-patterns.

# 3. SQL Object Summary
| Field | Value |
|-------|-------|
| Object Scope | Logical/Physical Schema Design & Transformation Mapping |
| Type | Fact/Dimension Tables, One Big Tables (OBT), Materialized Views, Secure Views, SCD Layers |
| Purpose | Align business grain with Snowflake execution mechanics, optimize query performance, enforce incremental boundaries |
| Source Objects | Cleaned staging tables, validated dimensions, provider shares, external feeds |
| Output Object | Presentation-layer datasets, incremental fact tables, governed views, materialized aggregates |
| Execution Mode | Batch (full rebuild), Incremental (`MERGE`/`INSERT`), Zero-Copy Clone (dev/test), Scheduled (mat view refresh) |

# 4. Architecture
Snowflake modeling shifts from row-store normalization to columnar denormalization with metadata-driven access. The architecture layers raw staging, validated dimensions, incremental facts, and presentation views, with optimization primitives attached based on query patterns.

```mermaid
graph TD
  A[Validated Staging] -->|Grain Alignment| B[Dimension Tables (SCD/Static)]
  A -->|Temporal/Event Mapping| C[Incremental Fact Tables]
  B -->|Denormalization/Flattening| D[One Big Table (OBT)]
  C -->|Aggregation/Materialization| E[Summary Tables / Materialized Views]
  D -->|Governed Exposure| F[Secure Views / BI Interface]
  E -->|Query Acceleration| F
  B -.->|Clustering / Search Opt| C
  C -.->|Clustering / Search Opt| C
  D -.->|Zero-Copy Cloning| G[Dev/Test Sandbox]
  F -.->|RLS / DDM / Sharing| H[Consumer Layer]
```

# 5. Data Flow / Process Flow
| Step | Input | Transformation | Output | Purpose |
|------|-------|----------------|--------|---------|
| 1. Grain Definition | Business requirements, source entity analysis | Cardinality mapping, event vs snapshot classification | Explicit grain contract | Prevents aggregation skew, join fan-out, and duplicate counting |
| 2. Schema Mapping | Staged data, dimension lookups | Surrogate key generation, type harmonization, null fallbacks | Fact/dimension table structures | Aligns source variance to consistent query interface |
| 3. Incremental Boundary Resolution | Validated facts, stream/task offsets | `MERGE` with `IS DISTINCT FROM`, `QUALIFY` dedup, late-arriving fact handling | Updated fact state | Enables bounded compute, prevents full rebuilds |
| 4. Optimization Attachment | Query profile, filter patterns, join frequency | Clustering keys, search optimization, materialized views | Optimized storage/metadata | Reduces scanned partitions, eliminates repeated joins |
| 5. Exposure Layer Creation | Curated tables, governance requirements | Secure views, RLS/DM policies, BI aliases | Query-ready datasets | Enforces access control, abstracts physical structure |

# 6. Logical Breakdown of the SQL
| Component | Responsibility | Inputs | Outputs | Dependencies | Failure Modes / Risks |
|-----------|----------------|--------|---------|--------------|-----------------------|
| Fact Table (Incremental) | Event/measurement storage, metric aggregation | Staged transactions, surrogate keys, timestamps | Appended/merged fact rows | Dimension availability, stream offset state | Grain mismatch causes double-counting; late-arriving facts break time windows |
| Dimension Table (SCD Type 2) | Historical attribute tracking | Source attributes, business keys, change detection | Current + historical rows, `START_DATE`/`END_DATE`/`CURRENT_FLAG` | Deterministic change hashing, timezone alignment | Overlapping date ranges if `MERGE` order incorrect; high update volume inflates Time Travel |
| One Big Table (OBT) | Pre-joined denormalized dataset | Fact + dimensions, flattening logic, type casting | Wide table, single scan resolution | Stable join keys, controlled storage growth | Join fan-out inflates rows; storage cost outweighs compute savings if dimensions change frequently |
| Materialized View | Pre-aggregated or filtered query cache | Base table query, `GROUP BY`/`WHERE` clauses | Incrementally refreshed result set | Base table stability, refresh warehouse | Refresh stalls if base DDL changes; cannot use `DISTINCT` or non-deterministic functions |
| Secure View | Governance & abstraction layer | Underlying tables/views, RLS/DM policies | Filtered/redacted projection | Role privileges, policy bindings | Optimizer disables predicate pushdown; forces full evaluation, may bypass pruning |
| Bridge/Mapping Table | Many-to-many resolution | Entity relationships, weight/effective dates | Junction rows, allocation keys | Source hierarchy stability | Missing bridge records cause orphaned facts; complex joins increase spill risk |

# 7. Data Model
| Entity | Role | Important Fields | Grain | Relationships | Keys | Null Handling |
|--------|------|------------------|-------|---------------|------|---------------|
| `FACT_TRANSACTION` | Event-level measurement storage | `SURROGATE_KEY`, `DIM_KEYS`, `EVENT_TS`, `METRIC_VALUE`, `LOAD_BATCH_ID` | 1 row = 1 discrete business event | Joins to dimensions via FK mapping; feeds OBTs/mat views | Surrogate key, natural event ID | `NULL` metrics excluded from aggregations via `COALESCE` or filter |
| `DIM_CUSTOMER_SCD2` | Historical attribute tracking | `BUSINESS_KEY`, `SURROGATE_KEY`, `ATTRIBUTES`, `START_TS`, `END_TS`, `IS_CURRENT` | 1 row = 1 attribute state period | Linked to fact via surrogate or business key | Surrogate key (PK), business key (lookup) | Historical rows preserve original state; nulls explicit, not inferred |
| `OBT_ANALYTICS` | Pre-joined consumption dataset | Flattened dimensions, metrics, effective dates, status flags | 1 row = 1 business event with resolved context | Standalone; replaces multi-table joins | Event surrogate key | Denormalized nulls preserved; no join resolution at query time |
| `MV_DAILY_AGGREGATE` | Materialized summary | `DATE_BUCKET`, `DIM_GROUP`, `SUM_METRIC`, `COUNT_EVENTS`, `LAST_REFRESH` | 1 row = 1 daily grouped summary | Derived from fact/dim base tables | Composite: `DATE_BUCKET` + `DIM_GROUP` | Null groups excluded or mapped to `UNKNOWN` during refresh |

**Output Grain**: Explicitly defined per entity. `FACT_TRANSACTION` = event-level. `DIM_CUSTOMER_SCD2` = state-period level. `OBT_ANALYTICS` = event-level with resolved context. `MV_DAILY_AGGREGATE` = daily grouped level. Grain mismatch between fact and dimension causes join fan-out or silent metric inflation.

# 8. Business Logic
| Rule | Effect | Implementation Pattern | Edge Case |
|------|--------|------------------------|-----------|
| **SCD Change Detection** | Triggers historical row creation | `MD5(CONCAT_WS('|', col1, col2))` comparison vs target | Non-deterministic null hashing produces false positives; requires `COALESCE` normalization |
| **Incremental Boundary** | Limits compute to new/changed data | `WHERE load_ts > (SELECT MAX(load_ts) FROM target)` + stream offset | Timezone drift drops recent records; late-arriving facts require separate merge window |
| **Denormalization Threshold** | Decides OBT vs Star/Snowflake | Join frequency vs storage growth vs dimension update velocity | High-update dimensions invalidate OBT cache, trigger stale reads until rebuild |
| **Late-Arriving Fact Handling** | Routes missing dimension context | `LEFT JOIN` with `COALESCE(dim_key, 'UNKNOWN')`, delayed reprocess batch | Facts assigned to `UNKNOWN` skew aggregations; requires reconciliation job |
| **Many-to-Many Allocation** | Distributes metrics across relationships | Bridge table join with weight/effective date filtering | Missing bridge records cause metric drop; overlapping weights double-count |
| **Materialized View Refresh** | Maintains pre-aggregated state | `ALTER MATERIALIZED VIEW ... REFRESH` on schedule or DML trigger | Base table `ALTER` pauses refresh; incompatible schema changes break view |

# 9. Transformations
| Source | Derived | Formula / Rule | Business Meaning | Impact |
|--------|---------|----------------|------------------|--------|
| Staged event rows | Surrogate keys | `MD5(CONCAT_WS('|', COALESCE(biz_key, 'UNKNOWN'), event_ts))` | Stable, deterministic join identifiers | Prevents drift across source updates; enables idempotent upserts |
| Dimension attributes | SCD Type 2 historical rows | `MERGE` with `END_DATE = CURRENT_TIMESTAMP()` on match, `INSERT` for new | Tracks state changes over time | Enables point-in-time reporting; inflates storage proportional to change velocity |
| Multi-table joins | OBT projection | `FACT JOIN DIM1 JOIN DIM2` with `COALESCE`, type alignment | Eliminates repeated join compute at query time | Reduces latency, increases storage; requires refresh strategy |
| Fact metrics | Daily/weekly aggregates | `GROUP BY date_trunc('DAY', event_ts), dim_group` with `SUM/COUNT` | Pre-computed summaries for BI dashboards | Materialized view refresh cost scales with base table churn |
| Raw timestamps | Effective dating | `TO_TIMESTAMP_TZ(event_ts) AT TIME ZONE 'UTC'`, `START_DATE`/`END_DATE` generation | Global time alignment for SCD and incremental windows | Eliminates timezone skew in historical comparisons |

# 10. Parameters / Variables / Macros
| Name | Type | Purpose | Allowed Format | Default | Usage | Effect on Output |
|------|------|---------|----------------|---------|-------|------------------|
| `INCREMENTAL_STRATEGY` | Enum | Model update pattern | `MERGE`, `INSERT`, `DELETE+INSERT`, `MAT_VIEW` | `MERGE` | dbt/pipeline config | Determines idempotency, Time Travel impact, and refresh cost |
| `SCD_TYPE` | Enum | Dimension change handling | `TYPE_1` (overwrite), `TYPE_2` (history), `TYPE_3` (limited history) | `TYPE_2` | Dimension DDL | Controls storage growth, historical query capability, merge complexity |
| `CLUSTERING_KEYS` | String | Micro-partition optimization | Column list (`col1, col2`) | None | Table DDL | Enables partition pruning; poor selection increases maintenance cost |
| `SEARCH_OPTIMIZATION` | Boolean | Equality/range filter acceleration | `TRUE` / `FALSE` | `FALSE` | Table/View DDL | Bypasses clustering maintenance; accelerates wide tables, incurs background credit cost |
| `REFRESH_WAREHOUSE` | String | Compute allocation for mat views | Existing warehouse name | None | Mat view definition | Controls refresh latency and credit consumption |
| `GRANULARITY` | String | Aggregation time window | `HOUR`, `DAY`, `WEEK`, `MONTH` | `DAY` | Aggregate models | Determines row count, storage footprint, and BI query alignment |

# 11. APIs / Interfaces
| Interface | Invocation Method | Input Structure | Output Structure | Error Behavior | Consumers |
|-----------|-------------------|-----------------|------------------|----------------|-----------|
| Secure View | `SELECT` | Role context, query predicates | Filtered/redacted rows | Fails on missing privileges; secure flag disables pushdown | BI tools, external consumers, governed dashboards |
| Materialized View | Automatic refresh / `ALTER ... REFRESH` | Base table DML | Aggregated/filtered result set | Stalls on base schema change; requires manual resume | High-frequency dashboards, summary reporting |
| dbt Model Contract | CI/CD validation | Schema YAML, `data_type`/`constraints` | Pass/fail deployment gate | Blocks deployment on mismatch | Engineering pipelines, cross-team data products |
| Snowflake Data Share | `GRANT ... TO SHARE` | Database, schemas, tables/views | Read-only consumer copy | Fails if role lacks `OWNERSHIP`; provider revokes access | External partners, cross-account analytics |
| Zero-Copy Clone | `CREATE TABLE ... CLONE` | Source table/view, point-in-time | Independent metadata reference | No data duplication; inherits Time Travel window | Dev/test environments, safe experimentation, rollback testing |

# 12. Execution / Deployment
- **Manual vs Scheduled**: Fact/dimension builds run via `TASK` or dbt. Materialized views refresh on schedule or base DML triggers. Clones created manually or via CI/CD hooks.
- **Batch vs Incremental**: Initial loads use full `CREATE TABLE AS SELECT`. Production runs use incremental `MERGE` with stream tracking or timestamp boundaries.
- **Orchestration**: CI/CD enforces contract testing, blocks deployments on grain mismatch, and automates zero-copy cloning for dev/test. Airflow/Dagster manage refresh ordering and warehouse scaling.
- **Upstream Dependencies**: Staging data availability, dimension SCD state, stream offset alignment, warehouse capacity, retention policies.
- **Environment Behavior**: Dev/test use clones, disabled search optimization, and aggressive Time Travel. Prod enforces strict contracts, incremental strategies, RLS/DM policies, and monitoring.
- **Runtime Assumptions**: `MERGE` requires exact join keys and `IS DISTINCT FROM` for idempotency. Materialized views cannot reference temporary tables or non-deterministic functions. Cloning preserves metadata, not physical data.

# 13. Observability
| Metric | Implementation | Detection Method | Operational Threshold |
|--------|----------------|------------------|------------------------|
| Model freshness | `MAX(LOAD_TS)` vs `CURRENT_TIMESTAMP()` per table | dbt freshness tests, scheduled validation query | >2x expected interval = pipeline stall or upstream failure |
| Grain validation | `COUNT(DISTINCT grain_keys)` vs `COUNT(*)` | Reconciliation query per batch | Mismatch >0% = join fan-out or duplicate ingestion |
| Storage growth rate | `ACTIVE_BYTES` trend over 30 days | `ACCOUNT_USAGE.TABLE_STORAGE_METRICS` | >15% weekly increase = uncontrolled SCD, missing retention, or OBT bloat |
| Materialized view lag | `CURRENT_TIMESTAMP() - LAST_REFRESH_TIME` | `SHOW MATERIALIZED VIEWS` or catalog query | >1 hour lag = warehouse contention or base table DDL change |
| Query pruning efficiency | `PARTITIONS_SCANNED / PARTITIONS_TOTAL` on model queries | `QUERY_HISTORY` aggregation | >0.4 scanned = clustering/search optimization ineffective or filter misalignment |

# 14. Failure Handling & Recovery
| Failure Scenario | What Breaks | Detection | Fallback Behavior | Recovery Approach |
|------------------|-------------|-----------|-------------------|-------------------|
| Grain mismatch in fact/dim join | Metric inflation, duplicate counting | Row count spike, `COUNT(*) > COUNT(DISTINCT grain_key)` | Downstream aggregations skew | Validate join keys pre-merge, add `QUALIFY` dedup, enforce bridge table constraints |
| Incremental offset drift | Pipeline loads stale or skips data | `MAX(load_ts)` in target < upstream source | Data gaps or duplicate inserts | Reset stream offset, use `MERGE` with `IS DISTINCT FROM`, add audit checkpoint table |
| Materialized view refresh failure | Stale aggregates, BI dashboards outdated | `REFRESH_STATE != 'SUCCESS'`, alerting | Queries fall back to base table (slow) | Resume warehouse, check base DDL compatibility, trigger manual refresh |
| SCD overlapping date ranges | Historical queries return ambiguous state | `END_TS` of row N > `START_TS` of row N+1 | Time-based reports produce incorrect snapshots | Enforce `END_TS = LEAD(START_TS) - INTERVAL 1 MS` in merge, add validation constraint |
| Schema drift on base table | Materialized view or clone breaks | `ALTER TABLE` drops column used in view/clone | View fails refresh, clone metadata invalid | Rebuild view with updated schema, use `CREATE OR REPLACE` with contract validation |
| Uncontrolled OBT growth | Storage costs spike, scan latency increases | `TABLE_STORAGE_METRICS` trend, BI query timeout | Performance degradation, budget overrun | Revert to star schema for high-update dimensions, implement tiered refresh, archive cold data |

# 15. Security & Access Control
| Control | Implementation | Effect |
|---------|----------------|--------|
| Row-Level Security (RLS) | `ROW ACCESS POLICY` on presentation tables/views | Filters output by role/domain; evaluated post-pruning, may reduce efficiency |
| Dynamic Data Masking (DDM) | `MASKING POLICY` on sensitive columns in OBTs/dimensions | Redacts PII at projection; preserves underlying storage for authorized roles |
| Secure View Abstraction | `SECURE VIEW` wrapping base tables | Prevents predicate pushdown leakage, enforces access boundaries, disables caching |
| Role-Based Schema Exposure | `GRANT USAGE/SELECT` by environment and domain | Isolates dev/test from prod, restricts cross-domain joins without approval |
| Data Sharing Governance | `SECURE SHARE` with explicit object grants | Exposes curated datasets without duplication; respects source RLS/DM policies |

# 16. Performance / Scalability Considerations
| Bottleneck | Cause | Tradeoff | Mitigation |
|------------|-------|----------|------------|
| Join fan-out in facts | Missing grain alignment, ambiguous FK mapping | Metric inflation, spill, timeout | Validate cardinality pre-join, use bridge tables for many-to-many, enforce `QUALIFY` |
| Unoptimized wide table scans | OBT without clustering/search opt, high null density | Full micro-partition scans, high credit consumption | Attach search optimization for equality/range predicates, cluster on high-filter columns |
| Materialized view refresh cost | Base table high DML volume, complex aggregation | Warehouse saturation, credit spike | Schedule refresh off-peak, use incremental mat views (when supported), filter base DML impact |
| Late filtering in CTEs | `WHERE` applied after join/group in model logic | Unnecessary compute, cache bypass | Push filters to source staging, use `QUALIFY` for post-window logic, flatten CTEs |
| SCD storage bloat | High-frequency attribute updates, missing retention policy | Time Travel + active storage inflation | Switch to Type 1 for non-audit attributes, archive historical rows, enforce `END_TS` cleanup |
| Secure view optimizer penalty | `SECURE` flag disables rewrite/caching | Consistent full evaluation, slower BI response | Use secure views only for governance; expose underlying tables to trusted roles for performance |

# 17. Assumptions & Constraints
- **Columnar storage favors denormalization**: Normalized star schemas work but incur join overhead. OBTs reduce compute until storage cost or dimension update frequency outweighs benefits.
- **Clustering is not indexing**: It’s metadata-driven pruning. Poor key selection increases maintenance cost without improving performance. Search Optimization Service accelerates wide tables without manual maintenance.
- **Grain is explicit, not inferred**: Snowflake does not enforce primary/foreign keys. Duplicate joins, missing bridges, or ambiguous keys cause silent metric inflation.
- **Incremental boundaries require deterministic ordering**: Late-arriving data, timezone drift, or non-idempotent `MERGE` breaks state consistency. `IS DISTINCT FROM` prevents unnecessary updates.
- **Materialized views have strict constraints**: Cannot use `DISTINCT`, `UDFs`, `TEMP` tables, or non-deterministic functions. Refresh pauses on base DDL changes.
- **Secure views disable optimizer rewrites**: `SECURE` prevents caching and predicate pushdown. Use only for governance; expose base objects to trusted consumers for performance.
- **Exam trap assumptions**: SnowPro Advanced tests grain alignment rules, SCD Type 2 implementation mechanics, clustering vs search optimization tradeoffs, incremental `MERGE` idempotency, materialized view constraints, secure view optimizer behavior, and zero-copy cloning semantics. Memorize defaults and architectural limits.

# 18. Future Enhancements
- **Automate grain validation in CI/CD**: Embed row count vs distinct key ratio checks into deployment pipelines. Block models that introduce join fan-out or duplicate counting.
- **Implement dynamic OBT generation**: Use metadata-driven flattening to auto-denormalize frequently joined star schemas. Refresh on dimension change thresholds, not fixed schedules.
- **Standardize SCD retention policies**: Archive historical Type 2 rows beyond audit window to external storage. Reduce active + Time Travel storage bloat.
- **Integrate search optimization recommendations**: Profile query patterns, auto-attach search optimization to high-frequency equality/range predicates. Monitor background credit cost vs query latency savings.
- **Harden incremental idempotency contracts**: Enforce `IS DISTINCT FROM` and deterministic `ORDER BY` across all `MERGE` operations. Add automated regression tests for offset drift.
- **Adopt incremental materialized views**: Leverage Snowflake’s evolving incremental refresh capabilities to reduce full re-computation cost. Align refresh cadence with upstream SLA, not arbitrary schedules.
