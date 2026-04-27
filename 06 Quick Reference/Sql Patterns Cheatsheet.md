# SQL Patterns Cheatsheet: SnowPro Core Data Analyst Exam Prep

## How to Use This
This isn't a tutorial. It's a tactical reference. If you're studying for the SnowPro Core Data Analyst exam, you need to recognize patterns, not memorize syntax. The exam tests application, not recall. Use this to:
- Spot what you don't know yet
- Verify your mental model against exam-relevant defaults
- Practice rewriting patterns from memory

If you can't explain *why* a pattern works, you're not ready. Let's fix that.

---

## 1. Core Query Patterns

### Basic SELECT with Filtering
```sql
-- Always specify columns; avoid SELECT * in production
SELECT customer_id, order_date, total_amount
FROM orders
WHERE order_date >= '2024-01-01'
  AND status IN ('completed', 'shipped')
  AND total_amount > 100
ORDER BY order_date DESC;
```
**Exam traps**:
- `WHERE` filters *before* aggregation; `HAVING` filters *after*
- `NULL` comparisons require `IS NULL`/`IS NOT NULL`; `= NULL` always returns `UNKNOWN`
- String comparisons are case-sensitive by default unless collation is applied

### JOIN Patterns
```sql
-- INNER JOIN: only matching rows
SELECT c.name, o.order_id
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- LEFT JOIN: all left rows, NULLs for unmatched right
SELECT c.name, COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name;

-- Qualify to filter after window calculation (no subquery needed)
SELECT *
FROM (
  SELECT customer_id, order_date,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
  FROM orders
)
QUALIFY rn = 1;  -- Gets most recent order per customer
```
**Exam traps**:
- Joining on nullable keys: `NULL = NULL` is `UNKNOWN`, so rows won't match. Use `IS NOT DISTINCT FROM` for null-safe joins if needed.
- `LEFT JOIN` + `WHERE right_col = 'X'` converts to `INNER JOIN`—filter in `ON` clause instead.
- `QUALIFY` executes *after* window functions, *before* final projection. Cannot reference `SELECT` aliases in same block in all contexts.

### Aggregation Essentials
```sql
-- COUNT variants: know the difference
SELECT 
  COUNT(*) AS total_rows,              -- counts all rows, including NULLs
  COUNT(order_id) AS non_null_orders,  -- counts non-NULL values only
  COUNT(DISTINCT customer_id) AS unique_customers
FROM orders;

-- Conditional aggregation (pivot-like logic)
SELECT 
  product_category,
  SUM(CASE WHEN region = 'West' THEN revenue END) AS west_revenue,
  SUM(CASE WHEN region = 'East' THEN revenue END) AS east_revenue,
  AVG(revenue) FILTER (WHERE revenue > 0) AS avg_positive_revenue  -- cleaner than CASE
FROM sales
GROUP BY product_category;
```
**Exam traps**:
- `AVG`, `SUM`, `COUNT(col)` ignore `NULL`; `COUNT(*)` does not
- `FILTER (WHERE ...)` is standard SQL; equivalent to `CASE` but more readable
- Grouping by expression: `GROUP BY DATE_TRUNC('month', order_date)` requires same expression in `SELECT` or alias

---

## 2. Snowflake-Specific Functions

### Date & Time
```sql
-- Current timestamp variants (know the timezone behavior)
SELECT 
  CURRENT_TIMESTAMP(),      -- TIMESTAMP_TZ, session timezone
  CURRENT_DATE(),           -- DATE, session timezone
  CURRENT_TIME(),           -- TIME, session timezone
  SYSDATE();                -- alias for CURRENT_TIMESTAMP

-- Date arithmetic
SELECT 
  DATEADD(day, 30, order_date) AS ship_deadline,
  DATEDIFF(day, order_date, CURRENT_DATE()) AS days_since_order,
  DATE_TRUNC('month', order_date) AS order_month;  -- groups by month start

-- Parsing & formatting
SELECT 
  TO_DATE('2024-01-15', 'YYYY-MM-DD'),
  TO_TIMESTAMP('2024-01-15 14:30:00', 'YYYY-MM-DD HH24:MI:SS'),
  TO_CHAR(order_date, 'YYYY-MM-DD') AS formatted_date;  -- for output only
```
**Exam traps**:
- `CURRENT_TIMESTAMP()` includes timezone; casting to `DATE` uses session timezone, not UTC
- `DATE_TRUNC` returns start of period; `EXTRACT` pulls component value
- `TO_DATE`/`TO_TIMESTAMP` fail on mismatched format; use `TRY_TO_DATE` for safe conversion

