# S4.1.25 — OAuth2 Client

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is an OAuth2 Client? ⭐⭐⭐⭐⭐

An **OAuth2 Client** is an application that obtains and/or uses OAuth2 access tokens to access protected resources on behalf of a resource owner or as itself, depending on the grant/flow.

```text
Client
   ↓
Authorization Server
   ↓
Access Token
   ↓
Resource Server
   ↓
Protected Resource
```

### Golden rule

```text
OAuth2 Client
→ Obtains/uses access tokens

Authorization Server
→ Issues tokens

Resource Server
→ Protects APIs/resources
```

An OAuth2 Client is **not the same thing as an OAuth2 Resource Server**.

---

# 2. OAuth2 Client vs Resource Server ⭐⭐⭐⭐⭐

| OAuth2 Client | Resource Server |
|---|---|
| Requests/uses access tokens | Receives access tokens |
| Calls protected APIs | Protects APIs |
| May redirect users to Authorization Server | Validates access tokens |
| May maintain authorized-client state | Creates Authentication from token |
| Uses OAuth2 client credentials/configuration | Uses resource-server configuration |

### Interview line

> **Client gets/uses the token; Resource Server validates the token and protects the resource.**

---

# 3. OAuth2 Roles ⭐⭐⭐⭐⭐

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

Entity that can grant access to protected resources, commonly a user.

### Client

Application requesting access to a protected resource.

### Authorization Server

Server responsible for issuing access tokens after the appropriate authorization process.

### Resource Server

Server hosting protected resources.

---

# 4. What Spring Security OAuth2 Client Provides ⭐⭐⭐⭐⭐

Spring Security provides OAuth2 Client support for scenarios such as:

- OAuth2 Login
- Authorization Code flow
- Calling protected APIs with authorized-client access tokens
- Client Credentials flow
- Managing authorized-client information
- Integrating with OAuth2/OpenID Connect providers

The exact behavior depends on the client registration and authorization grant being used.

---

# 5. OAuth2 Authorization Code Flow ⭐⭐⭐⭐⭐

The most important browser-based flow to understand:

```text
User
 ↓
Client Application
 ↓
Authorization Server
 ↓
User Authentication + Consent
 ↓
Authorization Code
 ↓
Client Backend
 ↓
Token Endpoint
 ↓
Access Token
 ↓
Resource Server
```

### Important

The authorization code is **not** the access token.

```text
Authorization Code
→ Short-lived intermediate credential

Access Token
→ Used to access protected resources
```

---

# 6. Authorization Code Flow with PKCE ⭐⭐⭐⭐⭐

For public clients, PKCE is an important protection for authorization-code flows.

```text
Client
 ↓
Generate code_verifier
 ↓
Create code_challenge
 ↓
Authorization Request
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Token Request + code_verifier
 ↓
Access Token
```

### Why PKCE?

It binds the authorization code exchange to the client instance that initiated the flow and reduces the impact of an intercepted authorization code.

### Memory

```text
code_verifier
      ↓
code_challenge
      ↓
Authorization Request
      ↓
Code
      ↓
code_verifier
      ↓
Token Endpoint
```

---

# 7. Client Credentials Grant ⭐⭐⭐⭐⭐

Used for machine-to-machine/service authentication where there is no end-user authorization step.

```text
Service A
   ↓
Client Credentials
   ↓
Authorization Server
   ↓
Access Token
   ↓
Service B / Resource Server
```

Conceptually:

```text
client_id
client_secret
     ↓
Token Endpoint
     ↓
Access Token
```

### Important

Client Credentials represents the **client/service identity**, not an end user's identity.

---

# 8. Authorization Code vs Client Credentials ⭐⭐⭐⭐⭐

| Authorization Code | Client Credentials |
|---|---|
| User/resource-owner context | No end-user context |
| Common for web login/consent | Common for service-to-service |
| Authorization code involved | Direct token request |
| Often used with OIDC login | OAuth2 authorization without user login |
| PKCE is important for public clients | PKCE is not the defining mechanism |

### Memory

```text
User involved?
   ↓ Yes → Authorization Code (+ PKCE where appropriate)
   ↓ No  → Client Credentials
```

---

# 9. OAuth2 Login vs OAuth2 Client ⭐⭐⭐⭐⭐

These concepts overlap but are not identical.

```text
OAuth2 Client
→ Generic client capabilities

OAuth2 Login
→ Spring Security feature using OAuth2/OIDC to authenticate a user
```

