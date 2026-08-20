# S4.1.21 — OAuth2 Fundamentals

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is OAuth 2.0? ⭐⭐⭐⭐⭐

OAuth 2.0 is an **authorization framework** that allows a client application to obtain limited access to protected resources on behalf of a resource owner without requiring the client to handle the user's password.

### Golden rule

> **OAuth 2.0 is primarily about authorization/delegated access, not authentication.**

For user authentication, OpenID Connect (OIDC) builds an identity layer on top of OAuth 2.0.

---

# 2. Why OAuth2? ⭐⭐⭐⭐⭐

Without delegated authorization:

```text
User
 ↓
Gives password to third-party application
 ↓
Third-party application accesses account
```

This is dangerous because the third-party application receives credentials.

With OAuth2:

```text
User
 ↓
Authorization Server
 ↓
Grant limited permission
 ↓
Access Token
 ↓
Client Application
 ↓
Resource Server
```

The client receives a token rather than the user's password.

---

# 3. OAuth2 Core Roles ⭐⭐⭐⭐⭐

OAuth2 defines four important roles.

## 3.1 Resource Owner

The entity that owns the protected resource and can grant access.

Usually:

```text
User
```

## 3.2 Client

The application requesting access to the protected resource.

Examples:

```text
Web application
Mobile application
Backend service
SPA
```

## 3.3 Authorization Server

Authenticates the resource owner and issues tokens.

Examples in real systems can include an enterprise identity provider or an OAuth authorization platform.

## 3.4 Resource Server

Hosts protected resources and validates access tokens.

Example:

```text
GET /api/orders
Authorization: Bearer <access-token>
```

---

# 4. OAuth2 Architecture ⭐⭐⭐⭐⭐

```text
                Resource Owner
                      │
                      │ Authorization
                      ↓
                Authorization Server
                      │
                      │ Access Token
                      ↓
Client ───────────────→ Resource Server
       Bearer Token        │
                           ↓
                     Protected Resource
```

### Memory

```text
Authorization Server → Token issue
Resource Server      → Token validate + resource
Client               → Token use
Resource Owner       → Permission grant
```

---

# 5. Access Token ⭐⭐⭐⭐⭐

An access token represents authorization to access protected resources.

Example request:

```http
GET /api/orders
Authorization: Bearer eyJ...
```

The resource server uses the access token to determine whether the request is authorized.

### Important

> The client should treat an access token as a credential and protect it accordingly.

---

# 6. Bearer Token ⭐⭐⭐⭐⭐

A bearer token means whoever possesses the token can potentially use it according to its scope and validity.

```text
Token stolen
    ↓
Attacker possesses token
    ↓
Attacker may call resource server
```

Therefore:

- Use HTTPS.
- Keep token lifetime appropriate.
- Minimize scopes.
- Protect token storage.
- Avoid logging tokens.

---

# 7. Scope ⭐⭐⭐⭐⭐

Scopes limit what the client is authorized to access.

Example:

```text
orders:read
orders:write
profile:read
```

Conceptually:

```text
Access Token
   ↓
Scopes
   ↓
Allowed operations
```

### Principle

> **Least privilege: request only the scopes the application actually needs.**

---

# 8. Authorization Code ⭐⭐⭐⭐⭐

An authorization code is a short-lived credential returned by the authorization server to the client after the authorization step.

It is then exchanged for tokens.

```text
Browser
  ↓
Authorization request
  ↓
Authorization Server
  ↓
Authorization Code
  ↓
Client
  ↓
Token endpoint
  ↓
Access Token
```

The code is not normally used directly to access APIs.

---

# 9. Authorization Code Flow ⭐⭐⭐⭐⭐

Conceptual flow:

```text
1. Client redirects user to Authorization Server

2. User authenticates

3. User grants consent if required

4. Authorization Server returns code

5. Client exchanges code for tokens

6. Client calls Resource Server using access token

7. Resource Server validates token

8. Protected resource returned
```

---

# 10. PKCE ⭐⭐⭐⭐⭐

**Proof Key for Code Exchange (PKCE)** protects the authorization-code flow against authorization-code interception.

Two values are created by the client:

```text
code_verifier
      ↓
code_challenge
```

Conceptually:

