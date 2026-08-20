# S4.1.15 — Authorization and Access Decisions

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture ⭐⭐⭐⭐⭐

Authorization ka purpose decide karna hai ki **authenticated user/principal ko requested resource ya operation access karne ki permission hai ya nahi**.

```text
Request
  ↓
Authentication
  ↓
Principal + GrantedAuthorities
  ↓
Authorization rules
  ↓
Access Decision
  ↓
ALLOW / DENY
```

### Golden rule

> **Authentication answers “Who are you?”; Authorization answers “What are you allowed to do?”**

---

# 2. Authentication vs Authorization ⭐⭐⭐⭐⭐

### Authentication

Identity verify karta hai:

```text
Username + Password
        ↓
Authentication
        ↓
Authenticated Principal
```

### Authorization

Access permission check karta hai:

```text
Authenticated Principal
        ↓
Authorities / Roles
        ↓
Authorization Rule
        ↓
ALLOW / DENY
```

Example:

```text
Nirbhay authenticated hai
        ↓
ROLE_USER
        ↓
/admin/users
        ↓
DENIED
```

---

# 3. What is an Access Decision? ⭐⭐⭐⭐⭐

Authorization system ultimately ek decision produce karta hai:

```text
GRANTED
```

or

```text
DENIED
```

Example:

```java
.requestMatchers("/admin/**")
.hasRole("ADMIN")
```

If user has:

```text
ROLE_ADMIN
```

then:

```text
Access = GRANTED
```

Otherwise:

```text
Access = DENIED
```

---

# 4. Where Authorization Happens ⭐⭐⭐⭐⭐

Spring Security applications mein authorization multiple layers par ho sakta hai:

```text
HTTP Request
    ↓
Request-level authorization

Service / Method
    ↓
Method-level authorization

Business ownership rules
    ↓
Application-level authorization
```

Examples:

```java
.requestMatchers("/admin/**").hasRole("ADMIN")
```

and:

```java
@PreAuthorize("hasAuthority('ORDER_WRITE')")
```

---

# 5. Request Authorization with SecurityFilterChain ⭐⭐⭐⭐⭐

