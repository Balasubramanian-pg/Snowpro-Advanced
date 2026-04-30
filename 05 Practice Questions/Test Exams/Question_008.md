# Question 008

![Question 008](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28179%29.png)

**Answer: Snowpipe Streaming writes rows directly to tables via the Ingest SDK without requiring staging files.**

**Why this is correct:**

| | Standard Snowpipe | Snowpipe Streaming |
|---|---|---|
| **Mechanism** | File-based (stages files in S3/GCS/Azure, then triggers COPY INTO via event notifications) | Row-based (uses the Snowflake Ingest SDK to push rows directly to a table buffer) |
| **Latency** | Seconds to minutes (depends on file arrival + micro-batch processing) | Sub-second (near real-time) |
| **Staging required** | Yes — files must land in a stage first | No — bypasses staging entirely |
| **Warehouse** | Serverless (no user-managed VW needed) | Also serverless |

**Why the others are wrong:**

- **Option A** — It's backwards. *Standard* Snowpipe uses file-based event notifications; *Snowpipe Streaming* uses the Ingest SDK.
- **Option B** — Completely reversed. Snowpipe Streaming is the one with sub-second latency; standard Snowpipe is the batch-oriented one.
- **Option D** — Neither requires a user-managed virtual warehouse. Both run on Snowflake's serverless compute.

**Practical rule of thumb:** If your pipeline produces discrete files (Kafka S3 sink, scheduled exports), use standard Snowpipe. If you need row-level, sub-second ingestion (Kafka with the Snowflake connector in streaming mode, CDC), use Snowpipe Streaming.
