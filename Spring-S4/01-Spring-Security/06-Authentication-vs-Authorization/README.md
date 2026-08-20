# S4.1.6 — Authentication vs Authorization

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Authentication vs Authorization ⭐⭐⭐⭐⭐

Spring Security interview ka sabse basic aur important distinction:

> **Authentication = Who are you?**  
> **Authorization = What are you allowed to do?**

Example:

```text
User logs in
   ↓
Authentication
   ↓
"This is Nirbhay"
   ↓
Authorization
   ↓
"Nirbhay can access /admin/reports"
```

---

# 2. Authentication

Authentication user/client ki identity verify karta hai.

Common credentials:

```text
Username + Password
JWT / Bearer Token
OAuth2/OIDC identity
API Key
Certificate
```

Question:

> **Who is this caller?**

Example:

```text
username = nirbhay
password = ********
        ↓
Credentials validated
        ↓
Authenticated identity
```

Authentication successful hone ke baad Spring Security mein generally an `Authentication` object represent karta hai ki caller authenticated hai aur uski authorities kya hain.

---

# 3. Authorization

Authorization decide karta hai ki authenticated caller ko requested resource/action perform karne ki permission hai ya nahi.

Question:

> **What can this caller access?**

Example:

```text
User = Nirbhay
Role = ADMIN

GET /admin/users
       ↓
Authorization decision
       ↓
Allowed
```

Agar user authenticated hai but required authority nahi hai:

```text
Authenticated
      ↓
Authorization
      ↓
Denied
```

Typically this results in **403 Forbidden** for an authenticated caller who is not permitted, depending on configuration.

---

# 4. Complete Flow ⭐⭐⭐⭐⭐

```text
HTTP Request
      ↓
Security Filter Chain
      ↓
Authentication
      ↓
Identity verified?
      ├── No → Authentication failure
      │
      └── Yes
            ↓
       SecurityContext
            ↓
       Authorization
            ↓
     Allowed / Denied
            ↓
       Controller
```

### Memory

```text
Authentication → WHO?
Authorization  → WHAT?
```

---

# 5. Real-World Example

Suppose banking application:

```text
Login
 ↓
Username/password or identity token
 ↓
Authentication
 ↓
User = Nirbhay
 ↓
Authorization
 ├── View account       → YES
 ├── Transfer money     → YES
 ├── Manage users       → NO
 └── Delete bank config → NO
```

Authentication tells the system **who the user is**.

Authorization tells the system **which operations that identity is allowed to perform**.

---

# 6. Authentication in Spring Security ⭐⭐⭐⭐⭐

Conceptual architecture:

```text
Request
   ↓
Authentication mechanism
   ↓
AuthenticationManager
   ↓
AuthenticationProvider
   ↓
UserDetailsService / token validation / other provider
   ↓
Authentication
   ↓
SecurityContext
```

Not every authentication mechanism uses exactly the same provider flow; JWT resource-server authentication, username/password authentication and OAuth2 login have different implementation paths.

### Important interview point

> **AuthenticationManager is an abstraction for authentication; AuthenticationProvider implementations perform specific authentication mechanisms.**

---

# 7. Username + Password Example

```text
Login Request
      ↓
UsernamePasswordAuthenticationToken
      ↓
AuthenticationManager
      ↓
AuthenticationProvider
      ↓
UserDetailsService
      ↓
PasswordEncoder
      ↓
Authentication successful
      ↓
SecurityContext
```

Password ko plain text mein compare/store nahi karna chahiye.

Typically:

```text
Raw password
     ↓
PasswordEncoder.matches(...)
     ↓
Stored password hash
```

---

# 8. JWT Authentication Example ⭐⭐⭐⭐⭐

For a JWT resource server:

```http
GET /api/orders
Authorization: Bearer <JWT>
```

Conceptual flow:

```text
Bearer Token
     ↓
JWT validation
     ↓
Claims / token authorities
     ↓
Authenticated principal
     ↓
SecurityContext
     ↓
Authorization
```

Important:

> JWT itself is a token format; **authentication** is the process of validating the token and establishing the caller identity.

---

# 9. Authorization Types

Authorization can be applied at different levels.

### URL/request level

```java
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/public/**").permitAll()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
);
```

