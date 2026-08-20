# S4.1.24 — OAuth2 Resource Server

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is an OAuth2 Resource Server? ⭐⭐⭐⭐⭐

An **OAuth2 Resource Server** is an application/API that hosts protected resources and accepts access tokens issued by a trusted Authorization Server.

```text
Client
   ↓
Access Token
   ↓
Resource Server
   ↓
Validate Token
   ↓
Authorize Request
   ↓
Protected Resource
```

### Golden rule

```text
Authorization Server
→ Issues tokens

Resource Server
→ Validates tokens + protects APIs

Client
→ Obtains and presents tokens
```

A resource server does **not** normally authenticate the user by showing a login page. Its primary responsibility is validating the presented access token and enforcing authorization.

---

# 2. OAuth2 Roles ⭐⭐⭐⭐⭐

```text
Resource Owner
      ↓
Client
      ↓
Authorization Server
      ↓
Access Token
      ↓
Resource Server
```

### Resource Owner

Entity that owns the protected resource, commonly the user.

### Client

Application requesting access to a protected resource.

### Authorization Server

Issues access tokens after the appropriate authorization process.

### Resource Server

Hosts protected resources and accepts valid access tokens.

---

# 3. Resource Server vs Authorization Server ⭐⭐⭐⭐⭐

| Authorization Server | Resource Server |
|---|---|
| Issues tokens | Validates access tokens |
| Authenticates/authorizes clients/users depending on flow | Protects APIs/resources |
| Provides token endpoint | Provides protected API endpoints |
| Manages authorization policies | Enforces resource-level authorization |
| Publishes discovery/JWKS when applicable | Consumes trusted metadata/keys |

### Interview line

> **The Authorization Server issues an access token; the Resource Server validates that token and decides whether the caller can access a protected resource.**

---

# 4. Resource Server Authentication vs Authorization ⭐⭐⭐⭐⭐

Resource server request processing can be remembered as:

```text
Authentication
→ Is this access token valid?

Authorization
→ Does this authenticated principal have permission?
```

Spring Security:

```text
Bearer Token
   ↓
Authentication
   ↓
Authorities / Scopes
   ↓
Authorization Decision
```

---

# 5. Protected API Flow ⭐⭐⭐⭐⭐

```text
Client
  │
  │ Authorization: Bearer <access-token>
  ↓
Spring Security Filter Chain
  ↓
BearerTokenAuthenticationFilter
  ↓
AuthenticationManager
  ↓
JwtAuthenticationProvider
  ↓
JwtDecoder
  ↓
JWT Validation
  ↓
Authentication
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
```

This connects directly with **S4.1.23 — JWT Authentication**.

---

# 6. Spring Security OAuth2 Resource Server ⭐⭐⭐⭐⭐

Spring Security provides Resource Server support for OAuth2 bearer tokens.

Typical Java configuration:

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

The exact configuration depends on whether the application validates JWTs using issuer discovery, a JWK Set URI, or an explicitly configured decoder.

---

# 7. Spring Boot Dependencies ⭐⭐⭐⭐⭐

For a JWT-based resource server, the application commonly uses the Spring Security OAuth2 Resource Server support and JWT support.

Conceptually:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

Spring Boot manages compatible dependency versions through its dependency management.

---

# 8. `issuer-uri` ⭐⭐⭐⭐⭐

A common production configuration is:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://issuer.example.com
```

The resource server can use the trusted issuer's metadata to discover the appropriate endpoints and signing keys and validate issuer information.

Conceptually:

```text
issuer-uri
    ↓
OIDC / OAuth Provider Metadata
    ↓
jwks_uri
    ↓
Public Keys
    ↓
JWT Validation
```

### Security rule

Never take the issuer from the incoming request and trust it dynamically.

---

# 9. `jwk-set-uri` ⭐⭐⭐⭐

A resource server can be configured with an explicit JWK Set URI when appropriate:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: https://issuer.example.com/.well-known/jwks.json
```

