**SECTION 2: SECURITY, ACCESS CONTROL, AND DATA GOVERNANCE**
**Logical Division:** Part A addresses authentication mechanisms, role hierarchy design, privilege inheritance, and network-level security controls. Part B addresses data-level protections including masking policies, row access controls, object tagging, and audit frameworks. This separation reflects the exam's distinction between infrastructure-level security and data-centric governance, requiring candidates to demonstrate competency in both domains independently.

**PART A: SECURITY FRAMEWORK, RBAC, AND AUTHENTICATION (Q201–Q300)**

**Q201.** A role hierarchy includes SYSADMIN owning CUSTOM_ROLE, which owns ANALYST_ROLE. A user is granted ANALYST_ROLE. Which privileges does the user inherit?
A. Only privileges granted directly to ANALYST_ROLE
B. Privileges granted to ANALYST_ROLE and CUSTOM_ROLE
C. Privileges granted to ANALYST_ROLE, CUSTOM_ROLE, and SYSADMIN
D. Only privileges explicitly granted to the user
**Answer:** B
**Explanation:** Role inheritance flows upward through the hierarchy. A user granted ANALYST_ROLE inherits privileges from ANALYST_ROLE and any parent roles (CUSTOM_ROLE). SYSADMIN is not inherited unless explicitly granted to the user or to a role in the chain above CUSTOM_ROLE.

**Q202.** Which privilege allows a role to grant object privileges to other roles without owning the object?
A. OWNERSHIP
B. GRANT OPTION
C. REFERENCES
D. APPLY
**Answer:** B
**Explanation:** The GRANT OPTION privilege enables a role to delegate privileges it possesses to other roles. This facilitates distributed access management without transferring object ownership.

**Q203.** A future grant is applied to a schema. Which objects receive the granted privileges automatically?
A. Existing tables only
B. Tables created after the grant is applied
C. All current and future tables in the schema
D. Views and functions only
**Answer:** B
**Answer:** B
**Explanation:** Future grants apply exclusively to objects created after the grant statement executes. Existing objects require explicit privilege assignment. This distinction prevents unintended access to historical data.

**Q204.** Which authentication method supports multi-factor authentication natively within Snowflake?
A. Username/password only
B. SAML 2.0 with IdP-initiated SSO
C. OAuth with external token provider
D. Key-pair authentication
**Answer:** B
**Explanation:** SAML 2.0 integrations leverage the identity provider's MFA capabilities. Snowflake delegates authentication to the IdP, which enforces MFA policies before issuing assertions. Other methods require external enforcement or additional configuration.

**Q205.** A network policy restricts access to specific IP ranges. Which connection type bypasses this restriction?
A. Snowsight web interface
B. SnowSQL CLI with valid credentials
C. PrivateLink endpoint within approved VPC
D. Partner connector with OAuth token
**Answer:** C
**Explanation:** Network policies apply to public endpoint connections. PrivateLink and similar private connectivity options operate outside the public network boundary, requiring separate security controls at the VPC level.

**Q206.** Which privilege is required to create a row access policy?
A. CREATE POLICY
B. CREATE ROW ACCESS POLICY
C. APPLY MASKING POLICY
D. MANAGE GRANTS
**Answer:** B
**Explanation:** Row access policies require the CREATE ROW ACCESS POLICY privilege at the schema or database level. This privilege is distinct from masking policy creation and governs row-level visibility logic.

**Q207.** A masking policy is applied to a column. Which role can view unmasked data by default?
A. Any role with SELECT privilege on the table
B. The role that created the masking policy
C. Roles explicitly granted the UNMASK privilege
D. SYSADMIN and ACCOUNTADMIN only
**Answer:** C
**Explanation:** Masking policies enforce data transformation at query time. Only roles granted the UNMASK privilege bypass the policy and access raw column values. Ownership or SELECT privilege alone does not override masking.

**Q208.** Which system function returns the current role context during query execution?
A. CURRENT_USER()
B. CURRENT_ROLE()
C. SESSION_ROLE()
D. ACTIVE_ROLE()
**Answer:** B
**Explanation:** CURRENT_ROLE() returns the active role for the session. CURRENT_USER() returns the authenticated user identity, which may differ when roles are switched. This distinction is critical for auditing and policy evaluation.

**Q209.** A secure view references a base table with a masking policy. Where is the masking applied?
A. During base table storage
B. During view compilation
C. During result projection from the secure view
D. During warehouse provisioning
**Answer:** C
**Explanation:** Masking policies execute during the projection phase. When a secure view is queried, the engine applies masking transformations before returning results, regardless of the underlying table's access controls.

**Q210.** Which privilege allows a role to modify a row access policy without owning it?
A. ALTER POLICY
B. APPLY ROW ACCESS POLICY
C. OWNERSHIP
D. MANAGE POLICIES
**Answer:** A
**Explanation:** The ALTER POLICY privilege permits modification of policy definitions. Ownership grants full control, but ALTER POLICY enables targeted updates without transferring ownership responsibilities.

**Q211.** A user authenticates via SAML SSO. Which Snowflake object stores the IdP metadata?
A. SECURITY INTEGRATION
B. NETWORK POLICY
C. SAML INTEGRATION
D. AUTHENTICATION POLICY
**Answer:** A
**Explanation:** Security integrations configure SAML, OAuth, and other external authentication methods. The integration object stores IdP endpoints, certificates, and assertion mapping required for federated authentication.

**Q212.** Which privilege is required to create a security integration for SAML?
A. CREATE INTEGRATION
B. CREATE SECURITY INTEGRATION
C. MANAGE ACCOUNT
D. APPLY AUTHENTICATION POLICY
**Answer:** B
**Explanation:** Creating security integrations requires the CREATE SECURITY INTEGRATION privilege. This privilege is typically restricted to security administrators to prevent unauthorized authentication configuration.

**Q213.** A row access policy references CURRENT_ROLE(). How does this affect policy evaluation?
A. The policy evaluates against the user's default role
B. The policy evaluates against the active session role
C. The policy ignores role context and applies globally
D. The policy fails if multiple roles are active
**Answer:** B
**Explanation:** Row access policies evaluate expressions in the context of the active session role. CURRENT_ROLE() returns the role executing the query, enabling dynamic row filtering based on role membership.

**Q214.** Which authentication method supports programmatic access without storing credentials in code?
A. Username/password
B. Key-pair authentication
C. OAuth with refresh tokens
D. SAML with IdP assertion
**Answer:** C
**Explanation:** OAuth enables token-based authentication where refresh tokens can be securely stored and rotated. Applications obtain access tokens without embedding long-lived credentials, reducing exposure risk.

**Q215.** A network policy is applied at the account level. Which users does it affect?
A. Only users with ACCOUNTADMIN role
B. All users authenticating via public endpoints
C. Only users connecting from external networks
D. Users with explicit policy grants
**Answer:** B
**Explanation:** Account-level network policies apply to all connections through public endpoints. Private connectivity options and internal Snowflake services operate outside this scope.

**Q216.** Which privilege allows a role to revoke privileges granted by other roles?
A. REVOKE OPTION
B. MANAGE GRANTS
C. OWNERSHIP
D. APPLY GRANTS
**Answer:** B
**Explanation:** The MANAGE GRANTS privilege enables a role to modify any grant on an object, including revoking privileges granted by other roles. This supports centralized access governance without requiring ownership.

**Q217.** A masking policy uses a conditional expression based on a user attribute. Which function retrieves the authenticated user name?
A. CURRENT_USER()
B. CURRENT_ROLE()
C. SESSION_USER()
D. AUTHENTICATED_USER()
**Answer:** A
**Explanation:** CURRENT_USER() returns the authenticated user identity for the session. This value remains constant even when roles are switched, enabling consistent policy evaluation based on user context.

**Q218.** Which object type supports dynamic data masking at the column level?
A. Table only
B. View only
C. Table and view columns
D. External table columns only
**Answer:** C
**Explanation:** Masking policies apply to columns in both base tables and views. When a view references a masked column, the policy executes during result projection unless explicitly overridden.

**Q219.** A role hierarchy includes multiple levels. Which statement accurately describes privilege propagation?
A. Privileges granted to child roles automatically propagate to parent roles
B. Privileges granted to parent roles automatically propagate to child roles
C. Privileges propagate bidirectionally through the hierarchy
D. Privileges do not propagate; each role requires explicit grants
**Answer:** B
**Explanation:** Privilege inheritance flows upward: child roles inherit privileges from parent roles. Grants to child roles do not affect parent roles. This unidirectional flow supports least-privilege design patterns.

**Q220.** Which privilege is required to attach a masking policy to a column?
A. APPLY MASKING POLICY
B. ATTACH POLICY
C. MODIFY COLUMN
D. MANAGE MASKING
**Answer:** A
**Explanation:** The APPLY MASKING POLICY privilege permits binding a policy to a column. This privilege is separate from policy creation and enables delegated policy management without granting full object ownership.

**Q221.** A secure view is created with the SECURE keyword. Which behavior distinguishes it from a regular view?
A. Query results are cached indefinitely
B. View definition is hidden from unauthorized roles
C. View executes with owner's privileges only
D. View cannot reference masked columns
**Answer:** B
**Explanation:** Secure views obscure metadata, including the view definition, from roles lacking explicit privileges. This prevents unauthorized users from inferring sensitive logic or underlying table structures.

**Q222.** Which authentication policy setting enforces MFA for specific user groups?
A. MFA_ENFORCEMENT_LEVEL
B. AUTHENTICATION_METHODS
C. CONDITIONAL_MFA_RULES
D. USER_GROUP_MFA_MAPPING
**Answer:** B
**Explanation:** Authentication policies define allowed authentication methods per user or role. The AUTHENTICATION_METHODS parameter specifies whether password, SSO, key-pair, or MFA is required, enabling granular enforcement.

**Q223.** A row access policy is applied to a table. Which query type bypasses the policy by default?
A. SELECT with WHERE clause
B. INSERT operation
C. COPY INTO external stage
D. Policy evaluation occurs for all data access operations
**Answer:** D
**Explanation:** Row access policies evaluate during any operation that reads data, including SELECT, COPY INTO, and view references. Policies do not apply to write operations like INSERT or UPDATE.

**Q224.** Which privilege allows a role to create future grants on a schema?
A. CREATE GRANT
B. MANAGE FUTURE GRANTS
C. OWNERSHIP
D. APPLY GRANTS
**Answer:** C
**Explanation:** Only roles with OWNERSHIP on a schema can define future grants for objects within that schema. This restriction prevents unauthorized privilege delegation to subsequently created objects.

**Q225.** A user authenticates via key-pair authentication. Which parameter specifies the public key fingerprint?
A. RSA_PUBLIC_KEY
B. RSA_PUBLIC_KEY_FP
C. KEY_FINGERPRINT
D. PUBLIC_KEY_HASH
**Answer:** B
**Explanation:** The RSA_PUBLIC_KEY_FP parameter stores the fingerprint of the registered public key. Snowflake validates the signature against this fingerprint during authentication, ensuring key integrity.