Typical login flow:

```text
Browser
 ↓
Application
 ↓
Authorization Server
 ↓
Login / Consent
 ↓
Code
 ↓
Application
 ↓
Tokens
 ↓
Authenticated User
```

OIDC adds identity information, including an ID Token.

---

# 10. Access Token vs ID Token ⭐⭐⭐⭐⭐

```text
Access Token
→ Authorization
→ Intended for Resource Server

ID Token
→ Authentication / identity
→ Intended for Client / Relying Party
```

### Critical rule

> Do not send an ID Token to an API just because it is a JWT. APIs should receive and validate access tokens intended for them.

---

# 11. Spring Boot OAuth2 Client Dependency ⭐⭐⭐⭐⭐

Typical dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

Spring Boot manages compatible transitive versions through its dependency management.

---

# 12. Client Registration ⭐⭐⭐⭐⭐

A **ClientRegistration** represents the configuration of an OAuth2/OpenID Connect provider/client relationship.

Conceptually:

```text
registrationId
clientId
clientSecret
authorizationGrantType
redirectUri
scopes
provider endpoints
```

Example:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          my-provider:
            client-id: ${CLIENT_ID}
            client-secret: ${CLIENT_SECRET}
            scope:
              - openid
              - profile
              - email
```

Provider-specific details depend on the identity provider.

---

# 13. Provider Configuration ⭐⭐⭐⭐⭐

Provider configuration can be defined explicitly or discovered from provider metadata when supported.

Conceptually:

```yaml
spring:
  security:
    oauth2:
      client:
        provider:
          my-provider:
            issuer-uri: https://issuer.example.com
```

The provider's metadata can expose endpoints such as:

```text
authorization_endpoint
token_endpoint
userinfo_endpoint
jwks_uri
end_session_endpoint (when supported)
```

---

# 14. `ClientRegistration` vs `OAuth2AuthorizedClient` ⭐⭐⭐⭐⭐

Common interview question.

### ClientRegistration

Describes **how the application is registered** with the provider.

```text
client-id
client-secret
scopes
provider
redirect URI
```

### OAuth2AuthorizedClient

Represents an **authorized client instance**, including token information obtained for that client/principal context.

```text
ClientRegistration
       ↓
Authorization
       ↓
OAuth2AuthorizedClient
       ↓
Access Token
```

### Memory

> **Registration = configuration. AuthorizedClient = authorization/token state.**

---

# 15. `OAuth2AuthorizedClientService` ⭐⭐⭐⭐⭐

This service manages authorized-client information.

Conceptually:

```text
OAuth2AuthorizedClientService
        ↓
Load / Save / Remove
        ↓
OAuth2AuthorizedClient
```

The storage implementation can be in-memory or backed by another store depending on application architecture.

---

# 16. `OAuth2AuthorizedClientRepository` ⭐⭐⭐⭐

For web applications, Spring Security can associate authorized clients with the current request/principal using an `OAuth2AuthorizedClientRepository`.

Conceptually:

```text
Request
  ↓
Principal
  ↓
Authorized Client Repository
  ↓
Access Token
```

### Difference

```text
Service
→ General authorized-client persistence/service abstraction

Repository
→ Web request/principal-oriented access
```

---

# 17. Authorization Endpoint ⭐⭐⭐⭐⭐

The client redirects the user to the Authorization Server's authorization endpoint.

Typical parameters include:

```text
response_type=code
client_id=...
redirect_uri=...
scope=...
state=...
code_challenge=...   (when PKCE is used)
```

### Important

`state` helps protect the authorization flow against request-forgery/mix-up style attacks by binding the callback to the initiating request.

---

# 18. Redirect URI ⭐⭐⭐⭐⭐

After authorization, the Authorization Server redirects back to a registered redirect URI.

```text
Browser
 ↓
Authorization Server
 ↓
redirect_uri
 ↓
Client callback
 ↓
Authorization code
```

### Security rule

Redirect URIs should be registered and validated. Avoid open redirects or attacker-controlled callback destinations.

---

# 19. `state` Parameter ⭐⭐⭐⭐⭐

The `state` parameter correlates the authorization request with the callback.

```text
Request
  ↓
Generate state
  ↓
Authorization Server
  ↓
Callback + state
  ↓
Verify state
  ↓
