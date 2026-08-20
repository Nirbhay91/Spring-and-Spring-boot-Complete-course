# S4.1.23 — JWT Authentication

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is JWT? ⭐⭐⭐⭐⭐

**JWT (JSON Web Token)** is a compact, URL-safe token format used to securely represent claims between parties. It is commonly used as a bearer access token in OAuth2-based systems, but JWT itself is a token format, not an authentication protocol.

### Golden rule

```text
JWT
→ Token format

OAuth2
→ Authorization framework

OIDC
→ Authentication + Identity layer

Spring Security
→ Security framework
```

JWT can be used for authentication/session propagation, but **JWT itself does not define the complete authentication protocol**.

---

# 2. Why JWT? ⭐⭐⭐⭐⭐

Traditional server-side session:

```text
Client
 ↓
Session ID
 ↓
Server Session Store
 ↓
User Session
```

JWT-based stateless API:

```text
Client
 ↓
Bearer JWT
 ↓
API
 ↓
Validate JWT
 ↓
Authenticated Request
```

A self-contained JWT can carry claims required by the resource server, reducing the need for a central session lookup for every request.

### Important

> **Stateless does not mean “no state exists anywhere.”** Token revocation, refresh-token state, key management, user state, and application state may still exist.

---

# 3. JWT Structure ⭐⭐⭐⭐⭐

A signed JWT commonly has three Base64URL-encoded parts:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example shape:

```text
xxxxx.yyyyy.zzzzz
```

### Three parts

```text
Header
   ↓
Payload
   ↓
Signature
```

---

# 4. JWT Header ⭐⭐⭐⭐⭐

The header describes information about the token, such as the token type and signing algorithm.

Example:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "key-123"
}
```

### Common fields

```text
alg → signing algorithm
kid → key identifier
typ → token type
```

### Important

> Never choose trust based only on an untrusted token's `alg` value. The accepted algorithms must be configured by the application/provider policy.

---

# 5. JWT Payload ⭐⭐⭐⭐⭐

The payload contains **claims**.

Example:

```json
{
  "iss": "https://issuer.example.com",
  "sub": "12345",
  "aud": "orders-api",
  "exp": 1780000000,
  "iat": 1779996400,
  "scope": "orders.read"
}
```

### Important

> JWT payload is normally **encoded, not encrypted**. Do not put passwords, secrets, or sensitive data into a normal signed JWT payload.

---

# 6. JWT Signature ⭐⭐⭐⭐⭐

The signature protects the integrity of the signed JWT.

Conceptually:

```text
Header + Payload
       ↓
Signing algorithm + Key
       ↓
Signature
```

On validation:

```text
JWT
 ↓
Signature verification
 ↓
Trusted?
```

A valid signature helps establish that the signed content was not modified and was signed by the expected key.

---

# 7. Encoding vs Encryption ⭐⭐⭐⭐⭐

This is a common interview question.

```text
Base64URL Encoding
→ Representation
→ Not security

JWS / Signed JWT
→ Integrity + authenticity of signed content

JWE / Encrypted JWT
→ Confidentiality
```

### Memory

> **JWT is not automatically encrypted.**

---

# 8. JWS vs JWE ⭐⭐⭐⭐

### JWS

A signed representation.

```text
Integrity
+
Authenticity
```

### JWE

An encrypted representation.

```text
Confidentiality
```

A normal three-part JWT often refers to a compact JWS representation.

---

# 9. Standard JWT Claims ⭐⭐⭐⭐⭐

Important registered claims include:

```text
iss
sub
aud
exp
nbf
iat
jti
```

### `iss` — Issuer

Identifies who issued the token.

### `sub` — Subject

Identifies the subject of the token.

### `aud` — Audience

Identifies the intended recipient/audience.

### `exp` — Expiration Time

Token must not be accepted after expiration.

### `nbf` — Not Before

Token must not be accepted before the specified time.

### `iat` — Issued At

Time when token was issued.

### `jti` — JWT ID

Unique identifier for the token when provided/used by the issuer.

---

# 10. Access Token JWT vs ID Token JWT ⭐⭐⭐⭐⭐

Both can be JWTs, but their purposes differ.

```text
JWT Access Token
→ OAuth2 authorization
→ Intended for Resource Server