**Q226.** Which system view lists active network policies applied to a user?
A. SNOWFLAKE.ACCOUNT_USAGE.NETWORK_POLICIES
B. INFORMATION_SCHEMA.NETWORK_POLICY_REFERENCES
C. SNOWFLAKE.CORE.USER_NETWORK_POLICIES
D. ACCOUNT_USAGE.USER_AUTHENTICATION_HISTORY
**Answer:** A
**Explanation:** The NETWORK_POLICIES view in ACCOUNT_USAGE schema documents policy definitions and assignments. Administrators reference this view to audit network-level access controls.

**Q227.** A masking policy references a secondary role. How does Snowflake evaluate role membership?
A. Only the primary role is considered
B. All active roles in the session are evaluated
C. Only roles granted with ADMIN OPTION are considered
D. Role membership is ignored; only user attributes apply
**Answer:** B
**Explanation:** Policy expressions can reference any role active in the session. Snowflake evaluates the logical expression against the complete set of enabled roles, supporting complex access scenarios.

**Q228.** Which privilege is required to create an authentication policy?
A. CREATE POLICY
B. CREATE AUTHENTICATION POLICY
C. MANAGE ACCOUNT
D. APPLY SECURITY POLICY
**Answer:** B
**Explanation:** Authentication policies require the CREATE AUTHENTICATION POLICY privilege at the account level. This privilege is typically restricted to security administrators due to its broad impact on access control.

**Q229.** A secure view references a table with a row access policy. How are the policies combined?
A. The view's policy overrides the table's policy
B. Both policies are evaluated; rows must satisfy both conditions
C. Only the table's policy applies; view policies are ignored
D. Policies are merged into a single logical expression
**Answer:** B
**Explanation:** Row access policies compose conjunctively. When multiple policies apply to a query, Snowflake evaluates all conditions, returning only rows that satisfy every applicable policy.

**Q230.** Which authentication method supports token-based access for server-to-server integrations?
A. Username/password
B. SAML SSO
C. OAuth client credentials flow
D. Key-pair authentication
**Answer:** C
**Explanation:** OAuth client credentials flow enables applications to obtain access tokens using client ID and secret. This pattern supports automated, credential-less authentication for service accounts and integrations.

**Q231.** A network policy includes an ALLOW_LIST parameter. Which format is valid for IP specification?
A. CIDR notation only
B. Single IP addresses only
C. CIDR notation or single IP addresses
D. Domain names only
**Answer:** C
**Explanation:** Network policies accept both CIDR blocks (e.g., 192.168.1.0/24) and individual IP addresses. This flexibility supports granular network segmentation and access control.

**Q232.** Which privilege allows a role to view the definition of a secure view?
A. SELECT on the view
B. OWNERSHIP of the view
C. REFERENCES privilege
D. VIEW DEFINITION privilege
**Answer:** B
**Explanation:** Secure views hide metadata from all roles except those with OWNERSHIP. Even users with SELECT privilege cannot view the underlying query definition, preserving logic confidentiality.

**Q233.** A row access policy uses CURRENT_ORGANIZATION_NAME(). Which context does this function provide?
A. The Snowflake account identifier
B. The organization account hierarchy name
C. The user's default role name
D. The network policy identifier
**Answer:** B
**Explanation:** CURRENT_ORGANIZATION_NAME() returns the organization-level account name in Snowflake's hierarchy. This enables policies that differentiate access based on organizational boundaries in multi-account deployments.

**Q234.** Which parameter in a security integration specifies the IdP logout URL for SAML?
A. SAML2_LOGOUT_URL
B. IDP_LOGOUT_ENDPOINT
C. SSO_LOGOUT_REDIRECT
D. ASSERTION_CONSUMER_URL
**Answer:** A
**Explanation:** The SAML2_LOGOUT_URL parameter configures the IdP endpoint for session termination. This enables single logout functionality across federated applications.

**Q235.** A masking policy applies different transformations based on role. Which SQL construct enables conditional logic?
A. CASE expression
B. IF statement
C. DECODE function
D. COALESCE clause
**Answer:** A
**Explanation:** Masking policy definitions support CASE expressions for conditional logic. This enables role-based or attribute-based masking rules within a single policy definition.

**Q236.** Which privilege is required to detach a row access policy from a table?
A. REMOVE POLICY
B. DETACH ROW ACCESS POLICY
C. APPLY ROW ACCESS POLICY
D. OWNERSHIP
**Answer:** D
**Explanation:** Detaching a row access policy requires OWNERSHIP of the table or the policy. This restriction prevents unauthorized removal of access controls that could expose sensitive data.

**Q237.** A user authenticates via OAuth. Which object stores the refresh token?
A. SECURITY INTEGRATION
B. USER PROFILE
C. SESSION CONTEXT
D. OAUTH REFRESH STORE
**Answer:** C
**Explanation:** Refresh tokens are maintained in session context during the authentication flow. Snowflake does not persist refresh tokens in user profiles or integration objects, reducing long-term exposure risk.

**Q238.** Which system view documents privilege grants on database objects?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.OBJECT_PRIVILEGES
C. SNOWFLAKE.CORE.ACCESS_HISTORY
D. ACCOUNT_USAGE.PRIVILEGE_AUDIT
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE provides a comprehensive audit of privilege assignments across roles and objects. This view supports compliance reporting and access reviews.

**Q239.** A row access policy references a tag value. Which function retrieves tag metadata?
A. GET_TAG()
B. SYSTEM$GET_TAG()
C. TAG_VALUE()
D. OBJECT_TAG()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG() retrieves tag values assigned to objects. Row access policies can leverage this function to implement tag-based access controls, enabling classification-driven security.

**Q240.** Which authentication policy parameter enforces password complexity requirements?
A. PASSWORD_POLICY
B. MIN_PASSWORD_LENGTH
C. AUTHENTICATION_METHODS
D. MFA_ENFORCEMENT
**Answer:** A
**Explanation:** Authentication policies reference separate password policy objects via the PASSWORD_POLICY parameter. This modular design allows independent management of complexity rules and authentication methods.

**Q241.** A secure view is queried by a role without SELECT privilege on the base table. What occurs?
A. The query succeeds if the role has SELECT on the view
B. The query fails due to insufficient base table privileges
C. The query succeeds with owner's privileges if the view is secure
D. The query returns empty results
**Answer:** A
**Explanation:** Secure views execute with the owner's privileges for base object access. A role with SELECT on the secure view can query it without direct privileges on underlying tables, enabling controlled data exposure.

**Q242.** Which privilege allows a role to modify a network policy?
A. ALTER NETWORK POLICY
B. MANAGE NETWORK POLICIES
C. OWNERSHIP
D. APPLY NETWORK RULES
**Answer:** C
**Explanation:** Network policy modification requires OWNERSHIP of the policy object. This restriction ensures that only authorized administrators can change network-level access controls.

**Q243.** A masking policy is applied to a column in a view. Which statement accurately describes evaluation order?
A. Base table masking applies first, then view masking
B. View masking overrides base table masking
C. Only the most recently applied policy is evaluated
D. Both policies are evaluated; the view's policy takes precedence for that column
**Answer:** D
**Explanation:** When multiple masking policies apply to the same column through different object layers, Snowflake evaluates the policy attached to the queried object. View-level policies take precedence for columns accessed through that view.

**Q244.** Which authentication method supports Just-In-Time user provisioning?
A. Username/password
B. SAML SSO with SCIM
C. Key-pair authentication
D. OAuth with external provider
**Answer:** B
**Explanation:** SAML integrations combined with SCIM enable automatic user and role provisioning upon first authentication. This reduces administrative overhead for large user populations.

**Q245.** A row access policy uses a subquery to reference another table. Which limitation applies?
A. Subqueries are not supported in row access policies
B. Subqueries must reference only system tables
C. Subqueries cannot reference columns with masking policies
D. Subqueries must be correlated to the main query
**Answer:** A
**Explanation:** Row access policy expressions do not support subqueries. Policy logic must use scalar functions, system context functions, and direct column references to ensure predictable evaluation performance.

**Q246.** Which privilege is required to create a tag-based row access policy?
A. CREATE TAG
B. CREATE ROW ACCESS POLICY
C. APPLY TAG
D. MANAGE TAGS
**Answer:** B
**Explanation:** Creating row access policies requires CREATE ROW ACCESS POLICY privilege. Tag references within the policy do not require additional privileges beyond those needed to read tag metadata.

**Q247.** A user authenticates via SAML. Which parameter maps IdP attributes to Snowflake roles?
A. ROLE_MAPPING
B. SAML2_ROLE_MAPPINGS
C. ASSERTION_ROLE_BINDINGS
D. IDP_ATTRIBUTE_ROLES
**Answer:** B
**Explanation:** The SAML2_ROLE_MAPPINGS parameter defines how IdP assertion attributes translate to Snowflake roles. This enables dynamic role assignment based on external identity attributes.

**Q248.** Which system view lists masking policy applications across columns?
A. ACCOUNT_USAGE.MASKING_POLICIES
B. INFORMATION_SCHEMA.MASKING_POLICY_REFERENCES
C. SNOWFLAKE.CORE.COLUMN_MASKING_HISTORY
D. ACCOUNT_USAGE.POLICY_AUDIT_LOG
**Answer:** B
**Explanation:** MASKING_POLICY_REFERENCES in INFORMATION_SCHEMA documents which columns have masking policies applied. This view supports impact analysis and compliance auditing.

**Q249.** A network policy is applied to a user. Which connection type is affected?
A. Only Snowsight web interface connections
B. Only programmatic connections via drivers
C. All connections through public endpoints
D. Only connections from untrusted networks
**Answer:** C
**Explanation:** User-level network policies apply to all public endpoint connections regardless of client type. Private connectivity options require separate VPC-level controls.

**Q250.** Which privilege allows a role to view access history for a table?
A. SELECT on ACCESS_HISTORY
B. MONITOR USAGE
C. VIEW AUDIT LOG
D. ACCESS HISTORY PRIVILEGE
**Answer:** B
**Explanation:** The MONITOR USAGE privilege enables access to ACCOUNT_USAGE views, including ACCESS_HISTORY. This privilege supports auditing without granting data access privileges.

**Q251.** A masking policy uses a UDF to transform data. Which requirement applies to the UDF?
A. The UDF must be secure
B. The UDF must be owned by the policy creator
C. The UDF must be immutable and deterministic
D. The UDF must reference only system functions
**Answer:** C
**Explanation:** Masking policy UDFs must be immutable and deterministic to ensure consistent evaluation. Non-deterministic functions could produce inconsistent results, violating policy enforcement guarantees.

**Q252.** Which authentication policy setting limits login attempts to prevent brute force attacks?
A. MAX_LOGIN_ATTEMPTS
B. LOCKOUT_DURATION
C. FAILED_LOGIN_ATTEMPTS
D. AUTHENTICATION_RETRY_LIMIT
**Answer:** C
**Explanation:** The FAILED_LOGIN_ATTEMPTS parameter in password policies specifies the threshold for account lockout. This mitigates brute force attacks by temporarily disabling authentication after repeated failures.

**Q253.** A row access policy references a session variable. Which function retrieves custom session context?
A. GETVARIABLE()
B. SYSTEM$GET_SESSION_VARIABLE()
C. SESSION_CONTEXT()
D. CURRENT_SETTING()
**Answer:** B
**Explanation:** SYSTEM$GET_SESSION_VARIABLE() retrieves custom session variables set via SET commands. Row access policies can leverage this for dynamic, session-specific access controls.

