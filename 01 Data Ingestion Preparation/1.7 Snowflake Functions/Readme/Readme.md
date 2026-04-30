# Snowflake Functions

# 1. Title
SnowPro Advanced: Snowflake Functions & Custom Execution Architecture

# 2. Overview
- **What it does**: Extends native SQL execution with procedural logic, custom language runtimes, external service integration, and secure computation boundaries. Enables complex parsing, validation, normalization, and enrichment during data ingestion.
- **Why it exists**: Native SQL functions cannot handle arbitrary business logic, regex-heavy parsing, cryptographic operations, or third-party API enrichment. Custom functions bridge this gap while maintaining transactional consistency, execution isolation, and auditability.
- **Where it fits**: Embedded within ingestion SQL (`COPY` transformations, staging CTEs, validation queries, and enrichment layers). Operates as inline compute within the query execution engine or as proxied external calls.
- **Intended consumer**: Data engineers, analytics engineers, platform architects, security teams, and SnowPro Advanced candidates evaluating execution contexts, privilege models, deterministic behavior, and external function architecture.

# 3. SQL Object Summary
| Field | Value |
|-------|-------|
| [Object Scope](SQL Object Summary/Object Scope.md) | Custom Function Execution & Integration Framework |
| [Type](SQL Object Summary/Type.md) | Scalar UDFs, Table Functions, External Functions, Secure UDFs, Stored Procedures |
| [Purpose](SQL Object Summary/Purpose.md) | Encapsulate reusable logic, enforce validation contracts, perform external enrichment, isolate sensitive computation |
| [Source Objects](SQL Object Summary/Source Objects.md) | SQL expressions, language handlers (JS/Python/Java/Scala), external API gateways, cloud credentials |
| [Output Object](SQL Object Summary/Output Object.md) | Computed columns, enriched rows, validation flags, error payloads, procedure execution states |
| [Execution Mode](SQL Object Summary/Execution Mode.md) | Inline (query engine), Vectorized (batch), Proxied (external API), Serverless/External (API gateway) |

# 4. Architecture
Functions operate as isolated execution contexts within Snowflake's query engine or as proxied external services. The architecture enforces strict boundaries between SQL execution, language runtimes, and network egress. Secure functions and external API proxies add authentication, logging suppression, and data masking layers.

```mermaid
graph TD
  A[Query Execution Engine] -->|Parameter Binding| B[Function Invocation]
  B -->|Context Selection| C{Execution Type}
  C -->|In-Warehouse| D[Language Runtime (JS/Python/Java/Scala)]
  C -->|External Proxy| E[API Gateway + IAM Role]
  D -->|Compute & Return| F[Result Row/Scalar]
  E -->|Network Egress + Auth| G[External Service]
  G -->|Response Parsing| F
  F -->|Integration| H[Query Pipeline / Staging Table]
  D -.->|Sandbox Isolation| I[No Side Effects / Stateless]
  E -.->|Secure UDF Flag| J[Suppressed Logging / Masked Payloads]
```

# 5. Data Flow / Process Flow
| Step | Input | Transformation | Output | Purpose |
|------|-------|----------------|--------|---------|
| [1. Invocation & Binding](Data Flow  Process Flow/1. Invocation & Binding.md) | SQL expression, column values, literal parameters | Parameter type validation, null propagation | Bound argument payload | Establish execution context with strict typing |
| [2. Context Initialization](Data Flow  Process Flow/2. Context Initialization.md) | Function definition, language handler, privilege scope | Runtime allocation, sandbox setup, security boundary enforcement | Isolated execution environment | Prevent cross-query state leakage, enforce least-privilege |
| [3. Execution](Data Flow  Process Flow/3. Execution.md) | Bound arguments, handler code | Language runtime evaluation, API call (if external), error routing | Scalar value or table rows | Perform custom logic outside native SQL operator set |
| [4. Return & Integration](Data Flow  Process Flow/4. Return & Integration.md) | Function output, query plan position | Type casting, row emission, caching evaluation | Inline column, derived table, or procedure state | Feed result back to ingestion pipeline without breaking transaction scope |
| [5. Audit & Telemetry](Data Flow  Process Flow/5. Audit & Telemetry.md) | Execution metadata, duration, error state | `QUERY_HISTORY` attachment, log suppression (secure), cost attribution | Execution trace | Enable debugging, monitor latency, enforce governance |

