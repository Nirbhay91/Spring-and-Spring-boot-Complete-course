# S4.1.18 — Session Management

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is Session Management? ⭐⭐⭐⭐⭐

HTTP is stateless. Session management allows a server-side application to maintain authentication/state across multiple requests.

```text
Login Request
     ↓
Authentication successful
     ↓
Server creates/uses session
     ↓
Session ID sent to browser
     ↓
Browser sends session cookie
     ↓
Server retrieves session
     ↓
User remains authenticated
```

In Spring Security, session management is closely related to the `SecurityContext`, authentication, session fixation protection, concurrency control, and session creation policy.

---

# 2. Session vs Authentication ⭐⭐⭐⭐⭐

Do not treat these as the same thing.

```text
Session
  ↓
State container across requests

Authentication
  ↓
Represents authenticated principal
```

A session can contain the security context that represents the authenticated user, depending on the application's configuration and authentication model.

---

# 3. `JSESSIONID` ⭐⭐⭐⭐⭐

Traditional servlet applications commonly use a session cookie such as:

```text
Cookie: JSESSIONID=abc123
```

Conceptually:

```text
Browser
   ↓
JSESSIONID
   ↓
Application server
   ↓
HTTP Session
   ↓
SecurityContext / application state
```

### Important

`JSESSIONID` is normally an identifier, not the user's password or complete authentication object.

---

# 4. Session-Based Authentication Flow ⭐⭐⭐⭐⭐

```text
POST /login
      ↓
AuthenticationManager
      ↓
Authentication successful
      ↓
SecurityContext populated
      ↓
Context associated with session
      ↓
JSESSIONID cookie
      ↓
Subsequent request
      ↓
SecurityContext restored
      ↓
Authenticated user
```

This is a common browser/server application model.

---

# 5. `SessionManagementConfigurer` ⭐⭐⭐⭐⭐

Spring Security exposes session-management configuration through `sessionManagement()`.

Example:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http
        .sessionManagement(session -> session
            .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
        );

    return http.build();
}
```

It can also configure:

- Session creation policy
- Session fixation protection
- Concurrent session control
- Invalid session handling
- Session-related security behavior

---

# 6. `SessionCreationPolicy` ⭐⭐⭐⭐⭐

Spring Security provides policies controlling when sessions may be created/used.

Main values:

```text
ALWAYS
IF_REQUIRED
NEVER
STATELESS
```

---

# 7. `ALWAYS` ⭐⭐⭐⭐

```java
.sessionCreationPolicy(SessionCreationPolicy.ALWAYS)
```

Spring Security will create an `HttpSession` if one does not already exist.

Conceptually:

```text
Request
  ↓
No session
  ↓
Create session
```

Use only when session creation is actually appropriate for the application.

---

# 8. `IF_REQUIRED` ⭐⭐⭐⭐⭐

```java
.sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
```

A session is created when required.

This is a common choice for traditional stateful web applications.

Memory:

> **IF_REQUIRED = session when necessary, not necessarily for every request.**

---

# 9. `NEVER` ⭐⭐⭐⭐

```java
.sessionCreationPolicy(SessionCreationPolicy.NEVER)
```

Spring Security will not create a session itself, but it can use an existing session.

Conceptually:

```text
Existing session? → Can use it
Need to create one? → Spring Security won't create it itself
```

---

# 10. `STATELESS` ⭐⭐⭐⭐⭐

```java
.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
```

The application does not use the HTTP session to obtain the `SecurityContext` for authentication in the normal stateless security model.

Common architecture:

```text
Request
  ↓
Authorization: Bearer JWT
  ↓
Authenticate token
  ↓
SecurityContext for request
  ↓
Response
  ↓
No server-side login session required
```

### Important

> **STATELESS is an authentication/session policy, not a synonym for “the server stores absolutely no state anywhere.”**

---

# 11. `STATELESS` vs `NEVER` ⭐⭐⭐⭐⭐

| Policy | Create session by Spring Security | Use existing session |
|---|---:|---:|
| `ALWAYS` | Yes | Yes |
| `IF_REQUIRED` | If required | Yes |
| `NEVER` | No | Yes |
| `STATELESS` | No | No for obtaining security context in the stateless model |

### Interview memory

> **NEVER can use an existing session; STATELESS does not use the session for security context persistence.**

---

# 12. Session Fixation Attack ⭐⭐⭐⭐⭐

Session fixation occurs when an attacker somehow causes a victim to authenticate using a session identifier already known to the attacker.

Conceptually:

```text
Attacker knows session ID
        ↓