**Q254.** Which privilege is required to create a future grant on tables in a schema?
A. CREATE TABLE
B. CREATE FUTURE GRANT
C. OWNERSHIP on schema
D. APPLY GRANTS
**Answer:** C
**Explanation:** Future grants require OWNERSHIP of the parent object (schema). This ensures that only authorized roles can define privilege inheritance for subsequently created objects.

**Q255.** A secure view includes a WHERE clause with sensitive logic. Which behavior protects this logic?
A. Query results are encrypted at rest
B. View definition is hidden from unauthorized roles
C. Query execution uses owner's warehouse only
D. Sensitive predicates are evaluated server-side only
**Answer:** B
**Explanation:** Secure views obscure the view definition metadata. Unauthorized roles cannot inspect the underlying SQL logic, protecting sensitive filtering or transformation rules.

**Q256.** Which authentication method supports certificate-based mutual TLS?
A. Username/password
B. SAML SSO
C. Key-pair authentication with mTLS
D. OAuth with client certificates
**Answer:** D
**Explanation:** OAuth integrations can be configured with client certificate requirements, enabling mutual TLS authentication. This provides strong cryptographic verification for service-to-service connections.

**Q257.** A row access policy is applied to a table with an external stage. Which operation triggers policy evaluation?
A. COPY INTO stage
B. COPY FROM stage
C. PUT command
D. GET command
**Answer:** B
**Explanation:** Row access policies evaluate during data read operations. COPY FROM stage reads data into a table, triggering policy evaluation. Write operations like COPY INTO stage do not evaluate row policies.

**Q258.** Which system view documents row access policy applications?
A. ACCOUNT_USAGE.ROW_ACCESS_POLICIES
B. INFORMATION_SCHEMA.ROW_ACCESS_POLICY_REFERENCES
C. SNOWFLAKE.CORE.POLICY_EVALUATION_LOG
D. ACCOUNT_USAGE.ACCESS_CONTROL_AUDIT
**Answer:** B
**Explanation:** ROW_ACCESS_POLICY_REFERENCES in INFORMATION_SCHEMA lists tables and columns with applied row access policies. This view supports governance audits and impact analysis.

**Q259.** A masking policy references CURRENT_ACCOUNT(). Which context does this function provide?
A. The Snowflake account locator
B. The organization account name
C. The user's default warehouse
D. The network policy identifier
**Answer:** A
**Explanation:** CURRENT_ACCOUNT() returns the account locator (e.g., xy12345). This enables policies that differentiate behavior based on account identity in multi-account deployments.

**Q260.** Which privilege allows a role to create a security integration for OAuth?
A. CREATE INTEGRATION
B. CREATE SECURITY INTEGRATION
C. MANAGE OAUTH
D. APPLY AUTHENTICATION POLICY
**Answer:** B
**Explanation:** Creating OAuth security integrations requires CREATE SECURITY INTEGRATION privilege. This privilege is typically restricted to security administrators to prevent unauthorized authentication configuration.

**Q261.** A user authenticates via key-pair. Which parameter specifies key rotation requirements?
A. KEY_ROTATION_INTERVAL
B. RSA_PUBLIC_KEY_2
C. KEY_EXPIRATION_DAYS
D. AUTHENTICATION_KEY_POLICY
**Answer:** B
**Explanation:** The RSA_PUBLIC_KEY_2 parameter enables key rotation by registering a secondary public key. Administrators can update the primary key without service interruption by pre-registering the replacement.

**Q262.** Which authentication policy parameter enforces session timeout?
A. SESSION_TIMEOUT_MINUTES
B. MAX_SESSION_DURATION
C. IDLE_SESSION_TIMEOUT
D. AUTHENTICATION_SESSION_LIMIT
**Answer:** C
**Explanation:** The IDLE_SESSION_TIMEOUT parameter in authentication policies specifies the inactivity period before automatic session termination. This reduces exposure from unattended sessions.

**Q263.** A row access policy uses a tag to determine visibility. Which privilege is required to read tag values in policy evaluation?
A. READ TAG
B. VIEW TAG METADATA
C. No additional privilege; tag metadata is accessible to policy evaluation
D. APPLY TAG POLICY
**Answer:** C
**Explanation:** Tag metadata is accessible to policy evaluation logic without additional privileges. The policy executes with sufficient context to retrieve tag values assigned to objects.

**Q264.** Which system view lists authentication events for audit purposes?
A. ACCOUNT_USAGE.AUTHENTICATION_HISTORY
B. INFORMATION_SCHEMA.LOGIN_AUDIT
C. SNOWFLAKE.CORE.USER_AUTH_EVENTS
D. ACCOUNT_USAGE.SECURITY_EVENT_LOG
**Answer:** A
**Explanation:** AUTHENTICATION_HISTORY in ACCOUNT_USAGE documents login attempts, methods, and outcomes. This view supports security monitoring and compliance reporting.

**Q265.** A masking policy applies to a VARIANT column. Which limitation applies?
A. Masking policies cannot be applied to VARIANT columns
B. Only top-level keys in VARIANT can be masked
C. Masking requires explicit path specification for nested keys
D. VARIANT columns must be flattened before masking
**Answer:** C
**Explanation:** Masking policies on VARIANT columns require explicit path expressions (e.g., col:field) to target specific nested keys. Whole-column masking is not supported for semi-structured data.

**Q266.** Which privilege allows a role to modify an authentication policy?
A. ALTER AUTHENTICATION POLICY
B. MANAGE AUTHENTICATION
C. OWNERSHIP
D. APPLY SECURITY SETTINGS
**Answer:** C
**Explanation:** Authentication policy modification requires OWNERSHIP of the policy object. This restriction ensures that only authorized administrators can change authentication requirements.

**Q267.** A secure view references a table with change tracking enabled. Which behavior applies?
A. Change tracking metadata is hidden from secure view queries
B. Streams on the base table capture changes from secure view queries
C. Change tracking operates independently of view security
D. Secure views disable change tracking for performance
**Answer:** C
**Explanation:** Change tracking operates at the base table level, independent of view security. Streams capture DML operations on the table regardless of whether queries access it through secure views.

**Q268.** Which authentication method supports token exchange for delegated access?
A. Username/password
B. SAML assertion exchange
C. OAuth token exchange
D. Key-pair delegation
**Answer:** C
**Explanation:** OAuth token exchange (RFC 8693) enables applications to obtain tokens on behalf of users or other services. This supports complex delegation scenarios in microservice architectures.

**Q269.** A row access policy references a user attribute from the IdP. Which function retrieves SAML assertion attributes?
A. GET_SAML_ATTRIBUTE()
B. SYSTEM$GET_SAML_ATTRIBUTE()
C. IDP_ATTRIBUTE()
D. ASSERTION_VALUE()
**Answer:** B
**Explanation:** SYSTEM$GET_SAML_ATTRIBUTE() retrieves attributes from the authenticated user's SAML assertion. This enables policies that leverage external identity metadata for access decisions.

**Q270.** Which privilege is required to create a network policy?
A. CREATE NETWORK POLICY
B. CREATE SECURITY INTEGRATION
C. MANAGE ACCOUNT
D. APPLY NETWORK RULES
**Answer:** A
**Explanation:** Creating network policies requires the CREATE NETWORK POLICY privilege at the account level. This privilege is typically restricted to network security administrators.

**Q271.** A masking policy uses a conditional expression with CURRENT_IP(). Which context does this function provide?
A. The user's home network IP
B. The client IP address for the current session
C. The Snowflake service endpoint IP
D. The IdP server IP address
**Answer:** B
**Explanation:** CURRENT_IP() returns the client IP address for the active session. Masking policies can leverage this for location-based or network-based data protection rules.

**Q272.** Which system view documents privilege grants to users?
A. ACCOUNT_USAGE.GRANTS_TO_USERS
B. INFORMATION_SCHEMA.USER_PRIVILEGES
C. SNOWFLAKE.CORE.USER_ACCESS_LOG
D. ACCOUNT_USAGE.USER_GRANT_AUDIT
**Answer:** A
**Explanation:** GRANTS_TO_USERS in ACCOUNT_USAGE provides a comprehensive audit of direct privilege assignments to users. This view supports user-level access reviews and compliance reporting.

**Q273.** A row access policy is applied to a table used in a materialized view. Which behavior applies?
A. Materialized view refresh bypasses row access policies
B. Row access policies evaluate during materialized view queries
C. Materialized views cannot reference tables with row access policies
D. Policies apply only to base table queries, not materialized view queries
**Answer:** B
**Explanation:** Row access policies evaluate during any query that reads data, including materialized view references. The policy ensures consistent row-level filtering regardless of query path.

**Q274.** Which authentication policy parameter enforces password history to prevent reuse?
A. PASSWORD_HISTORY_COUNT
B. MIN_PASSWORD_AGE_DAYS
C. PREVENT_PASSWORD_REUSE
D. PASSWORD_ROTATION_POLICY
**Answer:** A
**Explanation:** The PASSWORD_HISTORY_COUNT parameter in password policies specifies how many previous passwords are remembered to prevent reuse. This strengthens credential security over time.

**Q275.** A masking policy references a secondary database. Which privilege is required for policy evaluation?
A. USAGE on the secondary database
B. SELECT on referenced tables
C. No additional privilege; policy evaluation has implicit access
D. REFERENCES privilege on the database
**Answer:** A
**Explanation:** Masking policies that reference objects in other databases require USAGE privilege on those databases for policy evaluation. Without this, policy execution fails with permission errors.

**Q276.** Which privilege allows a role to view the evaluation results of a row access policy?
A. VIEW POLICY RESULTS
B. MONITOR POLICY
C. No privilege; policy evaluation is transparent
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Row access policy evaluation is transparent to authorized queries. Users with sufficient privileges to query the table see filtered results without explicit visibility into policy logic or evaluation details.

**Q277.** A secure view includes a JOIN with a dimension table. Which privilege model applies?
A. The role needs SELECT on both base and dimension tables
B. The role needs SELECT only on the secure view; owner privileges cover joins
C. The role needs REFERENCES privilege on joined tables
D. Secure views cannot include joins with external tables
**Answer:** B
**Explanation:** Secure views execute with the owner's privileges for all referenced objects. A role with SELECT on the secure view can access joined data without direct privileges on underlying tables.

**Q278.** Which authentication method supports hardware security module (HSM) backed keys?
A. Username/password
B. SAML SSO
C. Key-pair authentication with HSM
D. OAuth with external KMS
**Answer:** C
**Explanation:** Key-pair authentication can leverage HSM-backed private keys for enhanced security. The private key never leaves the HSM, reducing exposure risk during authentication operations.

**Q279.** A row access policy uses a time-based condition. Which function retrieves the current timestamp?
A. CURRENT_TIMESTAMP()
B. SYSDATE()
C. NOW()
D. All of the above
**Answer:** D
**Explanation:** Row access policies support standard timestamp functions including CURRENT_TIMESTAMP(), SYSDATE(), and NOW(). These enable time-based access controls such as business hours restrictions.

**Q280.** Which system view lists object tagging assignments for governance audits?
A. ACCOUNT_USAGE.TAG_REFERENCES
B. INFORMATION_SCHEMA.OBJECT_TAGS
C. SNOWFLAKE.CORE.TAG_AUDIT_LOG
D. ACCOUNT_USAGE.CLASSIFICATION_METADATA
**Answer:** A
**Explanation:** TAG_REFERENCES in ACCOUNT_USAGE documents tag assignments across database objects. This view supports classification-driven governance and compliance reporting.

