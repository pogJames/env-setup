https://claude.ai/public/artifacts/a7569c13-15b2-428e-936f-6c394029e52e
---

## Google Flow - Detailed Breakdown

```
User → GFE → Identity Service → Zanzibar → IAM → Backend
```

### Step 1-4: Authentication (Who are you?)

```
1. User sends request + credentials to Google Front End (GFE)
2. GFE forwards to Identity Service
3. Identity Service verifies (checks password + U2F key)
4. Identity Service returns "user context ticket"
```

**What's happening:**
- GFE is just a load balancer/reverse proxy. It doesn't know who you are.
- Identity Service is the actual login system. It checks your password against their database and verifies your hardware security key.
- "User context ticket" is basically a signed blob saying "this is user john@gmail.com, verified at 2:30pm"

**Why it's needed:**
- You can't access anything without proving who you are first
- The ticket travels with your request so downstream services know who's asking

---

### Step 5-8: Authorization via Zanzibar (Can you access THIS resource?)

```
5. GFE asks Zanzibar: "Can john@gmail.com view document XYZ?"
6. Zanzibar queries Spanner database for ACLs
7. Spanner returns: "Yes, john@gmail.com is in the 'editors' group for XYZ"
8. Zanzibar tells GFE: "Authorized"
```

**What is Zanzibar?**
Think of it as a massive database that stores relationships like:
```
document:XYZ#viewer@user:john@gmail.com      (john can view XYZ)
document:XYZ#editor@group:engineering        (engineering team can edit XYZ)
folder:ABC#parent@document:XYZ               (XYZ is inside folder ABC)
user:john@gmail.com#member@group:engineering (john is in engineering)
```

**Why it's needed:**
- Authentication only proves you're John. It doesn't say what John can do.
- Every Google Doc, Drive folder, Calendar event has its own permissions
- Zanzibar answers: "Does John have permission to do this specific action on this specific resource?"

**Why not just check this in the backend service?**
- Consistency. If Google Docs checks permissions differently than Google Drive, you get bugs.
- Performance. Zanzibar is optimized for this one job - billions of lookups per second.

---

### Step 9-10: IAM Policy Check (What ACTIONS are allowed?)

```
9. GFE checks IAM: "Can john@gmail.com call the 'deleteDocument' API?"
10. IAM returns: "Yes, his role allows delete operations"
```

**What is IAM (Identity and Access Management)?**

IAM is about **roles and permissions at the API/action level**, not individual resources.

Example IAM policy:
```json
{
  "role": "DocumentEditor",
  "permissions": [
    "documents.read",
    "documents.write",
    "documents.delete"    // <-- Can call delete API
  ]
}

{
  "role": "DocumentViewer", 
  "permissions": [
    "documents.read"       // <-- Can only read, not delete
  ]
}
```

**Wait, didn't Zanzibar already check permissions?**

Yes, but they check **different things**:

| System | Question it answers | Example |
|--------|-------------------|---------|
| **Zanzibar** | Can John access **this specific document**? | "Is John in the sharing list for doc XYZ?" |
| **IAM** | Can John perform **this type of action**? | "Is John's role allowed to call delete APIs at all?" |

**Real example:**
1. John has "Viewer" IAM role (can only read, not delete)
2. John is listed as an "editor" on document XYZ in Zanzibar
3. John tries to delete document XYZ

Result:
- Zanzibar says: ✅ "John has editor access to XYZ"
- IAM says: ❌ "John's role doesn't allow delete operations"
- **Request denied**