Victim uses same session
        ↓
Victim authenticates
        ↓
Attacker may attempt to reuse known session ID
```

The defense is to change the session identifier when authentication occurs.

---

# 13. Session Fixation Protection ⭐⭐⭐⭐⭐

Modern Spring Security provides session-fixation protection for relevant session-based authentication flows.

Conceptually:

```text
Before authentication
Session ID = ABC
       ↓
Authentication successful
       ↓
Session identity changes / session is migrated
       ↓
New session ID = XYZ
```

This prevents an attacker from continuing to rely on the pre-authentication session identifier.

---

# 14. `sessionFixation()` Configuration ⭐⭐⭐⭐⭐

Spring Security provides configuration options for session fixation behavior.

Conceptually:

```java
.sessionManagement(session -> session
    .sessionFixation(fixation -> fixation
        .changeSessionId()
    )
)
```

Depending on the servlet container and Spring Security version, available strategies can include changing the session ID or migrating/creating the session.

### Security principle

> **The post-authentication session identity should not remain attacker-controlled.**

---

# 15. Session Fixation Strategies ⭐⭐⭐⭐

Historically and across servlet environments, you may encounter strategies such as:

```text
changeSessionId()
migrateSession()
newSession()
none()
```

`changeSessionId()` changes the identifier while preserving the session's relevant state where supported by the servlet container.

`migrateSession()` creates a new session and copies session attributes.

`newSession()` creates a new session without copying application attributes.

`none()` disables Spring Security's session-fixation protection and should only be considered with a strong architectural reason.

---

# 16. Concurrent Session Control ⭐⭐⭐⭐⭐

A common security requirement is limiting how many sessions one user can have simultaneously.

Example:

```text
User A
 ├── Laptop session
 ├── Mobile session
 └── Another browser session
```

Application may choose a policy such as:

```text
Maximum sessions = 1
```

Then a new login can cause an older session to become invalid/expired, depending on configuration.

---

# 17. `maximumSessions()` ⭐⭐⭐⭐⭐

Conceptual configuration:

```java
.sessionManagement(session -> session
    .maximumSessions(1)
)
```

This is useful for applications that require concurrent-login restrictions.

### Important

Concurrency control requires session tracking infrastructure; it is not relevant to a purely stateless bearer-token model in the same way.

---

# 18. `maxSessionsPreventsLogin()` ⭐⭐⭐⭐

Two common policy styles are:

```text
Allow new login
→ expire previous session

OR

Prevent new login
→ keep existing session active
```

Conceptually:

```java
.maximumSessions(1)
.maxSessionsPreventsLogin(true)
```

The exact behavior should be tested for the authentication flow and Spring Security version being used.

---

# 19. Session Expiration ⭐⭐⭐⭐⭐

A session can become invalid because of:

- Explicit logout
- Timeout
- Concurrent-session policy
- Administrative invalidation
- Server/container restart in some deployments
- Session store failure

Typical flow:

```text
Authenticated session
       ↓
Session expires
       ↓
Next protected request
       ↓
Authentication no longer available
       ↓
Authentication entry point / login flow
```

---

# 20. Invalid Session Handling ⭐⭐⭐⭐

Applications can configure handling for invalid sessions.

Conceptually:

```java
.sessionManagement(session -> session
    .invalidSessionUrl("/login?invalid")
)
```

This is more common in browser-based applications where redirecting to a login page is appropriate.

For REST APIs, a JSON response is generally more appropriate than an HTML redirect.

---

# 21. REST API Session Strategy ⭐⭐⭐⭐⭐

For a stateless REST API:

```text
Client
  ↓
Authorization: Bearer JWT
  ↓
Authentication
  ↓
Request processing
  ↓
Response
```

No server-side login session is required.

Typical configuration:

```java
http.sessionManagement(session -> session
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
);
```

### Important

This should be aligned with the authentication mechanism. Do not set `STATELESS` blindly if the application relies on session-based authentication.

---

# 22. Session vs JWT ⭐⭐⭐⭐⭐

| Feature | Session-based | JWT bearer-based |
|---|---|---|
| Server session | Usually yes | Usually no |
| Credential | Session ID/cookie | Bearer token |
| Revocation | Session invalidation is straightforward | Requires token strategy/expiry/revocation design |
| Horizontal scaling | Needs shared session/sticky/session store strategy | Easier from session perspective |
| Browser cookie CSRF concern | Yes | Depends on token transport |
| Logout | Invalidate session | Client removes token; server-side revocation may be needed depending on requirements |

### Interview point

> **JWT does not automatically make an architecture better; the choice depends on revocation, client type, scaling, security and operational requirements.**

---

# 23. `SecurityContext` and Session ⭐⭐⭐⭐⭐

In session-based applications, Spring Security can persist the authenticated `SecurityContext` across requests.

Conceptually:

```text
Request 1
   ↓
