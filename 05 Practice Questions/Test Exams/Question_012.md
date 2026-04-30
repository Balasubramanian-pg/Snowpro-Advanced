# Question 012

![Question 012](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28183%29.png)

**Answer: It references a location in a supported cloud storage service outside of Snowflake**

**Why:** A named external stage is simply a **pointer** to an external cloud storage location (S3, GCS, or Azure Blob) that you own. Snowflake doesn't move or copy the data — it just references it. You create it once with the URL + credentials, and reuse it for both loading and unloading.

**Why the others are wrong:**

- **"Stores uploaded files within Snowflake's storage layer"** — That describes an **internal stage** (Snowflake-managed storage). External stages point *outside* Snowflake.
- **"Automatically created for each table"** — That describes a **table stage** (`@%table_name`), which is auto-provisioned per table. Named stages are explicitly created by a user.
- **"Can only be used for unloading"** — Wrong direction. Named external stages work for **both** `COPY INTO <table>` (loading) and `COPY INTO <location>` (unloading).

**Stage type cheat sheet:**

| Stage Type | Storage | Created By |
|---|---|---|
| Internal (named) | Snowflake-managed | User (`CREATE STAGE`) |
| Table stage | Snowflake-managed | Auto, per table |
| User stage | Snowflake-managed | Auto, per user |
| **External (named)** | **Your S3/GCS/Azure** | **User (`CREATE STAGE` + URL)** |