**Why have both?**
- Zanzibar = fine-grained, per-resource (this doc, that folder)
- IAM = coarse-grained, org-wide policies (interns can't delete anything, admins can do everything)

They work together. Both must say "yes" for the action to proceed.

---

## Netflix Flow - Detailed Breakdown

```
User → Zuul Gateway → Auth Server → Eureka → Microservices
```

### Steps 1-6: Login Flow

```
1. User sends username + password to Zuul Gateway
2. Zuul forwards to Auth Server (it doesn't handle auth itself)
3. Auth Server checks credentials against User DB
4. DB returns user data (hashed password matches)
5. Auth Server creates JWT token
6. Zuul returns JWT to user
```

**What is the JWT token?**
```json
{
  "sub": "user123",
  "name": "John Doe",
  "roles": ["premium_subscriber"],
  "exp": 1699999999,           // expires in 1 hour
  "iat": 1699996399            // issued at
}
```
Plus a signature that proves Netflix created it (not forged).

**Why does Zuul forward to Auth Server instead of doing it itself?**
- Separation of concerns. Zuul's job is routing, not authentication.
- Auth Server might have complex logic (check if account is locked, verify MFA, etc.)
- You can scale them independently

---

### Steps 7-13: Accessing a Protected Resource

```
7. User sends API request + JWT to Zuul
8. Zuul validates JWT locally (checks signature + expiration)
9. Zuul asks Eureka: "Where is the movie-service?"
10. Eureka responds: "movie-service is at 10.0.5.23:8080"
11. Zuul forwards request to movie-service with user context
12. Movie-service processes and returns data
13. Zuul returns data to user
```

**Step 8: Why can Zuul validate JWT "locally"?**

JWTs are **self-contained**. The signature is created with a secret key:
```
signature = HMAC-SHA256(header + payload, secret_key)
```

Zuul has the secret key (or public key for RSA). It can verify:
1. The token wasn't tampered with (signature matches)
2. The token hasn't expired (check `exp` claim)

No database call needed. This is why JWTs scale well.

**Step 9-10: Why is Eureka needed?**

Without Eureka:
```python
# Hardcoded - BAD
movie_service_url = "http://10.0.5.23:8080"
```

Problem: If that server dies or IP changes, everything breaks.

With Eureka:
```python
# Dynamic - GOOD
movie_service_url = eureka.get_instance("movie-service")
# Returns whichever instance is currently healthy
```

**Step 11: What is "user context"?**

Zuul extracts info from the JWT and forwards it as headers:
```
X-User-Id: user123
X-User-Roles: premium_subscriber
X-User-Name: John Doe
```

The movie-service doesn't need to decode the JWT again. It trusts Zuul already did that.

---

## AWS Cognito Flow - Detailed Breakdown

```
User → User Pool (get JWT) → Identity Pool (get AWS credentials) → S3/DynamoDB
```

### Why TWO pools?

**User Pool = Authentication**
- Stores usernames, passwords, emails
- Handles login, signup, password reset, MFA
- Returns JWT tokens (like any OAuth provider)

**Identity Pool = Authorization for AWS**
- Doesn't store users
- Takes a JWT and exchanges it for temporary AWS credentials
- Those credentials let you call AWS APIs directly

**Why not just use the JWT to access S3?**

S3 doesn't understand JWT. S3 understands AWS credentials:
```
AWS_ACCESS_KEY_ID=AKIAXXXXXXX
AWS_SECRET_ACCESS_KEY=xxxxxxxxxx
AWS_SESSION_TOKEN=xxxxxxxxxx
```

Identity Pool is the translator:
```
JWT (standard web token) → AWS credentials (AWS-specific format)
```

---

### Steps 5-9: Getting AWS Credentials

```
5. App sends JWT to Identity Pool
6. Identity Pool maps user to an IAM Role
7. Identity Pool calls STS (Security Token Service)
8. STS returns temporary credentials (15 min - 1 hour)
9. Identity Pool returns credentials to app
```

**Step 6: What is "mapping to IAM Role"?**

You configure rules like:
```
IF jwt.claims.groups CONTAINS "admin"
  THEN use role: arn:aws:iam::123456:role/AdminRole
  
IF jwt.claims.groups CONTAINS "user"
  THEN use role: arn:aws:iam::123456:role/UserRole
  
DEFAULT: arn:aws:iam::123456:role/GuestRole
```

Each role has different permissions:
```json
// AdminRole can do anything
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}

// UserRole can only access their own folder
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::mybucket/users/${cognito-identity.amazonaws.com:sub}/*"
}
```

**Step 7: What is STS?**

STS (Security Token Service) is AWS's service for generating temporary credentials.

Why temporary?
- If credentials are stolen, damage is limited (they expire)
- You don't need to rotate long-lived keys
- You can't accidentally commit them to git and have them work forever

---

## Spotify PKCE Flow - Detailed Breakdown

```
App generates secret → sends hash → gets code → proves it knows secret → gets token
```

### The Problem PKCE Solves

**Without PKCE (old way):**
```
1. App redirects to Spotify login
2. User logs in
3. Spotify redirects back with: myapp://callback?code=ABC123
4. App exchanges code for token
```

**The attack:**
A malicious app registers the same URL scheme `myapp://`. When Spotify redirects, the OS might open the malicious app instead. Now the attacker has the code and can get tokens.

**With PKCE:**
```
1. App generates random string: code_verifier = "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"
2. App hashes it: code_challenge = SHA256(code_verifier) = "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM"
3. App sends code_challenge to Spotify (not the verifier!)
4. User logs in
5. Spotify redirects with code=ABC123
6. App exchanges: code + code_verifier (the original secret)
7. Spotify verifies: SHA256(code_verifier) == code_challenge we stored
8. Only then: here's your token
```

**Why the attacker fails:**
- Attacker intercepts `code=ABC123`
- Attacker doesn't have `code_verifier` (it was never sent over the network until step 6)
- Attacker can't exchange the code without the verifier
- Attack failed

---

## Summary: Questions to Ask Yourself

When looking at any auth flow, ask:

1. **Authentication step**: "How does the system verify this is really John?"
   - Password? OAuth? Hardware key?

2. **Authorization step**: "How does the system decide what John can do?"
   - Per-resource (Zanzibar style)?
   - Per-role (IAM style)?
   - Both?

3. **Token/credential step**: "What proof does John carry around?"
   - JWT? Session cookie? Temporary AWS credentials?
   - How long does it last?
   - Can it be validated without a database call?

4. **Why this separation?**: "Why not combine steps X and Y?"
   - Usually: different concerns, different scaling needs, different update frequencies
