# Question 013

![Question 013](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28184%29.png)

**The selected answer is WRONG. Correct answer: Child tasks execute after their predecessor task completes successfully, without defining their own schedule**

**Why:** In a Snowflake Task DAG, only the **root task** has a schedule. Child tasks have no schedule of their own — they are triggered automatically when their immediate predecessor task **completes successfully**. This creates a dependency chain, not parallel execution.

**Why the selected answer is wrong:**

Child tasks do NOT run simultaneously with the root task. They wait for their predecessor to finish first — that's the entire point of a DAG (directed acyclic graph).

**Why the others are wrong:**

- **Each child task must define its own CRON schedule** — False. Only the root task has a schedule. Child tasks use `AFTER <predecessor_task>` instead.
- **Triggered by event notification from root task's virtual warehouse** — No such mechanism exists. Triggering is handled by Snowflake's Cloud Services layer, not the warehouse.

**How it actually works:**
```
Root Task (every 10 min)
    └─► Child Task A (runs after root succeeds)
            └─► Child Task B (runs after A succeeds)
                    └─► Child Task C (runs after B succeeds)
```

```sql
CREATE TASK child_task_a
  AFTER root_task  -- no SCHEDULE defined
AS ...;
```

The DAG only starts when the root fires; each child waits on its predecessor's success.