Login
   ↓
SecurityContext
   ↓
Session

Request 2
   ↓
Session
   ↓
SecurityContext restored
   ↓
Authentication available
```

In stateless bearer authentication:

```text
Each request
   ↓
Bearer token
   ↓
Authentication
   ↓
Request SecurityContext
```

---

# 24. `SecurityContextRepository` ⭐⭐⭐⭐⭐

`SecurityContextRepository` abstracts how the security context is loaded and saved across requests.

This becomes important when discussing modern Spring Security versions and stateless vs stateful security.

Conceptually:

```text
Request
 ↓
SecurityContextRepository
 ↓
Load SecurityContext
 ↓
Security filters
 ↓
Authentication
 ↓
Save according to policy
```

Common session-oriented implementations use the HTTP session.

---

# 25. `SecurityContextHolderFilter` / Persistence ⭐⭐⭐⭐

Modern Spring Security has evolved its security-context persistence architecture.

The important interview concept is:

> **SecurityContext persistence is separate from the mere existence of an HTTP session.**

Depending on the version/configuration, filters such as `SecurityContextHolderFilter` and `SecurityContextPersistenceFilter` may appear in discussions.

Always answer based on the Spring Security version used by the application rather than assuming every historical filter behaves identically.

---

# 26. Session Cookie Security ⭐⭐⭐⭐⭐

For browser session applications, protect the session cookie.

Important attributes include:

```text
Secure
HttpOnly
SameSite
```

### Secure

Cookie should be sent over HTTPS.

### HttpOnly

Helps prevent JavaScript from reading the cookie.

### SameSite

Controls cross-site cookie sending behavior.

### Important

These attributes reduce attack surface but do not replace complete session-security design.

---

# 27. Session Timeout ⭐⭐⭐⭐⭐

A timeout limits how long an inactive session remains valid.

Example Spring Boot configuration:

```properties
server.servlet.session.timeout=30m
```

The exact timeout semantics depend on the servlet container and application configuration.

### Security benefit

Shorter sessions reduce the window for misuse of an abandoned authenticated browser session.

But excessively short timeouts can hurt user experience.

---

# 28. Absolute vs Idle Timeout ⭐⭐⭐⭐

### Idle timeout

Session expires after a period without activity.

```text
No activity for 30 minutes
       ↓
Session expires
```

### Absolute timeout

Session expires after a maximum lifetime regardless of activity.

```text
Login
 ↓
Maximum lifetime reached
 ↓
Session expires
```

High-security systems may use both policies.

---

# 29. Logout and Session Invalidation ⭐⭐⭐⭐⭐

A session-based logout commonly performs operations such as:

```text
Logout request
   ↓
Clear authentication
   ↓
Invalidate session / clear session state
   ↓
Clear relevant cookies
   ↓
Logged out
```

Spring Security's logout support can coordinate these operations.

---

# 30. Session Logout vs JWT Logout ⭐⭐⭐⭐⭐

### Session

```text
Logout
 ↓
Invalidate server-side session
 ↓
Future request with old session ID
 ↓
Authentication unavailable
```

### JWT

```text
Logout
 ↓
Client removes token
```

If immediate server-side invalidation is required, JWT architecture needs additional mechanisms such as token revocation/denylisting or short-lived access tokens plus refresh-token controls.

---

# 31. Distributed Sessions ⭐⭐⭐⭐⭐

In multiple application instances:

```text
              Load Balancer
             /      |      \
            ↓       ↓       ↓
         App-1    App-2    App-3
            \       |       /
             \      |      /
               Shared Session Store
```

Possible approaches include:

- Spring Session
- Redis-backed session storage
- Database-backed session storage
- Sticky sessions (with operational trade-offs)

### Interview point

> **A stateful application can still scale horizontally, but session state needs an appropriate distribution strategy.**

---

# 32. Spring Session ⭐⭐⭐⭐⭐

Spring Session provides infrastructure for managing session state independently from the servlet container's local memory.

A common architecture:

```text
Application instances
       ↓
Spring Session
       ↓