Continue
```

### Interview line

> `state` helps protect the authorization flow against CSRF/request-forgery attacks by correlating the callback with the original request.

---

# 20. Token Endpoint ⭐⭐⭐⭐⭐

The client exchanges an authorization code for tokens.

```text
Client
 ↓
Authorization Code
 ↓
Token Endpoint
 ↓
Access Token
 +
Refresh Token (when issued)
```

For confidential clients, client authentication is performed according to the provider configuration and OAuth2 security requirements.

---

# 21. Refresh Token ⭐⭐⭐⭐⭐

A refresh token can be used to obtain a new access token without requiring the user to repeat the authorization step, when the authorization server issues one and the client is permitted to use it.

```text
Access Token
   ↓
Expires
   ↓
Refresh Token
   ↓
Token Endpoint
   ↓
New Access Token
```

### Security

Refresh tokens are highly sensitive credentials and need stronger protection than ordinary access tokens.

---

# 22. Access Token Lifecycle ⭐⭐⭐⭐⭐

```text
Obtain
  ↓
Store securely
  ↓
Use
  ↓
Expire
  ↓
Refresh / Re-authorize
```

A mature OAuth2 Client should avoid unnecessarily exposing tokens to application logs, browser code, URLs or untrusted downstream systems.

---

# 23. Spring Security Login Flow ⭐⭐⭐⭐⭐

Typical Spring Security web flow:

```text
GET /oauth2/authorization/{registrationId}
              ↓
Authorization Request
              ↓
Authorization Server
              ↓
Authentication / Consent
              ↓
Callback
              ↓
Authorization Code
              ↓
Token Exchange
              ↓
Authenticated Principal
              ↓
Application Session
```

For OIDC, Spring Security also validates the ID Token and can use UserInfo depending on configuration/provider support.

---

# 24. `oauth2Login()` ⭐⭐⭐⭐⭐

Modern Spring Security configuration:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/", "/public/**").permitAll()
            .anyRequest().authenticated()
        )
        .oauth2Login(Customizer.withDefaults());

    return http.build();
}
```

The authorization endpoint can be initiated using:

```text
/oauth2/authorization/{registrationId}
```

---

# 25. OAuth2 Authorization Request Redirect ⭐⭐⭐⭐

Spring Security can provide the authorization request endpoint automatically.

Example:

```text
/oauth2/authorization/google
```

Conceptually:

```text
Browser
 ↓
/oauth2/authorization/google
 ↓
Spring Security
 ↓
Google Authorization Endpoint
```

The exact provider and registration ID depend on configuration.

---

# 26. OAuth2 Login Callback ⭐⭐⭐⭐⭐

Spring Security handles the authorization response and token exchange through its OAuth2 client support.

Conceptually:

```text
Authorization Server
 ↓
GET /login/oauth2/code/{registrationId}?code=...
 ↓
Spring Security
 ↓
Token Endpoint
 ↓
Access Token
 ↓
Authenticated Principal
```

Avoid manually implementing the entire protocol unless there is a strong reason and a well-defined security review.

---

# 27. `OAuth2LoginAuthenticationFilter` ⭐⭐⭐⭐

For OAuth2 Login, Spring Security processes the authorization response through OAuth2 login authentication components.

Conceptually:

```text
Authorization Code Callback
        ↓
OAuth2 Login Authentication Filter
        ↓
Token Exchange
        ↓
User / OIDC Identity
        ↓
Authentication
```

This is different from the resource-server `BearerTokenAuthenticationFilter`.

---

# 28. Client Credentials in Spring Security ⭐⭐⭐⭐⭐

For service-to-service calls, configure the appropriate client registration using the `client_credentials` grant.

Conceptually:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          orders-client:
            authorization-grant-type: client_credentials
            client-id: ${CLIENT_ID}
            client-secret: ${CLIENT_SECRET}
            scope:
              - orders.read
```

Then the application can obtain an access token for the configured client identity.

---

# 29. `OAuth2AuthorizedClientManager` ⭐⭐⭐⭐⭐

`OAuth2AuthorizedClientManager` is a key abstraction for obtaining and managing authorized clients/access tokens.

Conceptually:

```text
Application
    ↓
OAuth2AuthorizedClientManager
    ↓
Authorization Grant
    ↓
Access Token
    ↓
Protected API
```

It can manage authorization and token acquisition for different OAuth2 grant types supported by the configured providers.

---

# 30. `OAuth2AuthorizedClientProvider` ⭐⭐⭐⭐

Providers represent supported authorization-grant mechanisms.

Conceptually:

```text
OAuth2AuthorizedClientProvider
       ↓
