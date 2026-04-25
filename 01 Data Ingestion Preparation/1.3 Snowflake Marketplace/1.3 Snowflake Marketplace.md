# Snowflake Marketplace

# 1. Title
SnowPro Advanced: Snowflake Marketplace Data Preparation & Secure Ingestion Architecture

# 2. Overview
- **What it does**: Defines the ingestion, validation, materialization, and governance workflow for third-party data acquired via Snowflake Marketplace. Focuses on secure data sharing consumption, reader account provisioning, and preparation for downstream analytics.
- **Why it exists**: Marketplace data operates under strict provider-controlled boundaries. Direct consumption without preparation leads to unpredictable query costs, schema drift failures, governance gaps, and exam-scenario misconfigurations. Structured ingestion ensures safe materialization, cost predictability, and compliance with provider terms.
- **Where it fits**: Sits between external data acquisition and internal transformation layers. Acts as a controlled gateway that validates, isolates, and prepares shared datasets before they enter dbt models, BI tools, or ML pipelines.
- **Intended consumer**: Data engineers, platform architects, FinOps analysts, compliance teams, and SnowPro Advanced candidates evaluating secure sharing, cross-region consumption, and marketplace billing mechanics.

# 3. SQL Object Summary
| Field | Value |
|-------|-------|
| Object Scope | Marketplace Data Preparation & Secure Consumption Pipeline |
| Type | Secure Share Integration + Consumer-Side Materialization |
| Purpose | Ingest, validate, and prepare third-party data without violating provider constraints or incurring unbounded compute costs |
| Source Objects | Provider Secure Share, Provider Secure Views, Cross-Region Replication Targets, Reader Account Shares |
| Output Object | Consumer Shared Database, Validated Staging Tables, Materialized Curated Tables, Cost/Usage Tracking Views |
| Execution Mode | On-Demand (`CREATE DATABASE FROM SHARE`), Scheduled (Materialization/Refresh), Event-Driven (Schema Drift Alerts) |

# 4. Architecture
Marketplace ingestion relies on secure data sharing, not data movement. The provider retains physical storage and compute control for shared views unless explicitly replicated or materialized. The consumer architecture isolates shared data, applies validation layers, and controls downstream exposure.

```mermaid
graph TD
  A[Provider Marketplace Listing] -->|Secure Share Grant| B[Consumer Account]
  B -->|CREATE DATABASE FROM SHARE| C[Shared Database (Read-Only)]
  C -->|Secure Views / Tables| D[Validation & Schema Contract Layer]
  D -->|Materialization / MERGE| E[Internal Staging / Curated Tables]
  E -->|Downstream Transformation| F[Analytics / BI / ML]
  B -.->|Reader Account Provisioning| G[Isolated Consumer Environment]
  D -.->|Cost & Usage Tracking| H[ACCOUNT_USAGE Integration]
  G -.->|Secure Data Clean Room| I[Privacy-Governed Join]
```

# 5. Data Flow / Process Flow
| Step | Input | Transformation | Output | Purpose |
|------|-------|----------------|--------|---------|
| 1. Listing Acceptance | Marketplace UI / API grant | Role assignment, share acceptance | Provider share accessible in consumer account | Establish legal and technical access boundary |
| 2. Database Creation | Accepted share | `CREATE DATABASE ... FROM SHARE` | Read-only shared database in consumer account | Isolate provider data, prevent accidental writes |
| 3. Schema Validation | Shared objects metadata | Column type checks, nullability audit, constraint verification | Validation report / staging schema | Detect provider schema drift before downstream integration |
| 4. Materialization Prep | Shared tables/views | `COPY INTO` (if replicated), `CREATE TABLE AS SELECT`, or `MERGE` | Consumer-owned staging tables | Decouple from provider compute limits, enable indexing/clustering |
| 5. Downstream Consumption | Staging/curated tables | dbt models, BI queries, ML feature extraction | Business datasets | Serve consumers with predictable performance and cost |

