# S4.1.14 — GrantedAuthority and Roles

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture ⭐⭐⭐⭐⭐

`GrantedAuthority` Spring Security mein authenticated user ke **permissions/authorities** represent karta hai. Roles bhi ultimately authorities ke form mein authorization mein use hote hain.

```text
UserDetails
    ↓
getAuthorities()
    ↓
GrantedAuthority
    ↓
Authorization decision
    ↓
ALLOW / DENY
```

### Memory trick

```text
Authentication → Tum kaun ho?
Authorization  → Tum kya kar sakte ho?

GrantedAuthority → Tumhe kya permission mili hai?
Role             → Authorities ko group karne ka common convention
```

---

# 2. What is GrantedAuthority?

`GrantedAuthority` ek Spring Security interface hai jo kisi authenticated principal ko granted authority represent karta hai.

Conceptually:

```java
public interface GrantedAuthority extends Serializable {
    String getAuthority();
}
```

Example:

```text
ROLE_USER
ORDER_READ
ORDER_WRITE
```

A user ke authorities:

```java
Collection<? extends GrantedAuthority> authorities;
```

---

# 3. Where Does GrantedAuthority Come From? ⭐⭐⭐⭐⭐

Common flow:

```text
Database / LDAP / External Identity Provider
                    ↓
             UserDetailsService
                    ↓
                UserDetails
                    ↓
             getAuthorities()
                    ↓
          GrantedAuthority objects
                    ↓
              Authentication
                    ↓
              Authorization
```

`UserDetails.getAuthorities()` security system ko user's authorities provide karta hai.

---

# 4. Role vs Authority ⭐⭐⭐⭐⭐

Spring Security mein role aur authority related concepts hain, but identical terms nahi hain.

Example:

```text
Role:
USER

Authority:
ROLE_USER
```

By convention, roles commonly `ROLE_` prefix ke saath represented authorities se map hote hain.

```java
.roles("USER")
```

commonly results in:

```text
ROLE_USER
```

Whereas:

```java
.authorities("ORDER_READ")
```

creates an authority with exactly:

```text
ORDER_READ
```

---

# 5. `GrantedAuthority` Example ⭐⭐⭐⭐⭐

```java
GrantedAuthority authority =
        new SimpleGrantedAuthority("ORDER_READ");
```

Multiple authorities:

```java
List<GrantedAuthority> authorities = List.of(
    new SimpleGrantedAuthority("ORDER_READ"),
    new SimpleGrantedAuthority("ORDER_WRITE")
);
```

User:

```java
UserDetails user = User.withUsername("nirbhay")
        .password(passwordEncoder.encode("secret"))
        .authorities(authorities)
        .build();
```

---

# 6. SimpleGrantedAuthority ⭐⭐⭐⭐⭐

`SimpleGrantedAuthority` ek common implementation hai.

```java
new SimpleGrantedAuthority("ORDER_READ");
```

Its value is returned through:

```java
getAuthority()
```

Example:

```java
GrantedAuthority authority =
    new SimpleGrantedAuthority("ORDER_READ");

System.out.println(authority.getAuthority());
```

Output:

```text
ORDER_READ
```

---

# 7. Roles with `User.withUsername()` ⭐⭐⭐⭐⭐

Example:

```java
UserDetails user = User.withUsername("nirbhay")
        .password(passwordEncoder.encode("secret"))
        .roles("USER")
        .build();
```

Conceptually:

```text
roles("USER")
      ↓
ROLE_USER
      ↓
GrantedAuthority
```

Multiple roles:

```java
.roles("USER", "ADMIN")
```

Conceptually:

```text
ROLE_USER
ROLE_ADMIN
```

---

# 8. Authorities with `User.withUsername()` ⭐⭐⭐⭐⭐

Exact authority values specify kar sakte ho:

```java
UserDetails user = User.withUsername("nirbhay")
        .password(passwordEncoder.encode("secret"))
        .authorities("ORDER_READ", "ORDER_WRITE")
        .build();
```

Result:

```text
ORDER_READ
ORDER_WRITE
```

Notice:

```java
.authorities("USER")
```

does **not** automatically mean:

```text
ROLE_USER
```

Whereas:

```java
.roles("USER")
```

uses the role convention.

---

# 9. `hasRole()` vs `hasAuthority()` ⭐⭐⭐⭐⭐

### `hasRole()`

```java
.requestMatchers("/admin/**")
.hasRole("ADMIN")
```

Conceptually Spring Security checks the role using its configured role prefix convention, commonly:

```text
hasRole("ADMIN")
       ↓
ROLE_ADMIN
```

### `hasAuthority()`

```java
.requestMatchers("/orders/**")
.hasAuthority("ORDER_READ")
```

This checks the authority value directly.

### Memory

