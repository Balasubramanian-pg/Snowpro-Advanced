# Question 015

![Question 015](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28186%29.png)

**Answer: The ACCOUNT_USAGE schema in the SNOWFLAKE database, which retains data for up to 365 days**

**Why:** 10 months falls within 365 days, so `SNOWFLAKE.ACCOUNT_USAGE` covers it. This is the standard schema for historical usage analysis — including `WAREHOUSE_METERING_HISTORY` for credit consumption trends.

**Why the others are wrong:**

| Option | Issue |
|---|---|
| `INFORMATION_SCHEMA` — 12 months | Sounds plausible, but **wrong** — INFORMATION_SCHEMA only retains data for **7 days to 6 months** depending on the view (e.g., `QUERY_HISTORY` is only 7 days). The "12 months" claim is fabricated. |
| `ORGANIZATION_USAGE` — "only schema that tracks credits" | False — ACCOUNT_USAGE also tracks credits. ORGANIZATION_USAGE is for **cross-account** reporting at the org level. |
| `QUERY_PROFILE` schema — 18 months | **Doesn't exist** as a schema. Query profiles are accessed via the UI or `QUERY_HISTORY`. Fabricated option. |

**Key retention comparison to remember:**

| Schema | Retention |
|---|---|
| `INFORMATION_SCHEMA` | 7 days – 6 months (view-dependent) |
| `ACCOUNT_USAGE` | **365 days** |
| `ORGANIZATION_USAGE` | 365 days (org-level) |

For anything beyond a few days of history, always use `ACCOUNT_USAGE`.
