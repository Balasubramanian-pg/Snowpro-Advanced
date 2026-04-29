# Querying And Working Data

# 1. Title
SnowPro Advanced: Query Execution, Optimization & Data Access Architecture

# 2. Overview
- **What it does**: Defines the end-to-end lifecycle of SQL query execution, including parsing, optimization, micro-partition pruning, join resolution, caching behavior, and result materialization.
- **Why it exists**: Queries operate against a distributed, metadata-driven storage engine. Without explicit understanding of pruning boundaries, cache invalidation rules, join strategy selection, and window evaluation order, pipelines suffer from unbounded scans, warehouse spill, cache misses, and non-deterministic performance.
- **Where it fits**: Sits between transformation logic and downstream consumption. Governs how SQL translates to compute operations, how data is accessed from storage, and how results are returned or cached.
- **Intended consumer**: Data engineers, analytics engineers, query tuning specialists, platform architects, and SnowPro Advanced candidates evaluating optimizer behavior, caching semantics, pruning mechanics, and execution profiling.

# 3. SQL Object Summary
| Field | Value |
|-------|-------|
| Object Scope | Query Execution Engine & Optimization Framework |
| Type | Runtime Execution Pipeline + Cost-Based Optimizer + Multi-Tier Caching |
| Purpose | Translate declarative SQL into deterministic, vectorized compute operations with predictable latency and cost |
| Source Objects | Micro-partitions, metadata catalogs, result cache, local disk cache, external stages |
| Output Object | Result sets, query execution profiles, cache state records, pruning metrics |
| Execution Mode | Compiled, Optimized, Vectorized Batch Execution with Multi-Layer Caching |

# 4. Architecture
Query execution follows a deterministic compilation pipeline. SQL is parsed, bound to metadata, optimized using cost-based heuristics, and executed across distributed warehouse nodes. Caching operates at three layers: metadata, result, and local disk. Pruning and join strategies dictate compute distribution.

```mermaid
graph TD
  A[SQL Statement] -->|Parse & Validate| B[AST Generation]
  B -->|Metadata Binding| C[Cost-Based Optimizer]
  C -->|Predicate Pushdown| D[Pruning Engine]
  C -->|Join Ordering| E[Join Strategy Selector]
  D -->|Micro-Partition Scan| F[Vectorized Execution Engine]
  E -->|Hash/Merge/Nested Loop| F
  F -->|Aggregation/Window| G[Result Materialization]
  G -->|Cache Evaluation| H{Result Cache Hit?}
  H -->|YES| I[Return Cached Payload]
  H -->|NO| J[Return Fresh Result + Cache Write]
  F -.->|Spill/Audit| K[Execution Profile + Metrics]
```

# 5. Data Flow / Process Flow
| Step | Input | Transformation | Output | Purpose |
|------|-------|----------------|--------|---------|
| 1. Parse & Bind | Raw SQL, session context | Syntax validation, identifier resolution, privilege check | Abstract Syntax Tree (AST) | Establish execution boundaries before optimization |
| 2. Optimization | AST, table metadata, statistics | Predicate pushdown, join reordering, CTE materialization decision | Execution plan with cost estimates | Minimize scanned partitions, select optimal join strategy |
| 3. Pruning & Cache Check | Optimized plan, metadata cache, result cache | Micro-partition filtering via min/max metadata, cache key matching | Pruned partition list or cached result | Eliminate irrelevant storage reads, skip execution when possible |
| 4. Execution | Pruned partitions, join/hash structures | Vectorized scan, hash join build/probe, window sort/aggregation | Intermediate row sets, spill files if memory exceeded | Compute transformations in parallel across warehouse nodes |
| 5. Projection & Return | Final row sets, cache evaluation | Column projection, type formatting, cache persistence | Result set to client, cache entry written | Deliver deterministic output, enable subsequent cache hits |