JWT ID Token
→ OIDC authentication/identity
→ Intended for Client/Relying Party
```

### Critical rule

> **Do not treat every JWT as interchangeable. Validate it for the protocol, issuer, audience and purpose it was issued for.**

---

# 11. Bearer Token ⭐⭐⭐⭐⭐

A JWT access token is commonly sent using the HTTP Authorization header:

```http
Authorization: Bearer <access-token>
```

Conceptually:

```text
Client
 ↓
Bearer JWT
 ↓
Spring Security
 ↓
Validate
 ↓
Authentication
```

Because a bearer token can be used by whoever possesses it, token theft is a serious security concern.

---

# 12. JWT Authentication Flow ⭐⭐⭐⭐⭐

```text
User / Client
     ↓
Authentication / Authorization Server
     ↓
JWT Access Token
     ↓
Client
     ↓
Authorization: Bearer <JWT>
     ↓
Spring Security Filter Chain
     ↓
Bearer Token Authentication
     ↓
JWT Decoder
     ↓
JWT Validators
     ↓
Authentication
     ↓
Authorization
     ↓
Controller
```

---

# 13. JWT Validation ⭐⭐⭐⭐⭐

Never accept a JWT simply because it can be decoded.

Typical validation includes:

```text
Signature
   ↓
Issuer
   ↓
Audience (when applicable)
   ↓
Expiration
   ↓
Not-Before (when applicable)
   ↓
Algorithm / key policy
   ↓
Required claims
   ↓
Application authorization
```

### Golden rule

> **Decode ≠ Validate.**

---

# 14. Signature Validation ⭐⭐⭐⭐⭐

For asymmetric signing such as RS256:

```text
Authorization Server
      ↓
Private Key
      ↓
Sign JWT
      ↓
JWT

Resource Server
      ↓
Public Key
      ↓
Verify JWT
```

The private signing key must remain protected by the issuer.

---

# 15. Symmetric vs Asymmetric Signing ⭐⭐⭐⭐⭐

## HS256 — Symmetric

```text
Same shared secret
       ↓
Sign + Verify
```

Both sides need the secret.

## RS256 — Asymmetric

```text
Private key
→ Sign

Public key
→ Verify
```

### Microservices advantage

A resource server can verify tokens with the public key without possessing the authorization server's private signing key.

---

# 16. RS256 vs HS256 Interview Question ⭐⭐⭐⭐⭐

### HS256

```text
One shared secret
```

Useful when trusted parties share the same secret and key-management boundaries are appropriate.

### RS256

```text
Private key → issuer
Public key  → consumers
```

Often preferable in distributed architectures because verification does not require distributing the signing private key.

### Important

> Algorithm choice is an architecture and key-management decision, not simply a performance preference.

---

# 17. JWKS ⭐⭐⭐⭐⭐

**JSON Web Key Set (JWKS)** is a set of public keys used by consumers to verify signed tokens.

```text
OIDC Discovery
      ↓
jwks_uri
      ↓
JWKS
      ↓
Public Keys
      ↓
JWT Signature Validation
```

---

# 18. `kid` and Key Rotation ⭐⭐⭐⭐⭐

`kid` identifies which key was used to sign a JWT.

```text
JWT Header
{
  "kid": "key-123"
}
        ↓
JWKS
        ↓
Find key-123
        ↓
Verify signature
```

When keys rotate:

```text
Old Key
   ↓
New Key
   ↓
JWKS publishes active keys
   ↓
Resource Server refreshes metadata
```

Never hard-code a single temporary public key when the provider supports rotation.

---

# 19. Issuer Validation ⭐⭐⭐⭐⭐

Validate `iss` against the trusted issuer configured for the resource server.

```text
JWT iss
   ↓
Expected issuer
   ↓
Match?
   ↓
Continue / Reject
```

This prevents accepting a valid token from an unintended issuer.

---

# 20. Audience Validation ⭐⭐⭐⭐⭐

Validate `aud` when the application requires a specific audience.

```text
JWT aud
   ↓
orders-api
   ↓