Authorization Code
Client Credentials
Refresh Token
Other supported grants
```

The manager uses configured providers to determine how an authorized client can be obtained or refreshed.

---

# 31. Calling a Protected API ⭐⭐⭐⭐⭐

After obtaining an access token:

```text
OAuth2 Client
    ↓
Access Token
    ↓
Authorization: Bearer ...
    ↓
Resource Server
```

Spring Security can integrate the authorized client with HTTP client infrastructure so the access token can be attached to outbound requests.

---

# 32. `RestClient` + OAuth2 Client ⭐⭐⭐⭐⭐

Modern Spring applications can use `RestClient` with Spring Security OAuth2 client support.

Conceptually:

```java
RestClient restClient = RestClient.builder()
    .requestInterceptor((request, body, execution) -> {
        // OAuth2 token integration/configuration
        return execution.execute(request, body);
    })
    .build();
```

For production code, prefer Spring Security's OAuth2-aware request interceptors/managers rather than manually copying token strings.

---

# 33. `WebClient` + OAuth2 Client ⭐⭐⭐⭐⭐

For reactive applications, Spring Security provides OAuth2 client integration with `WebClient`.

Conceptually:

```java
@Bean
WebClient webClient(OAuth2AuthorizedClientManager manager) {
    ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2 =
        new ServletOAuth2AuthorizedClientExchangeFilterFunction(manager);

    return WebClient.builder()
        .apply(oauth2.oauth2Configuration())
        .build();
}
```

The exact API differs between servlet and reactive applications.

---

# 34. `OAuth2ClientHttpRequestInterceptor` ⭐⭐⭐⭐⭐

For Spring's `RestClient`/HTTP client ecosystem, Spring Security provides OAuth2 client integration through an OAuth2-aware interceptor.

Conceptually:

```text
Outbound Request
      ↓
OAuth2 Client Interceptor
      ↓
AuthorizedClientManager
      ↓
Access Token
      ↓
Authorization Header
      ↓
Resource Server
```

This avoids manually implementing token acquisition and attachment logic in every API call.

---

# 35. Authorized Client Selection ⭐⭐⭐⭐

An OAuth2 client may have multiple registrations:

```text
registrationId
     ↓
provider/client configuration
     ↓
authorized client
     ↓
access token
```

The application must select the correct registration and token for the target resource.

### Security rule

Do not send a token obtained for one resource/provider to unrelated APIs unless its audience and authorization contract explicitly permit it.

---

# 36. Multiple OAuth2 Providers ⭐⭐⭐⭐⭐

Example:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          company-idp:
            client-id: ${COMPANY_CLIENT_ID}
            client-secret: ${COMPANY_CLIENT_SECRET}
            scope: openid,profile
          partner-idp:
            client-id: ${PARTNER_CLIENT_ID}
            client-secret: ${PARTNER_CLIENT_SECRET}
            scope: orders.read
```

Then:

```text
/oauth2/authorization/company-idp
/oauth2/authorization/partner-idp
```

Each registration can have different provider configuration and security requirements.

---

# 37. Confidential vs Public Client ⭐⭐⭐⭐⭐

### Confidential Client

Can securely maintain credentials in a protected backend environment.

```text
Backend
 ↓
client_secret
```

### Public Client

Cannot safely keep a client secret, such as a native/mobile or browser-based application.

```text
Public client
→ No trustworthy client secret
→ PKCE is important for authorization-code flow
```

### Important

Do not embed confidential client secrets in frontend/mobile application binaries.

---

# 38. Client Authentication ⭐⭐⭐⭐

Confidential clients authenticate to the token endpoint according to the provider's supported method.

Examples can include:

```text
Client secret authentication
Private key based authentication
mTLS
```

The selected method depends on provider and deployment security requirements.

---

# 39. OAuth2 Client Security Best Practices ⭐⭐⭐⭐⭐

```text
[ ] HTTPS everywhere
[ ] Use Authorization Code + PKCE where appropriate
[ ] Validate redirect URIs
[ ] Validate state
[ ] Protect client secrets
[ ] Never expose client secrets in frontend code
[ ] Protect refresh tokens
[ ] Use least-privilege scopes
[ ] Validate provider/issuer configuration
[ ] Do not log access/refresh tokens
[ ] Avoid putting tokens in URLs
[ ] Use secure session/cookie configuration for browser apps
[ ] Rotate credentials when supported
[ ] Avoid unnecessary token forwarding
[ ] Use correct audience/resource for downstream APIs
```