**Q281.** A masking policy applies to a column with a default value. Which behavior applies during INSERT?
A. The mask applies to stored data
B. The mask applies only during SELECT; raw data is stored
C. Default values bypass masking policy evaluation
D. INSERT operations fail if masking policy is applied
**Answer:** B
**Explanation:** Masking policies transform data only during query result projection. Underlying storage retains raw values, ensuring that authorized roles with UNMASK privilege access complete data.

**Q282.** Which privilege allows a role to create a future grant on views in a schema?
A. CREATE VIEW
B. CREATE FUTURE GRANT
C. OWNERSHIP on schema
D. APPLY GRANTS
**Answer:** C
**Explanation:** Future grants for any object type require OWNERSHIP of the parent schema. This ensures centralized control over privilege inheritance for subsequently created objects.

**Q283.** A user authenticates via OAuth with refresh token rotation. Which behavior applies?
A. Refresh tokens are single-use; new tokens are issued on each use
B. Refresh tokens remain valid indefinitely
C. Refresh tokens expire after 24 hours regardless of use
D. Refresh tokens can be reused without limitation
**Answer:** A
**Explanation:** OAuth refresh token rotation issues a new refresh token each time the current one is used. This mitigates token theft by limiting the window of exposure for compromised tokens.

**Q284.** Which authentication policy parameter enforces geographic restrictions on login?
A. GEO_RESTRICTION_LIST
B. ALLOWED_COUNTRIES
C. NETWORK_POLICY reference
D. LOCATION_BASED_ACCESS
**Answer:** C
**Explanation:** Geographic restrictions are enforced via network policies that specify allowed IP ranges by region. Authentication policies reference network policies to implement location-based access controls.

**Q285.** A row access policy references a user's department attribute. Which source provides this metadata?
A. Snowflake user profile
B. SAML assertion or SCIM attribute
C. Tag assigned to the user object
D. Session variable set at login
**Answer:** B
**Explanation:** User attributes like department typically originate from the identity provider via SAML assertions or SCIM provisioning. Policies reference these external attributes for dynamic access decisions.

**Q286.** Which privilege is required to create a masking policy?
A. CREATE MASKING POLICY
B. CREATE POLICY
C. MANAGE DATA GOVERNANCE
D. APPLY SECURITY POLICY
**Answer:** A
**Explanation:** Creating masking policies requires the CREATE MASKING POLICY privilege at the schema or database level. This privilege enables delegated policy management without granting full administrative access.

**Q287.** A secure view is created in a shared database. Which behavior applies to data consumers?
A. Consumers see the view definition for transparency
B. Consumers cannot query the secure view without base table privileges
C. Consumers query the view with provider's privileges; definition remains hidden
D. Secure views cannot be included in data shares
**Answer:** C
**Explanation:** Secure views in shared databases execute with the provider's privileges. Consumers query the view without accessing underlying tables or view logic, enabling controlled data exposure.

**Q288.** Which authentication method supports passwordless authentication via WebAuthn?
A. Username/password
B. SAML SSO
C. Key-pair authentication
D. Snowflake native passwordless with WebAuthn
**Answer:** D
**Explanation:** Snowflake supports WebAuthn for passwordless authentication using platform authenticators like biometrics or security keys. This provides strong, phishing-resistant authentication without passwords.

**Q289.** A row access policy uses a subquery-like pattern with EXISTS. Which alternative is supported?
A. EXISTS with correlated subquery
B. IN with list of values from system function
C. JOIN with system context table
D. Row access policies do not support subqueries; use scalar functions only
**Answer:** D
**Explanation:** Row access policy expressions do not support subqueries or JOINs. Policy logic must use scalar functions, system context functions, and direct column references for predictable evaluation.

**Q290.** Which system view documents policy evaluation errors for troubleshooting?
A. ACCOUNT_USAGE.POLICY_EVALUATION_ERRORS
B. INFORMATION_SCHEMA.POLICY_AUDIT
C. SNOWFLAKE.CORE.POLICY_ERROR_LOG
D. Policy evaluation errors appear in QUERY_HISTORY with error codes
**Answer:** D
**Explanation:** Policy evaluation errors manifest as query failures logged in QUERY_HISTORY. Administrators analyze error codes and messages in this view to diagnose policy configuration issues.

**Q291.** A masking policy applies to a column used in a GROUP BY clause. Which behavior applies?
A. Grouping operates on masked values, potentially aggregating incorrectly
B. Grouping operates on raw values; masking applies only to output
C. Queries fail if masked columns are used in GROUP BY
D. Masking policies automatically disable for aggregation contexts
**Answer:** B
**Explanation:** Masking policies apply during result projection, after aggregation and grouping operations. GROUP BY uses raw column values, ensuring accurate aggregation while masking output for unauthorized roles.

**Q292.** Which privilege allows a role to view the definition of a row access policy?
A. SELECT on policy object
B. OWNERSHIP of the policy
C. VIEW POLICY DEFINITION
D. MONITOR POLICY
**Answer:** B
**Explanation:** Row access policy definitions are visible only to roles with OWNERSHIP. This protects sensitive access logic from exposure to unauthorized administrators or users.

**Q293.** A user authenticates via SAML with attribute-based role mapping. Which parameter defines the attribute name?
A. SAML2_ATTRIBUTE
B. ROLE_ATTRIBUTE_NAME
C. IDP_ROLE_MAPPING_ATTRIBUTE
D. ASSERTION_ROLE_KEY
**Answer:** B
**Explanation:** The ROLE_ATTRIBUTE_NAME parameter in SAML security integrations specifies which IdP assertion attribute contains role information. This enables dynamic role assignment based on external identity metadata.

**Q294.** Which authentication policy setting enforces periodic password rotation?
A. PASSWORD_MAX_AGE_DAYS
B. PASSWORD_ROTATION_INTERVAL
C. MIN_PASSWORD_CHANGE_FREQUENCY
D. AUTHENTICATION_EXPIRATION
**Answer:** A
**Explanation:** The PASSWORD_MAX_AGE_DAYS parameter in password policies specifies the maximum duration a password remains valid. This enforces periodic rotation to mitigate credential compromise risk.

**Q295.** A row access policy references a tag on the querying role. Which function retrieves role tag values?
A. GET_ROLE_TAG()
B. SYSTEM$GET_TAG_ON_ROLE()
C. ROLE_TAG_VALUE()
D. CURRENT_ROLE_TAG()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG_ON_ROLE() retrieves tag values assigned to roles. Row access policies can leverage this for role classification-based access controls, enabling dynamic filtering based on role metadata.

**Q296.** Which privilege is required to attach a row access policy to a table?
A. APPLY ROW ACCESS POLICY
B. ATTACH POLICY
C. MODIFY TABLE
D. MANAGE ROW ACCESS
**Answer:** A
**Explanation:** The APPLY ROW ACCESS POLICY privilege permits binding a policy to a table. This privilege is separate from policy creation and enables delegated policy management without granting full object ownership.

**Q297.** A secure view includes a UNION with a base table query. Which privilege model applies?
A. The role needs SELECT on all UNION sources
B. The role needs SELECT only on the secure view; owner privileges cover all sources
C. Secure views cannot include UNION operations
D. UNION sources must all be secure views
**Answer:** B
**Explanation:** Secure views execute with the owner's privileges for all referenced objects. A role with SELECT on the secure view can access UNION results without direct privileges on underlying tables.

**Q298.** Which authentication method supports just-in-time role assignment based on group membership?
A. Username/password
B. SAML SSO with group attribute mapping
C. Key-pair authentication
D. OAuth with scope-based roles
**Answer:** B
**Explanation:** SAML integrations can map IdP group attributes to Snowflake roles via SAML2_ROLE_MAPPINGS. This enables automatic role assignment upon authentication without manual provisioning.

**Q299.** A masking policy uses a UDF that references an external function. Which requirement applies?
A. External functions cannot be used in masking policies
B. The external function must be secure and owned by the policy creator
C. Masking policies cannot call external functions due to latency constraints
D. External functions require additional UNMASK privilege
**Answer:** A
**Explanation:** Masking policy definitions cannot reference external functions. Policy logic must use built-in functions, UDFs (with restrictions), and system context functions to ensure deterministic, low-latency evaluation.

**Q300.** Which system view documents the evaluation context for row access policies during query execution?
A. ACCOUNT_USAGE.ROW_ACCESS_EVALUATION
B. INFORMATION_SCHEMA.POLICY_CONTEXT_LOG
C. SNOWFLAKE.CORE.ACCESS_POLICY_TRACE
D. Row access policy evaluation context is not logged; use QUERY_HISTORY for audit
**Answer:** D
**Explanation:** Snowflake does not log detailed policy evaluation context. Administrators audit query execution via QUERY_HISTORY and ACCESS_HISTORY, inferring policy impact from result sets and access patterns.

---

**PART B: DATA GOVERNANCE, MASKING, ROW ACCESS, AND AUDITING (Q301–Q400)**

**Q301.** A masking policy applies different transformations based on data classification tags. Which function retrieves tag values for policy logic?
A. GET_TAG()
B. SYSTEM$GET_TAG()
C. TAG_METADATA()
D. OBJECT_CLASSIFICATION()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG() retrieves tag values assigned to database objects. Masking policies can leverage this function to implement classification-driven masking rules, enabling dynamic protection based on data sensitivity labels.

**Q302.** Which privilege allows a role to view access history for a specific table?
A. SELECT on ACCESS_HISTORY
B. MONITOR USAGE on the table
C. VIEW AUDIT LOG
D. ACCESS HISTORY PRIVILEGE
**Answer:** B
**Explanation:** The MONITOR USAGE privilege on a table enables access to ACCESS_HISTORY entries for that object. This supports targeted auditing without granting broad account-level monitoring privileges.

**Q303.** A row access policy uses a time-based window to restrict data visibility. Which function provides timezone-aware timestamp evaluation?
A. CURRENT_TIMESTAMP()
B. CONVERT_TIMEZONE()
C. TIMESTAMP_TZ_FROM_PARTS()
D. All of the above
**Answer:** D
**Explanation:** Row access policies support standard timestamp functions including timezone conversion utilities. This enables policies that enforce business hours or regional access windows with proper timezone handling.

**Q304.** Which system view lists all masking policies applied to columns in a database?
A. ACCOUNT_USAGE.MASKING_POLICIES
B. INFORMATION_SCHEMA.MASKING_POLICY_REFERENCES
C. SNOWFLAKE.CORE.COLUMN_MASKING_REGISTRY
D. ACCOUNT_USAGE.DATA_PROTECTION_AUDIT
**Answer:** B
**Explanation:** MASKING_POLICY_REFERENCES in INFORMATION_SCHEMA documents column-level masking policy assignments. This view supports governance audits and impact analysis for policy changes.

**Q305.** A masking policy applies to a column with a CHECK constraint. Which behavior applies during INSERT?
A. The mask validates against the constraint before storage
B. Constraints evaluate raw values; masking applies only during SELECT
C. INSERT operations fail if masked values violate constraints
D. Masking policies automatically disable constraint evaluation
**Answer:** B
**Explanation:** Constraints evaluate raw data values during INSERT operations. Masking policies transform data only during query result projection, ensuring that data integrity rules operate on complete, unmasked values.