```text
hasRole("ADMIN")
      ↓
ROLE_ADMIN

hasAuthority("ORDER_READ")
      ↓
ORDER_READ
```

---

# 10. Why `ROLE_` Prefix? ⭐⭐⭐⭐⭐

Spring Security historically uses a role prefix convention to distinguish role-style authorities.

```text
Role name: ADMIN
       ↓
Authority: ROLE_ADMIN
```

This is why:

```java
hasRole("ADMIN")
```

and:

```java
hasAuthority("ROLE_ADMIN")
```

are commonly equivalent in the standard role-prefix configuration.

However, prefix handling can be customized in modern Spring Security, so don't treat `ROLE_` as an unchangeable universal rule.

---

# 11. Role Hierarchy ⭐⭐⭐⭐

Sometimes one role should inherit permissions of another.

Example:

```text
ADMIN
  ↓
MANAGER
  ↓
USER
```

With role hierarchy:

```text
ADMIN → MANAGER → USER
```

An ADMIN can therefore inherit permissions associated with lower roles depending on the configured hierarchy.

This avoids manually assigning every lower-level role/authority to higher roles.

---

# 12. Roles vs Permissions ⭐⭐⭐⭐⭐

Production systems often distinguish:

```text
Role
 ↓
Group of permissions
```

Example:

```text
ADMIN
 ├── USER_READ
 ├── USER_WRITE
 ├── USER_DELETE
 └── REPORT_VIEW
```

Instead of assigning all permissions directly to every user:

```text
User → Role → Permissions
```

This is commonly called RBAC — Role-Based Access Control.

---

# 13. RBAC Example ⭐⭐⭐⭐⭐

```text
User: Nirbhay
       ↓
Role: ADMIN
       ↓
Authorities:
 ├── USER_READ
 ├── USER_WRITE
 ├── USER_DELETE
 └── REPORT_VIEW
```

Authorization:

```text
GET /users
      ↓
USER_READ
      ↓
Allowed
```

```text
DELETE /users/10
      ↓
USER_DELETE
      ↓
Allowed / Denied
```

---

# 14. URL Authorization with Roles ⭐⭐⭐⭐⭐

