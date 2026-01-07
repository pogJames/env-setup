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