**Q306.** Which privilege is required to detach a masking policy from a column?
A. REMOVE MASKING POLICY
B. DETACH MASKING POLICY
C. APPLY MASKING POLICY
D. OWNERSHIP on the table or policy
**Answer:** D
**Explanation:** Detaching a masking policy requires OWNERSHIP of either the table or the policy object. This restriction prevents unauthorized removal of data protection controls that could expose sensitive information.

**Q307.** A row access policy references a user's clearance level stored in a tag. Which evaluation order applies?
A. Tag values are evaluated before policy logic
B. Policy logic retrieves tag values during expression evaluation
C. Tag metadata is cached at policy creation; runtime changes are ignored
D. Tags cannot be referenced in row access policies
**Answer:** B
**Explanation:** Row access policies evaluate expressions at query time, retrieving current tag values during execution. This enables dynamic access controls that respond to tag updates without policy redeployment.

**Q308.** Which system view documents the lineage of data accessed through secure views?
A. ACCOUNT_USAGE.OBJECT_DEPENDENCIES
B. INFORMATION_SCHEMA.VIEW_USAGE
C. SNOWFLAKE.CORE.SECURE_VIEW_AUDIT
D. ACCOUNT_USAGE.DATA_LINEAGE_LOG
**Answer:** A
**Explanation:** OBJECT_DEPENDENCIES in ACCOUNT_USAGE tracks references between database objects, including secure views and their base tables. This supports impact analysis and governance auditing for view-based data exposure.

**Q309.** A masking policy uses a conditional expression with CURRENT_ROLE(). How does role switching affect policy evaluation?
A. Policy evaluates against the user's default role only
B. Policy evaluates against the active session role at query time
C. Policy caches role context at policy creation; runtime changes are ignored
D. Role context is ignored; only user attributes apply
**Answer:** B
**Explanation:** Masking policies evaluate expressions in the context of the active session role. CURRENT_ROLE() returns the role executing the query, enabling dynamic masking based on role membership changes during a session.

**Q310.** Which privilege allows a role to create a tag-based masking policy?
A. CREATE TAG
B. CREATE MASKING POLICY
C. APPLY TAG
D. MANAGE TAGS
**Answer:** B
**Explanation:** Creating masking policies requires CREATE MASKING POLICY privilege. Tag references within the policy do not require additional privileges beyond those needed to read tag metadata during evaluation.

**Q311.** A row access policy is applied to a table used in a stream. Which behavior applies to change data capture?
A. Streams capture only rows visible to the stream owner's role
B. Streams capture all changes; row access policies filter query results
C. Row access policies prevent stream creation on protected tables
D. Streams bypass row access policies for performance
**Answer:** B
**Explanation:** Streams capture all DML changes at the storage layer, independent of row access policies. Policies filter query results during data consumption, ensuring that CDC pipelines receive complete change data while access controls govern visibility.

**Q312.** Which system view lists tag assignments for data classification audits?
A. ACCOUNT_USAGE.TAG_REFERENCES
B. INFORMATION_SCHEMA.OBJECT_TAGS
C. SNOWFLAKE.CORE.CLASSIFICATION_REGISTRY
D. ACCOUNT_USAGE.DATA_GOVERNANCE_LOG
**Answer:** A
**Explanation:** TAG_REFERENCES in ACCOUNT_USAGE documents tag assignments across database objects. This view supports classification-driven governance, compliance reporting, and impact analysis for tag-based policies.

**Q313.** A masking policy applies to a column with a foreign key constraint. Which behavior applies during JOIN operations?
A. Masked values participate in join conditions, potentially causing mismatches
B. Join conditions use raw values; masking applies only to output columns
C. Queries fail if masked columns are used in join predicates
D. Masking policies automatically disable for foreign key relationships
**Answer:** B
**Explanation:** Masking policies apply during result projection, after join evaluation. Foreign key relationships operate on raw column values, ensuring referential integrity while masking output for unauthorized roles.

**Q314.** Which privilege allows a role to view the evaluation history of a masking policy?
A. VIEW MASKING HISTORY
B. MONITOR POLICY
C. No privilege; masking evaluation is transparent
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Masking policy evaluation is transparent to authorized queries. Users with sufficient privileges to query the table see transformed results without explicit visibility into policy logic or evaluation details.

**Q315.** A row access policy references a secondary role's tag. Which function retrieves tag values for roles?
A. GET_ROLE_TAG()
B. SYSTEM$GET_TAG_ON_ROLE()
C. ROLE_TAG_METADATA()
D. CURRENT_ROLE_TAG()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG_ON_ROLE() retrieves tag values assigned to roles. Row access policies can leverage this for role classification-based access controls, enabling dynamic filtering based on role metadata.

**Q316.** Which system view documents privilege grants on masking policies?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.POLICY_PRIVILEGES
C. SNOWFLAKE.CORE.MASKING_POLICY_GRANTS
D. ACCOUNT_USAGE.POLICY_ACCESS_AUDIT
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE provides a comprehensive audit of privilege assignments, including privileges on masking policies. This view supports compliance reporting and access reviews.

**Q317.** A masking policy uses a UDF to hash sensitive values. Which requirement applies to ensure deterministic output?
A. The UDF must be marked IMMUTABLE
B. The UDF must reference only built-in functions
C. The UDF must be owned by the policy creator
D. The UDF must execute in the same warehouse as the query
**Answer:** A
**Explanation:** Masking policy UDFs must be marked IMMUTABLE to guarantee deterministic output. Non-deterministic functions could produce inconsistent results, violating policy enforcement guarantees and breaking query reproducibility.

**Q318.** Which privilege is required to create a future grant on masking policy applications?
A. CREATE FUTURE GRANT
B. APPLY MASKING POLICY
C. OWNERSHIP on schema
D. MANAGE MASKING GRANTS
**Answer:** C
**Explanation:** Future grants for any object type, including masking policy applications, require OWNERSHIP of the parent schema. This ensures centralized control over privilege inheritance for subsequently created objects.

**Q319.** A row access policy is applied to a table with a materialized view. Which behavior applies during view refresh?
A. Refresh operations bypass row access policies
B. Row access policies filter data during refresh based on refresh role context
C. Materialized views cannot reference tables with row access policies
D. Policies apply only to direct table queries, not materialized view operations
**Answer:** B
**Explanation:** Materialized view refresh executes with the privileges of the role triggering the refresh. Row access policies evaluate during refresh, filtering data based on that role's context, ensuring consistent row-level protection.

**Q320.** Which system view lists the evaluation context for masking policies during query execution?
A. ACCOUNT_USAGE.MASKING_EVALUATION
B. INFORMATION_SCHEMA.POLICY_CONTEXT_LOG
C. SNOWFLAKE.CORE.MASKING_TRACE
D. Masking policy evaluation context is not logged; use QUERY_HISTORY for audit
**Answer:** D
**Explanation:** Snowflake does not log detailed masking policy evaluation context. Administrators audit query execution via QUERY_HISTORY, inferring policy impact from result sets and access patterns.

**Q321.** A masking policy applies to a column used in a WHERE clause. Which behavior applies?
A. Filtering operates on masked values, potentially excluding relevant rows
B. Filtering operates on raw values; masking applies only to output
C. Queries fail if masked columns are used in predicates
D. Masking policies automatically disable for filter contexts
**Answer:** B
**Explanation:** Masking policies apply during result projection, after predicate evaluation. WHERE clauses use raw column values, ensuring accurate filtering while masking output for unauthorized roles.

**Q322.** Which privilege allows a role to view the definition of a masking policy?
A. SELECT on policy object
B. OWNERSHIP of the policy
C. VIEW POLICY DEFINITION
D. MONITOR POLICY
**Answer:** B
**Explanation:** Masking policy definitions are visible only to roles with OWNERSHIP. This protects sensitive transformation logic from exposure to unauthorized administrators or users.

**Q323.** A row access policy uses a tag on the table to determine visibility rules. Which evaluation timing applies?
A. Tag values are resolved at policy creation time
B. Tag values are resolved at query execution time
C. Tag values are cached for 24 hours to optimize performance
D. Tags cannot be referenced in row access policies
**Answer:** B
**Explanation:** Row access policies evaluate expressions at query time, retrieving current tag values during execution. This enables dynamic access controls that respond to tag updates without policy redeployment.

**Q324.** Which system view documents the application of row access policies to tables?
A. ACCOUNT_USAGE.ROW_ACCESS_POLICIES
B. INFORMATION_SCHEMA.ROW_ACCESS_POLICY_REFERENCES
C. SNOWFLAKE.CORE.POLICY_APPLICATION_LOG
D. ACCOUNT_USAGE.ACCESS_CONTROL_REGISTRY
**Answer:** B
**Explanation:** ROW_ACCESS_POLICY_REFERENCES in INFORMATION_SCHEMA lists tables and columns with applied row access policies. This view supports governance audits and impact analysis for policy changes.

**Q325.** A masking policy applies to a VARIANT column with nested keys. Which syntax targets a specific nested field?
A. column.field
B. column:field
C. column->field
D. column['field']
**Answer:** B
**Explanation:** Masking policies on VARIANT columns use colon notation (e.g., col:field) to target specific nested keys. This syntax aligns with Snowflake's semi-structured data querying conventions.

**Q326.** Which privilege is required to create a row access policy that references tags?
A. CREATE TAG
B. CREATE ROW ACCESS POLICY
C. READ TAG METADATA
D. MANAGE TAG POLICIES
**Answer:** B
**Explanation:** Creating row access policies requires CREATE ROW ACCESS POLICY privilege. Tag references within the policy do not require additional privileges beyond those needed to read tag metadata during evaluation.

**Q327.** A row access policy is applied to a table used in an external function. Which behavior applies?
A. External functions bypass row access policies for performance
B. Row access policies filter data before external function invocation
C. External functions cannot reference tables with row access policies
D. Policies apply only to SQL queries, not external function calls
**Answer:** B
**Explanation:** Row access policies evaluate during data retrieval, before external function invocation. This ensures that only authorized rows are passed to external processing, maintaining data protection boundaries.

**Q328.** Which system view lists the privilege grants required to apply masking policies?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.MASKING_POLICY_PRIVILEGES
C. SNOWFLAKE.CORE.POLICY_GRANT_AUDIT
D. ACCOUNT_USAGE.MASKING_ACCESS_LOG
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including APPLY MASKING POLICY grants. This view supports access reviews and compliance auditing for policy management.

**Q329.** A masking policy uses a conditional expression with CURRENT_USER(). How does user context affect policy evaluation?
A. Policy evaluates against the user's default role only
B. Policy evaluates against the authenticated user identity, independent of role
C. Policy caches user context at policy creation; runtime changes are ignored
D. User context is ignored; only role attributes apply
**Answer:** B
**Explanation:** CURRENT_USER() returns the authenticated user identity for the session. Masking policies can leverage this for user-specific transformations, independent of role membership changes during a session.

**Q330.** Which privilege allows a role to create a tag-based row access policy?
A. CREATE TAG
B. CREATE ROW ACCESS POLICY
C. APPLY TAG
D. MANAGE TAGS
**Answer:** B
**Explanation:** Creating row access policies requires CREATE ROW ACCESS POLICY privilege. Tag references within the policy do not require additional privileges beyond those needed to read tag metadata during evaluation.

