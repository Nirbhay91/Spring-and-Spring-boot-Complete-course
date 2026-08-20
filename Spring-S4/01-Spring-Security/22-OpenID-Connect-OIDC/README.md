# S4.1.22 — OpenID Connect (OIDC)

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is OpenID Connect? ⭐⭐⭐⭐⭐

OpenID Connect (OIDC) is an **identity layer built on top of OAuth 2.0**. It standardizes how a client application can authenticate a user and obtain information about that authenticated identity.

### Golden rule

```text
OAuth 2.0
→ Authorization
→ What can this client access?

OpenID Connect
→ Authentication + Identity
→ Who is the user?
```

OIDC uses OAuth 2.0 flows and adds standardized identity information, most importantly the **ID Token**.

---

# 2. Why OIDC? ⭐⭐⭐⭐⭐

OAuth2 by itself is primarily designed for delegated authorization.

A client may receive an access token, but OAuth2 alone does not standardize a user-login identity protocol.

OIDC adds:

- User authentication
- ID Token
- Standard identity claims
- UserInfo endpoint
- Discovery metadata
- Standard scopes such as `openid`

Conceptually:

```text
OAuth2
   ↓
Access to resources

OIDC
   ↓
User authentication + identity
```

---

# 3. OIDC Architecture ⭐⭐⭐⭐⭐

```text
                    User
                     ↓
              User Authentication
                     ↓
              OpenID Provider
                     ↓
        ┌────────────┴────────────┐
        ↓                         ↓
    ID Token                 Access Token
        ↓                         ↓
      Client                Resource Server
        ↓                         ↓
  User Identity            Protected API
```

### Main idea

The **OpenID Provider (OP)** authenticates the user and issues identity information. The client, called the **Relying Party (RP)**, consumes that identity information.

---

# 4. OIDC Roles ⭐⭐⭐⭐⭐

## 4.1 End-User

The human user who authenticates.

```text
User
```

## 4.2 Relying Party (RP)

The client application that relies on the identity information from the OpenID Provider.

```text
Web Application
SPA
Mobile App
Backend Application
```

## 4.3 OpenID Provider (OP)

The OAuth2 Authorization Server that supports OIDC authentication and issues OIDC identity information.

```text
Identity Provider
Authorization Server + OIDC capabilities
```

---

# 5. OpenID Provider vs Authorization Server ⭐⭐⭐⭐⭐

An OIDC provider builds on OAuth2 authorization-server functionality.

```text
Authorization Server
        ↓
OAuth2 authorization
        ↓
OIDC Provider
        ↓
OAuth2 + Authentication/Identity
```

### Interview line

> **An OpenID Provider is an OAuth 2.0 Authorization Server that also provides standardized user authentication and identity information according to OpenID Connect.**

---

# 6. ID Token ⭐⭐⭐⭐⭐

The ID Token is the central OIDC artifact.

It is a security token containing claims about the authentication event and the authenticated user/subject.

It is normally represented as a JWT.

Conceptually:

```text
ID Token
   ↓
JWT
   ↓
Header + Payload + Signature
```

### Important

> **ID Token is intended for the client/Relying Party. It is not a general-purpose API access token.**

---

# 7. ID Token vs Access Token ⭐⭐⭐⭐⭐

| ID Token | Access Token |
|---|---|
| OIDC | OAuth2 |
| Represents authentication/identity | Represents authorization to resources |
| Intended for client | Intended for resource server/API |
| Contains identity/authentication claims | Contains authorization information according to token format/provider |
| Not normally sent to APIs as API credential | Sent to resource APIs |

### Memory

```text
ID Token
→ Who is the user?

Access Token
→ What can the client access?
```

---

# 8. `openid` Scope ⭐⭐⭐⭐⭐

The `openid` scope is what distinguishes an OIDC authentication request from a normal OAuth2 authorization request.

Example:

```text
scope=openid profile email
```

Common OIDC scopes:

```text
openid
profile
email
address
phone
```

### Critical point

> **The `openid` scope requests OIDC processing and results in an ID Token according to the flow/provider configuration.**

---

# 9. Standard OIDC Scopes ⭐⭐⭐⭐

### `openid`

Required to request OIDC authentication.

### `profile`

Requests standard profile claims such as name-related information.

### `email`

Requests email-related claims.

### `address`

Requests address-related claims.