Modern Spring Security configuration example:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .requestMatchers("/orders/**").hasAuthority("ORDER_READ")
        .anyRequest().authenticated()
    );

    return http.build();
}
```

Important:

```text
/admin/**
   ↓
ROLE_ADMIN

/orders/**
   ↓
ORDER_READ
```

---

# 15. Method Security ⭐⭐⭐⭐⭐

Authorities can also be used at method level.

Enable method security:

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
    // ...
}
```

Authority example:

```java
@PreAuthorize("hasAuthority('ORDER_WRITE')")
public void updateOrder(Long id) {
    // ...
}
```

---

# 16. `hasAnyRole()` and `hasAnyAuthority()` ⭐⭐⭐⭐

Multiple roles:

```java
@PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
```

Multiple authorities:

```java
@PreAuthorize(
    "hasAnyAuthority('ORDER_READ', 'ORDER_ADMIN')"
)
```

Conceptually:

```text
hasAnyRole
    ↓
OR condition between roles

hasAnyAuthority
    ↓
OR condition between authorities
```

---

# 17. `getAuthorities()` ⭐⭐⭐⭐⭐

Authenticated principal se authorities retrieve ki ja sakti hain:

```java
Authentication authentication =
        SecurityContextHolder.getContext()
                .getAuthentication();

Collection<? extends GrantedAuthority> authorities =
        authentication.getAuthorities();
```

Example output:

```text
[ROLE_ADMIN, ORDER_READ, ORDER_WRITE]
```

---

# 18. Authentication vs Authorities ⭐⭐⭐⭐⭐

```text
Authentication
 ├── Principal
 ├── Credentials
 ├── Authorities ← authorization information
 └── Authenticated state
```

`GrantedAuthority` authentication object ke andar authorization-related information carry karta hai.

Important distinction:

```text
Authentication = current security identity/state
GrantedAuthority = granted permission/role authority
```

---

# 19. Database Design for Roles and Authorities ⭐⭐⭐⭐⭐

Simple RBAC schema:

```text
users
-----
id
username
password

roles
-----
id
name

permissions
----------
id
name

user_roles
----------
user_id
role_id

role_permissions
----------------
role_id
permission_id
```

Flow:

```text
User
 ↓
Role
 ↓
Permission
 ↓
GrantedAuthority
```

---

# 20. Mapping Database Roles to GrantedAuthority ⭐⭐⭐⭐⭐

Example database:

```text
role = ADMIN
```

Application mapping:

```java
new SimpleGrantedAuthority("ROLE_" + roleName);
```

Result:

```text
ROLE_ADMIN
```

For permissions:

```java
new SimpleGrantedAuthority(permissionName);
```

Example:

```text
USER_READ
USER_DELETE
REPORT_VIEW
```

---

# 21. Role + Permission Architecture ⭐⭐⭐⭐⭐

Recommended mental model:

```text
                 User
                   ↓
                 Roles
                   ↓
             Permissions
                   ↓
          GrantedAuthority
                   ↓
             Authorization
```

Example:

```text
Nirbhay
   ↓
ADMIN
   ↓
ROLE_ADMIN
USER_READ
USER_WRITE
USER_DELETE
REPORT_VIEW
```

---

# 22. JWT and GrantedAuthority ⭐⭐⭐⭐⭐

JWT-based systems commonly carry authorities/roles as claims, depending on the identity provider and token design.

Example conceptual JWT claims:

```json
{
  "sub": "nirbhay",
  "roles": ["ADMIN"],
  "permissions": ["ORDER_READ", "ORDER_WRITE"]
}
```

But JWT claims are not automatically equal to Spring Security `GrantedAuthority` objects.

A converter/mapping layer may be required:

```text
JWT Claims
    ↓
JwtAuthenticationConverter / custom mapping
    ↓
GrantedAuthority
    ↓
Authorization
```

### ⭐ Important interview point

> **A role claim in a JWT does not automatically mean Spring Security will recognize it as `ROLE_*`; the token-to-authority mapping must be configured correctly.**

---

# 23. OAuth2 / Resource Server and Authorities ⭐⭐⭐⭐

In OAuth2 resource-server applications:

```text
Bearer Token
     ↓
JWT validation
     ↓
JWT claims
     ↓
Authority mapping
     ↓
GrantedAuthority
     ↓
Authorization
```

Different identity providers may use different claim names such as:

```text
scope
scp
roles
groups
permissions
```

Application configuration determines how these become Spring Security authorities.

---

# 24. Scopes vs Roles ⭐⭐⭐⭐⭐

Don't blindly equate OAuth scopes with application roles.

Example:

```text
scope = orders.read
```

can become an authority such as:

```text
SCOPE_orders.read
```

depending on the resource-server configuration.

Whereas application role:

```text
ADMIN
```

may become:

```text
ROLE_ADMIN
```

They are different authorization concepts even though both can be represented as `GrantedAuthority`.

---

# 25. Custom Authority Mapping ⭐⭐⭐⭐⭐

If JWT contains:

```json
{
  "roles": ["ADMIN", "USER"]
}
```

you may need to map:

```text
ADMIN → ROLE_ADMIN
USER  → ROLE_USER
```

Then authorization can use:

```java
.hasRole("ADMIN")
```

The exact converter depends on the authentication mechanism and Spring Security configuration.

---

# 26. `ROLE_ADMIN` vs `ADMIN` ⭐⭐⭐⭐⭐

This is a common interview trap.

Suppose authorities are:

```text
ROLE_ADMIN
```

Then standard configuration:

```java
hasRole("ADMIN")
```

works through the role prefix convention.

But:

```java
hasAuthority("ADMIN")
```

does not match `ROLE_ADMIN` because it asks for the exact authority `ADMIN`.

Correct direct check:

```java
hasAuthority("ROLE_ADMIN")
```

or role-style:

```java
hasRole("ADMIN")
```

---

# 27. Role Prefix Customization ⭐⭐⭐⭐

Role prefix handling can be customized in Spring Security.

Therefore, for interviews say:

> **By default, role-based checks commonly use the `ROLE_` prefix, but role prefix behavior can be configured.**

Avoid saying:

> “Spring Security always requires ROLE_.”

That is too absolute.

---

# 28. SecurityContext Relationship ⭐⭐⭐⭐⭐

After successful authentication:

```text
Authentication
     ↓
SecurityContext
     ↓
SecurityContextHolder
```

Authentication contains authorities:

```text
Authentication
 ├── Principal
 ├── Authorities
 │     ├── ROLE_USER
 │     └── ORDER_READ
 └── Authenticated = true
```

Authorization components use these authorities to make access decisions.

---

# 29. Least Privilege ⭐⭐⭐⭐⭐

Production security mein user ko sirf required permissions deni chahiye.

Bad:

```text
Every user → ADMIN
```

Better:

```text
Normal user → ORDER_READ

Support     → ORDER_READ + ORDER_UPDATE

Admin       → USER_* + ORDER_* + REPORT_*
```

### Principle

> **Give users the minimum authority required to perform their job.**

---

# 30. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1 — Role and authority are exactly the same

❌ Not precise.

> Role is commonly represented as a role-prefixed authority; authorities can also represent fine-grained permissions.

### Trap 2 — `hasRole("ADMIN")` checks exact `ADMIN`

❌ Not under the standard role-prefix convention.

> It commonly checks `ROLE_ADMIN`.

### Trap 3 — `hasAuthority("ADMIN")` and `hasRole("ADMIN")` are always equivalent

❌ Wrong.

```text
hasRole("ADMIN")
    → ROLE_ADMIN

hasAuthority("ADMIN")
    → ADMIN
```

### Trap 4 — JWT role claim automatically becomes authority

❌ Not necessarily.

> Token claim mapping must produce the desired `GrantedAuthority` values.

### Trap 5 — Authentication and authorization are same

❌ No.

> Authentication identifies/authenticates; authorization decides access.

### Trap 6 — Give every user all permissions

❌ Violates least privilege.

---

# 31. Production Best Practices ⭐⭐⭐⭐⭐

- Use least privilege.
- Keep role names and authority names consistent.
- Prefer fine-grained permissions where business requirements need them.
- Keep authorization rules close to the protected resource/method.
- Avoid hardcoding large permission matrices throughout controllers.
- Map external JWT/OAuth claims to authorities explicitly.
- Test both positive and negative authorization scenarios.
- Don't expose sensitive role/permission information unnecessarily.
- Keep role hierarchy simple and documented.
- Treat authorization as server-side security; frontend role checks are only UX controls.

---

# 32. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is GrantedAuthority?

> It represents an authority granted to an authenticated principal and is used during authorization decisions.

### Q2. What is the difference between role and authority?

> A role is commonly represented as a role-prefixed authority such as `ROLE_ADMIN`, while an authority can represent a more specific permission such as `ORDER_READ`.

### Q3. What does `hasRole("ADMIN")` check?

> Under the standard role-prefix convention, it checks for the `ROLE_ADMIN` authority.

### Q4. What does `hasAuthority("ORDER_READ")` check?

> It checks for the authority `ORDER_READ`.

### Q5. What is `SimpleGrantedAuthority`?

> It is a simple implementation of `GrantedAuthority` that stores an authority string.

### Q6. Where are authorities stored after authentication?

> They are carried by the authenticated `Authentication` object, which is associated with the `SecurityContext`.

### Q7. How do roles reach UserDetails?

> A UserDetailsService or another authentication mechanism loads/maps the user's roles or permissions into `GrantedAuthority` values returned by UserDetails.

### Q8. What is RBAC?

> Role-Based Access Control assigns permissions through roles rather than directly assigning every permission to each user.

### Q9. Are JWT roles automatically GrantedAuthority?

> No. The JWT claims need to be mapped to Spring Security authorities by the appropriate converter/configuration.

### Q10. What is least privilege?

> Give each user or service only the minimum permissions required to perform its responsibilities.

---

# 33. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“GrantedAuthority is Spring Security's abstraction for representing an authority granted to an authenticated user. UserDetails exposes these authorities through `getAuthorities()`, and the authorization system uses them to decide whether access should be allowed. Roles are commonly represented as authorities with the `ROLE_` prefix, so `hasRole("ADMIN")` normally checks for `ROLE_ADMIN`, while `hasAuthority("ORDER_READ")` checks that exact permission. In a production RBAC design, users are assigned roles and roles are mapped to fine-grained permissions, which ultimately become GrantedAuthority values. With JWT or OAuth2, claims such as roles, groups or scopes need to be explicitly mapped into GrantedAuthority values; a JWT role claim does not automatically become a Spring Security authority. The overall principle is least privilege.”**

---

# 34. 30-Second Hinglish Answer

> **“GrantedAuthority Spring Security mein user ki granted permission ya role ko represent karta hai. UserDetails ke `getAuthorities()` se ye authorities milti hain aur authorization ke time use hoti hain. `hasRole("ADMIN")` standard setup mein `ROLE_ADMIN` ko check karta hai, jabki `hasAuthority("ORDER_READ")` exact authority check karta hai. JWT mein role ya scope claim automatically authority nahi ban jata; usko proper mapping/converter se GrantedAuthority mein convert karna padta hai. Production mein least privilege follow karte hain.”**

---

# 35. Whiteboard Memory ⭐⭐⭐⭐⭐

```text
                 USER
                   ↓
                 ROLE
                   ↓
              PERMISSIONS
                   ↓
          GrantedAuthority
                   ↓
             Authentication
                   ↓
             SecurityContext
                   ↓
             Authorization
                   ↓
              ALLOW / DENY
```

### Example

```text
Nirbhay
   ↓
ADMIN
   ↓
ROLE_ADMIN
USER_READ
USER_WRITE
ORDER_READ
ORDER_WRITE
   ↓
Authorization
```

### Final memory line

> **Role = common grouping | Authority = actual granted capability | Authorization checks authorities.**

---

## Navigation

[← S4.1.13 — PasswordEncoder](../13-PasswordEncoder/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.15 — Authorization and Access Decisions**