# 6. Logical Breakdown of the SQL
| Component | Responsibility | Inputs | Outputs | Dependencies | Failure Modes / Risks |
|-----------|----------------|--------|---------|--------------|-----------------------|
| `CREATE SHARE` / `GRANT ... TO SHARE` | Provider-side access control | Database, schemas, tables, secure views | Active share object | Provider role privileges, listing configuration | Over-granting exposes unintended data, revocation breaks consumer pipelines |
| `CREATE DATABASE FROM SHARE` | Consumer-side share ingestion | Share name, database name | Read-only database mirroring provider objects | Active share, consumer role privileges | Fails if share revoked or network/region sync incomplete |
| Secure View Projection | Controlled data exposure | Provider base tables | Filtered/masked columns, row-level policies | `ROW ACCESS POLICY`, `DYNAMIC DATA MASKING` | Predicate pushdown limits increase consumer query cost |
| Consumer Materialization (`CTAS`/`MERGE`) | Data decoupling & optimization | Shared tables/views, consumer staging | Consumer-owned tables with clustering/partitioning | Warehouse capacity, transformation logic | Stale data if refresh cadence misaligned, storage cost duplication |
| Reader Account Provisioning | Access for non-Snowflake users | Listing grant, account creation | Isolated account with predefined warehouse | Billing agreement, user provisioning flow | Fixed warehouse sizes limit performance, cross-region latency |
| Clean Room Secure Functions | Privacy-governed joins | Consumer data, provider shared data | Aggregated/privacy-safe results | `SECURE UDF`, differential privacy budget, policy grants | Budget exhaustion blocks queries, complex SQL triggers privacy violations |

# 7. Data Model
| Entity | Role | Important Fields | Grain | Relationships | Keys | Null Handling |
|--------|------|------------------|-------|---------------|------|---------------|
| `PROVIDER_SHARED_DB` | Read-only mirror of provider listing | `SCHEMA_NAME`, `OBJECT_TYPE`, `OWNER`, `CREATED_ON` | 1 row = 1 shared object (table/view/UDF) | Source for consumer staging | Internal object ID, fully qualified name | `NULL` if object dropped by provider |
| `CONSUMER_STAGING` | Validated, consumer-owned copy | `SOURCE_OBJECT`, `INGESTION_TS`, `VALIDATION_STATUS`, `SCHEMA_VERSION` | 1 row = 1 validated record batch | Feeds curated layer | `BATCH_ID`, `SOURCE_OBJECT` + `INGESTION_TS` | Explicit null checks on mandatory provider columns |
| `MATERIALIZED_CURATED` | Business-aligned, optimized dataset | Clustering keys, effective dates, business metrics | Defined by consumption pattern (e.g., daily snapshot) | Consumed by BI/ML | Surrogate key, business key | Coalesced, provider nulls mapped to defaults |
| `READER_ACCOUNT_INSTANCE` | Isolated consumer environment | `ACCOUNT_NAME`, `WAREHOUSE_SIZE`, `BILLING_MODEL`, `USER_COUNT` | 1 row = 1 provisioned account | Binds to provider share | Account locator, internal ID | `NULL` on deprovisioning |
| `CLEAN_ROOM_SESSION` | Privacy-governed execution context | `QUERY_ID`, `PRIVACY_BUDGET_USED`, `RESULT_SIZE_LIMIT`, `POLICY_BINDINGS` | 1 row = 1 clean room query execution | Joins consumer + provider data | `SESSION_ID`, `QUERY_ID` | Fails if budget exceeded, returns error instead of partial results |

**Output Grain**: Determined at `CONSUMER_STAGING` and `MATERIALIZED_CURATED`. Provider shared databases maintain provider-defined grain. Consumer materialization must explicitly align to business grain to prevent aggregation skew.

# 8. Business Logic
| Rule | Effect | Implementation Pattern | Edge Case |
|------|--------|------------------------|-----------|
| **Access Boundary Enforcement** | Prevents data export or provider bypass | `SECURE VIEW`, `ROW ACCESS POLICY`, disabled `COPY INTO` on shared objects | Consumers attempt `INSERT OVERWRITE` or `EXPORT`; blocked by share permissions |
| **Schema Drift Handling** | Detects provider column/type changes | `INFORMATION_SCHEMA.COLUMNS` comparison + automated validation task | Silent type coercion breaks downstream models if not caught |
| **Cost Allocation** | Maps marketplace usage to internal budgets | `ACCOUNT_USAGE.MARKETPLACE_TRANSACTIONS` + `WAREHOUSE_METERING_HISTORY` join | Free listings still consume compute; untracked queries inflate cost |
| **Privacy Budget Control** | Limits clean room query sensitivity | `SECURE FUNCTION` with differential privacy parameters, budget tracker | Exceeding budget returns error; retry requires provider approval or window reset |
| **Cross-Region Sync** | Ensures shared data availability globally | `ALTER SHARE ... ADD ACCOUNT` + data replication policy | Network latency increases query time; staging materialization recommended |
| **Reader Account Isolation** | Restricts non-Snowflake users to approved objects | Pre-configured database grants, warehouse size caps, disabled sharing | Users attempt to create objects or share data; blocked by account policy |

