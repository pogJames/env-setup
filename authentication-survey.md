# Auth API Standards

### The Stack Everyone Uses
```
OAuth 2.0/2.1  →  Authorization framework
OpenID Connect →  Identity layer (who is this user?)
JWT            →  Token format
PKCE           →  Security for public clients (NOW MANDATORY)
```

### OpenAPI Security Types (5 total)

| Type | Use Case | Example |
|------|----------|---------|
| `apiKey` | Simple service auth | `X-API-Key: abc123` |
| `http: basic` | Legacy (avoid) | Base64 username:password |
| `http: bearer` | **Most common** | `Authorization: Bearer <jwt>` |
| `oauth2` | Full auth flows | Login with Google, scoped access |
| `openIdConnect` | Enterprise SSO | Auto-discovers endpoints |

### Typical Auth Endpoints

```
POST /auth/register     → Create account (public)
POST /auth/login        → Get tokens (public)
POST /auth/logout       → Invalidate session
POST /auth/refresh      → Get new access token
POST /auth/password/forgot  → Send reset email
POST /auth/password/reset   → Use reset token
GET  /users/me          → Current user profile
```

### Key Takeaways

1. **Use Bearer + JWT** for most APIs
2. **PKCE is mandatory** in OAuth 2.1 (not optional anymore)
3. **Implicit flow is dead** — always use Authorization Code + PKCE for SPAs/mobile
4. **Short-lived access tokens** (5-60 min) + refresh tokens
5. **Document your security** in OpenAPI `securitySchemes`

## [COMMON AUTH ACTIVITIES](https://claude.ai/public/artifacts/1c6a2d56-2309-4c3c-969f-8e36281345df)

### Summary: Questions to Ask Yourself

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