# 6. Logical Breakdown of the SQL
| Component | Responsibility | Inputs | Outputs | Dependencies | Failure Modes / Risks |
|-----------|----------------|--------|---------|--------------|-----------------------|
| [Scalar UDF](Logical Breakdown of the SQL/Scalar UDF.md) | Row-level value transformation | Single/multiple columns, literals | Single scalar value | Language runtime, query context | Row-by-row execution overhead, non-vectorized bottleneck |
| [Table Function (UDTF)](Logical Breakdown of the SQL/Table Function (UDTF).md) | Row expansion or schema projection | Input table, parameters | Virtual table rows | Query engine, memory allocation | Memory spill on large expansion, join explosion risk |
| [External Function](Logical Breakdown of the SQL/External Function.md) | Third-party service integration | Query payload, API keys, endpoint | API response mapped to SQL types | API gateway, network egress, IAM proxy | Latency spikes, rate limiting, payload truncation, cold starts |
| [Secure UDF](Logical Breakdown of the SQL/Secure UDF.md) | Sensitive logic execution | Columns, cryptographic keys, PII | Masked/redacted output, secure scalar | Query engine, logging suppression config | No execution visibility, debugging requires audit logs |
| [Stored Procedure](Logical Breakdown of the SQL/Stored Procedure.md) | Procedural workflow & DML orchestration | Parameters, dynamic SQL, transaction scope | Result set, status codes, side effects | Warehouse, role privileges, transaction isolation | Caller vs owner rights mismatch, implicit commits, cursor leaks |
| [Language Handler](Logical Breakdown of the SQL/Language Handler.md) | Runtime binding & execution environment | JS/Python/Java/Scala code, dependencies | Executable function object | Runtime provisioning, dependency packaging | Version drift, sandbox crashes, dependency conflicts |

# 7. Data Model
| Entity | Role | Important Fields | Grain | Relationships | Keys | Null Handling |
|--------|------|------------------|-------|---------------|------|---------------|
| [`FUNCTION_CATALOG`](Data Model/FUNCTION_CATALOG.md) | Metadata registry | `FUNCTION_NAME`, `SCHEMA`, `LANGUAGE`, `IS_SECURE`, `PRIVILEGE_MODEL` | 1 row = 1 function definition | Maps to `INFORMATION_SCHEMA.FUNCTIONS`, `DEPENDENCIES` | `FUNCTION_ID`, fully qualified name | `NULL` if dropped or orphaned |
| [`EXECUTION_CONTEXT`](Data Model/EXECUTION_CONTEXT.md) | Runtime state snapshot | `QUERY_ID`, `HANDLER`, `MEMORY_LIMIT`, `TIMEOUT_MS` | 1 row = 1 execution instance | Links to `QUERY_HISTORY`, function definition | `QUERY_ID` + `EXECUTION_TIMESTAMP` | Context terminated on failure; no partial state persisted |
| [`PARAMETER_BINDING`](Data Model/PARAMETER_BINDING.md) | Input schema contract | `ARG_NAME`, `DATA_TYPE`, `DEFAULT_VALUE`, `IS_NULLABLE` | 1 row = 1 function argument | Validates against caller input, binds to handler | Composite: `FUNCTION_NAME` + `ARG_POSITION` | Strict type enforcement; nulls propagated unless guarded |
| [`RETURN_SCHEMA`](Data Model/RETURN_SCHEMA.md) | Output contract definition | `RETURN_TYPE`, `IS_TABLE`, `COLUMN_DEFINITIONS` | 1 row = 1 return signature | Consumed by query planner for type inference | `FUNCTION_NAME` | Mismatch causes runtime type error; no implicit casting post-execution |

**Output Grain**: Determined by function type. Scalar UDFs = 1:1 with input rows. UDTFs = 1:N (row expansion). External functions = 1:1 or 1:N depending on API response mapping. Grain mismatch causes downstream aggregation skew.

