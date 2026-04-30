# Question 019

![Question 019](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28190%29.png)

**The selected answer is WRONG. ✅ Correct answer: On the virtual warehouse used by the reporting team**

**Why:** The Query Acceleration Service (QAS) in Snowflake is a warehouse-level setting. It offloads portions of eligible analytic queries (large scans, filters, aggregations) to serverless compute, reducing contention on the warehouse. You enable it — and optionally cap its scale factor — directly on the warehouse object.

```sql
ALTER WAREHOUSE reporting_wh
  SET ENABLE_QUERY_ACCELERATION = TRUE
      QUERY_ACCELERATION_MAX_SCALE_FACTOR = 8;
```

**Why the selected answer is wrong:**

"On the database containing the queried tables" — QAS has nothing to do with the database object. Databases in Snowflake are purely logical namespaces for organizing schemas and tables. You cannot attach compute behavior or acceleration settings to a database.

**Why the other two are also wrong:**

- **Query-level hint** — Snowflake does not support query-level hints to enable QAS. There is no `/*+ QAS */` syntax or equivalent. QAS eligibility is determined automatically by Snowflake based on query shape.
- **On the schema where reporting views are stored** — Schemas, like databases, are logical containers only. No compute or acceleration configuration can be applied at the schema level.

**Key mental model:**

> Anything related to compute behavior (QAS, auto-suspend, auto-resume, scaling policy, concurrency) is always configured on the **virtual warehouse** — never on database/schema/table objects.

| Object | Can configure QAS? |
|---|---|
| Virtual Warehouse | ✅ Yes — this is where it lives |
| Database | ❌ No |
| Schema | ❌ No |
| Individual Query | ❌ No |
