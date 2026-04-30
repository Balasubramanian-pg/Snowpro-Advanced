**Overview**
External tables are Snowflake’s query-in-place interface for cloud storage. They don’t move or store data. Instead, they map directly to files sitting in an external stage, letting you run standard SQL against raw Parquet, JSON, CSV, or Avro files as if they were regular tables. Snowflake reads the files on demand, caches metadata for faster lookups, and applies your warehouse’s compute to push down predicates. It’s a bridge between data lake storage and analytical querying without the overhead of full ingestion.

**Key Characteristics**
- Zero data ingestion: Queries run directly against files in cloud storage. No COPY INTO required.
- Read-only & schema-on-read: You define column mappings or let Snowflake infer them from file metadata. Data isn’t stored in Snowflake.
- Partition-aware: Supports partition pruning using file path metadata or embedded columns, drastically cutting scan costs.
- Metadata caching: Snowflake caches file listings and column stats. Query latency drops after the first run, but isn’t real-time.
- Manual/auto refresh: Requires `ALTER EXTERNAL TABLE ... REFRESH` to sync with new/changed files. Can be automated via tasks or event-driven pipelines.
- Compute-bound: Scans run on your virtual warehouse. Performance scales with warehouse size, but partitioning and file format (Parquet/ORC) matter more.
- Limited DML: You can’t INSERT, UPDATE, or DELETE. It’s strictly for analytics, exploration, and lightweight transformations.

**Examples**

**1. Basic creation (Parquet with auto-inference)**
```sql
CREATE EXTERNAL TABLE ext_sales
WITH LOCATION = @ext_s3_stage/sales/
FILE_FORMAT = (TYPE = PARQUET)
AUTO_REFRESH = TRUE;
```
*Breakdown*: Maps to Parquet files in the stage. `VALUE` column holds the raw semi-structured row. `AUTO_REFRESH` triggers metadata sync when cloud storage notifications hit Snowflake (requires setup). Queries use `VALUE:column_name::type` to extract data.

**2. Explicit schema with partition pruning**
```sql
CREATE EXTERNAL TABLE ext_logs (
  log_date DATE AS TO_DATE(SPLIT_PART(metadata$filename, '/', 4)),
  app_name VARCHAR AS VALUE:app_name::VARCHAR,
  event_type VARCHAR AS VALUE:event_type::VARCHAR,
  payload OBJECT AS VALUE:payload
)
WITH LOCATION = @ext_stage/logs/
FILE_FORMAT = (TYPE = JSON)
PARTITION BY (log_date, app_name)
AUTO_REFRESH = FALSE;
```
*Breakdown*: Defines typed columns by parsing JSON and extracting path-based metadata. `PARTITION BY` enables Snowflake to skip files that don’t match WHERE clauses on `log_date` or `app_name`. `AUTO_REFRESH = FALSE` means you manually run `REFRESH` or schedule it.

**3. Querying with predicate pushdown**
```sql
SELECT log_date, app_name, COUNT(*) as event_count
FROM ext_logs
WHERE log_date BETWEEN '2024-01-01' AND '2024-01-31'
  AND event_type = 'CLICK'
GROUP BY 1, 2;
```
*Breakdown*: Standard SQL. Because of partitioning and file format optimization, Snowflake prunes irrelevant files and pushes filters down to the storage layer. Only matching data hits the warehouse compute.

**4. Refresh + metadata sync**
```sql
ALTER EXTERNAL TABLE ext_logs REFRESH;
ALTER EXTERNAL TABLE ext_logs REFRESH PREFIX = 'logs/2024/02/';
```
*Breakdown*: Syncs Snowflake’s metadata cache with the actual files in the stage. Prefix-based refresh saves time when you know exactly what changed. Required when `AUTO_REFRESH` isn’t configured or fails.

**5. Join with native Snowflake table**
```sql
SELECT s.customer_id, s.amount, l.app_name
FROM snowflake_db.dbo.orders s
JOIN ext_logs l ON s.customer_id = l.payload:cust_id::NUMBER
WHERE l.log_date = CURRENT_DATE();
```
*Breakdown*: External tables behave like regular tables in JOINs, CTEs, and views. Snowflake executes the join in-memory using warehouse compute. Keep the external side filtered to avoid full-stage scans.

**Notes**
External tables trade storage cost for compute latency. You pay for warehouse scan time instead of paying for stored, optimized micro-partitions. If you query the same data repeatedly or need sub-second response times, materialize it with COPY INTO or CTAS. External tables shine for exploratory analysis, late-binding data lake queries, archival access, or when ingestion pipelines are too slow or expensive to maintain.

Partitioning isn’t optional for scale. Without it, Snowflake scans every file in the stage regardless of your WHERE clause. Structure your cloud paths logically, map them in PARTITION BY, and let Snowflake prune. File format matters more than warehouse size here. Parquet/ORC with columnar encoding and snappy/zstd compression outperform CSV/JSON by orders of magnitude because Snowflake can read only the needed columns.

Metadata cache is stale by design. It doesn’t update instantly when files land. If you need near-real-time visibility, use Snowpipe auto-ingest into a staging table instead. External tables are batch-oriented with a refresh window. Don’t treat them as live pipelines.

Credential and stage management still apply. External tables inherit the security model of their underlying stage. Use storage integrations, not inline credentials. And remember: you can’t update external data from Snowflake. If you need ACID compliance, upserts, or streaming, external tables aren’t your tool. They’re a query lens, not a storage engine. Know the difference before you build on them.