# 9. Transformations
| Source | Derived | Formula / Rule | Business Meaning | Impact |
|--------|---------|----------------|------------------|--------|
| Provider shared view | Consumer staging table | `CREATE TABLE AS SELECT * FROM provider_db.schema.view` | Decouples query execution from provider compute | Enables clustering, caching, predictable warehouse sizing |
| Raw shared columns | Validated schema | `TRY_CAST(col AS target_type)`, `NOT NULL` checks | Enforces data contract before downstream use | Fails fast on drift, prevents silent pipeline corruption |
| Shared date/timestamp | Standardized timezone | `TO_TIMESTAMP_TZ(col, format) AT TIME ZONE 'UTC'` | Aligns cross-provider time references | Eliminates aggregation skew in global reports |
| Provider metrics | Business KPIs | Aggregation, filtering, window ranking | Translates raw shared data to consumption-ready format | Requires explicit grain definition to avoid double-counting |
| Clean room query result | Privacy-safe output | Differential noise injection, aggregation threshold enforcement | Complies with provider data usage terms | Limits row-level exports, ensures compliance |

# 10. Parameters / Variables / Macros
| Name | Type | Purpose | Allowed Format | Default | Usage | Effect on Output |
|------|------|---------|----------------|---------|-------|------------------|
| `SHARE_NAME` | String | Identify provider share for ingestion | `PROVIDER_ACCOUNT.SHARE_NAME` | N/A | `CREATE DATABASE FROM SHARE` | Determines which listing is consumed |
| `CONSUMER_DB_NAME` | String | Isolate provider data in consumer account | Valid database name | `MARKETPLACE_DATA` | Database creation, role grants | Prevents naming collisions, enforces RBAC |
| `MATERIALIZATION_CADENCE` | String | Schedule consumer data refresh | `HOURLY`, `DAILY`, `WEEKLY`, `ON_DEMAND` | `DAILY` | `TASK` schedule, dbt run config | Balances data freshness vs storage/compute cost |
| `PRIVACY_BUDGET_LIMIT` | Float | Clean room query sensitivity cap | `0.0 – 10.0` (epsilon/delta) | `1.0` | `SECURE FUNCTION` parameter | Exceeding limit blocks query execution |
| `CROSS_REGION_SYNC` | Boolean | Enable automatic share replication | `TRUE` / `FALSE` | `FALSE` | `ALTER SHARE`, replication policy | Impacts query latency, network egress cost |
| `VALIDATION_MODE` | Enum | Schema drift detection strategy | `STRICT`, `WARN`, `IGNORE` | `STRICT` | Pre-ingestion validation task | `STRICT` halts pipeline on mismatch, `WARN` logs only |

# 11. APIs / Interfaces
| Interface | Invocation Method | Input Structure | Output Structure | Error Behavior | Consumers |
|-----------|-------------------|-----------------|------------------|----------------|-----------|
| `CREATE DATABASE FROM SHARE` | SQL | Share identifier, target DB name | Read-only database schema | Fails if share invalid, revoked, or permissions missing | Data engineers, pipeline orchestrators |
| `INFORMATION_SCHEMA.SHARE_*` | SQL | Database/schema filters | Share grants, object mappings | Returns empty if no access granted | Audit teams, discovery pipelines |
| `ACCOUNT_USAGE.MARKETPLACE_TRANSACTIONS` | SQL | Date range, listing filters | Billing records, usage metrics | 45-min latency, requires `MONITOR` role | FinOps, cost allocation systems |
| Snowsight Marketplace UI | Web | Search, filters, provider terms | Listing details, pricing, access request | UI-only; backend handles share grants | Business users, data buyers |
| Reader Account Endpoint | Web/SQL | User provisioning, role assignment | Isolated account credentials, DB access | Fails if billing agreement incomplete | External partners, non-Snowflake consumers |