---

# 40. Common OAuth2 Client Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Confusing OAuth2 Login with Resource Server

Login authenticates the user; resource server protects APIs.

### Mistake 2 — Sending ID Token to API

An ID token is for the client/relying party, not a generic API credential.

### Mistake 3 — No PKCE for public clients

Authorization-code interception becomes more dangerous.

### Mistake 4 — Hard-coding client secrets

Secrets can leak through source control or artifacts.

### Mistake 5 — Logging tokens

Logs become credential leakage points.

### Mistake 6 — Treating access token as application session

Access tokens have a protocol-specific purpose and lifetime.

### Mistake 7 — Forwarding tokens everywhere

Only send a token to intended/trusted recipients.

### Mistake 8 — Open redirect URI

Attackers may exploit callback manipulation.

---

# 41. OAuth2 Client vs JWT ⭐⭐⭐⭐⭐

Another common interview trap.

```text
OAuth2 Client
→ OAuth2 protocol role/capability

JWT
→ Token representation/format
```

An OAuth2 access token **may be a JWT or opaque token**.

```text
OAuth2 Client
     ↓
Access Token
     ├── JWT
     └── Opaque
```

---

# 42. OAuth2 Client + OIDC ⭐⭐⭐⭐⭐

OIDC extends OAuth2 for authentication.

```text
OAuth2
→ Authorization

OIDC
→ Authentication + Identity
```

Typical login response can include:

```text
Authorization Code
      ↓
Tokens
      ├── Access Token
      ├── ID Token
      └── Refresh Token (if issued)
```

The exact token set depends on provider and requested scopes/flow.

---

# 43. OAuth2 Client + Session ⭐⭐⭐⭐⭐

A server-side web application may use OAuth2/OIDC to authenticate the user and then maintain a local application session.

```text
Browser
 ↓
OAuth2 Login
 ↓
Identity Provider
 ↓
Authentication
 ↓
Application Session
 ↓
Session Cookie
```

This is different from an API that receives a bearer access token on every request.

---

# 44. OAuth2 Client in Microservices ⭐⭐⭐⭐⭐

Service-to-service:

```text
Service A
   ↓
OAuth2 Client
   ↓
Client Credentials
   ↓
Authorization Server
   ↓
Access Token
   ↓
Service B
   ↓
Resource Server
```

### Key distinction

```text
Service A
→ OAuth2 Client

Service B
→ OAuth2 Resource Server
```

The same application can be both a Client and Resource Server when it calls another protected service and also exposes protected endpoints.

---

# 45. Application as Both Client and Resource Server ⭐⭐⭐⭐⭐

Very common in microservices.

```text
          Authorization Server
             ↙           ↘
       Token ↓             ↓ Token
             ↘           ↙
          Service A
          /       \
         /         \
Resource Server   OAuth2 Client
       ↓               ↓
  Inbound API      Outbound API
                       ↓
                  Service B
```

### Interview line

> A microservice can simultaneously act as a Resource Server for inbound requests and an OAuth2 Client for outbound calls.

---

# 46. Token Relay Security ⭐⭐⭐⭐⭐

If a service forwards a user access token:

```text
Client
 ↓
User Access Token
 ↓
Gateway / Service A
 ↓
Service B
```

Verify that:

```text
Service B
→ Is an intended audience
→ Needs delegated user context
→ Is trusted to receive the token
```

Otherwise prefer service identity/client credentials or another dedicated workload-authentication mechanism.

---

# 47. Error Handling ⭐⭐⭐⭐

Common OAuth2 client failures:

```text
Authorization denied
Invalid client
Invalid redirect URI
Invalid scope
Token endpoint failure
Provider unavailable
Token expired
Refresh failed
```

For browser login:

```text
Authorization failure
 ↓
Authentication failure handling
 ↓
Application response
```

For outbound API calls:

```text
Token acquisition failure
 ↓
Application error handling
```

Do not expose provider secrets or raw tokens in error responses.

---

# 48. Testing OAuth2 Client ⭐⭐⭐⭐

Test at least:

