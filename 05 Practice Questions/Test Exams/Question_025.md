# Question 025

![Question 025](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28196%29.png)

**The selected answer is WRONG. ✅ Correct answer: The table will inherit the "high" value from the database through tag inheritance**

**Why:** Snowflake tags support **hierarchical inheritance**. When a tag is applied to a container object (database or schema), all child objects created within it automatically inherit that tag value. So a table created inside a database tagged `confidentiality = 'high'` will itself report `confidentiality = 'high'` — no explicit assignment needed.

```sql
-- Tag applied at database level
ALTER DATABASE sensitive_db SET TAG confidentiality = 'high';

-- Any new table inside inherits it automatically
CREATE TABLE sensitive_db.public.new_table (...);

-- Verify inheritance
SELECT * FROM snowflake.account_usage.tag_references
WHERE object_name = 'NEW_TABLE';
-- Returns: confidentiality = 'high' (inherited from database)
```

**Why the selected answer is wrong:**

There is no "unclassified" system default value in Snowflake's tagging system. Snowflake does not inject any default tag values — inheritance flows from parent to child, not from a system default.

**Why the other two are wrong:**

- **Null until explicitly set at schema level** — Inheritance doesn't require schema-level configuration. Database → table inheritance is direct.
- **Tags must be explicitly set on each object** — Incorrect. The entire purpose of tag inheritance is to avoid having to tag every individual object manually.

**Inheritance chain in Snowflake:**

```
Database (tag: confidentiality = 'high')
    └─► Schema (inherits 'high')
            └─► Table (inherits 'high')
                    └─► Column (inherits 'high')
```

This is a key governance feature — tag once at the top, govern everything beneath it automatically.