# 8. Business Logic
| Rule | Effect | Implementation Pattern | Edge Case |
|------|--------|------------------------|-----------|
| [**Statelessness Enforcement**](Business Logic/Statelessness Enforcement.md) | Functions cannot retain state across invocations | No external storage, no global variables, deterministic flag validation | Caching may mask non-deterministic behavior; test with `IS_DETERMINISTIC = FALSE` |
| [**Privilege Execution Model**](Business Logic/Privilege Execution Model.md) | Controls data access during function run | `EXECUTE AS OWNER` vs `EXECUTE AS CALLER` | `CALLER` inherits query role; `OWNER` uses function creator role. Misalignment breaks access or leaks data |
| [**Deterministic Flag**](Business Logic/Deterministic Flag.md) | Influences query optimization & caching | `IS_DETERMINISTIC = TRUE/FALSE` | `TRUE` enables result caching and predicate pushdown; `FALSE` forces recomputation |
| [**External API Proxying**](Business Logic/External API Proxying.md) | Secures network egress & credentials | API integration + IAM role + function binding | Credentials never exposed to query engine; all traffic routed through Snowflake proxy |
| [**Secure UDF Logging Suppression**](Business Logic/Secure UDF Logging Suppression.md) | Prevents PII/crypto exposure in logs | `SECURE` keyword applied at creation | Execution details omitted from `QUERY_HISTORY`; debugging requires controlled test environment |
| [**Error Isolation**](Business Logic/Error Isolation.md) | Fails query vs skips row | `RAISE` / exception handling in handler | Unhandled exceptions abort entire query; handled exceptions can return fallback values |

# 9. Transformations
| Source | Derived | Formula / Rule | Business Meaning | Impact |
|--------|---------|----------------|------------------|--------|
| [Raw JSON string](Transformations/Raw JSON string.md) | Structured fields | Python/JS `json.loads()` + key extraction | Schema-on-read parsing during ingestion | Enables relational downstream processing; invalid JSON routes to error handling |
| [Unformatted identifiers](Transformations/Unformatted identifiers.md) | Standardized keys | Regex validation, hash normalization, case folding | Enforces contract compliance before staging | Reduces join failures, prevents duplicate entity creation |
| [External API payload](Transformations/External API payload.md) | Enriched attributes | HTTP POST/GET + response parsing | Augments raw data with third-party context | Introduces network latency; requires batch optimization to avoid N+1 calls |
| [Sensitive columns](Transformations/Sensitive columns.md) | Masked outputs | Cryptographic functions, tokenization, hashing | Complies with governance & PII regulations | Irreversible transformation; breaks deterministic joins if not keyed consistently |
| [Multi-column validation](Transformations/Multi-column validation.md) | Compliance flags | Conditional logic, threshold checks, anomaly detection | Gates ingestion into curated layer | Centralizes validation, reduces repeated `CASE` logic in SQL |

# 10. Parameters / Variables / Macros
| Name | Type | Purpose | Allowed Format | Default | Usage | Effect on Output |
|------|------|---------|----------------|---------|-------|------------------|
| [`ARGUMENTS`](Parameters  Variables  Macros/ARGUMENTS.md) | Typed parameters | Input binding for function execution | SQL data types, arrays, objects, variants | None (required) | Function invocation | Type mismatch raises error; nulls propagate unless guarded |
| [`LANGUAGE`](Parameters  Variables  Macros/LANGUAGE.md) | Enum | Runtime selection | `JAVASCRIPT`, `PYTHON`, `JAVA`, `SCALA` | `JAVASCRIPT` (legacy) | `CREATE FUNCTION` | Determines execution environment, dependency packaging, sandbox limits |
| [`HANDLER`](Parameters  Variables  Macros/HANDLER.md) | String | Entry point identification | Module/class.method or function name | N/A | Function definition | Routing to correct code block; missing handler causes runtime error |
| [`PACKAGES` / `IMPORTS`](Parameters  Variables  Macros/PACKAGES  IMPORTS.md) | Array/String | Dependency injection | Snowpark package names, stage file paths | None | Function creation | Enables third-party libraries; version conflicts cause sandbox failure |
| [`API_INTEGRATION`](Parameters  Variables  Macros/API_INTEGRATION.md) | Object Name | External service routing | Pre-created integration object | N/A | `EXTERNAL FUNCTION` | Binds proxy endpoint, IAM role, and network policy |
| [`IS_DETERMINISTIC`](Parameters  Variables  Macros/IS_DETERMINISTIC.md) | Boolean | Caching & optimization flag | `TRUE` / `FALSE` | `FALSE` | Function creation | `TRUE` enables result cache reuse; `FALSE` forces recomputation per row/query |
| [`RETURNS NULL ON NULL INPUT`](Parameters  Variables  Macros/RETURNS NULL ON NULL INPUT.md) | Boolean | Null propagation control | `TRUE` / `FALSE` | `FALSE` | Function definition | `TRUE` skips execution on null args; returns null immediately |