**Q331.** A row access policy references a user attribute from SCIM provisioning. Which function retrieves SCIM attribute values?
A. GET_SCIM_ATTRIBUTE()
B. SYSTEM$GET_SCIM_ATTRIBUTE()
C. SCIM_USER_ATTRIBUTE()
D. IDP_USER_METADATA()
**Answer:** B
**Explanation:** SYSTEM$GET_SCIM_ATTRIBUTE() retrieves attributes provisioned via SCIM integration. Row access policies can leverage this for dynamic access decisions based on externally managed user metadata.

**Q332.** Which system view documents the evaluation performance of row access policies?
A. ACCOUNT_USAGE.POLICY_PERFORMANCE_METRICS
B. INFORMATION_SCHEMA.POLICY_EXECUTION_STATS
C. SNOWFLAKE.CORE.POLICY_OPTIMIZATION_LOG
D. Policy evaluation performance is not separately logged; use QUERY_HISTORY for timing analysis
**Answer:** D
**Explanation:** Snowflake does not log detailed policy evaluation performance metrics. Administrators analyze query execution timing in QUERY_HISTORY to infer policy impact on performance.

**Q333.** A masking policy applies to a column with a unique constraint. Which behavior applies during INSERT?
A. The mask validates uniqueness before storage
B. Uniqueness constraints evaluate raw values; masking applies only during SELECT
C. INSERT operations fail if masked values violate uniqueness
D. Masking policies automatically disable uniqueness evaluation
**Answer:** B
**Explanation:** Uniqueness constraints evaluate raw data values during INSERT operations. Masking policies transform data only during query result projection, ensuring that data integrity rules operate on complete, unmasked values.

**Q334.** Which privilege allows a role to view the application history of a row access policy?
A. VIEW POLICY HISTORY
B. MONITOR POLICY
C. No privilege; policy application is transparent
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Row access policy application is transparent to authorized queries. Users with sufficient privileges to query the table see filtered results without explicit visibility into policy application history.

**Q335.** A row access policy uses a tag on the querying user's role. Which function retrieves role tag values?
A. GET_ROLE_TAG()
B. SYSTEM$GET_TAG_ON_ROLE()
C. ROLE_TAG_VALUE()
D. CURRENT_ROLE_TAG()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG_ON_ROLE() retrieves tag values assigned to roles. Row access policies can leverage this for role classification-based access controls, enabling dynamic filtering based on role metadata.

**Q336.** Which system view lists the dependency chain for masking policies applied through views?
A. ACCOUNT_USAGE.OBJECT_DEPENDENCIES
B. INFORMATION_SCHEMA.MASKING_POLICY_LINEAGE
C. SNOWFLAKE.CORE.POLICY_DEPENDENCY_GRAPH
D. ACCOUNT_USAGE.MASKING_LINEAGE_LOG
**Answer:** A
**Explanation:** OBJECT_DEPENDENCIES in ACCOUNT_USAGE tracks references between database objects, including masking policies applied through views. This supports impact analysis and governance auditing for policy changes.

**Q337.** A masking policy uses a UDF that references a secure UDF. Which requirement applies?
A. Secure UDFs cannot be used in masking policies
B. The secure UDF must be owned by the policy creator
C. Masking policies cannot call secure UDFs due to privilege escalation risk
D. Secure UDFs require additional UNMASK privilege for policy evaluation
**Answer:** A
**Explanation:** Masking policy definitions cannot reference secure UDFs. Policy logic must use non-secure UDFs (with IMMUTABLE marking) and built-in functions to ensure predictable, auditable evaluation.

**Q338.** Which privilege is required to create a future grant on row access policy applications?
A. CREATE FUTURE GRANT
B. APPLY ROW ACCESS POLICY
C. OWNERSHIP on schema
D. MANAGE ROW ACCESS GRANTS
**Answer:** C
**Explanation:** Future grants for any object type, including row access policy applications, require OWNERSHIP of the parent schema. This ensures centralized control over privilege inheritance for subsequently created objects.

**Q339.** A row access policy is applied to a table used in a task. Which behavior applies during task execution?
A. Task execution bypasses row access policies for performance
B. Row access policies filter data based on the task owner's role context
C. Tasks cannot reference tables with row access policies
D. Policies apply only to interactive queries, not automated tasks
**Answer:** B
**Explanation:** Tasks execute with the privileges of the owner role. Row access policies evaluate during task queries, filtering data based on that role's context, ensuring consistent row-level protection for automated workloads.

**Q340.** Which system view lists the privilege grants required to apply row access policies?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.ROW_ACCESS_POLICY_PRIVILEGES
C. SNOWFLAKE.CORE.POLICY_GRANT_AUDIT
D. ACCOUNT_USAGE.ROW_ACCESS_LOG
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including APPLY ROW ACCESS POLICY grants. This view supports access reviews and compliance auditing for policy management.

**Q341.** A masking policy applies to a column used in an ORDER BY clause. Which behavior applies?
A. Sorting operates on masked values, potentially altering result order
B. Sorting operates on raw values; masking applies only to output
C. Queries fail if masked columns are used in ORDER BY
D. Masking policies automatically disable for sorting contexts
**Answer:** B
**Explanation:** Masking policies apply during result projection, after sorting operations. ORDER BY uses raw column values, ensuring accurate result ordering while masking output for unauthorized roles.

**Q342.** Which privilege allows a role to view the evaluation metrics of a masking policy?
A. VIEW MASKING METRICS
B. MONITOR POLICY
C. No privilege; masking evaluation metrics are not exposed
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Snowflake does not expose detailed masking policy evaluation metrics. Administrators infer policy impact through query performance analysis in QUERY_HISTORY and access patterns in ACCESS_HISTORY.

**Q343.** A row access policy references a tag on the table with hierarchical values. Which evaluation logic applies?
A. Tag values are evaluated as exact matches only
B. Tag values support hierarchical inheritance if defined in policy logic
C. Hierarchical tag evaluation requires custom UDFs
D. Tags cannot represent hierarchical relationships in policies
**Answer:** B
**Explanation:** Row access policies can implement hierarchical logic using CASE expressions or conditional functions that interpret tag value hierarchies. This enables classification-driven access controls with inheritance semantics.

**Q344.** Which system view documents the application of masking policies to views?
A. ACCOUNT_USAGE.MASKING_POLICIES
B. INFORMATION_SCHEMA.MASKING_POLICY_REFERENCES
C. SNOWFLAKE.CORE.VIEW_MASKING_REGISTRY
D. ACCOUNT_USAGE.MASKING_APPLICATION_LOG
**Answer:** B
**Explanation:** MASKING_POLICY_REFERENCES in INFORMATION_SCHEMA documents masking policy assignments to columns in both tables and views. This view supports governance audits and impact analysis for policy changes.

**Q345.** A masking policy applies to a column with a computed default value. Which behavior applies during INSERT?
A. The mask validates the computed default before storage
B. Default computation uses raw values; masking applies only during SELECT
C. INSERT operations fail if masked values conflict with defaults
D. Masking policies automatically disable default value computation
**Answer:** B
**Explanation:** Default value computation operates on raw data during INSERT operations. Masking policies transform data only during query result projection, ensuring that default logic functions correctly while masking output for unauthorized roles.

**Q346.** Which privilege is required to create a masking policy that references system context functions?
A. CREATE MASKING POLICY
B. USE SYSTEM FUNCTIONS
C. ACCESS CONTEXT METADATA
D. MANAGE SYSTEM POLICIES
**Answer:** A
**Explanation:** Creating masking policies requires CREATE MASKING POLICY privilege. System context functions like CURRENT_ROLE() or CURRENT_USER() are available to policy logic without additional privileges.

**Q347.** A row access policy is applied to a table used in a secure view. Which behavior applies to policy composition?
A. The view's policy overrides the table's policy
B. Both policies are evaluated; rows must satisfy both conditions
C. Only the table's policy applies; view policies are ignored
D. Policies are merged into a single logical expression
**Answer:** B
**Explanation:** Row access policies compose conjunctively. When multiple policies apply to a query, Snowflake evaluates all conditions, returning only rows that satisfy every applicable policy.

**Q348.** Which system view lists the evaluation context for masking policies applied through secure views?
A. ACCOUNT_USAGE.MASKING_EVALUATION
B. INFORMATION_SCHEMA.SECURE_VIEW_MASKING_LOG
C. SNOWFLAKE.CORE.MASKING_CONTEXT_TRACE
D. Masking policy evaluation context is not logged; use QUERY_HISTORY for audit
**Answer:** D
**Explanation:** Snowflake does not log detailed masking policy evaluation context. Administrators audit query execution via QUERY_HISTORY, inferring policy impact from result sets and access patterns.

**Q349.** A masking policy uses a conditional expression with CURRENT_IP(). Which context does this function provide?
A. The user's home network IP
B. The client IP address for the current session
C. The Snowflake service endpoint IP
D. The IdP server IP address
**Answer:** B
**Explanation:** CURRENT_IP() returns the client IP address for the active session. Masking policies can leverage this for location-based or network-based data protection rules.

**Q350.** Which privilege allows a role to create a tag-based masking policy that references role tags?
A. CREATE TAG
B. CREATE MASKING POLICY
C. APPLY TAG ON ROLE
D. MANAGE ROLE TAGS
**Answer:** B
**Explanation:** Creating masking policies requires CREATE MASKING POLICY privilege. Tag references within the policy, including role tags, do not require additional privileges beyond those needed to read tag metadata during evaluation.

**Q351.** A row access policy references a user's department from a tag on the user object. Which function retrieves user tag values?
A. GET_USER_TAG()
B. SYSTEM$GET_TAG_ON_USER()
C. USER_TAG_VALUE()
D. CURRENT_USER_TAG()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG_ON_USER() retrieves tag values assigned to user objects. Row access policies can leverage this for user classification-based access controls, enabling dynamic filtering based on user metadata.

**Q352.** Which system view documents the privilege grants required to create masking policies?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.POLICY_CREATION_PRIVILEGES
C. SNOWFLAKE.CORE.MASKING_POLICY_GRANTS
D. ACCOUNT_USAGE.POLICY_CREATION_AUDIT
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including CREATE MASKING POLICY grants. This view supports access reviews and compliance auditing for policy management.

**Q353.** A masking policy applies to a column with a generated column dependency. Which behavior applies?
A. Generated columns evaluate masked values, potentially causing inconsistencies
B. Generated columns use raw source values; masking applies only to output
C. Queries fail if masked columns are used in generated column expressions
D. Masking policies automatically disable for generated column contexts
**Answer:** B
**Explanation:** Generated columns compute values from raw source data during INSERT or UPDATE operations. Masking policies transform data only during query result projection, ensuring that generated logic functions correctly while masking output for unauthorized roles.

**Q354.** Which privilege allows a role to view the dependency chain for a row access policy?
A. VIEW POLICY DEPENDENCIES
B. MONITOR POLICY
C. No privilege; policy dependencies are visible via OBJECT_DEPENDENCIES
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Policy dependencies are documented in ACCOUNT_USAGE.OBJECT_DEPENDENCIES. Roles with MONITOR USAGE privilege can query this view to analyze policy impact without requiring special policy-specific privileges.

