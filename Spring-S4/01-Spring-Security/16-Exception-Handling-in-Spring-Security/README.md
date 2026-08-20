# S4.1.16 — Exception Handling in Spring Security

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture ⭐⭐⭐⭐⭐

Spring Security mein exception handling ka main purpose hai authentication aur authorization failures ko correctly handle karke appropriate response dena.

```text
HTTP Request
    ↓
Security Filters
    ↓
Authentication / Authorization
    ↓
Exception
    ↓
ExceptionTranslationFilter
    ↓
┌─────────────────────────────┐
│ AuthenticationException     │ → AuthenticationEntryPoint
│ AccessDeniedException       │ → AccessDeniedHandler
└─────────────────────────────┘
```

### Golden rule

> **Authentication problem → AuthenticationEntryPoint**  
> **Authorization problem → AccessDeniedHandler**

---

# 2. Why Exception Handling? ⭐⭐⭐⭐⭐

Security failure ko application ke normal business exception se separate rakhna important hai.

Examples:

```text
No / invalid authentication
        ↓
AuthenticationException

Authenticated user
        ↓
Required authority missing
        ↓
AccessDeniedException
```

Spring Security in failures ko security-specific infrastructure ke through handle karta hai.

---

# 3. AuthenticationException ⭐⭐⭐⭐⭐

`AuthenticationException` authentication process ke failure ko represent karta hai.

Examples include:

- Invalid credentials
- Authentication required
- Authentication provider failure
- Invalid authentication token
- Account-related authentication failures

Conceptually:

```text
Credentials / Token
      ↓
Authentication attempt
      ↓
Failure
      ↓
AuthenticationException
```

---

# 4. AccessDeniedException ⭐⭐⭐⭐⭐

`AccessDeniedException` tab hota hai jab authorization fail hota hai.

Typical scenario:

```text
User authenticated
        ↓
Authorities = ROLE_USER
        ↓
Endpoint requires ROLE_ADMIN
        ↓
AccessDeniedException
```

Memory:

```text
Authentication fail → AuthenticationException
Authorization fail  → AccessDeniedException
```

---

# 5. ExceptionTranslationFilter ⭐⭐⭐⭐⭐

`ExceptionTranslationFilter` Spring Security filter chain ka important component hai jo security exceptions ko appropriate handling mechanism tak translate/route karta hai.

Conceptually:

```text
Security filter / authorization layer
             ↓
      Security exception
             ↓
 ExceptionTranslationFilter
             ↓
 ┌──────────────────────┐
 │ AuthenticationEx.    │ → AuthenticationEntryPoint
 │ AccessDeniedEx.      │ → AccessDeniedHandler
 └──────────────────────┘
```

### Important

> It is not the filter that authenticates the user. Its job is primarily to bridge security exceptions to the configured response mechanisms.

---

# 6. AuthenticationEntryPoint ⭐⭐⭐⭐⭐

`AuthenticationEntryPoint` ka role hai jab request ko authentication ki zarurat ho but valid authentication establish nahi hui ho.

Conceptually:

```java
public interface AuthenticationEntryPoint {
    void commence(
        HttpServletRequest request,
        HttpServletResponse response,
        AuthenticationException authException
    );
}
```

Common REST API use:

```text
401 Unauthorized
```

Example:

```java
@Component
public class CustomAuthenticationEntryPoint
        implements AuthenticationEntryPoint {

    @Override
    public void commence(
            HttpServletRequest request,
            HttpServletResponse response,
            AuthenticationException exception)
            throws IOException {

        response.sendError(
            HttpServletResponse.SC_UNAUTHORIZED,
            "Authentication required"
        );
    }
}
```

---

# 7. AccessDeniedHandler ⭐⭐⭐⭐⭐

`AccessDeniedHandler` authenticated user ke authorization failure ko handle karta hai.

Conceptually:

```java
public interface AccessDeniedHandler {
    void handle(
        HttpServletRequest request,
        HttpServletResponse response,
        AccessDeniedException accessDeniedException
    );
}
```

Typical REST response:

```text
403 Forbidden
```

Example:

```java
@Component
public class CustomAccessDeniedHandler
        implements AccessDeniedHandler {

    @Override
    public void handle(
            HttpServletRequest request,
            HttpServletResponse response,
            AccessDeniedException exception)
            throws IOException {

        response.sendError(
            HttpServletResponse.SC_FORBIDDEN,
            "Access denied"
        );
    }
}
```

---