Is this token intended for this API?
```

A token with a valid signature is not automatically valid for every API.

---

# 21. Expiration and Time Claims ⭐⭐⭐⭐⭐

### `exp`

Reject expired tokens.

### `nbf`

Reject tokens that are not yet valid.

### Clock Skew

Distributed systems can have small clock differences.

Validation libraries can support a configured clock-skew tolerance.

### Important

Do not make clock skew unnecessarily large because it extends the effective acceptance window.

---

# 22. Spring Security Resource Server ⭐⭐⭐⭐⭐

Spring Security provides OAuth2 Resource Server support for bearer tokens.

Typical configuration:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2
            .jwt(jwt -> {})
        );

    return http.build();
}
```

The exact setup depends on whether issuer metadata, JWKS, or an explicit decoder is configured.

---

# 23. `issuer-uri` Configuration ⭐⭐⭐⭐⭐

Spring Boot can configure a JWT resource server using an issuer URI.

Conceptually:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://issuer.example.com
```

Spring Security can use provider metadata to discover the appropriate key material and validate issuer information.

### Production rule

> Use a trusted issuer configuration; do not accept an arbitrary issuer from the incoming request.

---

# 24. `jwk-set-uri` Configuration ⭐⭐⭐⭐

A resource server can also be configured with a JWK Set URI when appropriate.

Conceptually:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: https://issuer.example.com/.well-known/jwks.json
```

This can be useful when discovery is not used or the deployment requires explicit key configuration.

### Difference

```text
issuer-uri
→ Provider identity + discovery
→ Issuer validation

jwk-set-uri
→ Direct key-set location
```

Use the configuration model appropriate to the provider and deployment.

---

# 25. `JwtDecoder` ⭐⭐⭐⭐⭐

Spring Security uses a `JwtDecoder` to decode and validate JWTs.

Conceptually:

```text
Bearer JWT
   ↓
JwtDecoder
   ↓
Signature validation
   ↓
Claim validation
   ↓
Jwt
```

Example concept:

```java
@Bean
JwtDecoder jwtDecoder() {
    // Configure according to the trusted issuer/provider.
    return ...;
}
```

Do not implement insecure JWT validation manually when Spring Security's supported validators can be used.

---

# 26. `JwtAuthenticationProvider` ⭐⭐⭐⭐⭐

For JWT bearer authentication, Spring Security uses a JWT authentication provider internally.

Conceptually:

```text
Bearer Token
    ↓
JwtAuthenticationProvider
    ↓
JwtDecoder
    ↓
JWT validation
    ↓
Authentication
```

The resulting authentication is placed into the Spring Security security context for authorization decisions.

---

# 27. `BearerTokenAuthenticationFilter` ⭐⭐⭐⭐⭐

Spring Security's bearer-token filter extracts bearer tokens from incoming requests and delegates authentication.

```text
HTTP Request
    ↓
Authorization: Bearer ...
    ↓
BearerTokenAuthenticationFilter
    ↓
AuthenticationManager
    ↓
JwtAuthenticationProvider
    ↓
JwtDecoder
```

This is an important filter-chain concept for interviews.

---

# 28. AuthenticationManager Flow ⭐⭐⭐⭐⭐

Conceptually:

```text
BearerTokenAuthenticationFilter
            ↓
AuthenticationManager
            ↓
JwtAuthenticationProvider
            ↓
JwtDecoder
            ↓
Validated JWT
            ↓
Authentication
            ↓
SecurityContext
```

This connects JWT authentication with the Spring Security architecture studied earlier.

---

# 29. JWT to Authorities ⭐⭐⭐⭐⭐

JWT claims can be converted into Spring Security authorities.

For OAuth2 scopes, Spring Security commonly creates authorities such as:

```text
scope = orders.read
       ↓
SCOPE_orders.read
```

Then:

```java
@PreAuthorize("hasAuthority('SCOPE_orders.read')")
```

Exact claim-to-authority mapping can be customized.

---

# 30. Scope vs Roles in JWT ⭐⭐⭐⭐⭐

JWT may contain:

```json
{
  "scope": "orders.read orders.write",
  "roles": ["ADMIN"]
}
```

But the meaning of claims is determined by the issuer/application contract.

```text
scope
→ OAuth2 permission

role
→ Application/domain permission
```

Do not assume every `roles` claim automatically becomes a Spring authority.

