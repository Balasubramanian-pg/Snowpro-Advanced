# Question 002

![Question 002](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28173%29.png)

**Answer: The warehouse cache was purged when the warehouse was suspended**

**Explanation:**

When a virtual warehouse (like in Snowflake) is suspended, its **local cache is cleared**. This cache includes:
- Data files cached from storage
- Intermediate query results
- Metadata about accessed data

When the warehouse resumes and queries run again, they must:
1. Fetch data from the slower remote storage layer (S3, Azure Blob, etc.)
2. Rebuild the cache with frequently accessed data
3. Only after the cache warms up do queries return to normal speed

This is why the **first several queries** are slower - they're experiencing cold cache performance. Once the cache is repopulated, performance returns to normal.

**Why the other options are incorrect:**
- **Metadata cache** - This is maintained at the system level, not warehouse-specific, and persists across suspension
- **Smaller warehouse size** - Warehouses resume at their configured size unless explicitly changed; this wouldn't be automatic
- **Query result cache invalidation** - Result caches are typically at the account/system level and aren't invalidated just by suspending/resuming a warehouse

This is a fundamental trade-off: suspending saves costs but causes a "cold start" performance penalty on resume.