### Method level

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) {
    // ...
}
```

Method-level security must be enabled/configured appropriately.

---

# 10. Role vs Authority ⭐⭐⭐⭐⭐

Authorization commonly uses authorities.

Example:

```text
ROLE_ADMIN
ORDER_READ
ORDER_WRITE
```

### Role

Spring Security's role-based expressions conventionally use the `ROLE_` prefix.

```java
.hasRole("ADMIN")
```

Conceptually corresponds to:

```text
ROLE_ADMIN
```

### Authority

Can be more granular:

```java
.hasAuthority("ORDER_WRITE")
```

Memory:

```text
Role      → broad permission group
Authority → specific permission/grant
```

---

# 11. Authentication vs Authorization Table ⭐⭐⭐⭐⭐

| Point | Authentication | Authorization |
|---|---|---|
| Main question | Who are you? | What can you do? |
| Purpose | Verify identity | Decide access |
| Happens | Before access decision | After identity is established / during access decision |
| Example | Password/JWT validation | Role/authority check |
| Result | Authenticated identity | Allow / deny |
| Spring concept | `Authentication` | Authorization decision/rules |
| Failure commonly | 401 | 403 |

### Important nuance

`401` and `403` are conventional HTTP outcomes, not a universal rule for every custom application/configuration.

---

# 12. 401 vs 403 ⭐⭐⭐⭐⭐

### 401 Unauthorized

Commonly means:

```text
Authentication is required or failed
```

Example:

```text
No valid credentials
      ↓
Authentication failure
      ↓
401
```

### 403 Forbidden

Commonly means:

```text
Caller is authenticated
but not permitted
```

Example:

```text
User = USER
Required = ADMIN
      ↓
Authorization denied
      ↓
403
```

### Interview memory

```text
401 → Who are you? → Can't establish valid authentication
403 → I know you, but you can't do this.
```

---

# 13. Principal ⭐⭐⭐⭐⭐

After successful authentication, Spring Security has an authenticated principal representing the caller.

Conceptually:

```text
Authentication
 ├── Principal
 ├── Credentials
 └── Authorities
```

For a user-based authentication, the principal may contain identity information such as username.

Example:

```java
Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();

Object principal = authentication.getPrincipal();
```

---

# 14. SecurityContext

Successful authentication is associated with the current security context.

Conceptually:

```text
Authentication
      ↓
SecurityContext
      ↓
SecurityContextHolder
```

Later application/security components can access the authenticated identity according to the current execution context and configured strategy.

Example:

```java
Authentication auth = SecurityContextHolder
        .getContext()
        .getAuthentication();
```

---

# 15. Authorization Does Not Mean Authentication Again

A common misunderstanding:

```text
Authorization
   ↓
Check password again
```

❌ Not normally.

Authentication establishes identity.

Authorization evaluates permissions associated with that identity.

```text
Authentication
   ↓
Identity + authorities
   ↓
Authorization
   ↓
Access decision
```

---

# 16. Authentication Is Not Only Login ⭐⭐⭐⭐⭐

Another important interview point:

> Authentication is broader than username/password login.

Examples:

```text
JWT
OAuth2/OIDC
HTTP Basic
Mutual TLS
API keys
Session-based authentication
```

Login form is only one authentication mechanism.

---

# 17. Authorization Is Not Only Roles

Authorization can use:

```text
Roles
Authorities
Scopes
Resource ownership
Request attributes
Business rules
Method-level expressions
```

Example:

```text
USER can update only own profile
```

This is more than a simple role check; it may require resource/business context.

---

# 18. Authentication vs Authorization in Microservices ⭐⭐⭐⭐⭐

Suppose:

```text
Client
  ↓
API Gateway
  ↓
Order Service
  ↓
Payment Service
```

Possible design:

```text
Authentication
   ↓
Validate caller identity/token

Authorization
   ↓
Can this identity perform this operation?
```

Important production principle:

> Do not automatically assume that authentication at the gateway removes the need for downstream authorization/security controls.

Each service's trust boundary and threat model should determine what it validates and authorizes.

---

# 19. Authentication vs Authorization in JWT Microservices

Example token claims:

```json
{
  "sub": "nirbhay",
  "scope": "orders.read orders.write"
}
```

Conceptually:

```text
JWT validation
      ↓
Authentication
      ↓
Principal = nirbhay
      ↓
Authorities/scopes
      ↓
Authorization
      ↓
orders.read → allowed
orders.delete → denied
```

The token's claims are only trusted after the token has been correctly validated according to the resource server's configuration.

---

# 20. Authentication and Authorization Order ⭐⭐⭐⭐⭐

Normally think:

```text
1. Establish identity
       ↓
2. Establish authenticated security context
       ↓
3. Evaluate access permissions
       ↓
