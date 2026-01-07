# Authentication Methods
> Authentication methods verify a user's identity before granting access to systems, applications, or resources

### Basics Authentications
1. **Basic** ~ Username/password sent in every request (Base64 encoded)
-  Simple but insecure — like shouting your credentials across the room
2. **Form** ~ Login page(username/password submitted via HTML form)
- Server creates a session (stored on server) and sends a cookie to your browser for future requests

> [!NOTE]
> These are single-app only — you log in separately for every site/app

### Adding Security Layers
**MFA** ~ adds a layer of security during login that requires users to provide more than one credential to prove their digital identity. Factors can be:
1. Something you are - like a biometric
2. Something you know - like a password
3. Something you own - like a device

### Improving UX
**SSO** ~ authentication mechanism that allows users to access multiple applications and websites using a single set of login credentials
1. Initial Login: You sign in once to a central authentication server (an "identity provider").
2. Token Issuance: The server verifies you and issues a secure token or certificate.
3. Seamless Access: When you try to access another connected application, it uses that token to verify you, granting immediate access without asking for a password again

## Authentication & Authorization Standards
### OAuth 2.0 
**Authorization framework**, protocol that allows a user to grant a third-party web site or application access to the user’s protected resources, without necessarily revealing their long-term credentials or even their identity
> the **client** (Application) requests access to resources controlled by the **resource owner** (End-user) and hosted by the **resource server** and is issued a different set of credentials than those of the resource owner. Instead of using the resource owner’s credentials to access protected resources, the client obtains an **access token** - a string denoting a specific scope, lifetime, and other access attributes. Access tokens are issued to third-party clients by an **authorization server** with the approval of the resource owner. Then the client uses the access token to access the protected resources hosted by the resource server

### Open ID Connect (IODC)
**Identity layer** built on top of the OAuth 2.0 framework. It allows third-party applications to verify the identity of the end-user and to obtain basic user profile information using **access tokens**
> While OAuth 2.0 is about resource access and sharing, OIDC is about user authentication. Its purpose is to give you one login for multiple sites. Each time you need to log in to a website using OIDC, you are redirected to your OpenID site where you log in, and then taken back to the website

### SAML (Security Assertion Markup Language) 
XML-based framework for authentication and authorization between two entities without a password
1. The user (Principal) attempts to access a **Service Provider** (e.g., Salesforce).
2. The **Service Provider** redirects the user to the **Identity Provider** for authentication.
3. The **Identity Provider** verifies the user's credentials (e.g., username, password, or multifactor authentication).
4. Upon successful authentication, the **Identity Provider** generates a **SAML Assertion**, an XML document containing authentication and user details.
5. The **SAML Assertion** is sent back to the **Service Provider**, confirming the user's identity.
6. The **Service Provider** grants the user access without requiring additional login credentials.

### JWT (JSON web token) 
compact and self-contained way for securely transmitting information between parties as a JSON object -> access token
> example token: header.payload.signature
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNzM2MjY1NjAwLCJleHAiOjE3MzYzNTIwMDAsInJvbGUiOiJhZG1pbiJ9.qL3XzK8fG5vY9pR2mN7jH6tW1bC4xD8uV0eA9sQ7rT`\
> decoded:
```bash
# header
{
  "alg": "HS256",
  "typ": "JWT"
}
# payload
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1736265600,
  "exp": 1736352000,
  "role": "admin"
}
# signature
a-string-secret-at-least-256-bits-long
```

# SWOT Analysis
## Basic Authentication (Base64 Encoding)
1. Strengths: Super simple to implement; no sessions or cookies needed.
2. Weaknesses: Credentials sent with every request (even Base64 is not encrypted); very insecure over HTTP; no built-in logout.
3. Opportunities: Rare use in internal APIs with HTTPS.
4. Threats: Highly vulnerable to interception, replay attacks, and credential stuffing; deprecated in modern apps.
```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant DB as Database

    Note over Client: Combine username:password<br>Base64-encode it
    Client->>Server: HTTP Request<br>Authorization: Basic <base64-string>
    Server->>Server: Decode Base64 to get credentials
    Server->>DB: Verify username & hashed password
    DB-->>Server: Valid / Invalid
    alt Valid
        Server-->>Client: 200 OK + Resource
    else Invalid
        Server-->>Client: 401 Unauthorized<br>WWW-Authenticate: Basic
    end
```

## Form-Based Authentication
1. Strengths: Familiar to users (login page); easy to customize; works with sessions/cookies.
2. Weaknesses: Session hijacking risks (cookie theft); server-side state needed; prone to CSRF without protection.
3. Opportunities: Still common in traditional web apps; easy to add MFA.
4. Threats: Phishing and brute-force attacks; being replaced by token-based systems.
```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Server
    participant DB as Database

    User->>Browser: Enter username/password in form
    Browser->>Server: POST /login (credentials)
    Server->>DB: Verify credentials
    DB-->>Server: Valid / Invalid
    alt Valid
        Server-->>Browser: 302 Redirect + Set-Cookie: sessionID
        Browser->>Server: GET protected resource (with Cookie)
        Server-->>Browser: 200 OK + Resource
    else Invalid
        Server-->>Browser: Login page + Error
    end