### `phone`

Requests phone-related claims.

The actual claims returned depend on provider policy and user consent.

---

# 10. OIDC Authorization Code Flow ⭐⭐⭐⭐⭐

Modern web applications commonly use Authorization Code Flow.

```text
Browser
   ↓
Client
   ↓
Authorization Request
   ↓
OpenID Provider
   ↓
User Login
   ↓
Consent / Authorization
   ↓
Authorization Code
   ↓
Client
   ↓
Token Endpoint
   ↓
ID Token + Access Token
   ↓
Authenticated Application
```

For public clients, **PKCE** should be used.

---

# 11. OIDC + PKCE ⭐⭐⭐⭐⭐

PKCE protects the authorization-code exchange.

```text
Client
 ↓
Generate code_verifier
 ↓
Generate code_challenge
 ↓
Authorization request
 ↓
OP
 ↓
Authorization code
 ↓
Token request + code_verifier
 ↓
OP verifies challenge
 ↓
Tokens
```

### Production rule

> **Use Authorization Code + PKCE for modern public clients rather than relying on legacy implicit authentication patterns.**

---

# 12. `state` Parameter ⭐⭐⭐⭐⭐

`state` binds the authorization request and callback together and provides protection against authorization-response CSRF attacks.

```text
Client generates state
       ↓
Authorization request
       ↓
OP
       ↓
Redirect callback + state
       ↓
Client validates state
```

The client must verify the returned value.

---

# 13. `nonce` Parameter ⭐⭐⭐⭐⭐

`nonce` is an important OIDC security parameter.

It associates the authentication request with the resulting ID Token and helps mitigate replay attacks.

```text
Authentication Request
        ↓
      nonce
        ↓
OpenID Provider
        ↓
ID Token contains nonce
        ↓
Client validates nonce
```

### `state` vs `nonce`

```text
state
→ Authorization request/callback correlation
→ CSRF protection

nonce
→ ID Token/authentication request correlation
→ Replay protection
```

---

# 14. ID Token Claims ⭐⭐⭐⭐⭐

Important registered claims can include:

```text
iss
sub
aud
exp
iat
nonce
```

### `iss` — Issuer

Identifies the OpenID Provider.

### `sub` — Subject

Stable identifier for the authenticated subject within the issuer context.

### `aud` — Audience

Identifies the intended audience, typically the client.

### `exp` — Expiration

Defines when the ID Token expires.

### `iat` — Issued At

Identifies when the token was issued.

### `nonce`

Binds the ID Token to the authentication request when used.

---

# 15. ID Token Validation ⭐⭐⭐⭐⭐

Never blindly trust an ID Token simply because it is JWT-shaped.

Typical validation includes:

```text
Signature
   ↓
Issuer
   ↓
Audience
   ↓
Expiration
   ↓
Issued-at/time rules where applicable
   ↓
Nonce (when used)
   ↓
Other provider/application requirements
```

### Critical interview point

> **JWT parsing is not JWT validation.**

---

# 16. Signature Validation ⭐⭐⭐⭐⭐

The client must verify that the ID Token was signed by the expected issuer using trusted key material.

Conceptually:

```text
ID Token
   ↓
Signature
   ↓
Provider public key / trusted key set
   ↓
Valid?
```

Never accept an arbitrary key supplied by an untrusted request.

---

# 17. Issuer Validation ⭐⭐⭐⭐⭐

The `iss` claim identifies the issuer.

The client should verify that it matches the expected OpenID Provider.

```text
Expected issuer
       ↓
Compare with ID Token iss
       ↓
Match?
```

This prevents accepting tokens from an unintended issuer.

---

# 18. Audience Validation ⭐⭐⭐⭐⭐

The `aud` claim identifies the intended audience.

For an ID Token, the client should verify that its own client identifier is an intended audience.

```text
ID Token
   ↓
aud = client-id
   ↓
Expected client?
```

A valid signature alone does not mean the token was issued for your application.

---

# 19. Expiration Validation ⭐⭐⭐⭐⭐

The client must reject an expired ID Token.

```text
exp < current time
      ↓
Reject
```

Clock skew may be handled according to the provider/library's configured validation rules.

---

# 20. UserInfo Endpoint ⭐⭐⭐⭐⭐

OIDC defines a UserInfo endpoint that can return claims about the authenticated user.

Typical request:

```http
GET /userinfo
Authorization: Bearer <access-token>
```

The access token authorizes access to UserInfo.

### Important distinction

```text
ID Token
→ Identity information delivered during authentication

UserInfo
→ User claims retrieved through an access token
```

---

# 21. ID Token vs UserInfo ⭐⭐⭐⭐⭐

| ID Token | UserInfo |
|---|---|
| JWT | HTTP endpoint response |
| Issued during token exchange | Retrieved using access token |
| Authentication event + identity claims | User claims |
| Validated as token | Protected API response |
| Intended for client | UserInfo resource accessed by client |

---

# 22. OIDC Discovery ⭐⭐⭐⭐⭐

OIDC provides a standardized discovery mechanism.

Typical endpoint:

```text
/.well-known/openid-configuration
```

The discovery document can provide metadata such as:

```text
authorization_endpoint
token_endpoint
userinfo_endpoint
jwks_uri
issuer
response_types_supported
scopes_supported
id_token_signing_alg_values_supported
```

The exact metadata depends on the provider.

---

# 23. `jwks_uri` ⭐⭐⭐⭐⭐

`jwks_uri` points to the provider's JSON Web Key Set endpoint.

Conceptually:

```text
OIDC Discovery
      ↓
jwks_uri
      ↓
JWKS
      ↓
Public signing keys
      ↓
ID Token signature validation
```

Applications should use trusted provider metadata and key rotation mechanisms rather than hard-coding temporary signing keys.

---

# 24. Key Rotation ⭐⭐⭐⭐⭐

OIDC providers can rotate signing keys.

```text
Old Key
   ↓
New Key
   ↓
JWKS publishes active keys
   ↓
Client refreshes key metadata
   ↓
New ID Tokens validated
```

### Production point

> **Do not assume the signing key is permanently fixed. Use the provider's supported JWKS/key-discovery mechanism.**

---

# 25. OpenID Provider Metadata ⭐⭐⭐⭐

Important metadata includes:

```text
issuer
authorization_endpoint
token_endpoint
userinfo_endpoint
jwks_uri
```

It can also advertise supported:

```text
scopes
claims
response types
response modes
signing algorithms
```

This reduces manual endpoint configuration and supports interoperability.

---

# 26. Claims ⭐⭐⭐⭐⭐

Claims are statements about an entity, commonly the authenticated user.

Examples:

```json
{
  "sub": "12345",
  "name": "Nirbhay",
  "email": "user@example.com"
}
```

### Important

Claims returned depend on scopes, provider configuration, consent and user information availability.

---

# 27. Claims vs Authorities ⭐⭐⭐⭐⭐

Do not automatically treat every identity claim as an authorization authority.

```text
Claim
→ User information

Authority
→ Permission used for authorization
```

Example:

```text
email = user@example.com

role = ADMIN

scope = orders.read
```

The application decides how claims/scopes are mapped to authorities.

---

# 28. OIDC Authentication Flow in Spring Security ⭐⭐⭐⭐⭐

Spring Security can act as an OAuth2/OIDC client.

Conceptually:

```text
Browser
 ↓
Spring Boot Application
 ↓
Authorization Request
 ↓
OpenID Provider
 ↓
Login
 ↓
Authorization Code
 ↓
Spring Boot callback
 ↓
Token Exchange
 ↓
ID Token validation
 ↓
User identity
 ↓
Authenticated session
```

---

# 29. Spring Security `oauth2Login()` ⭐⭐⭐⭐⭐

A Spring Boot application can use OAuth2 Login for user authentication.

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
        .oauth2Login(oauth2 -> {});

    return http.build();
}
```

The provider/client registration is configured separately.

---

# 30. OIDC Client Registration ⭐⭐⭐⭐⭐

Spring Boot configuration commonly contains provider/client information under:

```text
spring.security.oauth2.client.registration
spring.security.oauth2.client.provider
```

Conceptually:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          my-provider:
            client-id: ${OIDC_CLIENT_ID}
            client-secret: ${OIDC_CLIENT_SECRET}
            scope:
              - openid
              - profile
              - email
```

### Security rule

Never commit real client secrets to Git.

---

# 31. `openid` Scope in Spring Security ⭐⭐⭐⭐⭐

When `openid` is included in the requested scopes, Spring Security can recognize the provider as an OIDC provider and use OIDC-specific behavior.