**Q355.** A row access policy uses a tag with time-based expiration. Which evaluation behavior applies?
A. Tag expiration is evaluated at policy creation time
B. Tag expiration is evaluated at query execution time
C. Tag expiration is cached for 1 hour to optimize performance
D. Tags cannot include time-based expiration logic
**Answer:** B
**Explanation:** Row access policies evaluate expressions at query time, retrieving current tag values and metadata during execution. This enables dynamic access controls that respond to tag expiration without policy redeployment.

**Q356.** Which system view lists the application of masking policies to external tables?
A. ACCOUNT_USAGE.MASKING_POLICIES
B. INFORMATION_SCHEMA.MASKING_POLICY_REFERENCES
C. SNOWFLAKE.CORE.EXTERNAL_TABLE_MASKING_LOG
D. Masking policies cannot be applied to external tables
**Answer:** D
**Explanation:** Masking policies cannot be directly applied to external table columns. Data protection for external tables requires alternative approaches such as secure views or pre-processing transformations.

**Q357.** A masking policy uses a UDF that references a stage. Which requirement applies?
A. UDFs cannot reference stages in masking policies
B. The stage must be owned by the policy creator
C. Masking policies cannot call UDFs that access external resources
D. Stage access requires additional UNMASK privilege
**Answer:** C
**Explanation:** Masking policy UDFs cannot reference external resources like stages. Policy logic must be self-contained, using only built-in functions and deterministic UDFs to ensure predictable, low-latency evaluation.

**Q358.** Which privilege is required to create a future grant on tag assignments for masking policies?
A. CREATE FUTURE GRANT
B. APPLY TAG
C. OWNERSHIP on schema
D. MANAGE TAG GRANTS
**Answer:** C
**Explanation:** Future grants for any object type, including tag assignments, require OWNERSHIP of the parent schema. This ensures centralized control over privilege inheritance for subsequently created objects.

**Q359.** A row access policy is applied to a table used in a streamlit application. Which behavior applies?
A. Streamlit applications bypass row access policies for performance
B. Row access policies filter data based on the authenticated user's context
C. Streamlit cannot reference tables with row access policies
D. Policies apply only to SQL queries, not application-layer access
**Answer:** B
**Explanation:** Row access policies evaluate during data retrieval, regardless of client application. Streamlit applications inherit the authenticated user's context, ensuring consistent row-level protection across access methods.

**Q360.** Which system view lists the privilege grants required to apply row access policies to views?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.ROW_ACCESS_POLICY_PRIVILEGES
C. SNOWFLAKE.CORE.VIEW_POLICY_GRANTS
D. ACCOUNT_USAGE.VIEW_ACCESS_LOG
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including APPLY ROW ACCESS POLICY grants on views. This view supports access reviews and compliance auditing for policy management.

**Q361.** A masking policy applies to a column used in a window function. Which behavior applies?
A. Window calculations operate on masked values, potentially altering results
B. Window calculations use raw values; masking applies only to output
C. Queries fail if masked columns are used in window functions
D. Masking policies automatically disable for window function contexts
**Answer:** B
**Explanation:** Masking policies apply during result projection, after window function evaluation. Window calculations use raw column values, ensuring accurate analytical results while masking output for unauthorized roles.

**Q362.** Which privilege allows a role to view the performance impact of a masking policy?
A. VIEW MASKING PERFORMANCE
B. MONITOR POLICY
C. No privilege; masking performance impact is not separately exposed
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Snowflake does not expose detailed masking policy performance metrics. Administrators infer policy impact through query execution timing in QUERY_HISTORY and resource consumption analysis.

**Q363.** A row access policy references a tag with multi-value assignments. Which evaluation logic applies?
A. Tag values are evaluated as exact matches only
B. Tag values support multi-value logic if defined in policy expressions
C. Multi-value tag evaluation requires custom UDFs
D. Tags cannot represent multi-value relationships in policies
**Answer:** B
**Explanation:** Row access policies can implement multi-value logic using IN clauses or conditional functions that interpret tag value lists. This enables classification-driven access controls with flexible membership semantics.

**Q364.** Which system view documents the application of row access policies to secure views?
A. ACCOUNT_USAGE.ROW_ACCESS_POLICIES
B. INFORMATION_SCHEMA.ROW_ACCESS_POLICY_REFERENCES
C. SNOWFLAKE.CORE.SECURE_VIEW_POLICY_LOG
D. ACCOUNT_USAGE.SECURE_VIEW_ACCESS_LOG
**Answer:** B
**Explanation:** ROW_ACCESS_POLICY_REFERENCES in INFORMATION_SCHEMA lists tables and views with applied row access policies. This view supports governance audits and impact analysis for policy changes across object types.

**Q365.** A masking policy applies to a column with a materialized computed value. Which behavior applies during refresh?
A. Materialized computations use masked values, potentially causing inconsistencies
B. Materialized computations use raw source values; masking applies only to output
C. Refresh operations fail if masked columns are used in computations
D. Masking policies automatically disable for materialized computation contexts
**Answer:** B
**Explanation:** Materialized computed values derive from raw source data during refresh operations. Masking policies transform data only during query result projection, ensuring that materialized logic functions correctly while masking output for unauthorized roles.

**Q366.** Which privilege is required to create a masking policy that references external authentication attributes?
A. CREATE MASKING POLICY
B. ACCESS EXTERNAL AUTH METADATA
C. USE IDP ATTRIBUTES
D. MANAGE FEDERATED POLICIES
**Answer:** A
**Explanation:** Creating masking policies requires CREATE MASKING POLICY privilege. External authentication attributes accessed via system functions like SYSTEM$GET_SAML_ATTRIBUTE() are available to policy logic without additional privileges.

**Q367.** A row access policy is applied to a table used in a Python UDF. Which behavior applies?
A. Python UDFs bypass row access policies for performance
B. Row access policies filter data before Python UDF invocation
C. Python UDFs cannot reference tables with row access policies
D. Policies apply only to SQL queries, not UDF-layer access
**Answer:** B
**Explanation:** Row access policies evaluate during data retrieval, before UDF invocation. This ensures that only authorized rows are passed to Python processing, maintaining data protection boundaries across execution contexts.

**Q368.** Which system view lists the evaluation context for row access policies applied through secure views?
A. ACCOUNT_USAGE.ROW_ACCESS_EVALUATION
B. INFORMATION_SCHEMA.SECURE_VIEW_POLICY_LOG
C. SNOWFLAKE.CORE.POLICY_CONTEXT_TRACE
D. Row access policy evaluation context is not logged; use QUERY_HISTORY for audit
**Answer:** D
**Explanation:** Snowflake does not log detailed policy evaluation context. Administrators audit query execution via QUERY_HISTORY, inferring policy impact from result sets and access patterns.

**Q369.** A masking policy uses a conditional expression with CURRENT_ORGANIZATION_NAME(). Which context does this function provide?
A. The Snowflake account locator
B. The organization account hierarchy name
C. The user's default role name
D. The network policy identifier
**Answer:** B
**Explanation:** CURRENT_ORGANIZATION_NAME() returns the organization-level account name in Snowflake's hierarchy. Masking policies can leverage this for organization-specific transformations in multi-account deployments.

**Q370.** Which privilege allows a role to create a tag-based masking policy that references table tags?
A. CREATE TAG
B. CREATE MASKING POLICY
C. APPLY TAG ON TABLE
D. MANAGE TABLE TAGS
**Answer:** B
**Explanation:** Creating masking policies requires CREATE MASKING POLICY privilege. Tag references within the policy, including table tags, do not require additional privileges beyond those needed to read tag metadata during evaluation.

**Q371.** A row access policy references a user's clearance level from a tag on the role. Which function retrieves role tag values?
A. GET_ROLE_TAG()
B. SYSTEM$GET_TAG_ON_ROLE()
C. ROLE_TAG_VALUE()
D. CURRENT_ROLE_TAG()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG_ON_ROLE() retrieves tag values assigned to roles. Row access policies can leverage this for role classification-based access controls, enabling dynamic filtering based on role metadata.

**Q372.** Which system view documents the privilege grants required to create row access policies?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.POLICY_CREATION_PRIVILEGES
C. SNOWFLAKE.CORE.ROW_ACCESS_POLICY_GRANTS
D. ACCOUNT_USAGE.POLICY_CREATION_AUDIT
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including CREATE ROW ACCESS POLICY grants. This view supports access reviews and compliance auditing for policy management.

**Q373.** A masking policy applies to a column with a foreign key that references a masked column. Which behavior applies during JOIN?
A. Both sides of the join use masked values, potentially causing mismatches
B. Join conditions use raw values; masking applies only to output columns
C. Queries fail if masked columns are used in foreign key relationships
D. Masking policies automatically disable for foreign key join contexts
**Answer:** B
**Explanation:** Masking policies apply during result projection, after join evaluation. Foreign key relationships operate on raw column values, ensuring referential integrity while masking output for unauthorized roles.

**Q374.** Which privilege allows a role to view the lineage of masking policy applications?
A. VIEW MASKING LINEAGE
B. MONITOR POLICY
C. No privilege; policy lineage is visible via OBJECT_DEPENDENCIES
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Policy lineage is documented in ACCOUNT_USAGE.OBJECT_DEPENDENCIES. Roles with MONITOR USAGE privilege can query this view to analyze policy impact without requiring special policy-specific privileges.

**Q375.** A row access policy uses a tag with conditional inheritance. Which evaluation behavior applies?
A. Tag inheritance is evaluated at policy creation time
B. Tag inheritance is evaluated at query execution time based on policy logic
C. Tag inheritance is cached for 24 hours to optimize performance
D. Tags cannot represent inheritance relationships in policies
**Answer:** B
**Explanation:** Row access policies evaluate expressions at query time, interpreting tag metadata and inheritance logic during execution. This enables dynamic access controls that respond to tag hierarchy changes without policy redeployment.

**Q376.** Which system view lists the application of masking policies to tasks?
A. ACCOUNT_USAGE.MASKING_POLICIES
B. INFORMATION_SCHEMA.MASKING_POLICY_REFERENCES
C. SNOWFLAKE.CORE.TASK_MASKING_LOG
D. Masking policies apply to task queries based on task owner context
**Answer:** D
**Explanation:** Masking policies evaluate during query execution regardless of client context. Tasks inherit the owner's role context, ensuring consistent masking behavior for automated workloads without separate policy application tracking.

**Q377.** A masking policy uses a UDF that references a sequence. Which requirement applies?
A. UDFs cannot reference sequences in masking policies
B. The sequence must be owned by the policy creator
C. Masking policies cannot call UDFs that access non-deterministic resources
D. Sequence access requires additional UNMASK privilege
**Answer:** C
**Explanation:** Masking policy UDFs must be deterministic and immutable. References to sequences introduce non-determinism, violating policy evaluation guarantees and potentially producing inconsistent masking results.

**Q378.** Which privilege is required to create a future grant on tag-based masking policy applications?
A. CREATE FUTURE GRANT
B. APPLY MASKING POLICY
C. OWNERSHIP on schema
D. MANAGE TAG POLICY GRANTS
**Answer:** C
**Explanation:** Future grants for any object type, including tag-based masking policy applications, require OWNERSHIP of the parent schema. This ensures centralized control over privilege inheritance for subsequently created objects.

