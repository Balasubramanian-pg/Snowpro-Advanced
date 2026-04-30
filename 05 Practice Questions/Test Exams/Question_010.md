# Question 010

![Question 010](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28181%29.png)

**Answer: Row access policy**

**Why:** A Row Access Policy (RAP) in Snowflake filters rows transparently at query time based on the executing user's context (e.g., their mapped territory). Crucially, it works **without any changes to application queries** — the filter is enforced at the policy level, invisibly to the caller. Perfect fit for the scenario described.

**Why the others are wrong:**

- **Object tagging + column-level security** — Tags classify data; column-level security hides *columns*, not *rows*. Neither restricts which rows a user sees.
- **Dynamic data masking policy** — Also column-scoped. It obfuscates column *values* (e.g., masking SSNs), not which rows are returned.
- **Secure view with embedded WHERE clause** — This *could* work functionally, but it requires the application to query the view instead of the base table, meaning **existing queries would need to be redirected** — violating the "without modifying any existing application queries" constraint.

**Key distinction to remember:**
> Row Access Policy = row-level filter, transparent, policy-attached to the table itself.
> Dynamic Data Masking = column-level value obfuscation.
> Secure View = requires query redirection.
