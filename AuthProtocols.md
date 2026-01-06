# Authentication Methods
Authentication methods verify a user's identity before granting access to systems, applications, or resources
## Basic Authentication (Base64 Encoding)
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
