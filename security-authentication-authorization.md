# Security Essentials for Authentication & Authorization

## 1. Injection Attacks

**What:** Attacker inserts malicious code into your queries/commands

```
Your App: "SELECT * FROM users WHERE username = '" + input + "'"
Attacker input: "admin' OR '1'='1"
Result: Returns ALL users 💀
```

**Prevention (Pareto: these 2 cover 90% of cases):**

| Method | Example |
|--------|---------|
| **Parameterized queries** | `cursor.execute("SELECT * FROM users WHERE username = ?", (username,))` |
| **ORM usage** | `User.query.filter_by(username=username).first()` |

```python
# ❌ NEVER
query = f"SELECT * FROM users WHERE id = {user_input}"

# ✅ ALWAYS
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_input,))
```

---

## 2. Session Hijacking

**What:** Attacker steals session token → impersonates user

```
┌──────────┐    steals cookie    ┌──────────┐
│ Attacker │ ◄────────────────── │  User    │
└────┬─────┘                     └──────────┘
     │ uses stolen session
     ▼
┌──────────┐
│  Server  │  ← thinks attacker is user
└──────────┘
```

**Prevention:**

| Measure | Implementation |
|---------|----------------|
| **HttpOnly cookies** | JS can't access token |
| **Secure flag** | HTTPS only |
| **Short expiry** | Limits damage window |
| **Bind to IP/fingerprint** | Token valid only for original context |

```python
# FastAPI cookie example
response.set_cookie(
    key="session",
    value=token,
    httponly=True,   # ← Can't be read by JavaScript (blocks XSS theft)
    secure=True,     # ← HTTPS only (blocks network sniffing)
    samesite="lax"   # ← Blocks CSRF
)
```

---

## 3. Token Leaking Sensitive Info

**What:** JWT payload is base64 encoded, NOT encrypted — anyone can read it

```bash
# Anyone can decode your JWT payload:
echo "eyJzdWIiOiJib2IiLCJyb2xlIjoiYWRtaW4iLCJzc24iOiIxMjMtNDUtNjc4OSJ9" | base64 -d
# {"sub":"bob","role":"admin","ssn":"123-45-6789"}  ← SSN exposed! 💀
```

**Prevention:**

```python
# ❌ NEVER put in JWT
{
    "ssn": "123-45-6789",
    "credit_card": "4111...",
    "password": "...",
    "address": "..."
}

# ✅ ONLY put in JWT (minimal claims)
{
    "sub": "user_id_123",      # Reference ID only
    "role": "user",            # Access level
    "exp": 1699999999          # Expiry
}
```

**Rule of thumb:** If you wouldn't print it on a billboard, don't put it in a JWT.

---

## 4. Rate Limiting

**What:** Prevent brute force attacks by limiting requests

```
Without rate limit:
Attacker: POST /login (attempt 1)     → 10ms
Attacker: POST /login (attempt 2)     → 10ms
... 1 million attempts in minutes

With rate limit:
Attacker: POST /login (attempt 1-5)   → OK
Attacker: POST /login (attempt 6)     → 429 Too Many Requests (wait 60s)
```

**Implementation (FastAPI + slowapi):**

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/token")
@limiter.limit("5/minute")  # ← 5 login attempts per minute per IP
async def login(request: Request, ...):
    ...
```

**Recommended limits:**

| Endpoint | Limit | Rationale |
|----------|-------|-----------|
| `/login`, `/token` | 5-10/min | Prevent brute force |
| `/register` | 3/hour | Prevent spam accounts |
| `/api/*` (authenticated) | 100-1000/min | Prevent abuse |
| `/password-reset` | 3/hour | Prevent enumeration |

---

## 5. Hashing vs Plain-text

**Analogy:** 

```
Plain-text = Writing your PIN on your debit card
Hashing    = Memorizing your PIN (one-way, can't reverse)
```

**Visual:**

```
Password: "mypassword123"
                │
                ▼
┌─────────────────────────────────────────────────┐
│  bcrypt.hash("mypassword123")                   │
└─────────────────────────────────────────────────┘
                │
                ▼
Stored: "$2b$12$LQv3c1yqBw...R1u/XYZ"  ← Can't reverse to original
```

**Verification flow:**

```
User enters: "mypassword123"
                │
                ▼
bcrypt.verify("mypassword123", stored_hash) → True/False
```

**Algorithm hierarchy (2024):**

| Algorithm | Status | Use Case |
|-----------|--------|----------|
| **Argon2id** | ✅ Best | New projects (memory-hard) |
| **bcrypt** | ✅ Good | Most common, battle-tested |
| **scrypt** | ✅ Good | Alternative to bcrypt |
| SHA256/MD5 | ❌ Never | Too fast = easy to brute force |
| Plain-text | ☠️ Never | Just... no |

```python
# ✅ Using bcrypt (or passlib)
from passlib.hash import bcrypt

# Store
hashed = bcrypt.hash("user_password")

# Verify
if bcrypt.verify(input_password, stored_hash):
    grant_access()
```

---

## Quick Reference Checklist

```
□ Injection       → Parameterized queries / ORM
□ Session Hijack  → HttpOnly + Secure + Short expiry
□ Token Leaking   → Minimal claims (ID + role + exp only)
□ Rate Limiting   → 5-10/min on auth endpoints
□ Passwords       → bcrypt/Argon2 (NEVER plain-text)
```