# 8. 401 vs 403 ⭐⭐⭐⭐⭐

### 401 Unauthorized

Usually means:

```text
Authentication missing / invalid
```

Typical security flow:

```text
Protected resource
      ↓
No valid authentication
      ↓
AuthenticationEntryPoint
      ↓
401
```

### 403 Forbidden

Usually means:

```text
Authenticated
      ↓
Insufficient authority
```

Flow:

```text
Protected resource
      ↓
Authenticated
      ↓
Authorization denied
      ↓
AccessDeniedHandler
      ↓
403
```

### Interview memory

> **401 → authenticate first**  
> **403 → authenticated but not allowed**

Exact status behavior can be customized by the application and authentication mechanism.

---

# 9. SecurityFilterChain Exception Flow ⭐⭐⭐⭐⭐

```text
Client
  ↓
HTTP Request
  ↓
SecurityFilterChain
  ↓
Authentication filters
  ↓
Authentication established?
  ↓
Authorization
  ↓
Exception
  ↓
ExceptionTranslationFilter
  ↓
┌─────────────────────┬──────────────────────┐
│ AuthenticationEx.   │ AccessDeniedEx.      │
↓                     ↓
AuthenticationEP      AccessDeniedHandler
│                     │
↓                     ↓
401 / configured      403 / configured
response              response
```

---

# 10. Configuring `exceptionHandling()` ⭐⭐⭐⭐⭐

Modern configuration:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .anyRequest().authenticated()
        )
        .exceptionHandling(ex -> ex
            .authenticationEntryPoint(authenticationEntryPoint)
            .accessDeniedHandler(accessDeniedHandler)
        );

    return http.build();
}
```

Flow:

```text
Authentication failure
       ↓
authenticationEntryPoint()

Authorization failure
       ↓