4. Allow or deny
```

However, Spring Security has different mechanisms and filters, so don't describe the internals as one universal fixed sequence for every feature.

---

# 21. Common Interview Trap — “Authorization Before Authentication?”

If interviewer asks:

> Can authorization happen without authentication?

Best answer:

> **Authorization decisions can include public/anonymous access rules such as `permitAll`, but for protected resources the system generally needs an established identity before it can evaluate identity-based permissions.**

Example:

```java
.requestMatchers("/public/**").permitAll()
```

Here authenticated identity is not required for the authorization rule to allow access.

---

# 22. Common Interview Trap — `hasRole` vs `hasAuthority`

```java
.hasRole("ADMIN")
```

typically checks:

```text
ROLE_ADMIN
```

while:

```java
.hasAuthority("ADMIN")
```

checks the authority named exactly:

```text
ADMIN
```

Do not casually interchange them.

---

# 23. Common Interview Trap — “401 means no permission”

Not exactly.

Use the mental model:

```text
401 → authentication problem
403 → authenticated but forbidden
```

For exact behavior, always consider the application's Spring Security configuration and exception handling.

---

# 24. Common Interview Trap — JWT = Authorization

JWT can carry claims/scopes/roles used in authorization, but JWT itself is not the authorization process.

Correct model:

```text
JWT validation
     ↓
Authentication
     ↓
Authorities/scopes
     ↓
Authorization decision
```

---

# 25. Production Example: Admin API ⭐⭐⭐⭐⭐

Endpoint:

```http
DELETE /api/users/10
```

Requirement:

```text
Only ADMIN can delete users
```

Flow:

```text
Request
  ↓
JWT validation
  ↓
Authentication
  ↓
Principal = nirbhay
  ↓
Authorities = [ROLE_USER]
  ↓
Authorization check
  ↓
Required = ROLE_ADMIN
  ↓
Denied
  ↓
403 Forbidden
```

The user's identity was valid; the permission was insufficient.

---

# 26. Production Example: Missing Token

```http
GET /api/orders
```

Requirement:

```text
Authenticated user required
```

Flow:

```text
Request
  ↓
No valid authentication
  ↓
Authentication required
  ↓
AuthenticationEntryPoint
  ↓
401
```

Again, exact response behavior is configurable.

---

# 27. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“Authentication and authorization are two different security concerns. Authentication answers ‘Who are you?’ and verifies the identity of the caller, for example by validating username/password, a JWT, or another credential. After successful authentication, Spring Security represents the authenticated identity through an Authentication and security context. Authorization answers ‘What are you allowed to do?’ and evaluates roles, authorities, scopes or other access rules against the authenticated identity and requested resource. For example, a user may successfully authenticate as Nirbhay, but if the endpoint requires ADMIN and the user only has USER authority, authentication succeeds while authorization fails, commonly resulting in 403. If the caller cannot be authenticated for a protected endpoint, the common result is 401. In short: Authentication establishes identity; authorization makes the access decision.”**

---

# 28. 30-Second Hinglish Answer

> **“Authentication ka matlab hai user ki identity verify karna — ‘Who are you?’. Jaise username/password ya JWT validate karna. Successful authentication ke baad identity SecurityContext mein available hoti hai. Authorization ka matlab hai ‘What are you allowed to do?’. Jaise user authenticated hai but uske paas ADMIN authority nahi hai, to `/admin/**` access deny hoga. Simple memory: Authentication = WHO, Authorization = WHAT.”**

---

# 29. Whiteboard Memory Map ⭐⭐⭐⭐⭐

```text
                 REQUEST
                    │
                    ↓
              Authentication
              "WHO are you?"
                    │
             ┌──────┴──────┐
             ↓             ↓
          Failure        Success
             ↓             ↓
            401       Authentication
                          │
                          ↓
                   SecurityContext
                          │
                          ↓
                    Authorization
                  "WHAT can you do?"
                          │
                   ┌──────┴──────┐
                   ↓             ↓
                Allowed        Denied
                   ↓             ↓
              Controller        403
```

### One-line memory

> **Authentication proves identity; Authorization decides permission.**

---

# 30. Interview Follow-up Questions

1. What is authentication?
2. What is authorization?
3. Difference between authentication and authorization?
4. Why does authentication normally happen before protected-resource authorization?
5. What is the `Authentication` object?
6. What is a principal?
7. What is the SecurityContext?
8. Difference between 401 and 403?
9. Difference between role and authority?
10. What does `hasRole("ADMIN")` mean?
11. What does `hasAuthority("ORDER_READ")` mean?
12. Can `permitAll()` work without authentication?
13. Is JWT authentication or authorization?
14. How does JWT authentication work in a resource server?
15. Where does AuthenticationManager fit?
16. Where does AuthenticationProvider fit?
17. How does authorization work at URL level?
18. How does method-level authorization work?
19. How do authentication and authorization work in microservices?
20. Explain authentication vs authorization in 2 minutes on a whiteboard.

---

## Navigation

[← S4.1.5 — DelegatingFilterProxy](../05-DelegatingFilterProxy/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.7 — Principal, Credentials and Authorities**