Conceptually:

```text
JWT
 ↓
kid
 ↓
JWK Set
 ↓
Public Key
 ↓
Signature Validation
```

### `issuer-uri` vs `jwk-set-uri`

```text
issuer-uri
→ Provider identity + metadata discovery
→ Issuer validation

jwk-set-uri
→ Direct key-set location
```

Choose the configuration according to the provider and deployment architecture.

---

# 10. Startup Behavior and Discovery ⭐⭐⭐⭐

When issuer-based discovery is used, the application can depend on the authorization server's metadata and key endpoints during startup and/or key retrieval.

### Production consideration

Understand your provider availability and startup dependencies. Avoid designing an application that cannot start or rotate keys safely because discovery infrastructure is unavailable.

The exact startup/runtime behavior depends on Spring Security/Spring Boot configuration and provider metadata.

---

# 11. JWT vs Opaque Token ⭐⭐⭐⭐⭐

Spring Security Resource Server supports both common bearer-token validation models.

### JWT

```text
Bearer Token
 ↓
Local signature + claim validation
```

### Opaque Token

```text
Bearer Token
 ↓
Introspection Endpoint
 ↓
Authorization Server
 ↓
Active?
 ↓
Authentication
```

### Comparison

| JWT | Opaque Token |
|---|---|
| Self-contained | Reference-like token |
| Usually locally validated | Usually introspected |
| Can reduce validation network calls | Central validation possible |
| Revocation can be harder | Revocation can be more immediate |
| Key rotation required | Introspection availability required |

---

# 12. Opaque Token Configuration ⭐⭐⭐⭐

Conceptually:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        opaquetoken:
          introspection-uri: https://issuer.example.com/oauth2/introspect
          client-id: ${INTROSPECTION_CLIENT_ID}
          client-secret: ${INTROSPECTION_CLIENT_SECRET}
```

The exact property values depend on the authorization server.

### Flow

```text
API Request
   ↓
Bearer Token
   ↓
OpaqueTokenAuthenticationProvider
   ↓
Introspection
   ↓
Authorization Server
   ↓
Token Active?
   ↓
Authentication
```

---

# 13. Bearer Token Extraction ⭐⭐⭐⭐⭐

Standard API requests commonly send the token in:

```http
Authorization: Bearer eyJ...
```

Spring Security's bearer-token processing extracts the token and starts authentication.

### Important

Do not casually accept arbitrary custom headers as equivalent bearer credentials without understanding the security implications.

---

# 14. `BearerTokenAuthenticationFilter` ⭐⭐⭐⭐⭐

This filter processes bearer-token authentication in the Spring Security filter chain.

```text
HTTP Request
   ↓
BearerTokenAuthenticationFilter
   ↓
Bearer Token
   ↓
AuthenticationManager
```

For JWT authentication, this eventually reaches JWT-specific authentication components.

---

# 15. `JwtAuthenticationProvider` ⭐⭐⭐⭐⭐

For JWT bearer tokens:

```text
AuthenticationManager
        ↓
JwtAuthenticationProvider
        ↓
JwtDecoder
        ↓
JWT validation
        ↓
Jwt
        ↓
Authentication
```

The provider is responsible for converting a validated JWT into an authenticated principal representation.

---

# 16. `JwtDecoder` ⭐⭐⭐⭐⭐

`JwtDecoder` is responsible for decoding and validating signed JWTs according to its configuration.

Typical responsibilities include:

```text
Signature verification
Issuer validation
Timestamp validation
Other configured validators
```

### Important

> **Decoding is not the same as trusting. Validation is mandatory.**

---

# 17. JWT Claim Validation ⭐⭐⭐⭐⭐

A resource server should validate claims appropriate to its security contract.

```text
Signature
   ↓
Issuer
   ↓
Audience where applicable
   ↓
exp
   ↓