# 11. APIs / Interfaces
| Interface | Invocation Method | Input Structure | Output Structure | Error Behavior | Consumers |
|-----------|-------------------|-----------------|------------------|----------------|-----------|
| [`CREATE/ALTER/DROP FUNCTION`](APIs  Interfaces/CREATEALTERDROP FUNCTION.md) | SQL | Definition, handler, language, flags | Function metadata object | Fails on syntax error, privilege mismatch, or invalid handler | Pipeline engineers, CI/CD deployments |
| [`SELECT function_name(col)`](APIs  Interfaces/SELECT function_name(col).md) | SQL | Bound arguments, inline expressions | Scalar value or table rows | Raises exception on type error, timeout, or sandbox crash | Ingestion queries, validation CTEs, BI layers |
| [`INFORMATION_SCHEMA.FUNCTIONS`](APIs  Interfaces/INFORMATION_SCHEMA.FUNCTIONS.md) | SQL | Schema/function filters | Metadata, language, security flags | Returns empty if no access; requires `USAGE` on schema | Auditing, dependency mapping, governance checks |
| [`EXTERNAL FUNCTION` + API Integration](APIs  Interfaces/EXTERNAL FUNCTION + API Integration.md) | SQL | Payload columns, endpoint routing | Mapped API response fields | Returns HTTP error code on network/auth failure | Enrichment pipelines, third-party data augmentation |
| [`CALL procedure_name(...)`](APIs  Interfaces/CALL procedure_name(...).md) | SQL | Parameters, transaction context | Result set, status codes, DML side effects | Aborts on privilege error, implicit commit mismatch, or handler crash | Orchestration workflows, multi-step ingestion gating |

# 12. Execution / Deployment
- **Manual vs Scheduled**: Functions execute inline with queries. Procedures run via `CALL` or scheduled tasks. External functions trigger on query execution.
- **Batch vs Row-by-Row**: Scalar UDFs default to row-by-row execution. Snowpark Python/Java supports vectorized execution via `pandas`/`Spark` DataFrames. External functions batch payloads automatically when possible.
- **Orchestration**: CI/CD pipelines package dependencies, deploy to Snowflake stages, register functions via DDL, and run integration tests. Versioning via function naming or schema isolation.
- **Upstream Dependencies**: Language runtime availability, dependency package versions, API integration status, IAM role permissions, warehouse compute allocation.
- **Environment Behavior**: Dev/test use sandboxed handlers with mock API responses. Prod enforces secure functions, strict privilege models, network policies, and package version pinning.
- **Runtime Assumptions**: Execution context isolated per query. No persistent state across invocations. Memory limits enforced per handler. External function latency scales with API response time and batch size. Snowpark vectorization requires explicit handler implementation.