---

# 31. Custom JWT Claims ⭐⭐⭐⭐

Applications may use custom claims.

Example:

```json
{
  "sub": "123",
  "tenant": "tenant-a",
  "department": "finance"
}
```

These claims can be used for application-specific decisions, but sensitive authorization logic should be based on a well-defined and trusted token contract.

---

# 32. Multi-Tenant JWT Validation ⭐⭐⭐⭐⭐

Multi-tenant systems require careful issuer/key selection.

Unsafe concept:

```text
JWT says issuer = X
 ↓
Trust X automatically
```

Safer architecture:

```text
Trusted tenant configuration
        ↓
Resolve expected issuer
        ↓
Resolve trusted keys
        ↓
Validate JWT
        ↓
Authorize tenant
```

Never let an attacker choose an arbitrary issuer/JWK endpoint through an untrusted token or request parameter.

---

# 33. JWT Revocation Problem ⭐⭐⭐⭐⭐

Self-contained JWTs can remain valid until expiration unless the resource server has an additional revocation strategy.

```text
JWT issued
   ↓
User disabled
   ↓
JWT may still be cryptographically valid
   ↓
Until expiry / revocation strategy
```

Common strategies:

- Short access-token lifetime
- Refresh-token revocation
- Introspection for opaque tokens
- Token denylist where justified
- User/session version checks where appropriate

### Trade-off

```text
Statelessness ↔ Immediate revocation
```

---

# 34. JWT vs Opaque Token ⭐⭐⭐⭐⭐

| JWT | Opaque Token |
|---|---|
| Self-contained claims | No directly readable claims |
| Often locally validated | Often introspected |
| Can reduce network calls | Central validation possible |
| Key rotation required | Central token state |
| Revocation can be harder | Revocation can be more immediate |

Neither is universally superior; choose based on architecture, security and operational requirements.

---

# 35. JWT Security Best Practices ⭐⭐⭐⭐⭐

```text
[ ] HTTPS everywhere
[ ] Short-lived access tokens
[ ] Validate signature
[ ] Validate issuer
[ ] Validate audience where applicable
[ ] Validate exp / nbf
[ ] Restrict accepted algorithms
[ ] Use trusted JWKS/discovery
[ ] Handle key rotation
[ ] Do not log tokens
[ ] Do not put secrets in JWT payload
[ ] Protect refresh tokens
[ ] Use least-privilege scopes
[ ] Avoid unnecessarily large JWTs
[ ] Consider revocation requirements
[ ] Protect token transport and storage
```

---

# 36. JWT Storage ⭐⭐⭐⭐⭐

There is no universally correct storage strategy; it depends on the application architecture and threat model.

For browser applications, blindly storing sensitive tokens in `localStorage` can increase exposure to token theft through XSS.

For traditional server-rendered applications, a secure, HttpOnly, appropriately configured session cookie can be preferable.

For browser-based OAuth/OIDC clients, follow the current security guidance of the chosen architecture/provider rather than copying a generic storage recipe.

### Golden rule

> **Minimize token exposure and protect against XSS, CSRF, token leakage and insecure transport.**

---

# 37. JWT and CSRF ⭐⭐⭐⭐⭐

JWT does not automatically eliminate CSRF.

The risk depends heavily on how the credential is transported.

```text
Authorization header
→ Browser does not automatically attach it cross-site like a cookie

Cookie-based authentication
→ Browser automatically sends cookie
→ CSRF protection may be required
```

Therefore:

> **“We use JWT, so CSRF is always disabled” is not a valid universal rule.**

---

# 38. JWT and CORS ⭐⭐⭐⭐⭐

JWT authentication and CORS solve different problems.

```text
JWT
→ Authentication credential/token

CORS
→ Browser cross-origin request policy
```

A valid JWT does not automatically bypass browser CORS restrictions.

---

# 39. JWT vs Session ⭐⭐⭐⭐⭐

| Session | JWT |
|---|---|
| Server maintains session state | Can be stateless at API validation layer |
| Session ID is sent | Token is sent |
| Easy centralized invalidation | Immediate revocation is harder for self-contained tokens |
| Server-side session store | Token/key management required |
| Common for browser apps | Common for APIs/distributed systems |