nbf where applicable
   ↓
Required custom claims
```

A valid token from the wrong issuer or for the wrong audience should not automatically be accepted.

---

# 18. Audience Validation ⭐⭐⭐⭐⭐

Audience validation is especially important when multiple APIs share the same authorization server.

Example:

```text
Authorization Server
      ↓
Token for billing-api
      ↓
Billing API       ✓
Orders API        ?
```

If an API requires a specific audience, configure validation for it.

Conceptually:

```java
JwtIssuerValidator issuerValidator = ...;
JwtClaimValidator<List<String>> audienceValidator = ...;
```

Use Spring Security's supported validator mechanisms rather than writing insecure ad-hoc token checks.

---

# 19. Authorities from Scopes ⭐⭐⭐⭐⭐

OAuth2 access-token scopes can be mapped to Spring Security authorities.

Example token:

```json
{
  "scope": "orders.read orders.write"
}
```

Conceptually:

```text
orders.read
    ↓
SCOPE_orders.read

orders.write
    ↓
SCOPE_orders.write
```

Then:

```java
@PreAuthorize("hasAuthority('SCOPE_orders.read')")
```

This is a common Spring Security resource-server authorization pattern.

---

# 20. Scope Claim vs `scp` Claim ⭐⭐⭐⭐

OAuth providers can represent scopes using different claim conventions.

Common examples include:

```text
scope
scp
```

Spring Security provides standard support for common scope representations and can be customized when the provider uses a different contract.

### Important

Always confirm the actual token contract from your authorization server.

---

# 21. Custom Authority Mapping ⭐⭐⭐⭐⭐

If the token contains custom roles/permissions, configure a converter rather than assuming Spring Security will automatically interpret them.

Example concept:

```java
JwtAuthenticationConverter converter = new JwtAuthenticationConverter();

converter.setJwtGrantedAuthoritiesConverter(jwt -> {
    // Map provider-specific claims to GrantedAuthority
    return ...;
});
```

Possible token:

```json
{
  "roles": ["ADMIN", "REPORT_VIEWER"]
}
```

Application mapping:

```text
ADMIN
 ↓
ROLE_ADMIN

REPORT_VIEWER
 ↓