```text
Client
 ↓
Generate code_verifier
 ↓
Create code_challenge
 ↓
Authorization request
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Token request + code_verifier
 ↓
Server verifies challenge
 ↓
Tokens
```

### Interview point

> **PKCE is especially important for public clients and is now a standard best practice for authorization-code flows.**

---

# 11. State Parameter ⭐⭐⭐⭐⭐

The `state` parameter helps the client maintain request/response correlation and protect against CSRF-style authorization response attacks.

```text
Client generates state
      ↓
Authorization request
      ↓
Authorization Server
      ↓
Redirect with state
      ↓
Client verifies state
```

The client should validate that the returned state matches the expected value.

---

# 12. Nonce and OAuth2 vs OIDC ⭐⭐⭐⭐

`nonce` is primarily an OpenID Connect concept used to associate an authentication request with the resulting ID token.

Do not confuse:

```text
OAuth2 state
→ request correlation / CSRF protection

OIDC nonce
→ replay/request correlation for ID token authentication
```

---

# 13. Refresh Token ⭐⭐⭐⭐⭐

A refresh token can be used to obtain a new access token without requiring the resource owner to repeat the full authorization flow, subject to the authorization server's policies.

```text
Short-lived Access Token
          ↓ expires
Refresh Token
          ↓
Token Endpoint
          ↓
New Access Token
```

### Security principle

Refresh tokens are highly sensitive and generally deserve stronger protection than short-lived access tokens.

---

# 14. Access Token vs Refresh Token ⭐⭐⭐⭐⭐

| Feature | Access Token | Refresh Token |
|---|---|---|
| Purpose | Call resource server | Obtain new access token |
| Lifetime | Usually shorter | Usually longer |
| Sent to API | Yes | Normally no |
| Exposure | Higher frequency | Should be minimized |
| Protection | Strong | Very strong |
| Scope | Authorizes resources | Used at token endpoint |

### Golden rule

> **Never send a refresh token to a normal resource API unless the protocol/application explicitly requires it.**

---

# 15. Client Authentication ⭐⭐⭐⭐⭐

Confidential clients can authenticate to the authorization server.

Examples can include:

```text
client_secret
private_key_jwt
mTLS
```

Public clients cannot safely keep a client secret, such as browser-based applications and many mobile apps.

### Important

> Never embed a confidential client secret in a browser SPA or mobile application's distributed code.

---

# 16. Confidential vs Public Client ⭐⭐⭐⭐⭐

### Confidential client

Can securely maintain credentials in a trusted environment.

```text
Backend server
```

### Public client

Cannot securely keep a secret.

```text
SPA
Mobile app
Desktop app
```

---

# 17. Authorization Code + PKCE for Modern Applications ⭐⭐⭐⭐⭐

Typical modern user-login architecture:

```text
Browser / Mobile
       ↓
Authorization Code + PKCE
       ↓
Authorization Server
       ↓
Code
       ↓
Token Endpoint
       ↓
Access Token / ID Token / Refresh Token
```

The exact tokens returned depend on OAuth/OIDC configuration.

---

# 18. Client Credentials Grant ⭐⭐⭐⭐⭐

Used for machine-to-machine authorization where there is no end-user authorization step.

```text
Service A
   ↓
Client credentials
   ↓
Authorization Server
   ↓
Access Token
   ↓
Service B / Resource Server
```

Typical use case:

```text
Microservice → Microservice
```

### Important

Do not use Client Credentials when a user must authorize access.

---

# 19. Resource Owner Password Credentials Grant ⭐⭐⭐⭐

The password grant was historically defined in OAuth2 but is **not recommended for new applications**.

Reason:

```text
Client receives user password
        ↓
Breaks delegated-authorization model
        ↓
Higher credential exposure
```

Modern applications should use authorization-code based flows with PKCE where applicable.

---

# 20. Implicit Grant ⭐⭐⭐⭐

The implicit grant was historically used by browser applications but is **not recommended for modern applications**.

Modern browser applications should generally use:

```text
Authorization Code + PKCE
```

---

# 21. OAuth2 Grant Types — Interview View ⭐⭐⭐⭐⭐

```text
Authorization Code + PKCE
→ User-facing modern applications

Client Credentials
→ Machine-to-machine

Password Grant
→ Legacy / avoid for new systems

Implicit Grant
→ Legacy / avoid for new systems
```

---