```

## Multi-Factor Authentication
1. Strengths: Greatly improves security (something you know + have/are).
2. Weaknesses: Adds friction for users; can be bypassed (e.g., phishing for codes).
3. Opportunities: Essential today; integrates easily with most systems; push toward passwordless.
4. Threats: Social engineering or SIM-swapping attacks; user resistance to extra steps.
```mermaid
sequenceDiagram
    participant User
    participant Client
    participant Server
    participant MFA as MFA Service

    Client->>Server: Submit username & password
    Server->>Server: Verify password
    alt Password Valid
        Server->>MFA: Request second factor (e.g., send push/generate code)
        MFA->>User: Deliver code/push notification
        User->>Client: Enter MFA code/approve
        Client->>Server: Submit MFA response
        Server->>MFA: Validate
        MFA-->>Server: Valid
        Server-->>Client: Auth Success + Session/Token
    else Invalid
        Server-->>Client: Auth Failed
    end
```

## Single Sign-On
1. Strengths: Huge convenience—one login for many apps; better user experience.
2. Weaknesses: Single point of failure (if IdP compromised, all apps at risk).
3. Opportunities: Widely adopted in enterprises and cloud services.
4. Threats: Centralized attack target; dependency on identity provider uptime.
```mermaid
sequenceDiagram
    participant User as User/Browser
    participant SP as Service Provider (App)
    participant IdP as Identity Provider

    User->>SP: Access protected resource
    SP->>User: Redirect to IdP with AuthnRequest
    User->>IdP: AuthnRequest (authenticate if needed)
    IdP->>User: Login page (if no session)
    User->>IdP: Submit credentials
    alt Auth Success
        IdP->>User: POST SAML Assertion to SP ACS
        User->>SP: SAML Response (Assertion)
        SP->>SP: Validate Assertion
        SP-->>User: Grant Access + Resource
    else Auth Failed
        IdP-->>User: Error
    end
```
  
## OAuth
1. Strengths: Granular access, highly-scalable, and dev friendly (REST-based and lightweight JSON)
2. Weaknesses: Only for Authz and token thefts
3. Opportunities: evolving standards
4. Threats: primary target for bot attacks and phishing evolution
```mermaid
sequenceDiagram
    participant User
    participant Client as Client App
    participant Auth as Authorization Server
    participant RS as Resource Server (API)

    User->>Client: Initiate login/access
    Client->>Auth: Redirect to /authorize (code, PKCE, scopes)
    Auth->>User: Login + Consent
    User->>Auth: Credentials + Approve
    Auth->>Client: Redirect with auth code
    Client->>Auth: POST code + PKCE verifier for access/refresh token
    Auth-->>Client: Access Token + Refresh (optional ID Token if OIDC)
    Client->>RS: Request with Bearer Access Token
    RS-->>Client: Protected Resource
```

## OIDC
1. Strengths: Combines ASML Authn and OAuth mobile-friendly
2. Weaknesses: Vendor Divergence and Logout complexity
3. Opportunities: Universal Login and automatic key rotation
4. Threats: Single point of Failure and Privacy concerns
```mermaid
sequenceDiagram
    participant User
    participant RP as Relying Party (Client)
    participant OP as OpenID Provider

    User->>RP: Access app
    RP->>OP: Redirect to /authorize (scope=openid profile email, nonce, etc.)
    OP->>User: Authenticate (username/pw/MFA)
    User->>OP: Submit + Consent
    OP->>RP: Redirect with auth code
    RP->>OP: POST code for ID Token + Access Token
    OP-->>RP: ID Token (JWT with user claims) + Access Token
    RP->>RP: Validate ID Token signature/nonce/expiry
    Note right of RP: User authenticated
```

## SAML
1. Strengths: Enterprise standard and strong security
2. Weaknesses: Complex, heavy, and not mobile-friendly
3. Opportunities: Remains as gold standard for govtech, fintech, healthcare
4. Threats: Mordernization to OIDC and high maintenance cost
```mermaid
sequenceDiagram
    participant User
    participant SP as Service Provider
    participant IdP as Identity Provider

    User->>SP: Access protected resource
    SP->>IdP: Redirect with AuthnRequest (signed XML)
    IdP->>User: Login form (if not already authenticated)
    User->>IdP: Credentials + MFA
    IdP->>SP: POST SAML Assertion (signed XML with user attributes)
    SP->>SP: Validate signature, attributes, conditions
    SP-->>User: Grant access (session cookie)
```
## JWT
1. Strengths: Stateless, compact, built-in expiration
2. Weaknesses: Difficult to revoke and token bloat
3. Opportunities: Edge computing, cross domain and standardized
4. Threats: Algorithm confusion attacks or weak keys
```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: POST /login with credentials
    Server->>Server: Validate creds
    Server-->>Client: JWT (signed token)
    Client->>Server: Subsequent requests with Authorization: Bearer <JWT>
    Server->>Server: Verify signature, expiry, claims
    alt Valid
        Server-->>Client: Resource / Success
    else Invalid
        Server-->>Client: 401 Unauthorized
    end
```