# 13. Observability
| Metric | Implementation | Detection Method | Operational Threshold |
|--------|----------------|------------------|------------------------|
| [Function execution latency](Observability/Function execution latency.md) | `QUERY_HISTORY.FUNCTION_NAME` + `TOTAL_ELAPSED_TIME` | Query profile, alerting system | >50% baseline increase = handler degradation or external API slowdown |
| [Error rate](Observability/Error rate.md) | `COUNT(*) WHERE ERROR_MESSAGE LIKE '%function%' / TOTAL INVOCATIONS` | Query log parsing, `INFORMATION_SCHEMA` audit | >2% = type mismatch, dependency failure, or API timeout |
| [External function cold start](Observability/External function cold start.md) | First invocation duration after inactivity vs steady state | Timestamp comparison in `QUERY_HISTORY` | >3s cold start = API gateway provisioning delay |
| [Cache utilization](Observability/Cache utilization.md) | `IS_DETERMINISTIC=TRUE` hit rate | `QUERY_HISTORY.RESULT_SOURCE` vs function calls | <30% cache hits on deterministic functions = parameter variance too high |
| [Memory/spill events](Observability/Memoryspill events.md) | `QUERY_HISTORY.SPILLED_BYTES` + handler logs | Profile UI, warehouse metrics | Spill during UDF execution = row expansion exceeds memory limit |

# 14. Failure Handling & Recovery
| Failure Scenario | What Breaks | Detection | Fallback Behavior | Recovery Approach |
|------------------|-------------|-----------|-------------------|-------------------|
| [Unhandled exception in handler](Failure Handling & Recovery/Unhandled exception in handler.md) | Query aborts, transaction rolls back | `ERROR_MESSAGE` contains handler stack trace | No partial results; entire query fails | Add try/catch in handler, return fallback value, log to error table |
| [External API timeout/rate limit](Failure Handling & Recovery/External API timeoutrate limit.md) | Function stalls or returns HTTP error | `QUERY_HISTORY` shows long duration or error payload | Query fails or returns error code to caller | Implement retry logic in handler, add circuit breaker, batch payloads |
| [Privilege mismatch (`EXECUTE AS CALLER`)](Failure Handling & Recovery/Privilege mismatch (EXECUTE AS CALLER).md) | Access denied to referenced objects | `ORA-XXXXX` / `Insufficient privileges` error | Query halts at function invocation | Switch to `EXECUTE AS OWNER`, grant explicit role to caller, audit grants |
| [Non-deterministic caching trap](Failure Handling & Recovery/Non-deterministic caching trap.md) | Stale results returned despite data change | `IS_DETERMINISTIC=TRUE` but output incorrect | Query uses cached value; data inconsistency | Set `IS_DETERMINISTIC=FALSE`, invalidate cache, audit function logic |
| [Dependency version drift](Failure Handling & Recovery/Dependency version drift.md) | Handler crashes on import | `ModuleNotFoundError` / `ClassNotFoundException` | Query fails at execution start | Pin package versions in `CREATE FUNCTION`, test in staging before deploy |
| [Secure UDF debugging block](Failure Handling & Recovery/Secure UDF debugging block.md) | No execution visibility in logs | Query succeeds but output unexpected | Operators blind to internal state | Use test dataset, replicate in non-secure environment, add internal logging to error table |

# 15. Security & Access Control
| Control | Implementation | Effect |
|---------|----------------|--------|
| [Owner vs Caller Rights](Security & Access Control/Owner vs Caller Rights.md) | `EXECUTE AS OWNER` vs `EXECUTE AS CALLER` | `OWNER` uses creator role (safer for sensitive logic); `CALLER` inherits query role (stricter access control) |
| [Secure UDF Execution](Security & Access Control/Secure UDF Execution.md) | `SECURE` keyword at creation | Suppresses query text, parameter values, and results from logs; prevents PII leakage |
| [External Function Proxy](Security & Access Control/External Function Proxy.md) | API Integration + IAM Role + Network Policy | Credentials never exposed to SQL layer; all egress routed through Snowflake-managed gateway |
| [Package Isolation](Security & Access Control/Package Isolation.md) | Snowpark dependency sandboxing | Prevents arbitrary system access, restricts filesystem/network calls in handler |
| [Row-Level Policy Integration](Security & Access Control/Row-Level Policy Integration.md) | Functions called within `ROW ACCESS POLICY` context | Enforces dynamic filtering without exposing underlying logic to consumers |
| [Audit Logging](Security & Access Control/Audit Logging.md) | `ACCESS_HISTORY` + `QUERY_HISTORY` join | Tracks function invocation, role, warehouse, and execution state (unless secure) |