Conceptually:

```text
scope
 ↓
openid
 ↓
OIDC authentication
 ↓
ID Token / OIDC User
```

---

# 32. `OidcUser` ⭐⭐⭐⭐⭐

Spring Security provides `OidcUser` to represent an authenticated OIDC user.

Conceptually:

```java
OidcUser oidcUser = ...;

String subject = oidcUser.getSubject();
String email = oidcUser.getEmail();
```

`OidcUser` exposes identity information obtained through the OIDC authentication flow.

---

# 33. `OidcUserService` ⭐⭐⭐⭐⭐

`OidcUserService` is used by Spring Security to load OIDC user information, including interaction with the UserInfo endpoint when configured/required.

Conceptually:

```text
OIDC Authentication
      ↓
ID Token
      ↓
OidcUserService
      ↓
UserInfo (when applicable)
      ↓
OidcUser
```

It can be customized for application-specific user mapping.

---

# 34. `OAuth2User` vs `OidcUser` ⭐⭐⭐⭐⭐

```text
OAuth2User
→ Generic OAuth2 user representation

OidcUser
→ OIDC-specific authenticated user representation
→ Includes OIDC identity/token-related information
```

`OidcUser` extends the OAuth2 user model used by Spring Security.

---

# 35. Login Success Flow ⭐⭐⭐⭐⭐

```text
Authentication successful
        ↓
OidcUser created
        ↓
Authentication object
        ↓
SecurityContext
        ↓
HTTP Session (for typical stateful web login)
        ↓
Authenticated request
```

For stateless architectures, session persistence and token handling are designed differently.

---

# 36. OIDC Login vs OAuth2 Resource Server ⭐⭐⭐⭐⭐

### OIDC Login

```text
Browser
 ↓
Identity Provider
 ↓
User authentication
 ↓
Application session
```

### Resource Server

```text
Client
 ↓
Bearer Access Token
 ↓
API
 ↓
Token validation
```

A Spring application can implement one or both patterns depending on architecture.

---

# 37. OIDC Logout ⭐⭐⭐⭐⭐

OIDC logout can involve more than destroying the application's local session.

Conceptually:

```text
Application logout
      ↓
Local session cleared
      ↓
Optional provider logout
      ↓
Identity Provider session cleared
```

The exact behavior depends on the OIDC provider and supported logout specifications/features.

### Important

> **Local application logout does not automatically guarantee that the user's identity-provider session has ended.**

---

# 38. RP-Initiated Logout ⭐⭐⭐⭐

OIDC providers may support Relying Party initiated logout.

Conceptually:

```text
Application
   ↓
Logout request
   ↓
OpenID Provider
   ↓
Provider session logout
   ↓
Redirect back
```

Provider support and exact parameters must be verified against the provider's current OIDC implementation.

---

# 39. OIDC Security Best Practices ⭐⭐⭐⭐⭐

```text
[ ] HTTPS everywhere
[ ] Authorization Code + PKCE for public clients
[ ] Validate ID Token signature
[ ] Validate issuer
[ ] Validate audience
[ ] Validate expiration
[ ] Validate nonce when used
[ ] Validate state
[ ] Protect redirect URIs
[ ] Do not log ID/access/refresh tokens
[ ] Protect client credentials
[ ] Use secure session/cookie configuration
[ ] Minimize requested scopes
[ ] Use trusted discovery/JWKS metadata
[ ] Handle key rotation
[ ] Require step-up authentication for high-risk actions where appropriate
```

---

# 40. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — OIDC replaces OAuth2

❌ Wrong.

> OIDC is an identity layer built on OAuth2.

### Trap 2 — ID Token is an API access token

❌ Wrong.

> APIs normally receive access tokens.

### Trap 3 — JWT decoding means token is trusted

❌ Wrong.

> Signature and claims must be validated.

### Trap 4 — `state` and `nonce` are the same

❌ Wrong.

> `state` correlates authorization requests/callbacks; `nonce` binds the authentication request to the ID Token and helps mitigate replay.

### Trap 5 — `openid` is just another API permission

❌ Incomplete.

> It signals an OIDC authentication request.

### Trap 6 — ID Token can be sent to any microservice

❌ Wrong in general.

> Resource APIs should use access tokens intended for them.

### Trap 7 — One signing key forever

❌ Wrong.