accessDeniedHandler()
```

---

# 11. Custom JSON Error Response ⭐⭐⭐⭐⭐

REST APIs mein `sendError()` ki jagah structured JSON useful hota hai.

Example:

```java
@Component
public class RestAuthenticationEntryPoint
        implements AuthenticationEntryPoint {

    private final ObjectMapper objectMapper;

    public RestAuthenticationEntryPoint(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public void commence(
            HttpServletRequest request,
            HttpServletResponse response,
            AuthenticationException exception)
            throws IOException {

        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json");

        Map<String, Object> body = Map.of(
            "status", 401,
            "error", "Unauthorized",
            "message", "Authentication required"
        );

        objectMapper.writeValue(response.getOutputStream(), body);
    }
}
```

Example response:

```json
{
  "status": 401,
  "error": "Unauthorized",
  "message": "Authentication required"
}
```

---

# 12. Custom 403 JSON Response ⭐⭐⭐⭐⭐

```java
@Component
public class RestAccessDeniedHandler
        implements AccessDeniedHandler {

    private final ObjectMapper objectMapper;

    public RestAccessDeniedHandler(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public void handle(
            HttpServletRequest request,
            HttpServletResponse response,
            AccessDeniedException exception)
            throws IOException {

        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.setContentType("application/json");

        Map<String, Object> body = Map.of(
            "status", 403,
            "error", "Forbidden",
            "message", "Access denied"
        );

        objectMapper.writeValue(response.getOutputStream(), body);
    }
}
```

---

# 13. Avoid Sensitive Error Details ⭐⭐⭐⭐⭐

Bad response:

```json
{
  "message": "User nirbhay exists but password is wrong"
}
```

This can reveal account information.

Better:

```json
{
  "status": 401,
  "error": "Unauthorized",
  "message": "Authentication failed"
}
```

### Rule

> External security error messages should be useful without unnecessarily revealing authentication internals.

---

# 14. Authentication Failure vs Controller Exception ⭐⭐⭐⭐⭐

Important distinction.

### Security exception

```text
Authentication / Authorization
        ↓
Security infrastructure
        ↓
EntryPoint / AccessDeniedHandler
```

### Business exception

```text
Controller
   ↓
Service
   ↓
Business exception
   ↓
@RestControllerAdvice
```

`@ControllerAdvice` / `@RestControllerAdvice` is not a replacement for Spring Security's authentication and authorization exception handling.

---

# 15. `@RestControllerAdvice` vs Security Exception Handling ⭐⭐⭐⭐⭐

| Concern | Typical mechanism |
|---|---|
| Invalid/missing authentication | `AuthenticationEntryPoint` |
| Authenticated but forbidden | `AccessDeniedHandler` |
| Business/domain exception | `@RestControllerAdvice` |
| Validation errors | `@RestControllerAdvice` / validation handling |
| Database exception | Application exception handling |

### Interview line

> **Security exceptions should be handled by security infrastructure; application/business exceptions can be handled by controller advice.**

---

# 16. `AuthenticationFailureHandler` vs `AuthenticationEntryPoint` ⭐⭐⭐⭐⭐

These are commonly confused.

### AuthenticationFailureHandler

Often associated with an authentication mechanism such as form login and handles a failed authentication attempt.

Example concept:

```text
Login form submitted
      ↓
Authentication attempt
      ↓
Failure
      ↓
AuthenticationFailureHandler
```

### AuthenticationEntryPoint

Handles the situation where access to a protected resource requires authentication.

```text
Request protected resource
      ↓
Authentication required
      ↓
AuthenticationEntryPoint
```

### Memory

```text
Login attempt failed
      → FailureHandler

Protected resource needs authentication
      → EntryPoint
```

---

# 17. Form Login vs REST API ⭐⭐⭐⭐⭐

Form-login application often uses redirect-oriented behavior.

REST API generally prefers:

```text
401 JSON
403 JSON
```

instead of:

```text
302 redirect to /login
```

So API security configuration should choose handlers appropriate to API clients.

---

# 18. Bearer Token / JWT Exception Handling ⭐⭐⭐⭐⭐

JWT/resource-server applications have additional authentication failure handling concepts.

Typical flow:

```text
Authorization: Bearer <token>
          ↓
Bearer token authentication
          ↓
Token missing / invalid / expired
          ↓
Authentication failure
          ↓
Configured authentication entry point
```

For OAuth2 Resource Server, Spring Security provides bearer-token-specific infrastructure, including authentication entry point and access-denied handling customization.

Conceptually:

```text
JWT invalid
   ↓
Authentication failure
   ↓
401

JWT valid
   ↓
Authenticated
   ↓
Missing required authority
   ↓
403
```

---

# 19. OAuth2 Resource Server Configuration ⭐⭐⭐⭐

Conceptual configuration:

```java
http.oauth2ResourceServer(oauth2 -> oauth2
    .authenticationEntryPoint(authenticationEntryPoint)
    .accessDeniedHandler(accessDeniedHandler)
);
```

Exact DSL options can vary with the Spring Security version and resource-server setup, so always align implementation with the current API version.

---

# 20. Invalid JWT vs Insufficient Scope ⭐⭐⭐⭐⭐

Very common microservices interview question.

### Invalid JWT

```text
Token invalid / expired / malformed
        ↓
Authentication failure
        ↓
401
```

### Valid JWT but insufficient authority/scope

```text
JWT valid
   ↓
Authentication successful
   ↓
Required scope missing
   ↓
Authorization failure
   ↓
403
```

Memory:

> **Invalid token → authentication problem. Valid token + insufficient permission → authorization problem.**

---

# 21. `AccessDeniedHandler` and Anonymous User ⭐⭐⭐⭐

Spring Security's authorization handling has a distinction between an anonymous principal and an authenticated principal.

When an anonymous user attempts to access a protected resource, the framework can route the situation toward authentication-entry-point handling rather than treating it as a normal authenticated-user access denial.

This is why the same protected endpoint can produce different handling depending on whether the caller is authenticated.

---

# 22. Custom `AuthenticationEntryPoint` Best Practices ⭐⭐⭐⭐⭐

- Return consistent API format.
- Set the correct HTTP status.
- Set `Content-Type`.
- Avoid exposing sensitive details.
- Include correlation/request ID when appropriate.
- Do not log raw passwords or bearer tokens.
- Keep response handling lightweight.
- Avoid leaking internal stack traces.

---

# 23. Custom `AccessDeniedHandler` Best Practices ⭐⭐⭐⭐⭐

- Return consistent JSON.
- Use appropriate HTTP status.
- Avoid exposing internal authorization rules.
- Do not return the user's complete role/permission matrix unnecessarily.
- Add trace/correlation information if needed.
- Log security events carefully without secrets.

---

# 24. Logging Security Exceptions ⭐⭐⭐⭐

Security logs should help investigation without leaking secrets.

Good:

```text
requestId=abc123 userId=42 endpoint=/admin/users reason=access_denied
```

Bad:

```text
password=...
token=eyJ...
```

### Never log

- Raw passwords
- Access tokens
- Refresh tokens
- Sensitive authentication credentials

---

# 25. Exception Handling in Filter Chain ⭐⭐⭐⭐⭐

Remember that security exceptions can occur before the request reaches the controller.

Therefore:

```text
HTTP Request
   ↓
Security Filters
   ↓
Security exception
   ↓
Security exception handling
   ↓
Controller may never execute
```

This is why a controller-level `try/catch` or `@RestControllerAdvice` cannot be relied upon to handle every Spring Security exception.

---

# 26. Why `@ControllerAdvice` May Not Catch Security Exceptions ⭐⭐⭐⭐⭐

Example:

```text
Request
  ↓
SecurityFilterChain
  ↓
AccessDeniedException
  ↓
Security exception handling
  ↓
Controller never reached
```

So:

```java
@RestControllerAdvice
```

is not the primary mechanism for these filter-chain security failures.

---

# 27. Custom Error Response DTO ⭐⭐⭐⭐

Production APIs can standardize security errors:

```java
public record SecurityErrorResponse(
    int status,
    String error,
    String message,
    String path,
    String requestId
) {}
```

Then both handlers can use a consistent response contract.

Example:

```json
{
  "status": 403,
  "error": "Forbidden",
  "message": "Access denied",
  "path": "/admin/users",
  "requestId": "abc123"
}
```

Avoid returning sensitive internal policy details.

---

# 28. Exception Handling and CORS/Preflight ⭐⭐⭐⭐

In browser-based APIs, CORS preflight requests can interact with security configuration.

Typical public preflight handling may be required:

```text
OPTIONS request
     ↓
CORS processing
     ↓
Actual API request
```

Security error responses should also be compatible with the application's CORS policy.

This is an integration concern rather than a reason to bypass security broadly.

---

# 29. Exception Handling and CSRF ⭐⭐⭐⭐

CSRF failures are security failures but are conceptually different from authentication and authorization failures.

For example:

```text
Authenticated browser request
      ↓
Missing/invalid CSRF token
      ↓
CSRF protection rejects request
```

Spring Security provides CSRF-specific handling mechanisms/configuration.

Do not solve every security failure by globally disabling CSRF without understanding whether the application is a browser-based stateful application or a stateless API.

---

# 30. Exception Handling and Session Management ⭐⭐⭐⭐

Session expiration can lead to an authentication-related failure from the application's perspective.

Typical browser application:

```text
Session expires
      ↓
Request protected resource
      ↓
Authentication required
      ↓
Entry point / login flow
```

For stateless JWT APIs:

```text
Expired JWT
      ↓
Authentication failure
      ↓
401
```

---

# 31. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — 401 means user has insufficient permission

❌ Usually wrong.

> 401 is generally authentication-related; 403 is generally authorization-related.

### Trap 2 — `@RestControllerAdvice` handles every Spring Security exception

❌ Not reliable.

> Security filters can reject the request before controller invocation.

### Trap 3 — AuthenticationEntryPoint handles every security exception

❌ No.

> AccessDeniedHandler handles authenticated authorization failures.

### Trap 4 — AccessDeniedHandler handles invalid JWT

❌ Usually wrong.

> Invalid/expired JWT is an authentication failure and normally routes to authentication-entry-point handling.

### Trap 5 — Login failure and protected-resource authentication requirement are identical

❌ Not necessarily.

> AuthenticationFailureHandler and AuthenticationEntryPoint have different roles.

### Trap 6 — Return stack traces to API clients

❌ Security risk.

> Return safe, consistent error responses.

### Trap 7 — Log bearer tokens for debugging

❌ Dangerous.

> Never log access/refresh tokens.

---

# 32. Production Architecture ⭐⭐⭐⭐⭐

```text
                    CLIENT
                      ↓
                 HTTP REQUEST
                      ↓
              SecurityFilterChain
                      ↓
           ┌──────────┴──────────┐
           ↓                     ↓
     Authentication          Authorization
           ↓                     ↓
AuthenticationException   AccessDeniedException
           ↓                     ↓
AuthenticationEntryPoint  AccessDeniedHandler
           ↓                     ↓
         401*                  403*

* Typical REST convention; application configuration can customize behavior.
```

---

# 33. REST API Recommended Pattern ⭐⭐⭐⭐⭐

```text
Authentication failure:
HTTP 401
Content-Type: application/json

{
  "status": 401,
  "error": "Unauthorized",
  "message": "Authentication required"
}
```

Authorization failure:

```text
HTTP 403
Content-Type: application/json

{
  "status": 403,
  "error": "Forbidden",
  "message": "Access denied"
}
```

Keep the schema consistent across services.

---

# 34. Interview Questions ⭐⭐⭐⭐⭐

### Q1. How does Spring Security handle authentication exceptions?

> Authentication failures are handled through the configured authentication infrastructure, commonly resulting in an `AuthenticationEntryPoint` being invoked for a protected request that requires authentication.

### Q2. What is AccessDeniedException?

> It represents an authorization failure where access to a protected resource is denied.

### Q3. What is ExceptionTranslationFilter?

> It bridges security exceptions raised in the security filter chain to the configured `AuthenticationEntryPoint` or `AccessDeniedHandler`.

### Q4. AuthenticationEntryPoint vs AccessDeniedHandler?

> AuthenticationEntryPoint handles cases where authentication is required or has failed for protected access; AccessDeniedHandler handles authorization denial for an authenticated principal.

### Q5. Why 401 vs 403?

> 401 generally means authentication is missing or invalid; 403 generally means authentication exists but the principal is not permitted.

### Q6. Why doesn't RestControllerAdvice always catch Security exceptions?

> Because many security checks occur in the filter chain before controller invocation.

### Q7. What is AuthenticationFailureHandler?

> It handles a failed authentication attempt for authentication mechanisms such as form login, whereas AuthenticationEntryPoint is used when access to a protected resource requires authentication.

### Q8. How do you return JSON for 401/403?

> Implement custom AuthenticationEntryPoint and AccessDeniedHandler and configure them through `exceptionHandling()`.

### Q9. What happens when a JWT is expired?

> JWT authentication fails, so the request is normally treated as an authentication failure and handled through the resource-server authentication-entry-point mechanism, commonly resulting in 401.

### Q10. What happens when JWT is valid but scope is missing?

> Authentication succeeds, but authorization fails, commonly resulting in 403 through access-denied handling.

### Q11. Should bearer tokens be logged?

> No. Access and refresh tokens are sensitive credentials and should never be logged.

### Q12. Where should security exceptions be handled?

> In Spring Security's security infrastructure, using its entry-point and access-denied mechanisms; business exceptions remain the responsibility of application exception handling.

---

# 35. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“Spring Security separates authentication failures from authorization failures. When authentication is missing or invalid for a protected resource, the security infrastructure uses an AuthenticationEntryPoint. When the user is authenticated but does not have sufficient authority, an AccessDeniedException occurs and the AccessDeniedHandler handles it. ExceptionTranslationFilter is the key bridge that translates security exceptions raised in the filter chain to these handlers. In REST APIs we commonly return 401 for authentication problems and 403 for authorization problems, with a consistent JSON error structure. We should not depend on RestControllerAdvice because security filters can reject a request before the controller is invoked. For JWT resource servers, an invalid or expired token is an authentication failure, while a valid token without the required scope or authority is an authorization failure. Production handlers should avoid leaking credentials, tokens, stack traces or sensitive security details.”**

---

# 36. 30-Second Hinglish Answer

> **“Spring Security mein authentication aur authorization exceptions ko alag handle kiya jata hai. Authentication missing ya invalid ho to `AuthenticationEntryPoint` use hota hai, aur authenticated user ke paas required authority na ho to `AccessDeniedHandler` use hota hai. `ExceptionTranslationFilter` security exceptions ko in handlers tak route karta hai. REST API mein generally authentication failure ke liye 401 aur authorization failure ke liye 403 return karte hain. `@RestControllerAdvice` par depend nahi karna chahiye because security filter chain controller se pehle exception handle kar sakti hai.”**

---

# 37. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                    REQUEST
                       ↓
              SecurityFilterChain
                       ↓
                Security Failure
                       ↓
          ExceptionTranslationFilter
                       ↓
          ┌────────────┴────────────┐
          ↓                         ↓
 AuthenticationException      AccessDeniedException
          ↓                         ↓
AuthenticationEntryPoint     AccessDeniedHandler
          ↓                         ↓
       401-ish*                  403-ish*

* Typical REST convention; configuration can customize response.
```

### Final memory line

> **Authentication problem → EntryPoint | Authorization problem → AccessDeniedHandler | Security filters can fail before ControllerAdvice.**

---

## Navigation

[← S4.1.15 — Authorization and Access Decisions](../15-Authorization-and-Access-Decisions/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.17 — CSRF Protection**