JWT is not automatically better than sessions.

---

# 40. JWT in Microservices ⭐⭐⭐⭐⭐

Typical architecture:

```text
                  Authorization Server
                         ↓
                    Access Token
                         ↓
Client → API Gateway → Service A
                         ↓
                     Service B
```

Each service can validate the token independently when appropriate.

### Important design question

Should internal services:

```text
Forward user access token
OR
Use service-to-service credentials
```

The answer depends on whether downstream services need end-user authorization context and the organization's security architecture.

---

# 41. API Gateway + JWT ⭐⭐⭐⭐⭐

A gateway may validate JWTs as an initial security boundary.

But downstream services should not blindly trust headers merely because the request passed through the gateway.

```text
Gateway
 ↓
JWT validation
 ↓
Service
 ↓
Appropriate authorization
```

Use authenticated, integrity-protected service communication and clear trust boundaries.

---

# 42. JWT Error Handling ⭐⭐⭐⭐⭐

Common failures:

```text
Missing token
      ↓
401 Unauthorized

Invalid token
      ↓
401 Unauthorized

Expired token
      ↓
401 Unauthorized

Valid token but insufficient authority
      ↓
403 Forbidden
```

### Memory

```text
401
→ Authentication missing/invalid

403
→ Authenticated but not authorized
```

Exact response behavior can be customized by Spring Security configuration.

---

# 43. Common JWT Vulnerabilities / Mistakes ⭐⭐⭐⭐⭐

### 1. Accepting `alg=none`

Never allow an insecure/unintended algorithm through weak configuration.

### 2. Algorithm confusion

Do not dynamically trust the algorithm declared by an attacker-controlled token.

### 3. Missing issuer validation

A token from another trusted-looking issuer could be accepted incorrectly.

### 4. Missing audience validation

A token intended for another API may be accepted.

### 5. Ignoring expiration

Expired tokens remain usable.

### 6. Putting secrets in payload

Anyone who can read the JWT can decode its payload.

### 7. Logging tokens

Logs become a credential leak source.

### 8. Hard-coded signing keys

Key rotation becomes difficult and insecure.

### 9. Trusting arbitrary JWKS URLs

Can lead to malicious key injection / SSRF-style architecture problems.

---

# 44. JWT Authentication vs JWT Authorization ⭐⭐⭐⭐⭐

JWT authentication:

```text
Who/what is presenting this credential?
```

JWT authorization:

```text
What is this authenticated principal allowed to do?
```

Spring Security flow:

```text
JWT validation
    ↓
Authentication
    ↓
Authorities
    ↓
Authorization decision
```

---

# 45. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — JWT is encrypted

❌ Wrong.

> A typical signed JWT is encoded and signed, not encrypted.

### Trap 2 — JWT = OAuth2

❌ Wrong.

> JWT is a token format; OAuth2 is an authorization framework.

### Trap 3 — Decode JWT and trust it

❌ Wrong.

> Signature and relevant claims must be validated.

### Trap 4 — JWT is always stateless

❌ Oversimplified.

> JWT can support stateless resource-server validation, but systems can still maintain state for refresh tokens, revocation, sessions, users, etc.

### Trap 5 — JWT removes CSRF

❌ Wrong.

> CSRF depends on how credentials are transported and the application architecture.

### Trap 6 — JWT solves XSS

❌ Wrong.

> XSS can steal browser-accessible credentials and execute authenticated actions.

### Trap 7 — ID Token can be used as API token

❌ Wrong in general.

> APIs should validate access tokens intended for them.

### Trap 8 — Any valid JWT is valid for every API

❌ Wrong.

> Validate intended issuer/audience and token purpose.

---

# 46. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is JWT?

> JWT is a compact token format containing claims that can be digitally signed and optionally encrypted; it is commonly used as a bearer access token.

### Q2. What are the three parts of JWT?

> Header, payload and signature.

### Q3. Is JWT encrypted?

> Not by default. A normal signed JWT protects integrity but its payload can generally be decoded. JWE is used for encryption.

### Q4. JWT vs OAuth2?

> JWT is a token format; OAuth2 is an authorization framework.

### Q5. What should a resource server validate?

