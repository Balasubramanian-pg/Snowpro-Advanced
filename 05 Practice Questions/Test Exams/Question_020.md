# Question 020

![Question 020](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28191%29.png)

**Answer: API integration**

**Why:** Connecting Snowflake to an external Git provider (GitHub, GitLab, Bitbucket) requires an **API integration** object. It defines the allowed external endpoint and handles the authentication credentials (personal access token or secret) needed to reach the Git provider's API. You then create a `GIT REPOSITORY` object that references this API integration.

```sql
-- Step 1: Create the API integration
CREATE API INTEGRATION github_integration
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/your-org/')
  ALLOWED_AUTHENTICATION_SECRETS = ALL
  ENABLED = TRUE;

-- Step 2: Create the Git repository object
CREATE GIT REPOSITORY my_repo
  API_INTEGRATION = github_integration
  GIT_CREDENTIALS = my_github_secret
  ORIGIN = 'https://github.com/your-org/your-repo.git';
```

**Why the others are wrong:**

| Integration Type | Actual Purpose |
|---|---|
| **Security integration** | Handles SSO/SAML/OAuth for **user authentication** into Snowflake (e.g., Okta, ADFS) |
| **Storage integration** | Allows Snowflake to access **cloud storage** (S3, GCS, Azure Blob) for stages without embedding credentials |
| **API integration** | ✅ Connects Snowflake to **external HTTP/HTTPS APIs and Git providers** |
| **Notification integration** | Enables Snowflake to send/receive **event notifications** via SNS, Azure Event Grid, or GCS Pub/Sub |

**Memory anchor:** Any time Snowflake needs to reach out to an **external HTTP-based service** (Git, external functions via Lambda/Cloud Functions), the glue object is always an **API integration**.