# 22. Authorization Endpoint ⭐⭐⭐⭐⭐

The authorization endpoint handles the user authorization interaction.

Conceptually:

```text
GET /authorize
```

Typical parameters include:

```text
response_type=code
client_id=...
redirect_uri=...
scope=...
state=...
code_challenge=...
code_challenge_method=S256
```

Exact parameters depend on the flow/provider.

---

# 23. Token Endpoint ⭐⭐⭐⭐⭐

The token endpoint exchanges an authorization grant for tokens.

Conceptually:

```text
POST /token
```

For authorization code + PKCE:

```text
code
redirect_uri
client_id (where applicable)
code_verifier
```

The server validates the request and issues tokens according to policy.

---

# 24. Redirect URI ⭐⭐⭐⭐⭐

The redirect URI is where the authorization server sends the authorization response.

### Security rule

> **Redirect URIs should be registered and validated strictly.**

Avoid overly broad patterns.

Bad concept:

```text
https://example.com/*
```

Prefer exact registered redirect URIs where practical.

---

# 25. Authorization Server vs Resource Server ⭐⭐⭐⭐⭐

This distinction is extremely important in Spring Security.

### Authorization Server

```text
Authenticate/authorize
        ↓
Issue tokens
```

### Resource Server

```text
Receive access token
        ↓
Validate token
        ↓
Check authorities/scopes
        ↓
Return resource
```

One system can contain multiple roles, but architecturally they are distinct responsibilities.

---

# 26. OAuth2 Authentication vs Authorization ⭐⭐⭐⭐⭐

OAuth2 itself primarily answers:

> **“What is this client allowed to access?”**

OIDC adds:

> **“Who is the user?”**

Therefore:

```text
OAuth2
→ Authorization

OpenID Connect
→ Authentication / Identity layer
```

---

# 27. OAuth2 vs OpenID Connect ⭐⭐⭐⭐⭐

| OAuth2 | OpenID Connect |
|---|---|
| Authorization framework | Identity/authentication layer |
| Access Token | ID Token + Access Token |
| API/resource access | User authentication |
| Scope-based permissions | Identity claims |
| Resource Server | Client/Relying Party + Provider |

### Interview line

> **“OAuth2 tells us what a client can access; OIDC tells the client who authenticated.”**

---

# 28. ID Token ⭐⭐⭐⭐⭐

An ID token belongs to OpenID Connect, not base OAuth2.

It contains claims about the authenticated subject and is intended for the client.

Typical claims can include:

```text
iss
sub
aud
exp
iat
nonce
```

### Important

> **Do not use an ID token as a replacement for an access token when calling an API unless the architecture explicitly defines that behavior.**

---

# 29. JWT Access Token ⭐⭐⭐⭐⭐

An access token may be JWT-based, but OAuth2 does not require JWT access tokens.

```text
OAuth2
   ↓
Access Token
   ↓
Could be JWT
OR
Opaque token
```

This is an important interview distinction.

---

# 30. JWT vs Opaque Access Token ⭐⭐⭐⭐⭐

### JWT

Resource server can validate token claims/signature locally when it has the required key material.

```text
Token
 ↓
Signature verification
 ↓
Claims
 ↓
Authorization
```

### Opaque token

Resource server may call an introspection endpoint to determine token state.

```text
Opaque Token
 ↓
Introspection
 ↓
Authorization Server
 ↓
Active + claims
 ↓
Resource Server
```

---

# 31. Token Introspection ⭐⭐⭐⭐⭐

Introspection allows a resource server to ask the authorization server about an opaque token.

Conceptually:

```text
Resource Server
      ↓
POST /introspect
      ↓
Authorization Server
      ↓
active=true
scope=orders:read
sub=123
      ↓
Resource Server
```

Useful when centralized token state/revocation is important.

---

# 32. Token Revocation ⭐⭐⭐⭐⭐

Revocation invalidates a token before its natural expiration, subject to the authorization server's implementation and token type.

Conceptually:

```text
Refresh Token
     ↓
Revocation endpoint
     ↓
Authorization Server
     ↓
Token invalidated
```

Access-token revocation behavior depends on whether tokens are self-contained, opaque, short-lived, and how the authorization server implements revocation.

---

# 33. Scope vs Role ⭐⭐⭐⭐⭐

These are related but not identical.