**Q379.** A row access policy is applied to a table used in a Java UDF. Which behavior applies?
A. Java UDFs bypass row access policies for performance
B. Row access policies filter data before Java UDF invocation
C. Java UDFs cannot reference tables with row access policies
D. Policies apply only to SQL queries, not UDF-layer access
**Answer:** B
**Explanation:** Row access policies evaluate during data retrieval, before UDF invocation. This ensures that only authorized rows are passed to Java processing, maintaining data protection boundaries across execution contexts.

**Q380.** Which system view lists the privilege grants required to apply masking policies to secure views?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.MASKING_POLICY_PRIVILEGES
C. SNOWFLAKE.CORE.SECURE_VIEW_MASKING_GRANTS
D. ACCOUNT_USAGE.SECURE_VIEW_ACCESS_LOG
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including APPLY MASKING POLICY grants on secure views. This view supports access reviews and compliance auditing for policy management.

**Q381.** A masking policy applies to a column used in a CTE. Which behavior applies?
A. CTE evaluation operates on masked values, potentially altering intermediate results
B. CTE evaluation uses raw values; masking applies only to final output
C. Queries fail if masked columns are used in CTE definitions
D. Masking policies automatically disable for CTE contexts
**Answer:** B
**Explanation:** Masking policies apply during final result projection, after CTE evaluation. CTE logic uses raw column values, ensuring accurate intermediate computations while masking output for unauthorized roles.

**Q382.** Which privilege allows a role to view the resource consumption of masking policy evaluation?
A. VIEW MASKING RESOURCES
B. MONITOR POLICY
C. No privilege; masking resource consumption is not separately exposed
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Snowflake does not expose detailed masking policy resource metrics. Administrators infer policy impact through warehouse credit consumption analysis in WAREHOUSE_METERING_HISTORY and query profiling.

**Q383.** A row access policy references a tag with dynamic assignment via automation. Which evaluation behavior applies?
A. Tag assignments are evaluated at policy creation time
B. Tag assignments are evaluated at query execution time, reflecting current state
C. Tag assignments are cached for 1 hour to optimize performance
D. Dynamic tag assignment cannot be referenced in policies
**Answer:** B
**Explanation:** Row access policies evaluate expressions at query time, retrieving current tag assignments during execution. This enables dynamic access controls that respond to automated tag updates without policy redeployment.

**Q384.** Which system view documents the application of row access policies to materialized views?
A. ACCOUNT_USAGE.ROW_ACCESS_POLICIES
B. INFORMATION_SCHEMA.ROW_ACCESS_POLICY_REFERENCES
C. SNOWFLAKE.CORE.MATERIALIZED_VIEW_POLICY_LOG
D. ACCOUNT_USAGE.MATERIALIZED_VIEW_ACCESS_LOG
**Answer:** B
**Explanation:** ROW_ACCESS_POLICY_REFERENCES in INFORMATION_SCHEMA lists tables and views with applied row access policies, including materialized views. This view supports governance audits and impact analysis for policy changes.

**Q385.** A masking policy applies to a column with a time-travel query context. Which behavior applies?
A. Time-travel queries use historical masked values
B. Time-travel queries use historical raw values; masking applies based on current policy
C. Queries fail if masked columns are used in time-travel contexts
D. Masking policies automatically disable for time-travel queries
**Answer:** B
**Explanation:** Time-travel queries retrieve historical raw data values. Masking policies apply based on current policy definitions and session context, ensuring consistent data protection regardless of temporal query scope.

**Q386.** Which privilege is required to create a masking policy that references account-level tags?
A. CREATE MASKING POLICY
B. ACCESS ACCOUNT TAGS
C. USE ORGANIZATION METADATA
D. MANAGE ACCOUNT POLICIES
**Answer:** A
**Explanation:** Creating masking policies requires CREATE MASKING POLICY privilege. Account-level tags accessed via system functions are available to policy logic without additional privileges beyond policy creation rights.

**Q387.** A row access policy is applied to a table used in a Snowpark DataFrame operation. Which behavior applies?
A. Snowpark operations bypass row access policies for performance
B. Row access policies filter data based on the authenticated session context
C. Snowpark cannot reference tables with row access policies
D. Policies apply only to SQL queries, not DataFrame-layer access
**Answer:** B
**Explanation:** Row access policies evaluate during data retrieval, regardless of client API. Snowpark operations inherit the authenticated session context, ensuring consistent row-level protection across access methods.

**Q388.** Which system view lists the evaluation context for masking policies applied through materialized views?
A. ACCOUNT_USAGE.MASKING_EVALUATION
B. INFORMATION_SCHEMA.MATERIALIZED_VIEW_MASKING_LOG
C. SNOWFLAKE.CORE.MASKING_CONTEXT_TRACE
D. Masking policy evaluation context is not logged; use QUERY_HISTORY for audit
**Answer:** D
**Explanation:** Snowflake does not log detailed masking policy evaluation context. Administrators audit query execution via QUERY_HISTORY, inferring policy impact from result sets and access patterns.

**Q389.** A masking policy uses a conditional expression with CURRENT_WAREHOUSE(). Which context does this function provide?
A. The user's default warehouse name
B. The active warehouse for the current session
C. The warehouse used for policy evaluation
D. The organization's default warehouse
**Answer:** B
**Explanation:** CURRENT_WAREHOUSE() returns the active warehouse for the session. Masking policies can leverage this for warehouse-specific transformations, enabling environment-aware data protection rules.

**Q390.** Which privilege allows a role to create a tag-based masking policy that references database tags?
A. CREATE TAG
B. CREATE MASKING POLICY
C. APPLY TAG ON DATABASE
D. MANAGE DATABASE TAGS
**Answer:** B
**Explanation:** Creating masking policies requires CREATE MASKING POLICY privilege. Tag references within the policy, including database tags, do not require additional privileges beyond those needed to read tag metadata during evaluation.

**Q391.** A row access policy references a user's project assignment from a tag on the user. Which function retrieves user tag values?
A. GET_USER_TAG()
B. SYSTEM$GET_TAG_ON_USER()
C. USER_TAG_VALUE()
D. CURRENT_USER_TAG()
**Answer:** B
**Explanation:** SYSTEM$GET_TAG_ON_USER() retrieves tag values assigned to user objects. Row access policies can leverage this for user classification-based access controls, enabling dynamic filtering based on user metadata.

**Q392.** Which system view documents the privilege grants required to apply row access policies to materialized views?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.ROW_ACCESS_POLICY_PRIVILEGES
C. SNOWFLAKE.CORE.MATERIALIZED_VIEW_POLICY_GRANTS
D. ACCOUNT_USAGE.MATERIALIZED_VIEW_ACCESS_LOG
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including APPLY ROW ACCESS POLICY grants on materialized views. This view supports access reviews and compliance auditing for policy management.

**Q393.** A masking policy applies to a column with a clustering key dependency. Which behavior applies during reclustering?
A. Reclustering operates on masked values, potentially altering physical ordering
B. Reclustering uses raw values; masking applies only to query output
C. Reclustering operations fail if masked columns are used in clustering keys
D. Masking policies automatically disable for reclustering contexts
**Answer:** B
**Explanation:** Clustering operations use raw column values to determine micro-partition boundaries. Masking policies transform data only during query result projection, ensuring that physical storage optimization functions correctly while masking output for unauthorized roles.

**Q394.** Which privilege allows a role to view the dependency chain for masking policies applied through materialized views?
A. VIEW MASKING DEPENDENCIES
B. MONITOR POLICY
C. No privilege; policy dependencies are visible via OBJECT_DEPENDENCIES
D. ACCESS POLICY AUDIT
**Answer:** C
**Explanation:** Policy dependencies are documented in ACCOUNT_USAGE.OBJECT_DEPENDENCIES. Roles with MONITOR USAGE privilege can query this view to analyze policy impact without requiring special policy-specific privileges.

**Q395.** A row access policy uses a tag with role-based inheritance. Which evaluation behavior applies?
A. Role-based tag inheritance is evaluated at policy creation time
B. Role-based tag inheritance is evaluated at query execution time based on session roles
C. Role-based tag inheritance is cached for 1 hour to optimize performance
D. Tags cannot represent role-based inheritance in policies
**Answer:** B
**Explanation:** Row access policies evaluate expressions at query time, interpreting tag metadata and role context during execution. This enables dynamic access controls that respond to role membership changes without policy redeployment.

**Q396.** Which system view lists the application of masking policies to streams?
A. ACCOUNT_USAGE.MASKING_POLICIES
B. INFORMATION_SCHEMA.MASKING_POLICY_REFERENCES
C. SNOWFLAKE.CORE.STREAM_MASKING_LOG
D. Masking policies apply to stream queries based on consumer context
**Answer:** D
**Explanation:** Masking policies evaluate during query execution regardless of client context. Stream consumers inherit their session context, ensuring consistent masking behavior for change data capture workloads without separate policy application tracking.

**Q397.** A masking policy uses a UDF that references a file format. Which requirement applies?
A. UDFs cannot reference file formats in masking policies
B. The file format must be owned by the policy creator
C. Masking policies cannot call UDFs that access metadata objects
D. File format access requires additional UNMASK privilege
**Answer:** A
**Explanation:** Masking policy UDFs cannot reference metadata objects like file formats. Policy logic must be self-contained, using only built-in functions and deterministic UDFs to ensure predictable, low-latency evaluation.

**Q398.** Which privilege is required to create a future grant on tag-based row access policy applications?
A. CREATE FUTURE GRANT
B. APPLY ROW ACCESS POLICY
C. OWNERSHIP on schema
D. MANAGE TAG POLICY GRANTS
**Answer:** C
**Explanation:** Future grants for any object type, including tag-based row access policy applications, require OWNERSHIP of the parent schema. This ensures centralized control over privilege inheritance for subsequently created objects.

**Q399.** A row access policy is applied to a table used in a JavaScript UDF. Which behavior applies?
A. JavaScript UDFs bypass row access policies for performance
B. Row access policies filter data before JavaScript UDF invocation
C. JavaScript UDFs cannot reference tables with row access policies
D. Policies apply only to SQL queries, not UDF-layer access
**Answer:** B
**Explanation:** Row access policies evaluate during data retrieval, before UDF invocation. This ensures that only authorized rows are passed to JavaScript processing, maintaining data protection boundaries across execution contexts.

**Q400.** Which system view lists the privilege grants required to apply masking policies to materialized views?
A. ACCOUNT_USAGE.GRANTS_TO_ROLES
B. INFORMATION_SCHEMA.MASKING_POLICY_PRIVILEGES
C. SNOWFLAKE.CORE.MATERIALIZED_VIEW_MASKING_GRANTS
D. ACCOUNT_USAGE.MATERIALIZED_VIEW_ACCESS_LOG
**Answer:** A
**Explanation:** GRANTS_TO_ROLES in ACCOUNT_USAGE documents privilege assignments, including APPLY MASKING POLICY grants on materialized views. This view supports access reviews and compliance auditing for policy management.

---

**Progress Summary:** 400 of 1000 questions completed.  
**Next Section Preview:** Section 3 will address Performance Optimization, Query Tuning, and Workload Management (Q401–Q600), divided into query execution internals and resource governance strategies.
