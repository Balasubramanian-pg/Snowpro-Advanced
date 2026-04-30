**Overview**
Snowpipe is Snowflake's serverless, event-driven ingestion service for continuous data loading. Unlike `COPY INTO`, which runs on-demand with a virtual warehouse, Snowpipe automatically detects new files in cloud storage via event notifications (S3, Azure, GCS) and loads them using managed, ephemeral compute. It's designed for low-latency, incremental ingestion where files land irregularly and you need near-real-time availability without managing warehouse lifecycle or scheduling.

**Key Characteristics**
- Serverless compute: No virtual warehouse to provision or manage. Snowflake allocates compute automatically per load, billing per second of usage.
- Event-driven auto-ingest: Integrates with cloud storage notifications (S3 EventBridge, Azure Event Grid, GCS Pub/Sub) to trigger loads within seconds of file arrival.
- Micro-batching: Processes files as they arrive, not in large batches. Ideal for high-frequency, small-file ingestion patterns.
- Exactly-once semantics: Tracks loaded files via internal metadata. Re-processing the same file does not duplicate data unless `FORCE=TRUE` is explicitly used.
- COPY INTO syntax: Pipe definition uses standard `COPY INTO` options for format, error handling, and transformation. Familiar syntax, different execution model.
- Manual or auto refresh: Supports both notification-based auto-ingest and manual `INSERT FILES` via REST API for controlled scenarios.
- Cost model: Charges for serverless compute (per second) plus cloud notification costs. No warehouse credits burned, but high-volume small files can accumulate cost.
- Limited concurrency: Default 100 concurrent files per pipe. Scale via multiple pipes or larger files if throughput is bottlenecked.

**Examples**

**1. Basic Snowpipe with S3 auto-ingest**
```sql
CREATE OR REPLACE PIPE raw_events_pipe
  AUTO_INGEST = TRUE
  AWS_SNS_TOPIC = 'arn:aws:sns:us-east-1:123456789010:s3-events'
AS
COPY INTO raw_events
FROM @ext_s3_stage/events/
FILE_FORMAT = (TYPE = JSON)
ON_ERROR = 'SKIP_FILE';
```
*Breakdown*: Creates a pipe that listens to S3 notifications via SNS. When a new JSON file lands, Snowpipe auto-triggers a load into `raw_events`. `ON_ERROR` skips malformed files without stopping the pipeline.

**2. Snowpipe with on-the-fly transformation**
```sql
CREATE OR REPLACE PIPE parsed_logs_pipe
  AUTO_INGEST = TRUE
AS
COPY INTO parsed_logs (log_id, ts, severity, message)
FROM (
  SELECT 
    $1:log_id::VARCHAR,
    $1:timestamp::TIMESTAMP_NTZ,
    $1:level::VARCHAR,
    $1:msg::VARCHAR
  FROM @ext_stage/app_logs/
)
FILE_FORMAT = (TYPE = JSON);
```
*Breakdown*: Transforms semi-structured JSON into typed columns during ingestion. Avoids a separate staging + transformation step. Keeps pipeline lean for light parsing needs.

**3. Monitoring load history**
```sql
SELECT 
  pipe_name,
  status,
  file_name,
  TO_VARCHAR(last_load_time, 'YYYY-MM-DD HH24:MI:SS') AS loaded_at,
  error_count
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME => 'raw_events',
  START_TIME => DATEADD(HOUR, -24, CURRENT_TIMESTAMP())
))
WHERE pipe_name = 'RAW_EVENTS_PIPE';
```
*Breakdown*: Queries load history to audit file processing, check for errors, or build alerting. `COPY_HISTORY` is the primary observability surface for Snowpipe operations.

**4. Manual ingest via REST API**
```bash
POST /v0/pipes/insertFiles
{
  "files": [
    {"path": "s3://my-bucket/events/file_001.json"},
    {"path": "s3://my-bucket/events/file_002.json"}
  ],
  "request_id": "manual-batch-20240430"
}
```
*Breakdown*: Bypasses auto-ingest for controlled, programmatic file submission. Useful for backfills, testing, or environments where cloud notifications aren't feasible. Requires authentication via Snowflake JWT or key-pair.

**5. Error handling with dead-letter pattern**
```sql
-- Pipe with error capture
CREATE OR REPLACE PIPE events_pipe
  AUTO_INGEST = TRUE
AS
COPY INTO raw_events
FROM @ext_stage/events/
FILE_FORMAT = (TYPE = JSON)
ON_ERROR = 'CONTINUE'
VALIDATION_MODE = 'RETURN_ALL_ERRORS';

-- Query errors post-load
SELECT * 
FROM TABLE(VALIDATE(raw_events, JOB_ID => '_last'));
```
*Breakdown*: `ON_ERROR = 'CONTINUE'` loads valid rows and skips bad ones. `VALIDATE` function inspects the last job for parsing errors. Pair with a dead-letter table to capture and reprocess malformed records.

**Notes**
Snowpipe solves a specific problem: continuous, low-latency ingestion without operational overhead. If your data arrives in predictable, large batches (daily dumps, hourly aggregates), `COPY INTO` with a scheduled task is simpler and more cost-effective. Snowpipe shines when files land unpredictably, latency matters, or you want to decouple ingestion from warehouse management.

Cost awareness is critical. Serverless compute bills per second, but high-frequency small files incur overhead per file. If you're dropping 10KB files every second, you'll pay more in compute coordination than in actual processing. Batch files to 100MB-1GB where possible, or use compression. Partition paths and file naming conventions still matter for pruning and parallelism.

Credential and notification setup is the hardest part. Auto-ingest requires precise IAM/SNS/Event Grid configuration. A single misconfigured permission silently breaks the pipeline with no load activity. Test with manual `INSERT FILES` first, validate notifications with cloud CLI tools, then enable `AUTO_INGEST`.

Snowpipe does not replace streaming platforms. It's file-based, not row-based. If you need sub-second row ingestion, use Snowpipe Streaming (Kafka integration) or external streaming services. Snowpipe is for file-at-a-time, near-real-time workloads. Know the boundary before you architect on it.