# 6. Logical Breakdown of the SQL
| Component | Responsibility | Inputs | Outputs | Dependencies | Failure Modes / Risks |
|-----------|----------------|--------|---------|--------------|-----------------------|
| Parser & Binder | Syntax validation, object resolution | SQL text, session role, database/schema context | Validated AST, bound identifiers | Schema exists, privileges granted | Fails on syntax error, unresolved objects, or missing `USAGE` grants |
| Cost-Based Optimizer (CBO) | Plan generation, cost estimation | AST, column statistics, partition metadata, join predicates | Execution plan, join order, strategy selection | Up-to-date statistics, consistent data types | Stale stats cause suboptimal plans; implicit casts disable join reordering |
| Pruning Engine | Partition elimination | Filter predicates, clustering keys, search optimization, min/max metadata | Reduced micro-partition scan set | Native type filters, aligned clustering, exact match or range predicates | Non-sargable filters, timezone shifts, or string comparisons bypass pruning |
| Join Resolver | Algorithm selection & execution | Join keys, row counts, memory limits, sort state | Hash table build, sorted merge streams, nested loop pairs | Accurate row estimates, sufficient warehouse memory | Hot keys skew hash joins; missing sort keys force spill; cartesian joins timeout |
| Window/Aggregation Evaluator | Post-join computation | Partition keys, order clauses, frame definitions, aggregate functions | Windowed rows, grouped aggregates, ranked results | Deterministic ordering, memory for sort/hash, correct grain | Unbounded frames increase memory; non-deterministic `ORDER BY` breaks rank consistency |
| Cache Manager | Result/metadata caching | Query text, session params, timezone, warehouse config, plan hash | Cache hit/miss, result payload storage/retrieval | Exact match across all dimensions, no volatile functions | Dynamic filters, `CURRENT_TIMESTAMP`, or UDF state bypass result cache |

# 7. Data Model
| Entity | Role | Important Fields | Grain | Relationships | Keys | Null Handling |
|--------|------|------------------|-------|---------------|------|---------------|
| `QUERY_EXECUTION_PROFILE` | Execution state & performance tracking | `QUERY_ID`, `WAREHOUSE_NAME`, `EXECUTION_STATUS`, `TOTAL_ELAPSED_TIME`, `PARTITIONS_SCANNED`, `SPILLED_BYTES` | 1 row = 1 query execution instance | Maps to `ACCOUNT_USAGE.QUERY_HISTORY`, `WAREHOUSE_METERING_HISTORY` | `QUERY_ID` | `NULL` on canceled queries; partial metrics flushed on timeout |
| `CACHE_STATE_REGISTRY` | Cache hit/miss tracking | `QUERY_HASH`, `CACHE_TYPE`, `HIT_FLAG`, `EXPIRY_TS`, `SESSION_PARAMS_HASH` | 1 row = 1 cache evaluation event | Links to `QUERY_HISTORY`, `RESULT_CACHE` metadata | `QUERY_HASH` + `CACHE_TYPE` | `NULL` if cache disabled or bypassed by session state |
| `PRUNING_AUDIT` | Partition elimination metrics | `QUERY_ID`, `TABLE_NAME`, `TOTAL_PARTITIONS`, `PRUNED_PARTITIONS`, `FILTER_PREDICATE` | 1 row = 1 table scan pruning event | References `INFORMATION_SCHEMA.TABLES`, execution plan | `QUERY_ID` + `TABLE_NAME` | `NULL` if full scan forced or metadata unavailable |

**Output Grain**: 1 query execution = 1 deterministic result set. Row count changes based on join cardinality, filter selectivity, and window aggregation. Cache returns identical byte-for-byte payload when hit.

# 8. Business Logic
| Rule | Effect | Implementation Pattern | Edge Case |
|------|--------|------------------------|-----------|
| **Predicate Pushdown** | Filters applied before join/scan | `WHERE` on base columns, native types, range operators | `WHERE UPPER(col) = 'X'` or `WHERE CAST(col) = 1` disables pushdown, forces full scan |
| **Join Algorithm Selection** | Determines execution strategy | Hash join (default, unsorted), Merge join (sorted inputs), Nested loop (small datasets) | Skewed join keys cause hash build memory spill; missing sort keys prevent merge join |
| **Cache Invalidation** | Controls result reuse | Exact SQL text match + identical session params, timezone, warehouse config | `USE_CACHED_RESULT=FALSE`, `CURRENT_TIMESTAMP`, or dynamic table UDFs bypass cache |
| **Window Evaluation Order** | Defines computation sequence | `WHERE` -> `GROUP BY` -> `HAVING` -> Window (`QUALIFY` filters post-window) | `QUALIFY` applied after window calculation; cannot push into storage scan |
| **CTE vs Subquery Materialization** | Controls repeated scan behavior | Inline CTE (parsed once, reused in plan), `MATERIALIZED` hint (forces intermediate write) | Large reused CTEs without hint cause repeated scans; excessive materialization inflates temp storage |
| **Pruning Boundary Alignment** | Determines partition elimination | Filter on clustering/search key, exact type match, no function wrapping | Timezone offset on `TIMESTAMP_LTZ` breaks partition bounds; string comparison on numeric column bypasses pruning |

