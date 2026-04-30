# Question 014

![Question 014](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28185%29.png)

**Answer: QUERY_ATTRIBUTION_HISTORY**

**Why:** `QUERY_ATTRIBUTION_HISTORY` is specifically designed to attribute **credit costs to individual queries**. It breaks down the warehouse credit consumption at the query level, letting you pinpoint exactly which queries are the most expensive.

**Why the others are wrong:**

| View | What it actually shows |
|---|---|
| `WAREHOUSE_METERING_HISTORY` | Credit consumption aggregated at the **warehouse level**, by hour — no query-level detail |
| `WAREHOUSE_LOAD_HISTORY` | Warehouse **utilization/load** over time (queued, running, blocked) — no credit cost info |
| `QUERY_HISTORY` | Query execution metadata (duration, rows, bytes, status) — **no credit attribution** |

**Practical usage:**
```sql
SELECT 
    query_id,
    query_text,
    warehouse_name,
    credits_attributed_compute
FROM snowflake.account_usage.query_attribution_history
ORDER BY credits_attributed_compute DESC
LIMIT 20;
```

This is your go-to view for query-level cost optimization — pair it with `QUERY_HISTORY` (joined on `query_id`) to get the full picture of what a query did AND what it cost.