Redis / JDBC / other supported store
```

This helps multiple instances access the same logical session.

---

# 33. Session Replication vs Shared Store ⭐⭐⭐⭐

### Replication

Session data is replicated between application nodes.

### Shared store

All nodes access a central/distributed session store.

Modern deployments often prefer a dedicated shared store strategy when session state is required, depending on scale and operational requirements.

---

# 34. Session Concurrency and Distributed Systems ⭐⭐⭐⭐

Concurrent session control becomes more complex when multiple application instances are involved.

```text
Login on App-1
       ↓
Session registry/store
       ↓
Request on App-2
       ↓
Same session/security policy
```

A shared session/concurrency registry is needed when the policy must work consistently across nodes.

---

# 35. Session Security Threats ⭐⭐⭐⭐⭐

Important threats:

- Session fixation
- Session hijacking
- Session theft
- Session replay
- Excessive session lifetime
- Insecure transport
- Weak cookie configuration
- Session leakage through logs
- Missing logout invalidation

Mitigations:

```text
HTTPS
+ Secure cookie
+ HttpOnly
+ SameSite
+ Session fixation protection
+ Reasonable timeout
+ Proper logout
+ No session IDs in URLs
+ Secure logging
```

---

# 36. Never Put Session IDs in URLs ⭐⭐⭐⭐⭐

Avoid URL-based session tracking such as:

```text
https://example.com/account;jsessionid=ABC123
```

Why?

Session IDs in URLs can leak through:

- Browser history
- Logs
- Analytics
- Referrer information
- Screenshots/shared URLs

Cookie-based session tracking is generally preferred.

---

# 37. Session Hijacking ⭐⭐⭐⭐⭐

If an attacker obtains a valid session identifier:

```text
Attacker
   ↓
Stolen session ID
   ↓
Target application
   ↓
Server sees valid session
   ↓
Attacker impersonates victim
```

Mitigations include:

- HTTPS
- Secure/HttpOnly cookies
- SameSite policy
- Session fixation protection
- Short/appropriate timeouts
- Reauthentication for sensitive actions
- Avoiding session IDs in URLs

---

# 38. Session Management and CSRF ⭐⭐⭐⭐⭐

Session-based authentication commonly uses cookies.

Therefore:

```text
Session cookie
   ↓
Browser automatically sends cookie
   ↓
CSRF risk
```

Session security and CSRF protection should be designed together.

```text
Session security
      +
CSRF protection
      +
Cookie security
```

---

# 39. Session Management and CORS ⭐⭐⭐⭐

If a browser application uses cookies across origins, CORS configuration becomes security-sensitive.

Avoid broad configurations such as:

```text
Allow-Origin: *
```

when credentials are involved.

Explicitly define trusted origins and understand the browser's credential/CORS behavior.

---

# 40. Session Management with SSO ⭐⭐⭐⭐

In SSO systems, an application session and identity-provider session are separate concepts.

```text
Browser
  ↓
Application session
  ↓
Application

Identity Provider session
  ↓
SSO system
```

Logging out from one layer does not necessarily mean every related SSO session is immediately terminated unless the architecture supports coordinated logout.

---

# 41. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — `STATELESS` means no state anywhere

❌ Wrong.

> It means the security model does not use the HTTP session to maintain authentication state across requests in the normal stateless setup.

### Trap 2 — `NEVER` and `STATELESS` are identical

❌ Wrong.

> `NEVER` can use an existing session; `STATELESS` does not use the session for the security context in the stateless model.

### Trap 3 — JWT always means no session

❌ Not necessarily.

> An application can technically combine tokens with other state, and token transport/storage matters.

### Trap 4 — Session fixation means session timeout

❌ Different problems.

> Fixation concerns an attacker influencing the pre-authentication session identity; timeout limits session lifetime/inactivity.

### Trap 5 — Session timeout alone prevents hijacking

❌ No.

> It only reduces the lifetime/window; it does not protect the session ID itself.

### Trap 6 — Sticky sessions are the only scaling solution

❌ No.

> Shared/distributed session stores are another common approach.

### Trap 7 — `@RestControllerAdvice` manages session security

❌ No.

> Session/security behavior belongs to the security/session infrastructure.

---

# 42. Production Architecture — Stateful ⭐⭐⭐⭐⭐

```text
                 Browser
                    ↓
              HTTPS Request
                    ↓
              Session Cookie
                    ↓
           Load Balancer / Gateway
                    ↓
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    App-1         App-2        App-3
       \            |            /
        \           |           /
          Shared Session Store
                 ↓
          SecurityContext
                 ↓
             Authentication
```

---

# 43. Production Architecture — Stateless ⭐⭐⭐⭐⭐

```text
Client
  ↓
Authorization: Bearer JWT
  ↓
Load Balancer
  ↓
