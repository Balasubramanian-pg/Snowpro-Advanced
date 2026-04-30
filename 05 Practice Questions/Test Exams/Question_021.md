# Question 021

![Question 021](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28192%29.png)

**Answer: No data is copied; a shared database referencing the provider's data appears in the consumer's account**

**Why:** Snowflake Marketplace is built on **Secure Data Sharing** — a zero-copy architecture. When a consumer acquires a listing, Snowflake creates a read-only shared database in their account that **points directly to the provider's storage**. The data never physically moves. The consumer queries it live, and the provider's warehouse handles nothing — the consumer's own warehouse executes the query against the shared data.

**Why the others are wrong:**

- **A replicated copy is created in the consumer's account** — This is the most common misconception. No copy is made. Zero bytes are duplicated. This is the core value proposition of Snowflake's sharing architecture.

- **Data is exported to the consumer's internal stage for loading** — This would be a traditional ETL/extract pattern. Marketplace is specifically designed to eliminate this — no staging, no loading, no pipelines.

- **Data is extracted and loaded into new tables within an existing consumer database** — Again, no extraction, no loading, no new tables created. The shared database is a reference, not a materialization.

**Key mental model:**

| Aspect | Reality |
|---|---|
| Data movement | ❌ None — zero-copy |
| Storage cost to consumer | ❌ None |
| Data freshness | ✅ Always live (provider's data) |
| Consumer's warehouse used | ✅ Yes — for query execution |
| Provider's warehouse used | ❌ No |

This is one of Snowflake's flagship differentiators — live data sharing across accounts with no ETL, no replication lag, and no storage duplication.