### String & Conditional
```sql
-- Safe string handling
SELECT 
  UPPER(TRIM(customer_name)) AS clean_name,
  COALESCE(phone, email, 'no_contact') AS primary_contact,  -- first non-NULL
  NULLIF(discount_code, '') AS valid_discount,  -- returns NULL if empty string
  SUBSTR(order_id, 1, 8) AS order_prefix;

-- Conditional logic
SELECT 
  CASE 
    WHEN total_amount >= 1000 THEN 'platinum'
    WHEN total_amount >= 500 THEN 'gold'
    ELSE 'standard'
  END AS tier,
  IFF(status = 'cancelled', 0, total_amount) AS effective_amount;  -- Snowflake shorthand
```
**Exam traps**:
- `COALESCE` short-circuits; stops at first non-NULL
- `NULLIF(a, b)` evaluates both arguments before comparing
- `IFF(condition, true_val, false_val)` is Snowflake-specific; equivalent to `CASE WHEN`

### Semi-Structured Data (VARIANT)
```sql
-- Parse JSON, extract fields
SELECT 
  payload:customer_id::VARCHAR AS customer_id,
  payload:items[0].product_name::STRING AS first_product,
  payload:metadata.source::STRING AS source_system
FROM raw_events
WHERE payload:event_type::STRING = 'purchase';

-- Flatten arrays for row expansion
SELECT 
  e.order_id,
  f.value:product_id::VARCHAR AS product_id,
  f.value:quantity::NUMBER AS quantity
FROM orders e,
LATERAL FLATTEN(input => e.payload:items) f;

-- Check for key existence
SELECT *
FROM raw_data
WHERE IS_OBJECT(payload) 
  AND payload:required_field IS NOT NULL;
```
**Exam traps**:
- Path extraction (`:key`) returns `VARIANT`; cast explicitly to target type
- Missing paths return `NULL`, not error—use `TRY_CAST` for safe conversion
- `FLATTEN` produces one row per array element; `LATERAL` binds to outer row context
- `STRICT_JSON_OUTPUT = TRUE` (default) distinguishes `NULL` value vs missing key

---

## 3. Window Functions (High-Yield for Exam)

### Ranking & Deduplication
```sql
-- ROW_NUMBER: unique sequence, no ties
SELECT *
FROM (
  SELECT customer_id, order_date, total_amount,
         ROW_NUMBER() OVER (
           PARTITION BY customer_id 
           ORDER BY order_date DESC, order_id DESC  -- tie-breaker!
         ) AS rn
  FROM orders
)
QUALIFY rn = 1;  -- most recent order per customer

-- RANK vs DENSE_RANK: know the difference
SELECT 
  product_id,
  revenue,
  RANK() OVER (ORDER BY revenue DESC) AS rank_with_gaps,      -- 1,2,2,4
  DENSE_RANK() OVER (ORDER BY revenue DESC) AS rank_no_gaps   -- 1,2,2,3
FROM product_sales;
```
**Exam traps**:
- `ROW_NUMBER` is non-deterministic if `ORDER BY` has ties without unique tie-breaker
- `QUALIFY` filters *after* window calculation; cannot use in `WHERE`
- Default frame with `ORDER BY`: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (running calculation)

### Navigation & Aggregation Windows
```sql
-- LAG/LEAD for period-over-period
SELECT 
  order_date,
  daily_revenue,
  LAG(daily_revenue, 1) OVER (ORDER BY order_date) AS prev_day_revenue,
  daily_revenue - LAG(daily_revenue, 1) OVER (ORDER BY order_date) AS day_over_day_delta
FROM daily_metrics;

-- Running totals and moving averages
SELECT 
  order_date,
  revenue,
  SUM(revenue) OVER (
    ORDER BY order_date 
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total,
  AVG(revenue) OVER (
    ORDER BY order_date 
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS seven_day_moving_avg
FROM daily_sales;
```
**Exam traps**:
- `LAG`/`LEAD` default offset is 1; default return on boundary is `NULL` unless `default` specified
- Frame clauses: `ROWS` = physical rows; `RANGE` = logical value range (requires numeric/datetime `ORDER BY`)
- `IGNORE NULLS` modifier available for `LAG`/`LEAD`/`FIRST_VALUE`/`LAST_VALUE` only—not aggregates

---

## 4. Data Loading & Export Patterns

### COPY INTO <table> (Load)
```sql
COPY INTO orders
FROM @my_stage/orders/
FILE_FORMAT = (
  TYPE = 'CSV',
  FIELD_DELIMITER = ',',
  SKIP_HEADER = 1,
  NULL_IF = ['NULL', 'null', '']
)
ON_ERROR = 'CONTINUE'  -- logs errors, doesn't abort
PATTERN = '.*orders_.*\.csv\.gz';  -- regex for file selection
```
**Exam traps**:
- `ON_ERROR = 'CONTINUE'` logs to load history but does *not* populate custom error tables—manual routing required
- `NULL_IF` converts specified strings to `NULL` during load
- `PATTERN` uses regex; test with `LIST @stage` first
- File format options are case-sensitive: `TYPE`, not `type`