```text
Scope
→ OAuth client permission

Role
→ Application/domain authorization concept
```

Example:

```text
scope = orders.read
role  = ADMIN
```

A Spring application may map scopes to authorities, but this is an application design decision.

---

# 34. OAuth2 Security Best Practices ⭐⭐⭐⭐⭐

```text
[ ] HTTPS everywhere
[ ] Authorization Code + PKCE for user-facing public clients
[ ] Exact redirect URI validation
[ ] Short-lived access tokens
[ ] Protect refresh tokens strongly
[ ] Least-privilege scopes
[ ] Do not log tokens
[ ] Do not put client secrets in SPAs/mobile apps
[ ] Validate issuer/audience/signature/expiry as appropriate
[ ] Use secure state handling
[ ] Use OIDC for authentication when user identity is required
[ ] Rotate/revoke credentials according to risk
```

---

# 35. OAuth2 in Microservices ⭐⭐⭐⭐⭐

Typical architecture:

```text
                    Authorization Server
                           │
                           │ Access Token
                           ↓
Client ───────────────→ API Gateway
                           │
                           ↓
                    Resource Services
```

Each resource service can validate the access token and enforce authorization according to its responsibilities.

---

# 36. Spring Security Resource Server ⭐⭐⭐⭐⭐

Spring Security can configure a Spring Boot application as an OAuth2 Resource Server.

Conceptually:

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

The actual configuration depends on the authorization server and Spring Security version.

---

# 37. JWT Resource Server Flow ⭐⭐⭐⭐⭐

```text
Client
  ↓
Authorization: Bearer <JWT>
  ↓
Spring Security Filter Chain
  ↓
Bearer Token Authentication
  ↓
JWT Decoder / Validator
  ↓
Signature + claims validation
  ↓
Authentication
  ↓
Authorization
  ↓
Controller
```

Important validation typically includes appropriate checks for:

```text
Signature
Issuer
Audience (when required)
Expiration
Not-before / timestamps where applicable
```

---

# 38. Spring Security Authority Mapping ⭐⭐⭐⭐⭐

OAuth scopes can become Spring Security authorities.

Conceptually:

```text
scope = orders.read
       ↓
SCOPE_orders.read
       ↓
Spring Authorization decision
```

Example:

```java
@PreAuthorize("hasAuthority('SCOPE_orders.read')")
```

Exact authority mapping can be customized.

---

# 39. Client Registration ⭐⭐⭐⭐⭐

For an OAuth2 client, Spring Boot can represent provider/client configuration through client registration.

Conceptually:

```text
spring.security.oauth2.client.registration
        ↓
client-id
client-secret (confidential clients)
scope
redirect URI
provider
```

Secrets should come from secure configuration/secret management, not source control.

---

# 40. OAuth2 Login vs Resource Server ⭐⭐⭐⭐⭐

### OAuth2 Login

Application acts as an OAuth2/OIDC client and signs the user in.

```text
Browser
 ↓
Identity Provider
 ↓
Login
 ↓
Application
```

### Resource Server

Application protects APIs and validates bearer access tokens.

```text
Client
 ↓
Bearer Access Token
 ↓
API
 ↓
Token validation
```

These are different Spring Security use cases.

---

# 41. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — OAuth2 is authentication

❌ Incomplete.

> OAuth2 is primarily an authorization framework. OIDC provides authentication/identity on top of OAuth2.

### Trap 2 — OAuth2 always uses JWT

❌ Wrong.

> Access tokens can be JWTs or opaque tokens.

### Trap 3 — ID Token should be sent to APIs

❌ Wrong in general.

> APIs normally receive access tokens. ID tokens are intended for the client in OIDC.

### Trap 4 — Client secret can be embedded in React

❌ Wrong.

> Browser SPAs are public clients and cannot safely protect a static secret.

### Trap 5 — Implicit flow is best for SPA

❌ Outdated.

> Authorization Code + PKCE is the modern approach.

### Trap 6 — Password grant should be used because it is simple

❌ Bad practice for new systems.

> Avoid legacy password grant; use modern authorization flows.

### Trap 7 — JWT means no validation is needed

❌ Wrong.

> Signature, issuer, audience where required, expiration and other relevant claims must be validated.

### Trap 8 — Scope and role are identical

❌ Wrong.

