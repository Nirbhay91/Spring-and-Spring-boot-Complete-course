# S4.1.7 — Principal, Credentials and Authorities

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture ⭐⭐⭐⭐⭐

Spring Security mein authenticated request ko samajhne ke liye `Authentication` object ke 3 important concepts hain:

```text
Authentication
 ├── Principal      → WHO is the caller?
 ├── Credentials    → Proof used to authenticate
 └── Authorities    → WHAT is the caller allowed to do?
```

### Memory trick

```text
Principal    = Identity
Credentials  = Proof
Authorities  = Permissions
```

> Important: Authentication successful hone ke baad credentials ko generally expose/store karna avoid kiya jata hai; Spring Security implementations may erase sensitive credentials after authentication.

---

# 2. Principal

**Principal** authenticated caller ki identity ko represent karta hai.

Simple question:

> **Who is this user/client?**

Example:

```text
User login:
nirbhay / password

After authentication:
Principal = nirbhay
```

Spring Security mein principal ka concrete type authentication mechanism par depend kar sakta hai.

Example:

```java
Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();

Object principal = authentication.getPrincipal();
```

`principal` ko blindly `String` assume nahi karna chahiye; application/configuration ke according ye `UserDetails`, JWT-related principal, custom principal, etc. ho sakta hai.

---

# 3. Credentials

**Credentials** identity prove karne ke liye use hone wali information hoti hai.

Examples:

```text
Username + Password
JWT / Bearer Token
Client Secret
Certificate
```

Conceptually:

```text
Principal    → Nirbhay
Credentials  → Password / Token
```

### Important security point ⭐⭐⭐⭐⭐

Credentials are **sensitive data**.

Never:

```text
❌ Log raw password
❌ Return password in API response
❌ Store plaintext password
❌ Put secrets unnecessarily in logs
```

Password authentication mein password ko secure hash ke form mein store karna chahiye, normally a suitable `PasswordEncoder` ke through.

---

# 4. Authorities

**Authorities** authenticated principal ke granted permissions ko represent karti hain.

Question:

> **What is this caller allowed to do?**

Examples:

```text
ROLE_ADMIN
ROLE_USER
ORDER_READ
ORDER_WRITE
PAYMENT_REFUND
```

Example:

```text
Principal = nirbhay
Authorities =
    ROLE_USER
    ORDER_READ
    ORDER_WRITE
```

Authorization rules in authorities ko evaluate kar sakti hain.

---

# 5. Complete Authentication Object ⭐⭐⭐⭐⭐

Conceptually:

```text
Authentication
│
├── Principal
│     └── Who?
│
├── Credentials
│     └── Proof?
│
├── Authorities
│     └── Permissions?
│
└── Authenticated state
```

Example:

```text
Authentication
 ├── Principal    = nirbhay
 ├── Credentials  = <sensitive / normally not exposed>
 └── Authorities  = [ROLE_USER, ORDER_READ]
```

---

# 6. `Authentication` Interface

Spring Security ka `Authentication` interface security identity ko represent karta hai.

Commonly useful methods:

```java
Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();

authentication.getPrincipal();
authentication.getCredentials();
authentication.getAuthorities();
authentication.isAuthenticated();
```

### Interview point

> `Authentication` is both an authentication request/result representation and, after successful authentication, the representation of the authenticated principal together with its authorities.

Exact lifecycle depends on the authentication mechanism.

---

# 7. Principal vs Credentials vs Authorities ⭐⭐⭐⭐⭐

| Concept | Meaning | Example |
|---|---|---|
| Principal | Identity | `nirbhay` |
| Credentials | Proof of identity | Password / token |
| Authorities | Granted permissions | `ROLE_ADMIN`, `ORDER_READ` |

### One-line answer

> **Principal tells us who the caller is, credentials prove that identity, and authorities tell us what that caller is allowed to access.**

---

# 8. `UserDetails` as Principal

Username/password authentication often uses `UserDetails` to represent application user information used by Spring Security.

Example:

```java
public class CustomUserDetails implements UserDetails {

    private final String username;
    private final String password;

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(
            new SimpleGrantedAuthority("ROLE_USER")
        );
    }

    // other UserDetails methods...
}
```

Then conceptually:

```text
UserDetails
     ↓
Authentication
     ↓
SecurityContext
```

`UserDetails` is one possible principal representation; it is not the only possible principal type.

---

# 9. `GrantedAuthority`

Spring Security permissions are represented through `GrantedAuthority`.

Example:

```java
GrantedAuthority authority =
        new SimpleGrantedAuthority("ORDER_READ");
```

Multiple authorities:

```java
List<GrantedAuthority> authorities = List.of(
    new SimpleGrantedAuthority("ROLE_USER"),
    new SimpleGrantedAuthority("ORDER_READ"),
    new SimpleGrantedAuthority("ORDER_WRITE")
);
```