Modern Spring Security configuration:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/public/**").permitAll()
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .requestMatchers("/orders/**")
            .hasAuthority("ORDER_READ")
        .anyRequest().authenticated()
    );

    return http.build();
}
```

Conceptually:

```text
Request
  ↓
SecurityFilterChain
  ↓
authorizeHttpRequests
  ↓
Matching authorization rule
  ↓
Decision
```

---

# 6. `permitAll()` ⭐⭐⭐⭐⭐

Allows everyone to access the matched endpoint.

```java
.requestMatchers("/login", "/public/**")
.permitAll()
```

Important:

> `permitAll()` means authorization does not require the caller to be authenticated for that request. It does not mean the application should expose sensitive business operations.

---

# 7. `authenticated()` ⭐⭐⭐⭐⭐

Requires an authenticated principal:

```java
.anyRequest().authenticated()
```

This means:

```text
Authenticated → allowed by this rule
Unauthenticated → denied
```

It does not require a particular role.

---

# 8. `hasRole()` ⭐⭐⭐⭐⭐

Example:

```java
.requestMatchers("/admin/**")
.hasRole("ADMIN")
```

Under the standard role-prefix convention:

```text
hasRole("ADMIN")
       ↓
ROLE_ADMIN
```

So the authenticated principal needs the corresponding role authority.

---

# 9. `hasAuthority()` ⭐⭐⭐⭐⭐

Checks an authority value directly:

```java
.requestMatchers("/orders/**")
.hasAuthority("ORDER_READ")
```

The user needs:

```text
ORDER_READ
```

### Memory

```text
hasRole("ADMIN")
      ↓
role-style check

hasAuthority("ORDER_READ")
      ↓
exact authority check
```

---

# 10. `hasAnyRole()` / `hasAnyAuthority()` ⭐⭐⭐⭐

```java
.requestMatchers("/reports/**")
.hasAnyRole("ADMIN", "MANAGER")
```

or:

```java
.requestMatchers("/orders/**")
.hasAnyAuthority("ORDER_READ", "ORDER_ADMIN")
```

Conceptually:

```text
ADMIN OR MANAGER
```

or:

```text
ORDER_READ OR ORDER_ADMIN
```

---

# 11. `denyAll()` ⭐⭐⭐⭐

Explicitly denies access:

```java
.requestMatchers("/internal/**")
.denyAll()
```

Useful when an endpoint should never be exposed through the current security configuration.

---

# 12. Rule Ordering ⭐⭐⭐⭐⭐

Authorization rules are evaluated according to their matching order/semantics, so specific rules should generally be placed before broad catch-all rules.

Example:

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .requestMatchers("/**").authenticated()
)
```

The catch-all rule should not accidentally make the specific authorization policy unreachable.

### Interview point

> **Authorization configuration should be ordered from specific rules to broader fallback rules.**

---

# 13. `anyRequest()` ⭐⭐⭐⭐⭐

Common fallback:

```java
.anyRequest().authenticated()
```

This means any request not matched by an earlier authorization rule must be authenticated.

Example:

```text
/public/** → permitAll
/admin/**  → ADMIN
/orders/** → ORDER_READ
Everything else → authenticated
```

---

# 14. Request Matcher ⭐⭐⭐⭐⭐

Authorization rules need to identify which requests they apply to.

Example:

```java
.requestMatchers("/admin/**")
.hasRole("ADMIN")
```

Common matching dimensions can include:

- URL path
- HTTP method
- Request characteristics
- Custom request matchers

Example with HTTP method:

```java
.requestMatchers(HttpMethod.GET, "/orders/**")
.hasAuthority("ORDER_READ")
```

---

# 15. HTTP Method-Based Authorization ⭐⭐⭐⭐⭐

Different operations can require different authorities:

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers(HttpMethod.GET, "/orders/**")
        .hasAuthority("ORDER_READ")
    .requestMatchers(HttpMethod.POST, "/orders/**")
        .hasAuthority("ORDER_CREATE")
    .requestMatchers(HttpMethod.DELETE, "/orders/**")
        .hasAuthority("ORDER_DELETE")
)
```

This provides finer-grained authorization than simply protecting the entire URL with one role.

---

# 16. Method-Level Authorization ⭐⭐⭐⭐⭐

Enable it:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {
}
```

Then:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) {
    // business logic
}
```

Authority example:

```java
@PreAuthorize("hasAuthority('ORDER_WRITE')")
public void updateOrder(Long id) {
    // business logic
}
```

---

# 17. Why Method Security? ⭐⭐⭐⭐⭐

URL-level security says:

```text
Who can call this endpoint?
```

Method-level security can express:

```text
Who can execute this business operation?
```

This is useful when authorization depends on service-level business rules.

Example:

```java
@PreAuthorize("hasAuthority('ORDER_READ')")
public Order getOrder(Long orderId) {
    ...
}
```

---

# 18. `@PreAuthorize()` ⭐⭐⭐⭐⭐

Checks an authorization expression **before method invocation**.

Example:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) {
}
```

More complex expression:

```java
@PreAuthorize("hasAuthority('ORDER_READ')")
public Order getOrder(Long id) {
    ...
}
```

---

# 19. `@PostAuthorize()` ⭐⭐⭐⭐

Checks authorization after method execution, which can be useful when the decision depends on the returned object.

Conceptually:

```text
Method executes
      ↓
Return object
      ↓
Authorization expression
      ↓
Allow / Deny
```

Example concept:

```java
@PostAuthorize("returnObject.owner == authentication.name")
public Order findOrder(Long id) {
    ...
}
```

### Important

Because the method executes before the authorization check, it must be used carefully for operations with side effects.

---

# 20. `@Secured` ⭐⭐⭐⭐

Spring Security also supports `@Secured` for role-style authorization when method security is configured appropriately.

Example:

```java
@Secured("ROLE_ADMIN")
public void deleteUser(Long id) {
}
```

Compared with `@PreAuthorize`, `@Secured` is less expressive because `@PreAuthorize` supports authorization expressions.

---

# 21. `@RolesAllowed` ⭐⭐⭐⭐

JSR/Jakarta-style role authorization can be enabled/configured depending on the application stack.

Example:

```java
@RolesAllowed("ADMIN")
public void deleteUser(Long id) {
}
```

Exact behavior and role-prefix expectations should be verified against the application's Spring Security version/configuration.

---

# 22. Authorization Manager ⭐⭐⭐⭐⭐

Modern Spring Security uses the **AuthorizationManager** abstraction for authorization decisions.

Conceptually:

```text
Authentication
      +
Secured Object / Request
      ↓
AuthorizationManager
      ↓
Authorization Decision
      ↓
GRANTED / DENIED
```

This is an important modern Spring Security concept.

---

# 23. AuthorizationManager — Why Important? ⭐⭐⭐⭐⭐

Older Spring Security versions used authorization concepts such as:

```text
AccessDecisionManager
AccessDecisionVoter
```

Modern Spring Security favors:

```text
AuthorizationManager
```

So for current-version interviews:

> **Know AuthorizationManager as the modern authorization abstraction, while understanding AccessDecisionManager/Voter as legacy/older architecture concepts.**

---

# 24. Authorization Decision ⭐⭐⭐⭐⭐

Modern authorization APIs conceptually produce an `AuthorizationDecision`.

```text
AuthorizationManager
        ↓
AuthorizationDecision
        ↓
Granted / Not Granted
```

The decision is based on the current authentication and the protected resource/request.

---

# 25. Request Authorization Flow ⭐⭐⭐⭐⭐

```text
HTTP Request
     ↓
Security Filters
     ↓
Authentication established?
     ↓
Authentication object
     ↓
AuthorizationFilter / authorization infrastructure
     ↓
AuthorizationManager
     ↓
Match rule
     ↓
Evaluate authorities
     ↓
AuthorizationDecision
     ↓
 ┌───────────────┐
 │               │
GRANTED        DENIED
 │               │
 ↓               ↓
Controller   AccessDenied
```

For unauthenticated access to a protected resource, Spring Security's exception handling can route the request through the appropriate authentication-entry-point behavior.

---

# 26. AccessDenied vs AuthenticationEntryPoint ⭐⭐⭐⭐⭐

Very important distinction.

### Unauthenticated

User has not established authentication but accesses a protected resource.

```text
No authenticated principal
        ↓
Authentication required
        ↓
AuthenticationEntryPoint
```

### Authenticated but unauthorized

User is authenticated but lacks required authority.

```text
Authenticated
      ↓
Required authority missing
      ↓
AccessDeniedHandler
```

### Memory

```text
401-ish authentication problem → AuthenticationEntryPoint
403-ish authorization problem  → AccessDeniedHandler
```

Exact response status can also be influenced by application configuration and authentication mechanism.

---

# 27. 401 vs 403 ⭐⭐⭐⭐⭐

Common REST API convention:

```text
401 Unauthorized
```

usually means authentication is missing/invalid.

```text
403 Forbidden
```

usually means the request is authenticated but access is forbidden.

### Interview trap

> **401 is about authentication; 403 is about authorization.**

---

# 28. Role-Based Authorization Example ⭐⭐⭐⭐⭐

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/login", "/public/**")
            .permitAll()
        .requestMatchers("/admin/**")
            .hasRole("ADMIN")
        .requestMatchers("/manager/**")
            .hasAnyRole("ADMIN", "MANAGER")
        .anyRequest()
            .authenticated()
    );

    return http.build();
}
```

---

# 29. Permission-Based Authorization Example ⭐⭐⭐⭐⭐

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http.authorizeHttpRequests(auth -> auth
        .requestMatchers(HttpMethod.GET, "/orders/**")
            .hasAuthority("ORDER_READ")
        .requestMatchers(HttpMethod.POST, "/orders/**")
            .hasAuthority("ORDER_CREATE")
        .requestMatchers(HttpMethod.DELETE, "/orders/**")
            .hasAuthority("ORDER_DELETE")
        .anyRequest()
            .authenticated()
    );

    return http.build();
}
```

This is closer to fine-grained permission-based authorization.

---

# 30. Ownership / Business Authorization ⭐⭐⭐⭐⭐

Role-based access alone is not always enough.

Example:

```text
User A → ORDER_READ
```

But should User A read **every** order?

Maybe only their own orders.

Business rule:

```text
User can read order
IF
user has ORDER_READ
AND
order.ownerId == authenticatedUserId
```

This can be expressed using method security or service-layer business authorization.

---

# 31. RBAC vs ABAC ⭐⭐⭐⭐⭐

### RBAC

```text
User → Role → Permission
```

Example:

```text
Nirbhay → ADMIN → USER_DELETE
```

### ABAC

Authorization can depend on attributes:

```text
User attributes
Resource attributes
Action
Environment
```

Example:

```text
Allow UPDATE
IF
user.department == order.department
AND
user.clearance >= order.classification
```

Spring Security expressions/custom authorization components can support more advanced business policies.

---

# 32. JWT Authorization Flow ⭐⭐⭐⭐⭐

```text
Client
  ↓
Bearer JWT
  ↓
JWT validation
  ↓
Authentication
  ↓
Claims → GrantedAuthority
  ↓
AuthorizationManager
  ↓
Authorization decision
  ↓
ALLOW / DENY
```

Important:

> JWT validation proves/establishes the token-based authentication context; authorization still requires evaluating the resulting authorities/policies.

---

# 33. OAuth2 Scopes and Authorization ⭐⭐⭐⭐⭐

A resource server may receive:

```text
scope = orders.read
```

Depending on configuration, this can map to:

```text
SCOPE_orders.read
```

Then:

```java
.hasAuthority("SCOPE_orders.read")
```

can be used.

Don't assume all identity providers use the same claim names or prefixes.

---

# 34. Custom AuthorizationManager ⭐⭐⭐⭐⭐

Complex business authorization can be implemented with a custom `AuthorizationManager`.

Conceptual structure:

```java
AuthorizationManager<RequestAuthorizationContext>
```

It can evaluate:

```text
Authentication
        +
Request / context
        ↓
Business policy
        ↓
AuthorizationDecision
```

Use this when simple role/authority expressions are not enough and a reusable authorization policy is required.

---

# 35. Authorization in Microservices ⭐⭐⭐⭐⭐

In microservices:

```text
API Gateway
    ↓
Authentication / token validation
    ↓
Service
    ↓
Authorization
    ↓
Business operation
```

Important:

> **Don't rely only on the frontend or gateway for authorization of sensitive operations. Each service should enforce the authorization required for its own protected resources.**

---

# 36. Frontend Authorization vs Backend Authorization ⭐⭐⭐⭐⭐

Frontend:

```text
Hide Delete button
```

This improves UX but is not security.

Backend:

```text
DELETE /users/10
     ↓
Authorization check
     ↓
ROLE_ADMIN?
     ↓
ALLOW / DENY
```

### Golden rule

> **Frontend authorization is a UI concern; backend authorization is the security boundary.**

---

# 37. Authorization Testing ⭐⭐⭐⭐⭐

Test at least:

```text
Anonymous user
Authenticated normal user
Manager
Admin
User with individual permission
User without permission
Expired/invalid token
```

Example test scenarios:

```text
GET /orders/1 + ORDER_READ → 200
GET /orders/1 without ORDER_READ → 403
GET /admin/users as USER → 403
GET /admin/users without authentication → authentication challenge / configured failure
```

---

# 38. Common Authorization Mistakes ⭐⭐⭐⭐⭐

### Mistake 1
Only hiding UI buttons.

❌ Not security.

### Mistake 2
Using `hasRole("ADMIN")` while storing only `ADMIN` as a direct authority without matching prefix configuration.

❌ May fail because role prefix conventions matter.

### Mistake 3
Assuming JWT `roles` automatically become authorities.

❌ Not necessarily.

### Mistake 4
Protecting gateway but not individual services.

❌ Dangerous for sensitive operations.

### Mistake 5
Using one broad role for everything.

❌ Poor least-privilege design.

### Mistake 6
Putting all authorization logic into controllers.

❌ Can become duplicated and difficult to maintain.

### Mistake 7
Confusing 401 and 403.

❌ Authentication and authorization failures are different.

---

# 39. Production Best Practices ⭐⭐⭐⭐⭐

- Follow least privilege.
- Prefer explicit authorization rules.
- Keep public endpoints intentionally small.
- Use method security for business-sensitive operations.
- Validate authorization server-side.
- Map JWT/OAuth claims explicitly to authorities.
- Test negative authorization paths.
- Avoid overly broad `permitAll()` patterns.
- Keep authorization policy centralized/reusable where practical.
- Protect every microservice's sensitive business operations.
- Keep role/permission naming consistent.
- Audit important authorization failures where appropriate.
- Avoid exposing sensitive internal authorization details in API errors.

---

# 40. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is authorization?

> Authorization determines whether an authenticated principal has permission to access a resource or perform an operation.

### Q2. Authentication vs authorization?

> Authentication verifies identity; authorization decides access based on roles, authorities, or policies.

### Q3. What is AuthorizationManager?

> It is the modern Spring Security abstraction used to make authorization decisions based on the current authentication and protected context/resource.

### Q4. What is the difference between `hasRole()` and `hasAuthority()`?

> `hasRole()` uses the role-prefix convention, commonly checking `ROLE_ADMIN` for `hasRole("ADMIN")`; `hasAuthority()` checks the supplied authority value directly.

### Q5. Difference between 401 and 403?

> 401 generally indicates authentication is missing or invalid; 403 generally indicates the user is authenticated but not permitted to access the resource.

### Q6. What is AccessDeniedHandler?

> It handles authorization failures for an authenticated principal who is denied access.

### Q7. What is AuthenticationEntryPoint?

> It handles cases where authentication is required but the request has not established valid authentication.

### Q8. Why use method security?

> It allows authorization to be enforced at the business-operation level rather than relying only on URL patterns.

### Q9. What is RBAC?

> Role-Based Access Control assigns users roles, and roles provide permissions/authorities.

### Q10. Can authorization depend on resource ownership?

> Yes. For example, a user may have `ORDER_READ` but be allowed to access only orders they own. This requires a business/resource-level authorization policy.

### Q11. Does JWT authentication automatically authorize the request?

> No. JWT validation establishes the authentication context; authorization still evaluates the resulting authorities and policies.

### Q12. Why should every microservice enforce authorization?

> Because the service owning the business resource should not rely solely on a gateway or frontend to protect sensitive operations.

---

# 41. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“Authorization in Spring Security decides whether an authenticated principal can access a resource or perform an operation. After authentication, the Authentication object contains the principal and GrantedAuthorities. Spring Security evaluates these authorities against authorization rules, such as `hasRole`, `hasAuthority`, `authenticated`, or `permitAll`. In modern Spring Security, AuthorizationManager is the key authorization abstraction. Authorization can happen at request level through SecurityFilterChain and at method level using `@PreAuthorize` or related annotations. For an unauthenticated request, authentication-entry-point handling is used; for an authenticated user without sufficient authority, access-denied handling is used. In JWT/OAuth2 systems, token claims must be mapped to GrantedAuthority values before authorization rules can evaluate them. For production systems, we should follow least privilege, enforce authorization on the backend and inside services, and use resource ownership/business policies when role checks alone are insufficient.”**

---

# 42. 30-Second Hinglish Answer

> **“Authorization ka matlab hai authenticated user ko kya access allowed hai. Authentication ke baad Authentication object mein GrantedAuthorities hoti hain, aur Spring Security unko authorization rules ke against check karta hai. `hasRole()`, `hasAuthority()`, `authenticated()` aur `permitAll()` common checks hain. Modern Spring Security mein AuthorizationManager important abstraction hai. Unauthenticated request authentication handling mein jaati hai, jabki authenticated but unauthorized request AccessDenied handling mein jaati hai. JWT mein claims ko pehle GrantedAuthority mein map karna padta hai. Production mein backend-side least-privilege authorization mandatory hai.”**

---

# 43. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                    HTTP REQUEST
                         ↓
                Security Filter Chain
                         ↓
                   Authentication
                         ↓
              Principal + Authorities
                         ↓
                AuthorizationManager
                         ↓
               Authorization Rule
                         ↓
                AuthorizationDecision
                    ↙          ↘
               GRANTED        DENIED
                  ↓              ↓
             Controller      AccessDenied
```

### Final memory line

> **Authentication establishes identity → Authorities describe access → AuthorizationManager evaluates policy → Decision allows or denies the request.**

---

## Navigation

[← S4.1.14 — GrantedAuthority and Roles](../14-GrantedAuthority-and-Roles/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.16 — Exception Handling in Spring Security**
