# The Root Problem

Imagine a company with 50 different apps — email, file storage, HR system, internal tools. If each app manages its own users, you get:

- Employees juggling 50 passwords
- IT manually creating/deleting accounts in 50 places when someone joins or leaves
- Security nightmares (forgot to revoke access somewhere? breach.)

Identity Providers solve this: **one source of truth for "who exists" and "what can they access."**

---

## LDAP

Think of LDAP as a **phone book protocol**. Not a product itself — a standardized way to query a directory.

The directory is hierarchical, like a file system:

```
dc=company,dc=com
├── ou=engineering
│   ├── cn=alice
│   └── cn=bob
└── ou=finance
    └── cn=charlie
```

When an app needs to verify a user, it asks the LDAP server: "Does alice exist in engineering? Is her password correct?"

You won't interact with raw LDAP much. It's the foundation that products like Active Directory and OpenLDAP build on. Know it exists, know what it does, move on.

## AWS IAM

**What it is**: Free AWS service for managing identities and permissions to control access to AWS resources securely (authN + authZ).

**Key components**:
- Users: Individual accounts with creds (passwords, keys, MFA).
- Groups: Bundle users for shared policies.
- Roles: Temporary perms for apps/services (e.g., EC2 to S3).
- Policies: JSON rules for actions/resources/conditions.

**Main use cases**:
- Least-privilege access for teams/apps.
- Cross-account delegation.
- App creds without hardcoding.
- MFA/compliance enforcement.

**Quick examples**:
- Role for EC2 to read S3 bucket.
- MFA-required policy for instance termination.
- Group for read-only analysts.

## Active Directory (AD)

**What it is**: Microsoft's on-prem directory service for Windows networks, storing users/groups/devices and handling authN/authZ via Kerberos/LDAP.

**Key components**:
- Domains/Forests: Logical/security boundaries.
- Domain Controllers: Replicated servers with DB.
- OUs: Organize objects for policies.
- Group Policy: Enforce settings/security.

**Main use cases**:
- Windows domain logins and management.
- Policy enforcement (passwords, software).
- Resource access control (files/printers).
- Hybrid with cloud (Azure AD).

**Quick examples**:
- Domain login applies desktop policies.
- Group Policy restricts USBs in OUs.
- Group-based file share perms.

## Auth0

**What it is**: Developer-focused identity platform (now Okta-owned) for app authN/authZ, handling logins, SSO, and security via APIs/SDKs—cloud-native, scalable.

**Key components**:
- Universal Login: Customizable auth flows.
- Connections: Integrate social/enterprise IDPs.
- Actions/Rules: Custom code for workflows.
- MFA/Security: Adaptive auth, anomaly detection.

**Main use cases**:
- Quick app auth (web/mobile/SPA).
- SSO across apps/IDPs.
- CIAM for customer logins.
- Secure APIs with JWTs.

**Quick examples**:
- Social login (Google) for app signup.
- Custom action emails MFA codes.
- Role-based API access via tokens.

## Okta

**What it is**: Cloud IAM platform for workforce/customer identities, centralizing SSO, MFA, and lifecycle management across apps/devices.

**Key components**:
- Universal Directory: User store with integrations.
- SSO: One login for 7k+ apps.
- MFA: Adaptive factors (push, biometrics).
- Lifecycle: Auto provision/deprovision.

**Main use cases**:
- Hybrid/cloud SSO.
- Zero-trust security.
- User automation (onboard/offboard).
- Multi-cloud integration.

**Quick examples**:
- Auto-create Slack/Salesforce accounts on hire.
- MFA for sensitive app access.
- Dashboard SSO to all tools.

## Social IdPs (Google, Microsoft, GitHub, etc.)

**What they are**: Third-party services (Google, Facebook/Meta, Apple, GitHub, etc.) that let users "Login with..." your app using their existing accounts—no new passwords needed.

**Key components**:
- OAuth 2.0 + OIDC: For secure delegated auth (code flow + ID/access tokens).
- Scopes: Requested permissions (e.g., email, profile)—user consents.
- Claims: User data in ID token JWT (sub, email, name).

**Main use cases**:
- Consumer/B2C apps for fast signup/login.
- Reducing password fatigue/abandonment.
- Leveraging provider's MFA/security.

**Quick examples**:
- "Sign in with Google" → consent screen → app gets email/name via claims.
- GitHub login for dev tools → scopes for repo access.
- Apple Sign-In → privacy-focused (hide email option).

**Pros/Cons**: Super convenient, trusted security; but dependency on external providers, limited customization, privacy concerns.

**When to use**: Ideal for public-facing apps wanting quick adoption—combine with your own auth for hybrids.

# Privileges
### What is Fine-Grained Authorization?
Fine-Grained Authorization (FGA) implies the ability to grant specific users permission to perform certain actions in specific resources.

Well-designed FGA systems allow you to manage permissions for millions of objects and users. These permissions can change rapidly as a system continually adds objects and updates access permissions for its users.

A notable example of FGA is Google Drive: access can be granted either to documents or to folders, as well as to individual users or users as a group, and access rights regularly change as new documents are created and shared with specific users or groups.

### What is Role-Based Access Control?
In Role-Based Access Control (RBAC), permissions are assigned to users based on their role in a system. For example, a user needs the editor role to edit content.

RBAC systems enable you to define users, groups, roles, and permissions, then store them in a centralized location. Applications access that information to make authorization decisions.

### What is Attribute-Based Access Control?
In Attribute-Based Access Control (ABAC), permissions are granted based on a set of attributes that a user or resource possesses. For example, a user assigned both marketing and manager attributes is entitled to publish and delete posts that have a marketing attribute.

Applications implementing ABAC need to retrieve information stored in multiple data sources - like RBAC services, user directories, and application-specific data sources - to make authorization decisions.

### What is Policy-Based Access Control?
Policy-Based Access Control (PBAC) is the ability to manage authorization policies in a centralized way that’s external to the application code. Most implementations of ABAC are also PBAC.

### What is Relationship-Based Access Control?
Relationship-Based Access Control (ReBAC) enables user access rules to be conditional on relations that a given user has with a given object and that object's relationship with other objects. For example, a given user can view a given document if the user has access to the document's parent folder.

ReBAC is a superset of RBAC: you can fully implement RBAC with ReBAC. ReBAC also lets you natively solve for ABAC when attributes can be expressed in the form of relationships. For example ‘a user’s manager’, ‘the parent folder’, ‘the owner of a document’, ‘the user’s department’ can be defined as relationships.

Auth0 FGA extends ReBAC by making it simpler to express additional ABAC scenarios using Conditions or Contextual Tuples.

ReBAC can also be considered PBAC, as authorization policies are centralized.

### What is Zanzibar?
Zanzibar is Google's global authorization system across Google's product suite. It’s based on ReBAC and uses object-relation-user tuples to store relationship data, then checks those relations for a match between a user and an object. For more information, see Zanzibar Academy.

ReBAC systems based on Zanzibar store the data necessary to make authorization decisions in a centralized database. Applications only need to call an API to make authorization decisions.

Auth0 FGA is an example of a Zanzibar-based authorization system.