┌────────┬────────┬────────┐
│ App-1  │ App-2  │ App-3  │
└────────┴────────┴────────┘
     ↓
Authenticate token per request
     ↓
Request SecurityContext
     ↓
Response
```

No shared login session is required for the normal authentication flow.

---

# 44. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is session management?

> It is the mechanism used to maintain state, commonly authentication state, across multiple HTTP requests in a web application.

### Q2. What is `SessionCreationPolicy.STATELESS`?

> It tells Spring Security not to use the HTTP session to obtain/store the security context for the normal authentication flow, which is common for bearer-token APIs.

### Q3. `NEVER` vs `STATELESS`?

> `NEVER` does not create a session through Spring Security but can use an existing session; `STATELESS` does not use the session for the security context in the stateless model.

### Q4. What is session fixation?

> An attack where an attacker attempts to make a victim authenticate using a session identifier known to the attacker. Session-fixation protection changes/migrates the session identity after authentication.

### Q5. How do you prevent multiple logins?

> Configure concurrent session control such as `maximumSessions()` for session-based authentication.

### Q6. How do you scale session-based applications?

> Use shared/distributed session storage such as Spring Session with Redis/JDBC, or another suitable session distribution strategy.

### Q7. Session vs JWT?

> Session authentication keeps server-side authentication state and identifies it through a session cookie; JWT bearer authentication carries authentication information in the token and normally avoids a server-side login session.

### Q8. Does JWT eliminate session management?

> It can eliminate the need for a server-side authentication session, but token lifecycle, refresh tokens, revocation and client storage still require security management.

### Q9. How does logout work with sessions?

> Typically the server clears authentication and invalidates the session so subsequent requests cannot reuse the authenticated session.

### Q10. How do you protect session cookies?

> Use HTTPS, Secure, HttpOnly and appropriate SameSite settings, plus fixation protection, sensible timeouts and secure session lifecycle management.

### Q11. Why shouldn't session IDs be placed in URLs?

> They can leak through history, logs, analytics and referrer information.

### Q12. What happens when a session expires?

> The next protected request no longer has the required authenticated security context and the application normally starts a new authentication flow, such as a login redirect or 401 response depending on the client/application.

---

# 45. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“Spring Security session management controls how HTTP sessions are created, used and secured for authentication. In a stateful browser application, successful authentication can be associated with an HTTP session and represented to the browser through a session cookie such as JSESSIONID. Spring Security provides SessionCreationPolicy values such as ALWAYS, IF_REQUIRED, NEVER and STATELESS. The key difference is that NEVER can use an existing session, whereas STATELESS does not use the HTTP session for the security context in the normal stateless model. Session fixation protection changes or migrates the session identity after authentication. Spring Security also supports concurrent session control through maximumSessions for stateful applications. For distributed deployments, stateful sessions need a shared/distributed strategy such as Spring Session with Redis or JDBC. Session security should include HTTPS, Secure/HttpOnly/SameSite cookies, appropriate timeouts, proper logout, and no session IDs in URLs. For stateless JWT APIs, authentication is typically reconstructed from the bearer token on each request instead of relying on a server-side login session.”**

---

# 46. 30-Second Hinglish Answer

> **“Session Management ka purpose multiple HTTP requests ke across authentication state maintain karna hai. Stateful application mein user login ke baad session create hota hai aur usually JSESSIONID cookie ke through next requests identify hote hain. Spring Security mein `IF_REQUIRED`, `NEVER`, `ALWAYS` aur `STATELESS` policies hoti hain. `NEVER` existing session use kar sakta hai, jabki `STATELESS` authentication ke liye HTTP session use nahi karta. Session fixation protection login ke baad session identity change karta hai. Multiple logins control karne ke liye `maximumSessions()` use kar sakte hain. JWT bearer-based stateless API mein generally server-side authentication session ki need nahi hoti.”**

---

# 47. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
             STATEFUL
                 ↓
          Login / Authentication
                 ↓
             HTTP Session
                 ↓
            JSESSIONID
                 ↓
      SecurityContext restored
                 ↓
          Authenticated user


             STATELESS
                 ↓
       Authorization: Bearer JWT
                 ↓
       Authenticate each request
                 ↓
        Request SecurityContext
                 ↓
              Response
```

### Final memory line

> **STATEFUL → session carries authentication across requests | STATELESS → authentication is reconstructed per request | Fixation protection + cookie security + timeout + logout = secure session lifecycle.**

---

## Navigation

[← S4.1.17 — CSRF Protection](../17-CSRF-Protection/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.19 — Remember-Me Authentication**
