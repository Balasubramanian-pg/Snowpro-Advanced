# 📋 Exam Breakdown — What to Expect

## Exam Format

| Property | Detail |
|----------|--------|
| **Exam Name** | SnowPro Advanced: Data Analyst |
| **Prerequisite** | SnowPro Core (must be active) |
| **Format** | Multiple choice + scenario-based questions |
| **Duration** | 115 minutes |
| **Passing Score** | ~750 out of 1000 (scaled scoring) |
| **Delivery** | Online proctored (Examity) or testing center |
| **Validity** | 2 years from certification date |

---

## What Makes This Exam Hard

1. **Scenario-based questions** — They describe a business problem and ask which SQL/approach is correct. You must understand the *outcome*, not just the syntax.
2. **Tricky distractors** — Wrong answers look almost right. Often, two answers use the correct function but with flipped arguments (e.g., `REGR_SLOPE(y, x)` vs `REGR_SLOPE(x, y)`).
3. **Snowflake-specific behavior** — Questions test Snowflake quirks, not generic SQL knowledge (e.g., how NULL handling works in Snowflake, how caching behaves).
4. **Breadth** — It spans everything from raw data loading to visualization in Snowsight.

---

## Domain Deep Dive

### Domain 1: Data Ingestion & Data Preparation (17%)
Tests your ability to:
- Load structured, semi-structured, and unstructured data
- Use COPY INTO, PUT, stages
- Work with PARSE_JSON, INFER_SCHEMA
- Set up pipelines with scheduling
- Understand data lineage and auditing

**Key Snowflake Objects:** Stage, Pipe, Task, Stream, File Format

---

### Domain 2: Data Transformation & Data Modeling (23%)
Tests your ability to:
- Clean and reshape data (NULLs, deduplication, type casting)
- Write complex queries (CTEs, window functions, PIVOT)
- Choose the right data model (star schema, Data Vault, flat)
- Optimize queries (clustering, caching, pruning)

**Key Snowflake Objects:** Materialized Views, Transient Tables, Clone, Data Metric Functions

---

### Domain 3: Data Analysis (32%) — THE BIG ONE
Tests your ability to:
- Write and use UDFs, UDTFs, Stored Procedures
- Use Views (regular, secure, materialized)
- Perform descriptive analysis using aggregations and stats functions
- Find root causes of anomalies (diagnostic)
- Use REGR_SLOPE, REGR_INTERCEPT for forecasting

**Key Snowflake Objects:** Notebooks, Worksheets, Snowsight Dashboards, ML Functions

---

### Domain 4: Data Presentation & Visualization (28%)
Tests your ability to:
- Build dashboards in Snowsight
- Pick the right chart type for a business question
- Apply row access policies and dynamic data masking
- Connect BI tools to Snowflake
- Share and manage dashboards

**Key Snowflake Objects:** Dashboards, Tiles, Filters, Row Access Policies, Dynamic Data Masking

---

## How Questions Are Structured

Most questions follow this pattern:

> **"A Data Analyst needs to [goal]. The table has [structure]. Which query/approach achieves this?"**

You will then see 4 options that are all plausible-looking SQL or configuration choices.

**Common traps:**
- Flipped function arguments (dependent vs independent variable)
- Using `UNION` instead of `UNION ALL` (or vice versa)
- Using a regular view when a secure view is required
- Wrong role/context causing a dashboard to not refresh

---

## Scoring

Snowflake uses **scaled scoring** — not all questions are worth the same points. Harder questions (scenario-based) are worth more. Don't panic if you see a hard question. Don't skip them.

---

## Resources to Bookmark Now

- [Snowflake Documentation](https://docs.snowflake.com)
- [Snowsight UI Guide](https://docs.snowflake.com/en/user-guide/ui-snowsight)
- [Snowflake Quickstarts](https://quickstarts.snowflake.com)
- [Sample Questions (Official)](https://learn.snowflake.com/en/certifications/snowpro-advanced-dataanalyst/)

---

> Next: Open `60-day-study-plan.md` — that's your day-by-day roadmap.