> At minimum, the expected signature/key, issuer and relevant time claims, plus audience and other claims required by the application/provider contract.

### Q6. HS256 vs RS256?

> HS256 uses a shared secret for signing and verification; RS256 uses a private key for signing and public key for verification.

### Q7. Why is RS256 useful in microservices?

> Resource services can verify tokens using public keys without receiving the issuer's private signing key.

### Q8. What is JWKS?

> A JSON Web Key Set containing public keys used to verify signed tokens.

### Q9. What is `kid`?

> A key identifier in the JWT header that helps select the correct signing key from the provider's key set.

### Q10. Why is JWT revocation difficult?

> A self-contained token can remain cryptographically valid until expiry unless the architecture provides an additional revocation or validation mechanism.

### Q11. How does Spring Security validate JWT?

> The bearer token filter extracts the token, authentication is delegated through Spring Security, a JWT decoder verifies the signature and validators verify relevant claims, and the resulting Authentication is placed into the SecurityContext.

### Q12. JWT vs session?

> JWT can enable locally validated bearer credentials and distributed API validation, while sessions keep server-side session state. The right choice depends on architecture and security requirements.

---

# 47. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“JWT is a compact token format consisting of a header, payload and signature. It is commonly used as a bearer access token in OAuth2-based APIs, but JWT itself is not an authentication protocol. A signed JWT normally provides integrity and authenticity of the signed content; its payload is not confidential because it is encoded rather than encrypted. For a resource server, I would validate the signature using trusted key material, issuer, expiration and not-before where applicable, audience when required, and the accepted algorithm/key policy. In asymmetric signing such as RS256, the authorization server signs with a private key and resource services verify with public keys, usually discovered through JWKS, which also supports key rotation using `kid`. In Spring Security, BearerTokenAuthenticationFilter extracts the token, authentication is handled through the authentication manager/provider architecture, JwtDecoder validates the JWT, and the resulting Authentication is stored in the SecurityContext. JWT can make API validation stateless, but revocation, refresh tokens and other application state may still exist. I would also distinguish JWT from OAuth2, OIDC and sessions, and consider token leakage, XSS, CSRF, CORS, key rotation and least-privilege authorization.”**

---

# 48. 30-Second Hinglish Answer

> **“JWT ek token format hai, OAuth2 nahi. Iske three parts hote hain—header, payload aur signature. Normal JWT encrypted nahi hota, sirf signed/encoded hota hai, isliye payload mein secret data nahi rakhna chahiye. API mein JWT Bearer token ke through aata hai, Spring Security ka bearer-token filter usko extract karta hai, JwtDecoder signature aur claims validate karta hai, phir Authentication SecurityContext mein set hota hai. Production mein issuer, audience, expiry, algorithm aur signature validate karna important hai. RS256 mein private key token sign karti hai aur public key services token verify karti hain; JWKS aur `kid` key rotation mein help karte hain. JWT stateless API validation de sakta hai, lekin revocation/refresh-token state phir bhi ho sakti hai.”**

---

# 49. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
CLIENT
  │
  │ Authorization: Bearer JWT
  ↓
┌─────────────────────────────┐
│ Spring Security Filter Chain│
└─────────────────────────────┘
  │
  ↓
BearerTokenAuthenticationFilter
  │
  ↓
AuthenticationManager
  │
  ↓
JwtAuthenticationProvider
  │
  ↓
JwtDecoder
  │
  ├── Signature ✓
  ├── Issuer ✓
  ├── Audience ✓
  ├── Expiry ✓
  └── Other validators ✓
  │
  ↓
Authentication
  │
  ↓
SecurityContext
  │
  ↓
Authorization
  │
  ↓
CONTROLLER
```

### JWT signing memory

```text
Authorization Server
       │
Private Key
       ↓
    Sign JWT
       ↓
    JWT + kid
       ↓
Resource Server
       ↓
JWKS → Public Key
       ↓
Verify Signature
```

### Final memory line

> **JWT = Header + Payload + Signature → Bearer Token → Spring Security validates signature + claims → Authentication → Authorization.**

---

## Navigation

[← S4.1.22 — OpenID Connect (OIDC)](../22-OpenID-Connect-OIDC/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.24 — OAuth2 Resource Server**