### COPY INTO <location> (Export)
```sql
COPY INTO s3://my-bucket/export/orders/
FROM orders
STORAGE_INTEGRATION = my_s3_integration
FILE_FORMAT = (TYPE = 'PARQUET', COMPRESSION = 'SNAPPY')
MAX_FILE_SIZE = 1073741824  -- 1GB
OVERWRITE = FALSE;  -- default; fails if files exist
```
**Exam traps**:
- Export order is non-deterministic; `ORDER BY` in source query does *not* guarantee file row order
- `SINGLE = TRUE` forces serial export (slow); avoid for large datasets
- Compression reduces transfer size but adds CPU; SNAPPY balances speed/size, GZIP maximizes compression

### Staging Basics
```sql
-- Internal stage (Snowflake-managed)
CREATE STAGE my_internal_stage;
LIST @my_internal_stage;
GET @my_internal_stage/file.csv file://local/path/;  -- download
PUT file://local/path/file.csv @my_internal_stage;   -- upload

-- External stage (cloud storage)
CREATE STAGE my_s3_stage
  URL = 's3://my-bucket/data/'
  STORAGE_INTEGRATION = my_s3_integration
  FILE_FORMAT = my_csv_format;
```
**Exam traps**:
- `PUT`/`GET` are client-side commands; executed in SnowSQL, not Snowsight
- External stages require `STORAGE_INTEGRATION` (never embed credentials in DDL)
- `LIST` shows files; `REMOVE` deletes from stage (irreversible)

---

## 5. Performance & Optimization (Exam Focus)

### Pruning & Clustering
```sql
-- Check pruning potential
SELECT SYSTEM$CLUSTERING_INFORMATION('orders', 'order_date >= ''2024-01-01''');

-- Sargable vs non-sargable predicates
WHERE order_date >= '2024-01-01'                    -- ✓ pruning eligible
WHERE DATE_TRUNC('day', order_timestamp) = '2024-01-01'  -- ✗ full scan
WHERE customer_id::VARCHAR = '123'                  -- ✗ implicit cast breaks pruning

-- Cluster large tables on high-selectivity filters
ALTER TABLE orders CLUSTER BY (order_date, region);
```
**Exam traps**:
- `SYSTEM$CLUSTERING_INFORMATION` returns `NULL` if table not clustered or expression non-deterministic
- Function-wrapped columns bypass pruning; pre-compute derived columns if needed
- `CLUSTER BY` max 4 expressions; leftmost column dominates pruning effectiveness

### Result Caching
```sql
-- Cache is automatic; control with session param
ALTER SESSION SET RESULT_CACHE_ACTIVE = TRUE;  -- default

-- Check cache status for a query
SELECT SYSTEM$RESULT_CACHE_INFO('<query_hash>');

-- Bypass cache for freshness-critical queries
ALTER SESSION SET RESULT_CACHE_ACTIVE = FALSE;
SELECT ...;  -- forces re-execution
```
**Exam traps**:
- Cache keyed by: query text + role + warehouse + database + schema
- Non-deterministic functions (`RANDOM()`, `CURRENT_TIMESTAMP()`) always bypass cache
- Cache TTL: 24 hours default; changing any session context invalidates cache

---

## 6. Security & Governance (Must-Know)

### RBAC Basics
```sql
-- Role hierarchy: parent inherits child privileges
GRANT ROLE analyst TO ROLE manager;

-- Grant object access
GRANT SELECT ON TABLE orders TO ROLE analyst;
GRANT USAGE ON WAREHOUSE compute_wh TO ROLE analyst;

-- Set default role for user
ALTER USER jsmith SET DEFAULT_ROLE = 'analyst';
```
**Exam traps**:
- `CURRENT_ROLE()` returns primary role only; secondary roles require explicit `USE ROLE`
- Privileges are not inherited *down* the hierarchy—only up
- `OWNERSHIP` grants full control; use `GRANT ... WITH GRANT OPTION` sparingly

