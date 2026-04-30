# Question 016

![Question 016](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28187%29.png)

**Answer: GET_PRESIGNED_URL**

**Why:** A presigned URL grants **time-limited, unauthenticated access** to a specific file in a Snowflake stage. The external application can access the file directly via HTTP — no Snowflake credentials or session required. Perfect for sharing files with external apps temporarily.

**Why the others are wrong:**

| Function | What it does | Auth required? |
|---|---|---|
| `GET_PRESIGNED_URL` | Generates a temporary, token-embedded URL for direct file access | ❌ No Snowflake auth needed |
| `BUILD_SCOPED_FILE_URL` | Generates a URL scoped to the user's session/token | ✅ Requires Snowflake auth |
| `BUILD_STAGE_FILE_URL` | Generates a permanent URL to a staged file | ✅ Requires Snowflake auth |
| `GET_STAGE_LOCATION` | Returns the cloud storage path of a stage (not a file URL) | N/A — wrong purpose entirely |

**Usage example:**
```sql
SELECT GET_PRESIGNED_URL(
    @my_internal_stage, 
    'report.pdf', 
    3600  -- expires in 1 hour
);
```

**Key rule:** If the question says "no Snowflake authentication" or "external access" → `GET_PRESIGNED_URL`. If it says "within Snowflake" or session-based → `BUILD_SCOPED_FILE_URL`.