# 12. Execution / Deployment
- **Manual vs Scheduled**: Share acceptance is manual or API-driven. Consumer materialization runs on `TASK` schedules or dbt orchestration.
- **Batch vs Incremental**: Initial `CREATE DATABASE` is full sync. Subsequent refreshes use incremental `MERGE` or `CTAS` with time-window filters. Cross-region shares sync automatically but incur network latency.
- **Orchestration**: Native `TASK` graphs, Airflow, dbt Cloud. Dependencies: Share availability, warehouse resume, role grants, schema validation pass.
- **Upstream Dependencies**: Provider listing status, share revocation/updates, network connectivity for cross-region, billing agreement validity.
- **Environment Behavior**: Dev/test often use free listings or sample shares. Prod requires paid agreements, strict RBAC, and materialization controls. Reader accounts isolated from main consumer account.
- **Runtime Assumptions**: Provider retains compute control for direct shared queries. Materialization shifts cost to consumer. `ACCOUNT_USAGE` latency up to 45 min. Clean room budgets reset per billing cycle unless configured otherwise.

# 13. Observability
| Metric | Implementation | Detection Method | Operational Threshold |
|--------|----------------|------------------|------------------------|
| Share access latency | `MAX(START_TIME)` in `ACCOUNT_USAGE.GRANTS` vs acceptance timestamp | Query `GRANT` history | >24h gap = provisioning bottleneck or role misconfiguration |
| Materialization success rate | `TASK_HISTORY` + `QUERY_HISTORY` status | Count `SUCCESS` vs `FAILED` per cadence | <95% success = schema drift, warehouse contention, or invalid filters |
| Marketplace cost tracking | `SUM(TRANSACTION_COST)` + `WAREHOUSE_CREDITS` for materialization | `ACCOUNT_USAGE` join | Exceeds budget = optimize cadence, reduce scope, or renegotiate terms |
| Clean room budget consumption | `PRIVACY_BUDGET_USED` vs `PRIVACY_BUDGET_LIMIT` | Secure function logs, `QUERY_HISTORY` | >80% consumed = restrict query frequency, increase limit, or adjust privacy parameters |
| Schema drift detection | `COLUMNS` hash comparison between `INFORMATION_SCHEMA` snapshots | Automated validation task | Any mismatch in `VALIDATION_MODE = STRICT` = halt pipeline, alert provider |

# 14. Failure Handling & Recovery
| Failure Scenario | What Breaks | Detection | Fallback Behavior | Recovery Approach |
|------------------|-------------|-----------|-------------------|-------------------|
| Provider revokes share | Consumer database becomes inaccessible | `QUERY_HISTORY` returns `object does not exist` | Materialized staging remains available until next refresh | Request provider reinstatement, document retention policy for staging data |
| Cross-region sync delay | Stale shared data, query results outdated | `LAST_ALTERED` timestamp lag vs expected cadence | Queries return old data until sync completes | Switch to materialized staging for predictable freshness, monitor replication logs |
| Schema drift on provider | Materialization fails, downstream models break | Validation task flags type/column mismatch | Pipeline halts (`STRICT` mode) or logs warning (`WARN` mode) | Update consumer transformation logic, coordinate with provider on versioning |
| Clean room budget exhaustion | Queries fail, privacy violations blocked | Secure function returns budget error | No results returned, query logged for audit | Request provider reset, optimize query aggregation, adjust differential privacy parameters |
| Reader account warehouse limit | Slow queries, timeout errors | `QUERY_HISTORY` spill/timeout metrics | Degraded performance for end users | Upgrade reader account tier, materialize data locally, implement caching |
| Billing agreement lapse | Share access suspended, queries fail | `ACCOUNT_USAGE` transaction errors, provider notification | Consumer loses access to listing | Renew agreement, verify role permissions, restore materialized copies if permitted |