### Row Access Policies & Dynamic Data Masking
```sql
-- Row Access Policy example
CREATE ROW ACCESS POLICY region_policy AS
  CASE 
    WHEN CURRENT_ROLE() IN ('ADMIN', 'EXEC') THEN TRUE
    WHEN CURRENT_ROLE() = 'REGIONAL_ANALYST' THEN region = CURRENT_USER_REGION()
    ELSE FALSE
  END;

ALTER TABLE sales ADD ROW ACCESS POLICY region_policy ON (region);

-- Dynamic Data Masking example
CREATE MASKING POLICY email_mask AS
  CASE
    WHEN CURRENT_ROLE() IN ('ADMIN', 'SUPPORT') THEN val
    ELSE SHA2(val, 256)  -- or '***' for VARCHAR
  END;

ALTER TABLE customers MODIFY COLUMN email SET MASKING POLICY email_mask;
```
**Exam traps**:
- Policies evaluate at query time, *after* filter substitution; filters cannot bypass policies
- Masking expressions must return same data type as source column
- Shared dashboards apply recipient's role context, not sharer's—policies follow the viewer

---

## 7. Common Exam Traps & Defaults (Memorize These)

| Concept | Default / Behavior | Exam Relevance |
|---------|-------------------|----------------|
| `COUNT(*)` vs `COUNT(col)` | `*` counts all rows; `col` excludes NULLs | Aggregation questions |
| `NULL` comparison | `= NULL` → `UNKNOWN`; use `IS NULL` | Filtering logic |
| `CURRENT_ROLE()` | Returns primary role only | RBAC/policy questions |
| Result cache TTL | 24 hours | Performance/caching questions |
| `CLUSTER BY` limit | Max 4 expressions | Optimization questions |
| `COPY INTO` error handling | `ON_ERROR = 'ABORT_STATEMENT'` default | Loading questions |
| Semi-structured path | Missing key → `NULL`, not error | JSON/VARIANT questions |
| `QUALIFY` execution order | After window functions, before final projection | Window function questions |
| `DATE_TRUNC` timezone | Uses session timezone, not UTC | Date/time questions |
| String comparison | Case-sensitive by default | Filtering/join questions |
| `TRY_CAST` vs `CAST` | `TRY_CAST` returns `NULL` on failure; `CAST` throws error | Type conversion questions |
| `FLATTEN` output | One row per array element | Semi-structured questions |
| `LAG`/`LEAD` default | Offset=1, return `NULL` at boundaries | Window navigation questions |
| `SYSTEM$CLUSTERING_INFORMATION` | Returns `NULL` if unclustered or non-deterministic filter | Pruning questions |
| Shared dashboard policies | Recipient's role context applies, not sharer's | Governance questions |

---

## 8. Practice Prompts (Test Yourself)

1. *Rewrite this non-sargable filter to enable pruning*:
   ```sql
   WHERE DATE_TRUNC('day', created_at) = '2024-01-15'
   ```
   <details><summary>Answer</summary>
   
   ```sql
   WHERE created_at >= '2024-01-15' AND created_at < '2024-01-16'
   ```
   </details>

2. *Why does this `LEFT JOIN` return fewer rows than the left table?*
   ```sql
   SELECT c.*, o.order_id
   FROM customers c
   LEFT JOIN orders o ON c.customer_id = o.customer_id
   WHERE o.status = 'completed';
   ```
   <details><summary>Answer</summary>
   
   The `WHERE o.status = 'completed'` filters *after* the join, removing rows where `o.status` is `NULL` (unmatched orders). Move the condition to the `ON` clause: `ON c.customer_id = o.customer_id AND o.status = 'completed'`.
   </details>

3. *What's the difference between these two counts?*
   ```sql
   SELECT COUNT(*), COUNT(discount_code) FROM orders;
   ```
   <details><summary>Answer</summary>
   
   `COUNT(*)` includes all rows; `COUNT(discount_code)` excludes rows where `discount_code` is `NULL`. If 100 orders exist and 20 have `NULL` discounts, results are 100 and 80.
   </details>

4. *Why might this window function produce non-deterministic results?*
   ```sql
   ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY total_amount DESC)
   ```
   <details><summary>Answer</summary>
   
   If multiple orders have the same `total_amount` for a customer, the order among ties is undefined. Add a tie-breaker: `ORDER BY total_amount DESC, order_id DESC`.
   </details>

---

## Final Reality Check

If you're cramming:
- Focus on patterns you can *recognize*, not syntax you can recite
- Practice rewriting non-sargable predicates—it's a guaranteed exam topic
- Memorize the defaults table above; exam questions love to test "what happens if you don't specify X"
- Run these queries in Snowsight; muscle memory beats flashcards

But if you're actually learning:
- Ask *why* `QUALIFY` executes after window functions (it needs the computed value to filter)
- Understand *how* pruning works (micro-partition min/max metadata) so you can debug slow queries
- Trace *where* policies evaluate (query compilation) so you don't waste time trying to bypass them

The exam tests whether you can *apply* Snowflake as an analyst—not whether you can quote documentation. Build mental models, not memorization.

You've got this. Now go break something in a dev environment and fix it. That's how you actually learn.