Authorization mechanisms can use these authorities to make access decisions.

---

# 10. Role vs Authority ⭐⭐⭐⭐⭐

Role is commonly represented as an authority with the `ROLE_` prefix.

```java
.hasRole("ADMIN")
```

conventionally checks:

```text
ROLE_ADMIN
```

While:

```java
.hasAuthority("ORDER_READ")
```

checks the authority value directly.

### Memory

```text
Role      → ROLE_ADMIN
Authority → ORDER_READ
```

Don't confuse role and authority as completely different underlying concepts; roles are commonly expressed using authorities with a naming convention.

---

# 11. Principal, Credentials & Authorities in Username/Password Flow ⭐⭐⭐⭐⭐

```text
Login Request
     ↓
username + password
     ↓
Authentication request
     ↓
AuthenticationProvider
     ↓
UserDetailsService
     ↓
PasswordEncoder
     ↓
Successful Authentication
     ↓
Authentication
 ├── Principal = UserDetails
 ├── Credentials = sensitive / may be erased
 └── Authorities = granted roles/permissions
     ↓
SecurityContext
```

---

# 12. JWT Flow

For JWT resource-server authentication:

```text
Authorization: Bearer <JWT>
             ↓
        Token validation
             ↓
      Authentication created
             ↓
Principal + Authorities
             ↓
       SecurityContext
             ↓
        Authorization
```

JWT claims may be converted into authorities depending on the resource-server configuration.

Example conceptual token:

```json
{
  "sub": "nirbhay",
  "scope": "orders.read orders.write"
}
```

Conceptually:

```text
sub
 ↓
Principal / identity

scope
 ↓
Authorities
```

The exact mapping is configuration-dependent.

---

# 13. Getting the Current Principal

Example:

```java
Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();

Object principal = authentication.getPrincipal();
```

If the principal is known to implement `UserDetails`:

```java
UserDetails user = (UserDetails) authentication.getPrincipal();
String username = user.getUsername();
```

### Better controller approach

Spring MVC can inject the authenticated principal:

```java
@GetMapping("/profile")
public String profile(Principal principal) {
    return principal.getName();
}
```

Or use Spring Security-specific abstractions when their additional information is needed.

---

# 14. `Principal.getName()`

Java's `Principal` abstraction provides:

```java
principal.getName();
```

This gives the principal's name as defined by the authentication mechanism.

Important:

> `Principal` is a general Java security identity abstraction; Spring Security's `Authentication` provides richer security information such as authorities.

---

# 15. Credentials Are Not the Same as Principal

Common mistake:

```text
Principal = password
```

❌ Wrong.

Correct:

```text
Principal    = identity
Credentials  = proof
```

Example:

```text
Principal    = nirbhay
Credentials  = password/token
```

---

# 16. Authorities Are Not Credentials

Another common mistake:

```text
ROLE_ADMIN = credential
```

❌ No.

`ROLE_ADMIN` is an authority used for authorization.

```text
Password / token → credential
ROLE_ADMIN        → authority
```

---

# 17. SecurityContext Relationship ⭐⭐⭐⭐⭐

The authenticated `Authentication` is associated with the `SecurityContext`.

```text
SecurityContext
      │
      └── Authentication
             ├── Principal
             ├── Credentials
             └── Authorities
```

Access example:

```java
SecurityContext context =
        SecurityContextHolder.getContext();

Authentication authentication =
        context.getAuthentication();
```

This is why downstream application/security code can access the current authenticated identity.

---

# 18. Anonymous User

For unauthenticated requests, Spring Security can represent an anonymous user depending on configuration.

Do not assume that:

```java
getAuthentication() == null
```

is always the way an unauthenticated request is represented.

Spring Security may use an anonymous authentication mechanism.

Interview point:

> **Unauthenticated and anonymous are related but should not be treated as identical implementation concepts.**

---

# 19. SecurityContextHolder ⭐⭐⭐⭐⭐

`SecurityContextHolder` provides access to the current `SecurityContext`.

```java
Authentication auth =
    SecurityContextHolder.getContext().getAuthentication();
```

Conceptually:

```text
Current execution context
        ↓
SecurityContextHolder
        ↓
SecurityContext
        ↓
Authentication
        ↓
Principal + Authorities
```

Its context propagation strategy matters in asynchronous/concurrent execution; don't assume security context automatically behaves identically across every thread boundary.

---

# 20. Credentials Erasure ⭐⭐⭐⭐⭐

After successful authentication, sensitive credentials may be erased to reduce the chance of accidental exposure.

Conceptually:

```text
Before authentication completes
credentials available
        ↓
Authentication successful
        ↓
credentials may be erased
        ↓
principal + authorities remain useful
```

This is one reason you should not design application code around retrieving raw passwords from the `Authentication` object.

---

# 21. Why Authorities Matter for Authorization

Suppose:

