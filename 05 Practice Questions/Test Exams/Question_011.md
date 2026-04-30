# Question 011

![Question 011](https://raw.githubusercontent.com/Balasubramanian-pg/Snowpro-Advanced/main/05%20Practice%20Questions/Test%20Exams/Assets/Screenshot%20%28182%29.png)

**Answer: Network policies**

**Why:** Snowflake Network Policies let you define allowed/blocked IP address ranges (CIDRs) and attach them at the account or user level. Only connections originating from whitelisted IPs are permitted — exactly what the compliance requirement describes.

**Why the others are wrong:**

- **Resource monitors** — Control credit/cost consumption on virtual warehouses. Nothing to do with network access.
- **Federated authentication** — Deals with SSO/SAML identity providers (how you authenticate), not *where* you connect from.
- **Object tagging** — Labels Snowflake objects for governance/classification purposes. No access restriction capability.

**Quick syntax reminder:**
```sql
CREATE NETWORK POLICY corp_policy
  ALLOWED_IP_LIST = ('10.0.0.0/8', '192.168.1.0/24')
  BLOCKED_IP_LIST = ('192.168.1.100');

ALTER ACCOUNT SET NETWORK_POLICY = corp_policy;
```

Network policies can be scoped to the entire account or to individual users, giving you fine-grained control.
