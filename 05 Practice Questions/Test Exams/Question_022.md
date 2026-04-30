# Question 022

![Question 022](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28193%29.png)

**Answer: Alert**

**Why:** A Snowflake **Alert** is purpose-built for exactly this pattern — it runs on a schedule, evaluates a SQL condition, and fires an action (notification via email, Slack, PagerDuty, etc.) when that condition is true. It's the only Snowflake object that combines **scheduled evaluation + condition check + notification action** in one construct.

```sql
CREATE ALERT row_count_alert
  WAREHOUSE = monitor_wh
  SCHEDULE = '15 MINUTE'
  IF (EXISTS (
      SELECT 1 FROM sensitive_table
      HAVING COUNT(*) < 1000
  ))
  THEN CALL SYSTEM$SEND_EMAIL(
      'governance_team@company.com',
      'Alert: Row count below threshold',
      'Sensitive table has dropped below 1000 rows.'
  );
```

**Why the others are wrong:**

| Object | What it actually does | Why it fails here |
|---|---|---|
| **Task** | Executes SQL on a schedule | Runs SQL but has **no condition evaluation or notification mechanism** — it just runs, always |
| **Resource Monitor** | Tracks **credit consumption** on warehouses | Only monitors compute costs, not data/table conditions |
| **Stream** | Captures **CDC (change data)** on a table — inserts, updates, deletes | Records changes but doesn't evaluate conditions or send notifications |

**Key distinction — Alert vs Task:**
> A Task *does* something on a schedule. An Alert *checks* something on a schedule and *reacts* only when a condition is met. For monitoring + notification use cases, always reach for Alert.
