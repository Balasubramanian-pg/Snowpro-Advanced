# Overview
`COPY INTO` is Snowflake’s bulk data mover. It reads files from a stage or cloud storage, parses them in parallel across your virtual warehouse, and writes them into table micro-partitions (or does the reverse when unloading). It’s not a row-by-row inserter, and it’s not built for real-time streaming. Think of it as a distributed batch loader: you hand it files, it spins up parallel workers, and it dumps structured data into your table as fast as the compute and I/O allow.

If you’re treating it like a simple file import script, you’ll run into friction. It’s designed around cloud storage semantics, stage management, and parallel execution. Keep that in mind and you’ll avoid the common pitfalls around duplicates, credential rotation, and warehouse sizing.

**Key Characteristics**
- **Stage-first architecture**: Data must sit in an internal stage, external stage, or cloud path before `COPY INTO` touches it. Snowflake doesn’t read directly from your local machine.
- **Parallel by default**: Each file is processed by a separate thread. More files = more parallelism (up to warehouse limits).
- **Format-aware**: Handles CSV, JSON, Parquet, Avro, ORC, XML. You define or auto-detect the schema via `FILE_FORMAT`.
- **Error tolerance**: `ON_ERROR` controls behavior (`CONTINUE`, `SKIP_FILE`, `ABORT_STATEMENT`). You can also capture bad records with `VALIDATION_MODE` or `COPY_HISTORY`.
- **No built-in idempotency**: Running it twice on the same files doubles your data. You manage deduplication or use `PURGE` + staging tables.
- **Compute-bound**: Requires an active warehouse. Unloading still spins up compute because Snowflake rewrites data into columnar files.

---

**Examples**

**1. Load from internal named stage (CSV)**
```sql
COPY INTO sales_data
FROM @my_s3_stage/sales/
FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1)
ON_ERROR = 'SKIP_FILE'
PATTERN = '.*sales_2024.*\\.csv';
```
*Breakdown*: Pulls CSVs matching the regex from `@my_s3_stage`, skips the header row, skips any malformed file instead of failing the whole batch. `PATTERN` is optional but saves you from manually curating file lists.

**2. Load from external S3 with explicit credentials**
```sql
COPY INTO customer_raw
FROM 's3://my-bucket/exports/customers/'
CREDENTIALS = (AWS_KEY_ID='AKIA...' AWS_SECRET_KEY='...')
FILE_FORMAT = (TYPE = JSON);
```
*Breakdown*: Direct cloud path bypasses Snowflake stages. Credentials go in the query (not recommended for prod—use storage integrations or IAM roles instead). Good for quick tests or restricted environments.

**3. Load with on-the-fly transformation**
```sql
COPY INTO transformed_orders
FROM (
  SELECT 
    $1:order_id::NUMBER,
    $1:order_date::TIMESTAMP_NTZ,
    $1:amount::FLOAT,
    CURRENT_TIMESTAMP() AS loaded_at
  FROM @stg_orders
)
FILE_FORMAT = (TYPE = JSON);
```
*Breakdown*: `COPY INTO` isn’t just a dumb pipe. You can select, cast, and add columns before writing. This avoids staging intermediate tables when transformations are light.

**4. Unload table to external Parquet files**
```sql
COPY INTO 's3://my-bucket/archive/sales/'
FROM sales_data
STORAGE_INTEGRATION = my_s3_int
FILE_FORMAT = (TYPE = PARQUET COMPRESSION = SNAPPY)
HEADER = TRUE
OVERWRITE = TRUE;
```
*Breakdown*: Pushes table data back to cloud storage. `OVERWRITE` drops existing files in that path. Parquet + Snappy is the standard for downstream analytics. `HEADER = TRUE` adds column names to the first row.

**5. Dry run + error capture**
```sql
COPY INTO staging_logs
FROM @log_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_10_ROWS';
```
*Breakdown*: Doesn’t load anything. Returns up to 10 rows that would be parsed, letting you validate format, delimiters, and data types before committing to a full run. Pair this with `COPY_HISTORY` to track past runs.

---

**Notes**
First-principles check: `COPY INTO` is a **batch parallelizer**, not a streaming pipeline. If you’re dropping files into S3 every few seconds, you’ll burn warehouse credits and hit concurrency limits. For that, use **Snowpipe** (auto-ingest via notifications) or **streaming ingestion** (Kafka/Snowpipe Streaming). `COPY INTO` shines when you have bounded batches: daily dumps, backfills, archival loads, or ad-hoc bulk imports.

Credential management matters. Hardcoding keys in queries works for sandboxing but breaks in prod. Use `STORAGE_INTEGRATION` (Snowflake-managed OAuth/IAM) or IAM roles with trust policies. It’s cleaner, auditable, and doesn’t leak into query logs.

Monitor it. `COPY_HISTORY()` and `INFORMATION_SCHEMA.LOAD_HISTORY` show you files processed, rows loaded, and errors. If you’re seeing high `SKIP_FILE` rates, your format definition or encoding is off. Fix the source or tighten the `FILE_FORMAT` before scaling up.

Duplicates are your problem, not Snowflake’s. `COPY INTO` doesn’t track what’s already loaded unless you manage it. Common patterns: load into a staging table, `MERGE` into the target, then `PURGE` the stage files. Or use `FORCE = TRUE` only when you explicitly want to reprocess.

If you’re prepping for interviews or certs, don’t just memorize syntax. Understand the execution model: stage → parallel file readers → warehouse compute → micro-partition writes. That mental model explains why warehouse size matters, why file count impacts speed, and why Snowpipe exists as a separate service.

What’s your actual use case here? Are you loading daily CSV dumps, unloading for a data lake, or prepping for a scenario-based question? I’ll help you trim the fat and focus on what actually shows up in real workflows.
