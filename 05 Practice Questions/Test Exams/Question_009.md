# Question 009

![Question 009](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28180%29.png)

**Answers: Infrastructure management for virtual warehouse provisioning + Role-based access control enforcement**

Snowflake's architecture has 3 layers — here's what each owns:

| Layer | Responsibilities |
|---|---|
| **Cloud Services** | Authentication, access control (RBAC), query optimization, metadata management, infrastructure provisioning |
| **Query Processing** | Virtual warehouse compute, parallel query execution across nodes |
| **Storage** | Columnar storage, micro-partition management, data compression |

**Why the others are wrong:**

- **Data compression and columnar storage** → Storage layer
- **Automatic clustering of micro-partitions** → Storage layer (managed there, not by Cloud Services)
- **Parallel query execution across warehouse nodes** → Query Processing layer

The Cloud Services layer is essentially Snowflake's "brain" — it coordinates everything but doesn't touch raw data or run queries itself.
