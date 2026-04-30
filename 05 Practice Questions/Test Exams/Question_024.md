# Question 024

![Question 024](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28195%29.png)

**Selected answer is CORRECT. ✅**

**Why:** Clustering keys work by co-locating similar values in the same micro-partitions, enabling **partition pruning** — Snowflake skips irrelevant micro-partitions entirely rather than scanning the full 5TB. The benefit is maximized when the clustering key matches the columns used in `WHERE` filters and `JOIN` predicates, which is exactly `transaction_date` and `region_id` in this scenario.

```sql
ALTER TABLE fact_transactions
CLUSTER BY (transaction_date, region_id);
```

**Why the others are wrong:**

- **Cluster on the primary key column** — Primary keys in Snowflake are not enforced and have no physical ordering effect. Clustering on a high-cardinality surrogate key (like an ID) is essentially useless for pruning — every micro-partition would contain a wide spread of values with no grouping benefit.

- **Cluster on lowest cardinality column** — This is backwards. You want **medium-to-high cardinality** in the clustering key so Snowflake can meaningfully separate data into distinct partition ranges. Very low cardinality (e.g., a boolean flag) means too few distinct ranges — pruning is minimal.

- **Cluster on every column** — Completely counterproductive. Clustering on many columns destroys clustering effectiveness, massively increases reclustering overhead and cost, and provides no meaningful pruning benefit. Snowflake recommends **1–3 columns maximum**.

**Golden rule for clustering keys:**
> Pick columns that are: (1) frequently in `WHERE`/`JOIN` predicates, (2) have reasonable cardinality (not boolean, not unique ID), and (3) are queried together. Date + one dimension column is the classic pharma/analytics pattern.