> Providers can rotate signing keys; use JWKS/discovery mechanisms.

### Trap 8 — Local logout always logs out of Google/IdP

❌ Wrong.

> Provider logout must be handled separately when supported and required.

---

# 41. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is OIDC?

> OpenID Connect is an identity/authentication layer built on OAuth2 that standardizes user authentication and identity information.

### Q2. OAuth2 vs OIDC?

> OAuth2 is primarily authorization; OIDC adds authentication and standardized identity claims.

### Q3. What is an ID Token?

> An OIDC security token, normally a JWT, containing claims about the authentication event and authenticated subject, intended for the client.

### Q4. ID Token vs Access Token?

> ID Token is for client-side identity/authentication; access token authorizes access to protected resources.

### Q5. Why is `openid` scope important?

> It requests OIDC authentication rather than only OAuth2 authorization.

### Q6. What is `nonce`?

> A value that binds an OIDC authentication request to the resulting ID Token and helps mitigate replay attacks.

### Q7. What is OIDC Discovery?

> A standardized metadata mechanism, commonly exposed through `/.well-known/openid-configuration`, that describes provider endpoints and capabilities.

### Q8. What is `jwks_uri`?

> The endpoint advertised by provider metadata where clients can obtain public keys used to validate signed tokens.

### Q9. What is `OidcUser` in Spring Security?

> A Spring Security representation of an authenticated OIDC user containing OIDC identity information.

### Q10. What is `oauth2Login()`?

> A Spring Security configuration mechanism that lets an application authenticate users through an OAuth2/OIDC provider.

---

# 42. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“OpenID Connect is an identity layer built on top of OAuth2. OAuth2 mainly provides delegated authorization, while OIDC standardizes user authentication and identity information. The main OIDC artifact is the ID Token, normally a JWT, which is intended for the client and contains claims such as issuer, subject, audience and expiration. The access token has a different purpose: it authorizes access to protected resources and is normally sent to the resource server. An OIDC authentication request uses the `openid` scope, and modern applications commonly use Authorization Code with PKCE. `state` protects the authorization request/callback correlation, while `nonce` binds the authentication request to the ID Token and helps mitigate replay. Clients must validate the ID Token signature, issuer, audience, expiration and nonce where applicable. OIDC also provides discovery metadata and JWKS for endpoint and signing-key discovery. In Spring Security, `oauth2Login()` can authenticate users, `OidcUser` represents the authenticated OIDC user, and `OidcUserService` handles loading OIDC user information. In production I would protect redirect URIs, secrets and tokens, use HTTPS, handle key rotation, and distinguish local logout from identity-provider logout.”**

---

# 43. 30-Second Hinglish Answer

> **“OIDC OAuth2 ke upar identity layer hai. OAuth2 mainly authorization ke liye hota hai—client kya access kar sakta hai—while OIDC batata hai user kaun hai. OIDC ka main artifact ID Token hai, normally JWT, jo client ko user/authentication information deta hai. Access Token API/resource server ke liye hota hai. `openid` scope OIDC authentication request ko indicate karta hai. `state` callback correlation/CSRF protection ke liye aur `nonce` ID Token ko authentication request se bind karke replay protection mein help karta hai. ID Token ko blindly decode nahi karna; signature, issuer, audience, expiry aur applicable nonce validate karna hota hai. Spring Security mein `oauth2Login()`, `OidcUser` aur `OidcUserService` important hain.”**

---

# 44. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                    USER
                     ↓
                 Login
                     ↓
              OpenID Provider
                     ↓
          Authorization Code + PKCE
                     ↓
                Token Endpoint
                     ↓
          ┌──────────┴──────────┐
          ↓                     ↓
      ID Token              Access Token
          ↓                     ↓
     CLIENT / RP          RESOURCE SERVER
          ↓                     ↓
    User Identity         Protected API
```

### Security validation memory

```text
ID Token
   ↓
Signature
   ↓
Issuer
   ↓
Audience
   ↓
Expiry
   ↓
Nonce
   ↓
Authenticated User
```

### Final memory line

> **OIDC = OAuth2 + Identity → `openid` scope → ID Token → validate signature/issuer/audience/expiry/nonce → authenticated user.**

---

## Navigation

[← S4.1.21 — OAuth2 Fundamentals](../21-OAuth2-Fundamentals/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.23 — JWT Authentication**