> Scope is an OAuth authorization concept; roles are generally application/domain authorization concepts.

---

# 42. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is OAuth2?

> OAuth2 is an authorization framework that enables delegated access to protected resources without sharing the resource owner's password with the client.

### Q2. What are OAuth2's four roles?

> Resource Owner, Client, Authorization Server and Resource Server.

### Q3. Access Token vs ID Token?

> Access tokens authorize access to resources; ID tokens are an OpenID Connect authentication artifact intended for the client.

### Q4. Why PKCE?

> PKCE binds the authorization-code exchange to the client-generated proof key and protects the flow against authorization-code interception, especially for public clients.

### Q5. What is Client Credentials?

> A machine-to-machine OAuth flow where a client obtains an access token using its own credentials without an end-user authorization step.

### Q6. JWT vs opaque token?

> A JWT can often be validated locally using its signature and claims, while an opaque token commonly requires introspection or another authorization-server interaction.

### Q7. OAuth2 vs OIDC?

> OAuth2 provides authorization; OIDC adds standardized user authentication and identity claims on top of OAuth2.

### Q8. What is scope?

> Scope represents the permissions the client requests/receives for accessing protected resources.

### Q9. Why shouldn't a SPA have a client secret?

> Because the browser cannot securely keep a static secret; the code and configuration are ultimately accessible to the user.

### Q10. What is Spring Security Resource Server?

> It allows a Spring application to protect APIs and authenticate bearer-token requests by validating OAuth2 access tokens, commonly JWTs or opaque tokens.

---

# 43. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“OAuth2 is an authorization framework that allows a client to obtain limited access to protected resources without sharing the user's password with that client. Its four core roles are Resource Owner, Client, Authorization Server and Resource Server. The authorization server issues access tokens, and the resource server validates those tokens and enforces authorization. For modern user-facing applications, Authorization Code with PKCE is the preferred flow, while Client Credentials is commonly used for machine-to-machine communication. OAuth2 access tokens can be JWTs or opaque tokens; OAuth2 itself does not require JWT. When user authentication and identity are required, OpenID Connect adds an identity layer and provides an ID token. In Spring Security, an API can be configured as an OAuth2 Resource Server to validate bearer access tokens and map scopes to authorities. In production I would enforce HTTPS, use exact redirect URI validation, least-privilege scopes, short-lived access tokens, strong refresh-token protection, secure state/PKCE handling, and never log tokens or expose client secrets in public clients.”**

---

# 44. 30-Second Hinglish Answer

> **“OAuth2 mainly authorization ke liye hai, authentication ke liye directly nahi. Isme Resource Owner, Client, Authorization Server aur Resource Server four main roles hain. Authorization Server access token issue karta hai aur Resource Server token validate karke API access deta hai. Modern user login ke liye Authorization Code + PKCE important hai, aur service-to-service communication ke liye Client Credentials use hota hai. OAuth2 token JWT hona mandatory nahi hai; opaque token bhi ho sakta hai. Agar user ki identity authenticate karni hai to OIDC use hota hai, jisme ID Token milta hai. Spring Security mein Resource Server bearer access token validate karke scopes ko authorities mein map kar sakta hai.”**

---

# 45. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                 USER
            Resource Owner
                  ↓
            Authorization
                  ↓
        ┌─────────────────────┐
        │ Authorization       │
        │ Server              │
        └─────────────────────┘
                  ↓
             Access Token
                  ↓
               CLIENT
                  ↓
        Authorization: Bearer
                  ↓
        ┌─────────────────────┐
        │ Resource Server     │
        └─────────────────────┘
                  ↓
           Protected API
```

### Modern user flow

```text
Client
 ↓
Authorization Code + PKCE
 ↓
Authorization Server
 ↓
Code
 ↓
Token Endpoint
 ↓
Access Token
 ↓
Resource Server
```

### Machine-to-machine

```text
Service A
 ↓
Client Credentials
 ↓
Authorization Server
 ↓
Access Token
 ↓
Service B
```

### Final memory line

> **OAuth2 = Delegated Authorization → Authorization Server issues Access Token → Resource Server validates it → OIDC adds User Authentication/Identity.**

---

## Navigation

[← S4.1.20 — Security Headers](../20-Security-Headers/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.22 — OpenID Connect (OIDC)**