# 9. Transformations
| Source | Derived | Formula / Rule | Business Meaning | Impact |
|--------|---------|----------------|------------------|--------|
| Raw filter predicate | Pruned micro-partitions | Min/max metadata comparison + clustering key alignment | Eliminates irrelevant storage reads | Reduces scanned bytes from TB to GB; non-aligned filters trigger full scan |
| Join key mapping | Hash table / sorted streams | Key hashing, bucket distribution, sort merge alignment | Matches rows across datasets without nested iteration | Hash joins scale linearly; skew triggers spill and latency spikes |
| Window frame definition | Ranked/aggregated rows | `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, partition sort | Computes running totals, ranks, or peer-group aggregates | Large partitions increase sort memory; unbounded frames prevent streaming |
| Aggregation projection | Grouped metrics | `GROUP BY` + `SUM/AVG/COUNT`, `DISTINCT` elimination | Collapses detail rows to business summary | Late `GROUP BY` inflates intermediate row count; early filtering reduces compute |
| Cache payload | Stored result set | Byte-for-byte serialization + metadata hash | Skips execution on identical subsequent query | Cache hit returns <100ms; bypass forces full re-execution cost |

# 10. Parameters / Variables / Macros
| Name | Type | Purpose | Allowed Format | Default | Usage | Effect on Output |
|------|------|---------|----------------|---------|-------|------------------|
| `USE_CACHED_RESULT` | Boolean | Enable/disable result cache | `TRUE` / `FALSE` | `TRUE` | Session/Query | `FALSE` forces execution; disables cache write for session |
| `QUERY_TAG` | String | Execution tracking & profiling | Text label | `NULL` | Session/Query | Groups related queries in `QUERY_HISTORY`; enables cost attribution |
| `OPTIMIZER_MODE` | Enum | CBO behavior control | `COST_BASED` (default), `RULE_BASED` (legacy) | `COST_BASED` | Session | Alters plan generation; `RULE_BASED` ignores statistics, forces fixed strategies |
| `DATE_INPUT_FORMAT` | String | Temporal parsing consistency | Format pattern | `AUTO` | Session | Affects filter predicate evaluation; mismatched format causes pruning bypass |
| `WAREHOUSE_SIZE` | Enum | Compute allocation for execution | XSMALL → 6XLARGE | Defined at creation | Warehouse config | Directly impacts parallelism, memory limits, spill threshold, and credit consumption |
| `ENABLE_QUERY_ACCELERATION` | Boolean | Allow Snowflake auto-allocating compute for eligible queries | `TRUE` / `FALSE` | `FALSE` | Warehouse/Session | Enables query acceleration service; reduces runtime for cache-miss scans, incurs separate credits |

# 11. APIs / Interfaces
| Interface | Invocation Method | Input Structure | Output Structure | Error Behavior | Consumers |
|-----------|-------------------|-----------------|------------------|----------------|-----------|
| `EXPLAIN` | SQL | Query text | Execution plan tree, cost estimates, partition scan info | Returns plan only; does not execute | Query tuning, plan validation, CI/CD gating |
| `QUERY_HISTORY` / `ACCOUNT_USAGE` | SQL | Date range, warehouse, query ID filters | Execution metrics, status, cache source, spill bytes | 14-day retention in `ACCOUNT_USAGE`; requires `MONITOR` | Performance auditing, cost tracking, alerting |
| Snowsight Query Profile UI | Web | `QUERY_ID` | Visual DAG, operator timings, spill/pruning metrics, join strategy | Requires `MONITOR`/`USAGE`; loads only completed queries | Engineering debugging, join/spill diagnosis |
| JDBC/ODBC `ResultSet` | Driver API | Query execution | Paginated result set, fetch size control | Network timeout on large fetches; cursor state loss | BI tools, Python/R clients, data apps |
| `SYSTEM$EXPLAIN_PLAN_JSON` | SQL | Query text | Machine-readable plan structure | Fails on syntax error; returns JSON blob | Automated plan analysis, optimizer regression testing |

# 12. Execution / Deployment
- **Manual vs Scheduled**: Interactive queries run ad-hoc via client tools. Batch queries execute via `TASK`, dbt, or orchestration layers with explicit warehouse assignment.
- **Batch vs Interactive**: Batch favors larger warehouses, result caching disabled for volatile schedules. Interactive leverages result cache, smaller warehouses, and query acceleration for latency-sensitive workloads.
- **Orchestration**: CI/CD pipelines run `EXPLAIN` against staging, block deployments on full-scan regressions, enforce `QUERY_TAG` for cost attribution. Airflow/Dagster manage warehouse scaling and query retries.
- **Upstream Dependencies**: Warehouse availability, metadata freshness, clustering/search optimization state, session parameter consistency, privilege grants.
- **Environment Behavior**: Dev/test use default caches, smaller data volumes, and relaxed pruning validation. Prod enforces strict clustering, search optimization on hot predicates, cache validation, and query tag enforcement.
- **Runtime Assumptions`: Result cache is deterministic only for identical SQL + session state + timezone + warehouse config. Metadata cache persists ~24h per warehouse or until DDL change. Vectorized execution processes rows in batches, not row-by-row. Spill occurs when hash/window memory exceeds warehouse limit.