```text
Principal = nirbhay
Authorities =
    ROLE_USER
    ORDER_READ
```

Request:

```http
GET /orders
```

Required:

```text
ORDER_READ
```

Result:

```text
Allowed
```

But:

```http
DELETE /orders/10
```

Required:

```text
ORDER_DELETE
```

If authority is missing:

```text
Authorization denied
```

Identity is still valid.

---

# 22. Principal vs User vs UserDetails

These terms are often mixed up in interviews.

### Principal

General concept:

```text
Authenticated identity
```

### Application User

Your domain object:

```java
User
```

May contain:

```text
id
name
email
status
roles
```

### UserDetails

Spring Security abstraction used by username/password authentication to expose security-related user information.

They can be related but are not automatically the same object.

```text
Domain User
    ↓
Custom UserDetails
    ↓
Authentication principal
```

---

# 23. Production Example

Suppose an e-commerce application:

```text
Principal:
nirbhay

Authorities:
ROLE_USER
ORDER_READ
ORDER_CREATE
```

Requests:

```text
GET /orders
   → ORDER_READ → ALLOW

POST /orders
   → ORDER_CREATE → ALLOW

DELETE /users/42
   → ROLE_ADMIN required → DENY
```

Authentication established the identity once; authorization evaluates the permissions for protected operations.

---

# 24. Microservices Example ⭐⭐⭐⭐⭐

```text
Client
  ↓
JWT
  ↓
Order Service
  ↓
Token validation
  ↓
Authentication
  ├── Principal = nirbhay
  └── Authorities = orders.read
  ↓
Authorization
  ↓
GET /orders → ALLOW
```

For another service:

```text
Payment Service
  ↓
Required authority = payment.write
  ↓
Token/user lacks authority
  ↓
DENY
```

The exact trust model and token validation strategy should be designed per service boundary.

---

# 25. Common Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is Principal?

> Principal represents the identity of the authenticated caller.

### Q2. What are credentials?

> Credentials are proof used to establish or verify identity, such as a password or token.

### Q3. What are authorities?

> Authorities are permissions/grants associated with the authenticated identity and used during authorization.

### Q4. What is the relationship between Authentication and Principal?

> `Authentication` represents the security identity and authentication state; its `getPrincipal()` exposes the authenticated identity.

### Q5. Is UserDetails the same as Principal?

> `UserDetails` can be used as a principal representation in username/password authentication, but principal can have other types depending on the authentication mechanism.

### Q6. What is GrantedAuthority?

> It is Spring Security's abstraction representing a granted authority used in authorization decisions.

### Q7. Why should credentials not be logged?

> Because passwords, tokens and secrets are sensitive and logging them can cause credential leakage.

### Q8. What is the difference between Role and Authority?

> A role is commonly represented as an authority with the `ROLE_` prefix; authorities can also represent fine-grained permissions such as `ORDER_READ`.

### Q9. How do you get the current authenticated user?

```java
Authentication auth =
    SecurityContextHolder.getContext().getAuthentication();
Object principal = auth.getPrincipal();
```

### Q10. Can the principal be a JWT?

> The principal representation depends on the authentication mechanism and configuration. In a JWT resource server, Spring Security creates an authenticated representation from the validated token; it should not be assumed to be a simple username string.

---

# 26. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“In Spring Security, Principal, Credentials and Authorities are three important parts of the authentication model. Principal represents who the caller is, for example a username or a security-specific user object. Credentials are the proof used to authenticate that identity, such as a password or bearer token, and they are sensitive and should not be logged or exposed. Authorities represent what the authenticated principal is allowed to do, such as ROLE_ADMIN, ORDER_READ or ORDER_WRITE. These concepts are available through the Authentication object, which is associated with the SecurityContext after successful authentication. Authorization uses the authorities to make access decisions. So my simple memory is: Principal equals identity, Credentials equal proof, and Authorities equal permissions.”**

---

# 27. 30-Second Hinglish Answer

> **“Principal batata hai WHO user hai, Credentials identity ka proof hota hai jaise password ya token, aur Authorities batati hain user WHAT kar sakta hai. Spring Security mein ye information Authentication object ke through represent hoti hai aur successful authentication ke baad SecurityContext se associated hoti hai. Simple memory: Principal = Identity, Credentials = Proof, Authorities = Permissions.”**

---

# 28. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                 Authentication
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Principal     Credentials  Authorities
          │            │            │
        WHO?         PROOF?       WHAT?
          │            │            │
       nirbhay      JWT/pass      ORDER_READ
                                   ROLE_ADMIN
                       │
                       ↓
                Authorization
                       │
                 ALLOW / DENY
```

### Final memory line

> **Principal = WHO, Credentials = PROOF, Authorities = WHAT.**

---

## Navigation

[← S4.1.6 — Authentication vs Authorization](../06-Authentication-vs-Authorization/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.8 — SecurityContext and SecurityContextHolder**