ROLE_REPORT_VIEWER
```

The prefix and mapping are application configuration decisions.

---

# 22. Method Security ⭐⭐⭐⭐⭐

Resource-server authorization can be applied at controller/service methods.

Enable method security:

```java
@Configuration
@EnableMethodSecurity
class SecurityConfig {
}
```

Then:

```java
@PreAuthorize("hasAuthority('SCOPE_orders.read')")
@GetMapping("/orders")
public List<Order> getOrders() {
    return ...;
}
```

### Why method security?

URL-level security protects routes, while method security allows authorization close to business operations.

---

# 23. Request-Level vs Method-Level Authorization ⭐⭐⭐⭐⭐

```text
Request Authorization
→ /orders/** requires authentication

Method Authorization
→ getOrder() requires orders.read
→ deleteOrder() requires orders.delete
```

A mature application often uses both where appropriate.

---

# 24. `401 Unauthorized` vs `403 Forbidden` ⭐⭐⭐⭐⭐

### 401

Authentication is missing or invalid.

```text
No token
Invalid token
Expired token
Bad credentials
```

### 403

The caller is authenticated but does not have sufficient permission.

```text
Valid token
      ↓
Missing required authority
      ↓
403
```

### Memory

```text
401 → Who are you? Authentication failed.
403 → I know who you are, but you cannot do this.
```

---

# 25. `BearerTokenAuthenticationEntryPoint` ⭐⭐⭐⭐

When bearer authentication fails and the resource server needs to challenge the client, Spring Security can use bearer-token-specific authentication entry-point behavior.

Conceptually:

```text
Authentication failure
       ↓
AuthenticationEntryPoint
       ↓
401 response
```

Exact response headers/body depend on configuration.

---

# 26. AccessDeniedHandler ⭐⭐⭐⭐

When authentication succeeds but authorization fails:

```text
Authenticated
     ↓
Authorization denied
     ↓
AccessDeniedHandler
     ↓
403 response
```

This is different from `AuthenticationEntryPoint`.

---

# 27. Resource Server and CSRF ⭐⭐⭐⭐⭐

For a typical stateless API using bearer tokens in the `Authorization` header, CSRF protection may not be required in the same way as cookie-based browser authentication because the browser does not automatically attach an Authorization header cross-site.

But this is architecture-dependent.

### Do not memorize:

```text
JWT → disable CSRF always
```

Instead remember:

```text
Credential transport
      ↓
Browser behavior
      ↓
CSRF risk
      ↓
Security configuration
```

If the API uses cookies for authentication, CSRF considerations change.

---

# 28. Resource Server and CORS ⭐⭐⭐⭐⭐

CORS and OAuth2 authentication solve different problems.

```text
CORS
→ Browser cross-origin policy

OAuth2 Resource Server
→ Access-token authentication/authorization
```

A valid access token does not bypass browser CORS restrictions.

Configure allowed origins, methods and headers according to the application's actual clients.

---

# 29. Stateless API ⭐⭐⭐⭐⭐

A typical JWT resource server can operate without an HTTP session for bearer-token authentication.

Conceptually:

```java
http
    .sessionManagement(session ->
        session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
    );
```

Then each request carries its own bearer credential.

### Important

Stateless request authentication does not mean the overall system contains no state.

---

# 30. Session vs Resource Server ⭐⭐⭐⭐⭐

### Browser Login

```text
User
 ↓
OIDC Login
 ↓
Session Cookie
 ↓
Application
```

### API Resource Server

```text
Client
 ↓
Bearer Access Token
 ↓
API
 ↓
Validate
```

These are different security patterns and can coexist in different applications/services.

---

# 31. Resource Server + API Gateway ⭐⭐⭐⭐⭐

Typical microservice architecture:

```text
                 Authorization Server
                         ↓
                    Access Token
                         ↓
Client → API Gateway → Orders Service
                         ↓
                    Payments Service
```

Possible responsibilities:

```text
Gateway
→ Edge authentication / routing / rate limiting

Resource Service
→ Resource-specific authorization
```

### Critical rule

Do not blindly trust identity headers from the gateway. Establish authenticated and integrity-protected service trust.

---

# 32. Resource Server + Microservices ⭐⭐⭐⭐⭐

Each microservice can independently validate access tokens.

```text
                 Authorization Server
                         ↓
                      JWT
                         ↓
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   Order Service   Payment Service   User Service
        ↓                ↓                ↓
     Validate          Validate          Validate
```

### Benefits

- Independent authorization
- No central session lookup for every request
- Better fault isolation
- Horizontal scalability

### Trade-offs

- Key distribution/rotation
- Consistent authorization policies
- Token revocation challenges
- Clock synchronization
- Operational complexity

---

# 33. Service-to-Service Authentication ⭐⭐⭐⭐⭐

Do not assume that an end-user access token is always the best credential for every internal service call.

Two common patterns:

```text
User Delegation
→ Forward user access token
→ Downstream service authorizes user context

Service Identity
→ Client Credentials / workload identity
→ Service authenticates as itself
```

Choose based on whether downstream services need the end-user's delegated permissions.

---

# 34. Token Relay ⭐⭐⭐⭐

A gateway or service may propagate an access token downstream when the architecture requires delegated user context.

```text
Client
 ↓
Access Token
 ↓
Gateway
 ↓
Token Relay
 ↓
Service B
```

### Security concern

Forwarding tokens increases their exposure surface. Only relay tokens to trusted downstream services that are intended audiences/recipients.

---

# 35. Audience Design in Microservices ⭐⭐⭐⭐⭐

A common mistake is issuing one broad token that every service accepts.

Better principle:

```text
Token
 ↓
Specific intended audience/scope
 ↓
Specific APIs
```

Use least privilege.

Example:

```text
orders.read
orders.write
payments.read
```

A payment API should not automatically accept a token intended only for an unrelated resource.

---

# 36. Token Introspection ⭐⭐⭐⭐⭐

Opaque-token resource servers can call an introspection endpoint.

```text
Resource Server
      ↓
Introspection Request
      ↓
Authorization Server
      ↓
active=true?
      ↓
Scopes / Subject / Claims
      ↓
Authentication
```

### Advantages

- Central token status
- Easier immediate revocation
- Centralized token policy

### Trade-offs

- Network dependency
- Authorization Server availability
- Latency
- Caching complexity

---

# 37. JWT vs Introspection Decision ⭐⭐⭐⭐⭐

Use JWT-style local validation when:

```text
Low latency
Distributed services
High request volume
Provider supports reliable key distribution
```

Consider introspection when:

```text
Immediate token status matters
Central control is important
Token revocation is frequent/critical
Opaque tokens fit the architecture
```

The decision should be based on security and operational requirements, not fashion.

---

# 38. Resource Server Security Best Practices ⭐⭐⭐⭐⭐

```text
[ ] HTTPS
[ ] Validate trusted issuer
[ ] Validate signature
[ ] Validate audience when applicable
[ ] Validate exp / nbf
[ ] Restrict algorithms
[ ] Use trusted JWKS/discovery
[ ] Support key rotation
[ ] Use least-privilege scopes
[ ] Protect client/introspection credentials
[ ] Do not log access tokens
[ ] Avoid trusting arbitrary forwarded identity headers
[ ] Define service-to-service trust explicitly
[ ] Configure CORS deliberately
[ ] Configure CSRF according to credential transport
[ ] Keep access tokens short-lived where appropriate
[ ] Design for revocation requirements
```

---

# 39. Common Resource Server Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Only checking signature

A validly signed token may still have the wrong issuer/audience or be expired.

### Mistake 2 — Trusting any issuer

Issuer must be trusted/configured, not selected by the attacker.

### Mistake 3 — No audience validation

A token intended for one API can potentially be replayed against another if the resource server accepts it without checking audience/purpose.

### Mistake 4 — Treating roles automatically

A custom `roles` claim is not magically a Spring authority unless configured/mapped.

### Mistake 5 — Gateway-only security

A downstream service should not blindly trust gateway-provided identity headers.

### Mistake 6 — Disabling CSRF blindly

CSRF depends on credential transport and browser behavior.

### Mistake 7 — Logging bearer tokens

Logs can become credential stores.

### Mistake 8 — Sharing signing private keys

Resource servers should verify with public keys where asymmetric signing is used; the issuer's private signing key must remain protected.

---

# 40. Testing Resource Server APIs ⭐⭐⭐⭐

Typical request:

```bash
curl -H "Authorization: Bearer <access-token>" \
     https://api.example.com/orders
```

Test cases:

```text
No token              → 401
Malformed token       → 401
Expired token         → 401
Wrong issuer          → 401
Wrong audience        → 401 / rejected
Valid token           → 200
Valid token + no scope→ 403
Valid token + scope   → 200
```

Exact status/response details depend on the application's exception configuration and protocol behavior.

---

# 41. Observability ⭐⭐⭐⭐

Useful resource-server metrics/logging include:

```text
Authentication failures
Authorization failures
Latency
Token introspection latency
Key retrieval/rotation events
Provider availability
```

### Security rule

Never log raw bearer tokens, refresh tokens, client secrets or sensitive token claims unnecessarily.

---

# 42. Resource Server vs OAuth2 Client vs OIDC Login ⭐⭐⭐⭐⭐

| Capability | OAuth2 Client | Resource Server | OIDC Login |
|---|---|---|---|
| Obtains tokens | Yes | Usually no | Yes, as part of login flow |
| Accepts bearer tokens | Not its primary role | Yes | Not its primary role |
| Protects APIs | No | Yes | Not its primary role |
| User login | Possible depending on setup | No login UI by default | Yes |
| Token validation | As needed | Core responsibility | Validates OIDC identity tokens during login |

### Memory

```text
Client
→ Gets token

Resource Server
→ Protects API

OIDC Login
→ Logs user in
```

---

# 43. Modern Spring Security Configuration ⭐⭐⭐⭐⭐

A clean resource-server configuration can look like:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

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
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );

        return http.build();
    }
}
```

The application should add CSRF/CORS configuration based on its actual client and credential-transport model rather than blindly copying a template.

---

# 44. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“An OAuth2 Resource Server is an API that protects resources using access tokens issued by a trusted Authorization Server. The client obtains an access token and sends it as a Bearer token in the Authorization header. In Spring Security, the BearerTokenAuthenticationFilter extracts the token and authentication is delegated through the AuthenticationManager to a JWT authentication provider for JWT tokens. The JwtDecoder validates the signature and relevant claims such as issuer, expiration and, where required, audience. After successful authentication, the Authentication is stored in the SecurityContext and authorization is performed using scopes or mapped authorities. Spring Security supports JWT-based local validation as well as opaque-token introspection. JWT gives low-latency distributed validation but revocation can be harder, while introspection provides central token status at the cost of network dependency. In production I would use HTTPS, trusted issuer/JWKS discovery, key rotation, audience validation where applicable, least-privilege scopes, protected introspection credentials, deliberate CORS/CSRF configuration and avoid logging or blindly forwarding bearer tokens.”**

---

# 45. 30-Second Hinglish Answer

> **“OAuth2 Resource Server basically protected API hoti hai jo Authorization Server se issued access token ko validate karke resource access deti hai. Client `Authorization: Bearer <token>` bhejta hai. Spring Security ka BearerTokenAuthenticationFilter token extract karta hai, JWT case mein JwtAuthenticationProvider aur JwtDecoder signature, issuer, expiry aur required claims validate karte hain. Valid hone ke baad Authentication SecurityContext mein aata hai aur scopes/authorities ke basis par authorization hoti hai. Resource Server JWT ko locally validate kar sakta hai ya opaque token ko introspection endpoint se verify kar sakta hai. Production mein issuer/audience validation, JWKS key rotation, least privilege aur token leakage protection important hain.”**

---

# 46. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                  AUTHORIZATION SERVER
                          │
                          │ Access Token
                          ↓
CLIENT ───────────────→ RESOURCE SERVER
  │                         │
  │ Bearer Token            │
  │                         ↓
  │               BearerTokenAuthenticationFilter
  │                         │
  │                         ↓
  │                 AuthenticationManager
  │                         │
  │                         ↓
  │                JwtAuthenticationProvider
  │                         │
  │                         ↓
  │                     JwtDecoder
  │                         │
  │              ┌──────────┴──────────┐
  │              ↓                     ↓
  │         Signature ✓          Claims ✓
  │              │                     │
  │              └──────────┬──────────┘
  │                         ↓
  │                  Authentication
  │                         ↓
  │                  SecurityContext
  │                         ↓
  │                   Authorization
  │                         ↓
  └──────────────────→ Protected API
```

### JWT vs Introspection memory

```text
JWT
→ Local validation
→ Fast
→ Key rotation
→ Revocation harder

Opaque
→ Introspection
→ Central status
→ Revocation easier
→ Network dependency
```

### Final memory line

> **Resource Server = Receive Access Token → Validate → Authenticate → Map Authorities → Authorize → Serve Protected Resource.**

---

## Navigation

[← S4.1.23 — JWT Authentication](../23-JWT-Authentication/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.25 — OAuth2 Client**