# 13. Observability
| Metric | Implementation | Detection Method | Operational Threshold |
|--------|----------------|------------------|------------------------|
| Pruning efficiency | `PARTITIONS_SCANNED / NULLIF(PARTITIONS_TOTAL, 0)` | `QUERY_HISTORY` or Profile UI | >0.3 scanned = clustering/search optimization ineffective or non-sargable filter |
| Cache utilization | `RESULT_SOURCE IN ('LOCAL_DISK', 'REMOTE_DISK', 'CLIENT')` | `QUERY_HISTORY.RESULT_SOURCE` | <30% cache hits on repeated queries = dynamic filters, parameter variance, or `USE_CACHED_RESULT=FALSE` |
| Spill volume | `BYTES_SPILLED_TO_REMOTE_STORAGE` | Profile UI / `QUERY_HISTORY` | >5GB per query = undersized warehouse, skewed joins, or unbounded window frames |
| Join strategy accuracy | `QUERY_PROFILE` operator type (Hash vs Merge vs Nested Loop) | Visual plan inspection | Hash join on sorted small tables = optimizer misestimation; forces merge join hint or stats refresh |
| Execution latency drift | `TOTAL_ELAPSED_TIME` vs baseline for identical query | Trend analysis, alerting | >50% increase without data volume growth = cache bypass, plan regression, or warehouse contention |

# 14. Failure Handling & Recovery
| Failure Scenario | What Breaks | Detection | Fallback Behavior | Recovery Approach |
|------------------|-------------|-----------|-------------------|-------------------|
| Full scan due to non-sargable filter | Pruning bypassed, warehouse saturates | `PARTITIONS_SCANNED = PARTITIONS_TOTAL` | Query succeeds but incurs high cost/latency | Rewrite filter to native type, remove wrapping functions, add search optimization or clustering |
| Join explosion / memory spill | Hash build exceeds warehouse memory, spills to remote disk | `BYTES_SPILLED` spikes, execution time multiplies | Query may timeout or succeed with degraded performance | Filter before join, pre-aggregate dimensions, validate key cardinality, scale warehouse or use query acceleration |
| Result cache bypass | Identical query re-executes, credits wasted | `RESULT_SOURCE = 'WAREHOUSE'` despite repetition | No fallback; full compute charged | Remove volatile functions (`CURRENT_TIMESTAMP`), stabilize session params, enable `USE_CACHED_RESULT=TRUE` |
| Window frame memory overflow | Unbounded partition sort exhausts memory | `SPILLED_BYTES` on window operator, query timeout | Query fails or completes slowly | Limit partition size with pre-filter, use `ROWS BETWEEN`, materialize intermediate results, push filters earlier |
| Plan regression after stats change | CBO selects suboptimal join order | `EXPLAIN` shows different strategy vs baseline | Query runs slower or spills | Run `ANALYZE TABLE`, use `MATERIALIZED` CTE hint, validate join predicates, pin query tag for comparison |
| Metadata cache invalidation | Frequent DDL drops cache, forces metadata reload | Metadata cache miss, slight query startup delay | Minimal performance impact; execution proceeds normally | Schedule DDL during off-peak, consolidate schema changes, avoid frequent `ALTER TABLE` in prod pipelines |