# 16. Performance / Scalability Considerations
| Bottleneck | Cause | Tradeoff | Mitigation |
|------------|-------|----------|------------|
| [Row-by-row execution overhead](Performance  Scalability Considerations/Row-by-row execution overhead.md) | Scalar UDFs invoked per record | High CPU, slow throughput on large datasets | Vectorize via Snowpark Python/Java, push logic to SQL operators, cache deterministic results |
| [External function network latency](Performance  Scalability Considerations/External function network latency.md) | API round-trip per invocation | Query timeout, warehouse idle time | Batch payloads in handler, use async patterns where supported, implement client-side caching |
| [Non-sargable function predicates](Performance  Scalability Considerations/Non-sargable function predicates.md) | `WHERE function(col) = 'value'` | Disables pruning, forces full table scan | Push function to post-filter stage, create computed column with clustering, rewrite as native SQL |
| [Memory spill during UDTF expansion](Performance  Scalability Considerations/Memory spill during UDTF expansion.md) | Table functions emitting excessive rows | Warehouse disk I/O, query timeout | Limit output rows, use `QUALIFY`, pre-filter input, switch to incremental staging |
| [Cold start latency](Performance  Scalability Considerations/Cold start latency.md) | Serverless/external handlers after inactivity | First invocation delayed, schedule drift | Keep warm with dummy queries, align schedule with expected load, accept tradeoff for burst workloads |
| [Deterministic flag misuse](Performance  Scalability Considerations/Deterministic flag misuse.md) | Marking non-deterministic logic as `TRUE` | Incorrect caching, stale results in ingestion pipeline | Audit functions for time/random/sequence dependencies, enforce `FALSE` where applicable |

# 17. Assumptions & Constraints
- **No concrete SQL provided**: Documentation reflects canonical function architecture for SnowPro Advanced. Exact behavior depends on language handler, privilege model, external API configuration, and execution flags.
- **UDFs are stateless**: Functions cannot retain variables, open connections, or write to external storage between invocations. Each call operates in an isolated sandbox.
- **Deterministic flag controls caching**: `IS_DETERMINISTIC=TRUE` enables result cache reuse. `FALSE` forces recomputation. Mislabeling causes data inconsistency or wasted compute.
- **External functions require API Integration**: Cannot call endpoints directly. Must route through Snowflake proxy with IAM role binding and network policy enforcement.
- **Owner vs Caller rights dictate access**: `EXECUTE AS OWNER` uses function creator's privileges. `EXECUTE AS CALLER` uses the querying role. Exam frequently tests privilege escalation risks and access boundary failures.
- **Secure UDFs suppress logging**: Execution details, parameters, and outputs are hidden from `QUERY_HISTORY`. Debugging requires controlled replication or internal error routing.
- **SnowPro Advanced exam traps**: Tests UDF statelessness guarantees, deterministic caching behavior, owner/caller privilege models, external function proxy architecture, secure UDF logging suppression, vectorization requirements, and row-by-row vs batch execution tradeoffs. Memorize defaults and engine constraints.

# 18. Future Enhancements
- **Standardize vectorized execution contracts**: Replace row-by-row scalar UDFs with Snowpark DataFrame handlers. Enables batch processing, reduces context switching overhead, improves warehouse utilization.
- **Implement external function batching middleware**: Build handler logic that aggregates row payloads, makes single API call, and maps responses back. Cuts network latency from O(N) to O(1) per batch.
- **Automate deterministic flag auditing**: Scan `INFORMATION_SCHEMA.FUNCTIONS` for `IS_DETERMINISTIC=TRUE` with non-deterministic patterns (`CURRENT_TIMESTAMP`, `RANDOM`, `SEQUENCE`). Flag for remediation.
- **Harden error routing contracts**: Replace unhandled exceptions with structured fallback returns. Log failures to dedicated error table with query context, parameter snapshot, and retry flag.
- **Integrate computed column clustering**: Materialize function outputs as persistent columns, apply clustering/search optimization. Eliminates repeated execution during pruning-heavy queries.
- **Package version pinning pipeline**: Enforce dependency version locks in CI/CD, test handler compatibility across Snowflake runtime upgrades, prevent production sandbox crashes.