```text
[ ] Successful authorization code flow
[ ] Invalid state
[ ] Invalid redirect URI
[ ] Authorization denied
[ ] Token endpoint failure
[ ] Expired access token
[ ] Refresh token flow
[ ] Provider unavailable
[ ] Insufficient scope
[ ] Multiple provider registrations
[ ] Client credential failure
[ ] Token not leaked to logs
```

For automated tests, use mocks/test identity providers where practical instead of relying on production identity infrastructure.

---

# 49. Modern Spring Security Configuration ⭐⭐⭐⭐⭐

Basic OAuth2 Login:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(Customizer.withDefaults());

        return http.build();
    }
}
```

For outbound service calls, configure an `OAuth2AuthorizedClientManager` and integrate it with the HTTP client used by the application.

---

# 50. Important Spring Security Classes ⭐⭐⭐⭐⭐

```text
ClientRegistration
        ↓
OAuth2AuthorizedClient
        ↓
OAuth2AuthorizedClientService / Repository
        ↓
OAuth2AuthorizedClientManager
        ↓
OAuth2AuthorizedClientProvider
        ↓
Access Token
        ↓
Resource Server
```

### Remember

```text
ClientRegistration
→ Configuration

OAuth2AuthorizedClient
→ Authorized client + token state

AuthorizedClientManager
→ Orchestrates acquisition/refresh/use

OAuth2AuthorizedClientProvider
→ Supports grant-specific authorization
```

---

# 51. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“An OAuth2 Client is an application that obtains or uses OAuth2 access tokens to access protected resources. The Authorization Server issues the token and the Resource Server protects the API. For user-facing applications, the common flow is Authorization Code, and PKCE is an important protection for public clients. The user authenticates with the Authorization Server, the client receives an authorization code, exchanges it at the token endpoint and obtains an access token, and optionally a refresh token. In Spring Security, OAuth2 Client support models provider configuration using ClientRegistration and token/authorization state using OAuth2AuthorizedClient. OAuth2AuthorizedClientManager coordinates obtaining and refreshing authorized clients. For machine-to-machine communication, Client Credentials is commonly used, where the service authenticates as itself rather than as an end user. Spring Security can also integrate OAuth2 clients with outbound HTTP calls so access tokens are attached without manually copying token logic everywhere. In production I would protect client secrets, use Authorization Code with PKCE where appropriate, validate state and redirect URIs, use least-privilege scopes, protect refresh tokens, avoid logging tokens, and never confuse an ID Token with an API access token.”**

---

# 52. 30-Second Hinglish Answer

> **“OAuth2 Client wo application hai jo Authorization Server se access token obtain/use karke protected API ko call karti hai. User-based flow mein commonly Authorization Code + PKCE use hota hai; user Authorization Server par authenticate karta hai, client ko code milta hai aur client token endpoint se access token leta hai. Service-to-service case mein Client Credentials use kar sakte hain, jahan user involved nahi hota. Spring Security mein ClientRegistration provider configuration rakhta hai, OAuth2AuthorizedClient token/authorization state represent karta hai aur OAuth2AuthorizedClientManager token acquisition/refresh manage karta hai. Production mein client secret protect karna, PKCE, state, redirect URI validation, least-privilege scopes aur token leakage avoid karna important hai.”**

---

# 53. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                     AUTHORIZATION SERVER
                       /              \
                      /                \
             Authorization Code       Client Credentials
                    ↓                       ↓
                 CLIENT APPLICATION        │
                    ↓                       │
              Token Endpoint               │
                    ↓                       │
                Access Token ←─────────────┘
                    ↓
             Authorization: Bearer
                    ↓
              RESOURCE SERVER
                    ↓
             Protected Resource
```

### User login memory

```text
Browser
 ↓
/oauth2/authorization/{registrationId}
 ↓
Authorization Server
 ↓
Login + Consent
 ↓
Authorization Code
 ↓
Token Endpoint
 ↓
Access Token / ID Token
 ↓
Authenticated User
 ↓
Application Session
```

### Service-to-service memory

```text
Service A
 ↓
OAuth2 Client
 ↓
Client Credentials
 ↓
Authorization Server
 ↓
Access Token
 ↓
Service B
 ↓
OAuth2 Resource Server
```

### Final memory line

> **OAuth2 Client = Obtain/Manage Access Token → Use Token for Protected Resource; Authorization Code (+ PKCE) for user authorization, Client Credentials for service identity.**

---

## Navigation

[← S4.1.24 — OAuth2 Resource Server](../24-OAuth2-Resource-Server/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.26 — OAuth2 Login**