# 15. Security & Access Control
| Control | Implementation | Effect |
|---------|----------------|--------|
| Row-Level Security (RLS) | `ROW ACCESS POLICY` attached to tables/views | Filters rows during scan; pruning applies after policy evaluation, may reduce efficiency |
| Dynamic Data Masking (DDM) | `MASKING POLICY` on sensitive columns | Redacts output at projection stage; does not affect pruning or storage scan |
| Query Text Logging | `ACCESS_HISTORY` + `QUERY_HISTORY` | Stores executed SQL; sensitive payloads visible unless `SECURE` objects used |
| Role-Based Execution | `WAREHOUSE` usage grants, `SELECT` on objects | Prevents unauthorized queries; execution inherits caller role privileges |
| Secure View Inlining | `SECURE VIEW` prevents predicate pushdown leakage | Optimizer cannot rewrite or cache securely; forces full evaluation, may impact pruning |

# 16. Performance / Scalability Considerations
| Bottleneck | Cause | Tradeoff | Mitigation |
|------------|-------|----------|------------|
| Non-sargable predicates | Functions on filtered columns (`WHERE DATE(col) = ...`) | Disables micro-partition pruning, forces full scan | Filter on native type, add computed column with clustering, use search optimization |
| Late filtering | `WHERE` applied after `JOIN` or `GROUP BY` in CTE | Unnecessary compute on discarded rows, cache miss | Push filters to source, use `QUALIFY` post-window, flatten CTEs before aggregation |
| Repeated joins on large CTEs | Inline CTE referenced multiple times without materialization | Re-scans same data, inflates runtime | Add `/*+ MATERIALIZE */` hint, create temp table, or refactor to join-once pattern |
| Skewed join keys | Single value dominates join column | Hash partition imbalance, remote spill, long tail execution | Pre-filter skew, use broadcast hint for small tables, salt keys, or use `MERGE` with sorted inputs |
| Unbounded window frames | `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` on large partitions | Memory pressure, sort overflow, timeout | Limit frame, use `GROUP BY` pre-aggregation, partition by higher cardinality key |
| Cross-region egress | Query joins data across cloud regions | Network latency, egress charges, cache invalidation | Co-locate warehouses with data, replicate to target region, avoid cross-region joins |

# 17. Assumptions & Constraints
- **No concrete SQL provided**: Documentation reflects canonical query execution mechanics for SnowPro Advanced. Exact behavior depends on warehouse configuration, clustering state, session parameters, and data distribution.
- **Result cache requires exact match**: SQL text, whitespace, comments, session parameters, timezone, warehouse config, and role must match. Any deviation forces re-execution.
- **Metadata cache is warehouse-scoped**: Persists ~24h or until DDL change. Invalidated automatically; no manual flush required.
- **Pruning depends on metadata alignment**: Min/max values, clustering keys, and search optimization drive elimination. String comparisons on numeric columns, timezone shifts, or function-wrapped columns bypass pruning.
- **Optimizer is cost-based, not rule-based**: Uses column statistics, partition metadata, and row estimates. Stale stats or implicit casts produce suboptimal plans.
- **Window functions evaluate after `WHERE`/`GROUP BY`**: `QUALIFY` filters results after window calculation. Cannot push window logic into storage scan.
- **Exam trap assumptions**: SnowPro Advanced tests result cache invalidation triggers, pruning bypass conditions, join algorithm defaults, `QUALIFY` execution order, non-sargable filter impact, metadata cache scope, and `EXPLAIN` vs actual execution divergence. Memorize cache boundaries and pruning requirements.

# 18. Future Enhancements
- **Enforce search optimization on high-frequency predicates**: Replace manual clustering with Search Optimization Service for equality/range filters on wide tables. Reduces pruning latency without maintenance overhead.
- **Automate plan regression detection**: Compare `EXPLAIN` output across deployments, flag join strategy shifts, partition scan increases, or cache bypass rates. Block CI/CD on performance regression.
- **Standardize query tagging & cost attribution**: Require `QUERY_TAG` in all pipelines. Map tags to business units, enforce budget alerts, disable untagged execution in prod.
- **Materialize heavy CTEs strategically**: Replace repeated inline CTE references with `/*+ MATERIALIZE */` hints or staging tables. Reduces redundant scans, stabilizes cache utilization.
- **Harden non-sargable filter prevention**: Lint queries during development, reject `WHERE FUNCTION(col) = value` patterns, enforce computed column clustering or search optimization.
- **Integrate query acceleration service**: Enable for eligible read-heavy workloads with unpredictable cache hit rates. Reduces latency on cache misses, scales compute automatically without warehouse resize.