# 15. Security & Access Control
| Control | Implementation | Effect |
|---------|----------------|--------|
| Provider-enforced boundaries | `SECURE VIEW`, `ROW ACCESS POLICY`, disabled `COPY` on shared objects | Prevents data exfiltration, enforces usage terms |
| Consumer RBAC | Role-based grants on `CONSUMER_DB`, `CONSUMER_STAGING` | Restricts access to validated data, isolates raw shared objects |
| Dynamic data masking | `MASKING POLICY` on sensitive columns in consumer staging | Hides PII from unauthorized roles, complies with governance |
| Reader account isolation | Fixed warehouse, disabled sharing, predefined grants | Ensures non-Snowflake users access only approved data |
| Cross-region encryption | TLS for data in transit, encryption at rest per region | Maintains compliance across geographic boundaries |
| Clean room privacy guards | Differential privacy, aggregation thresholds, budget tracking | Prevents re-identification, enforces statistical disclosure control |

# 16. Performance / Scalability Considerations
| Bottleneck | Cause | Tradeoff | Mitigation |
|------------|-------|----------|------------|
| Direct shared query cost | Provider compute allocation, unoptimized secure views | Unpredictable billing, latency spikes | Materialize data locally, push filters early, cluster staging tables |
| Cross-region network latency | Secure share replication distance | Slower query response, higher egress cost | Use regional listings, materialize in target region, cache results |
| Secure view predicate pushdown limits | Complex filters, joins, UDFs in provider views | Full scan by provider, increased cost | Simplify consumer queries, request provider view optimization |
| Materialization storage duplication | Multiple staging/curated layers | Increased storage cost, sync complexity | Implement tiered retention, archive cold data, use zero-copy cloning where possible |
| Reader account warehouse caps | Fixed compute allocation for non-Snowflake users | Performance bottlenecks under load | Pre-aggregate data, use materialized views, upgrade account tier if justified |
| Clean room query overhead | Differential privacy computation, budget tracking | Slower results, complex query planning | Pre-aggregate consumer data, use secure functions with optimized parameters, batch queries |

# 17. Assumptions & Constraints
- **No concrete SQL provided**: Documentation reflects canonical Marketplace ingestion patterns for SnowPro Advanced. Exact behavior depends on provider listing configuration, consumer role grants, and billing agreements.
- **Provider retains control**: Secure shares do not transfer ownership. Consumers cannot alter, drop, or export provider objects without explicit permission or materialization.
- **Cross-region sharing requires replication**: Standard shares operate within a cloud region. Cross-region/cross-cloud listings use data replication or cross-cloud secure shares, incurring network latency and cost.
- **Reader accounts are isolated**: They do not inherit consumer account roles or configurations. Provisioning requires separate billing and warehouse management.
- **Clean room privacy budgets are enforced**: Exceeding epsilon/delta limits blocks query execution. Resets follow provider-defined billing cycles or policy configurations.
- `ACCOUNT_USAGE` latency applies to marketplace transaction tracking. Real-time billing requires provider APIs or external cost monitoring tools.
- **Exam trap assumptions**: SnowPro Advanced tests secure share immutability, consumer materialization cost shift, cross-region sync mechanics, reader account limitations, clean room privacy budgets, and `ACCOUNT_USAGE` marketplace tracking. Memorize defaults and constraint boundaries.

# 18. Future Enhancements
- **Automate share provisioning**: Implement IaC (Terraform/dbt) for `CREATE SHARE`, role grants, and database creation. Reduces manual errors, ensures environment parity.
- **Implement consumer-side caching**: Deploy materialized views with incremental refresh for high-frequency shared queries. Cuts provider compute cost, improves latency.
- **Standardize schema drift contracts**: Require provider versioned releases, enforce `VALIDATION_MODE = STRICT` in CI/CD pipelines, automate rollback on mismatch.
- **Optimize cross-region replication**: Use Snowflake Data Cloud replication policies for predictable sync cadence, monitor egress cost, align materialization windows.
- **Integrate clean room with custom privacy libraries**: Extend secure UDFs with advanced differential privacy models, expose budget utilization via `ACCOUNT_USAGE` views.
- **Harden reader account performance**: Pre-aggregate provider data, deploy clustered staging tables, implement query routing to optimal warehouse sizes based on workload type.
