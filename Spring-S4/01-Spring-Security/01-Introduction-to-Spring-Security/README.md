# S4.1.1 — Introduction to Spring Security

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is Spring Security?

**Spring Security** is a framework in the Spring ecosystem used to secure Java/Spring applications.

It primarily provides support for:

```text
Authentication
Authorization
Protection against common web attacks
Session management
Security context management
Password storage
OAuth 2.0 / OpenID Connect integration
JWT / Resource Server security
Method-level security
Security testing
```

Simple definition:

> **Spring Security is a framework that provides authentication, authorization, and protection mechanisms for Spring applications.**

---

# 2. Why Do We Need Security?

Suppose we have an Employee API:

```http
GET /employees
POST /employees
DELETE /employees/10
```

Without security, anyone who can reach the API might attempt to call these endpoints.

We usually need rules such as:

```text
Who are you?
        ↓
Authentication
        ↓
What are you allowed to do?
        ↓
Authorization
```

Example:

```text
ADMIN
 ├── GET /employees
 ├── POST /employees
 └── DELETE /employees/10

USER
 └── GET /employees
```

---

# 3. Authentication vs Authorization ⭐⭐⭐⭐⭐

This is one of the most important Spring Security concepts.

## Authentication

Authentication answers:

> **“Who are you?”**

Examples:

```text
Username + Password
JWT
OAuth2 Login
Certificate
API credentials
```

Flow:

```text
Request
  ↓
Credentials / Token
  ↓
Authentication
  ↓
Identity established
```

## Authorization

Authorization answers:

> **“What are you allowed to access?”**

Example:

```text
User = Nirbhay
Role = ADMIN

Allowed:
DELETE /employees/10
```

### Memory trick

> **Authentication = Who?**  
> **Authorization = What can you do?**

---

# 4. Spring Security at a High Level

A simplified request flow is:

```text
Client
  ↓
HTTP Request
  ↓
Servlet Container
  ↓
Spring Security Filter Chain
  ↓
Authentication / Authorization checks
  ↓
Controller
  ↓
Service
  ↓
Repository / Database
```

The key point is:

> **Spring Security normally performs security processing before the request reaches the controller.**

This is why security logic does not need to be manually repeated inside every controller.

---

# 5. Security Filter Chain ⭐⭐⭐⭐⭐

The **Security Filter Chain** is one of the central concepts in Spring Security.

A request passes through security filters before reaching the application endpoint.

Conceptually:

```text
HTTP Request
     ↓
┌───────────────────────┐
│ Security Filter Chain │
├───────────────────────┤
│ Security Filter 1     │
│ Security Filter 2     │
│ Security Filter 3     │
│ ...                   │
└───────────────────────┘
     ↓
Authentication
     ↓
Authorization
     ↓
Controller
```

Different filters have different responsibilities.

Examples include processing authentication mechanisms, security context, CSRF, authorization, and exception handling.

**Important:** Do not memorize a fixed universal filter order. The actual chain depends on the application's configuration and enabled features.

---

# 6. Spring Security Does Not Mean Only Username/Password

Spring Security supports multiple security models.

```text
Username / Password
        ↓
Form Login

HTTP Basic
        ↓
Basic Authentication

JWT
        ↓
Stateless API Security

OAuth 2.0
        ↓
Delegated Authorization

OpenID Connect
        ↓
Authentication Identity Layer
```

Therefore, Spring Security is much broader than a login page.

---

# 7. Authentication Flow — Basic Concept

Consider a username/password login.

```text
Client
  ↓
Username + Password
  ↓
Spring Security
  ↓
AuthenticationManager
  ↓
AuthenticationProvider
  ↓
UserDetailsService
  ↓
PasswordEncoder
  ↓
Authentication successful?
  ├── No  → Authentication failure
  └── Yes → Authenticated user
```

Later topics will explain each component in detail.

---

# 8. Passwords Must Not Be Stored as Plain Text ⭐⭐⭐⭐⭐

Never store:

```text
password = "Nirbhay@123"
```

as plain text in a database.

Instead, passwords should be stored using a suitable one-way password hashing mechanism.

Common Spring Security choice:

```java
PasswordEncoder
```

For example:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

The important principle is:

```text
Raw password
     ↓
Password hashing
     ↓
Stored password hash
```

During login, the submitted password is checked against the stored hash rather than decrypting a stored password.

---

# 9. SecurityContext — Basic Idea ⭐⭐⭐⭐⭐

After successful authentication, Spring Security needs to keep track of the authenticated identity for the current security execution context.

This is represented through the **SecurityContext**.

Conceptually:

```text
Successful Authentication
          ↓
SecurityContext
          ↓
Authenticated Principal
          ↓
Authorization decisions / application code
```

Application code can access the current authentication through Spring Security APIs.

Example:

```java
Authentication authentication =
        SecurityContextHolder
                .getContext()
                .getAuthentication();
```

We will study `SecurityContextHolder` deeply in a later topic.

---

# 10. Authentication Principal

The **principal** represents the currently authenticated identity.

Conceptually:

```text
Authentication
 ├── Principal
 ├── Credentials
 └── Authorities
```

For example:

```text
Principal = nirbhay
Authorities = ROLE_ADMIN
```

Do not confuse:

```text
Principal → identity
Authority → permission granted to identity
```

---

# 11. Roles and Authorities — Basic Idea

Spring Security can make authorization decisions using authorities.

Example:

```text
ROLE_ADMIN
ROLE_USER
READ_EMPLOYEE
WRITE_EMPLOYEE
DELETE_EMPLOYEE
```

A rule might conceptually be:

```text
DELETE /employees/**
        ↓
Requires ROLE_ADMIN
```

We will later cover the important difference between **roles and authorities**.

---

# 12. Stateful vs Stateless Security ⭐⭐⭐⭐⭐

Spring Security can be used in both stateful and stateless application designs.

## Stateful

The server maintains authentication state, commonly using an HTTP session.

```text
Client
  ↓
Session ID
  ↓
Server-side session
  ↓
Authenticated state
```

Common example:

```text
Traditional web application
Form login + session
```

## Stateless

The server does not depend on a server-side session to remember the authentication state for each request.

A common API approach is:

```text
Client
  ↓
Authorization: Bearer <JWT>
  ↓
Server validates token
  ↓
Authentication
  ↓
Authorization
```

Stateless does **not** mean “no state exists anywhere.” It means the server does not maintain the authentication state in a traditional session for each client request.

---

# 13. Spring Security and REST APIs

For a REST API, a common architecture is:

```text
Frontend / Mobile Client
          ↓
     HTTPS Request
          ↓
Authorization: Bearer <JWT>
          ↓
Spring Security
          ↓
JWT validation
          ↓
Authentication
          ↓
Authorization
          ↓
REST Controller
```

The exact architecture depends on whether the application is acting as an OAuth2 client, resource server, authorization server, or using another authentication mechanism.

---

# 14. Spring Security in Microservices ⭐⭐⭐⭐⭐

In a microservices architecture, security becomes distributed.

Example:

```text
                 ┌───────────────┐
Client ─────────→│ API Gateway   │
                 └───────┬───────┘
                         ↓
                ┌─────────────────┐
                │ Authentication  │
                │ / Token checks  │
                └────────┬────────┘
                         ↓
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
     Order Service   User Service   Payment Service
```

Important production questions include:

```text
Who authenticates the user?
Who validates the token?
How are service-to-service calls secured?
Where are authorization rules enforced?
How are tokens propagated?
```

These will be covered later under S4 microservices security.

---

# 15. Common Spring Security Components — Roadmap

At a high level, remember these components:

| Component | Basic Responsibility |
|---|---|
| SecurityFilterChain | Applies web security filters/rules |
| Authentication | Represents authentication request/result |
| AuthenticationManager | Coordinates authentication |
| AuthenticationProvider | Performs a particular authentication mechanism |
| UserDetailsService | Loads user information |
| UserDetails | Represents user details used by authentication |
| PasswordEncoder | Secure password hashing/matching |
| SecurityContext | Holds current authentication context |
| SecurityContextHolder | Provides access to SecurityContext |
| GrantedAuthority | Represents an authority/permission |
| AccessDeniedHandler | Handles authenticated-but-forbidden access |
| AuthenticationEntryPoint | Handles unauthenticated access |

We will cover these one by one rather than trying to memorize the entire architecture at once.

---

# 16. Modern Spring Security Configuration ⭐⭐⭐⭐⭐

For modern Spring Security, configuration is commonly based on a `SecurityFilterChain` bean.

Example:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

The exact configuration depends on the application and authentication model.

### Important modern-version point

Older Spring Security tutorials often show:

```java
extends WebSecurityConfigurerAdapter
```

That approach is obsolete in modern Spring Security. Current applications commonly configure security using beans such as `SecurityFilterChain`.

---

# 17. What Spring Security Protects Against

Spring Security provides mechanisms that help address common application-security concerns, including:

```text
Authentication attacks
Session-related attacks
CSRF
Authorization failures
Password-storage risks
Security headers
Common web-security concerns
```

But remember:

> **Using Spring Security does not automatically make an application secure.**

Developers still need secure configuration, HTTPS, proper authorization, secure password handling, dependency updates, input validation, secrets management, logging/monitoring, and secure architecture.

---

# 18. Spring Security vs Authentication System

Spring Security is a **security framework**, not just a user database.

It can integrate with:

```text
Database
LDAP
OAuth2 / OIDC providers
JWT resource servers
Custom authentication providers
In-memory users
Other identity systems
```

So:

> **Spring Security provides the security infrastructure; the actual identity source can vary.**

---

# 19. Why Spring Security Is Preferred in Spring Applications

### Without a security framework

Developers may end up writing:

```text
Authentication logic
Authorization checks
Password handling
Session handling
Security filters
Exception handling
CSRF protection
Security headers
```

manually.

### With Spring Security

```text
Spring Security
      ↓
Standard security abstractions
      ↓
Configurable security pipeline
      ↓
Reusable authentication + authorization mechanisms
```

This improves consistency and reduces the need to reinvent security infrastructure.

---

# 20. Important Interview Question

### Q: What is Spring Security?

### 2-minute answer

> **“Spring Security is a security framework in the Spring ecosystem used primarily for authentication, authorization, and protection against common web-security threats. It integrates with Spring applications through a security filter chain and provides abstractions such as AuthenticationManager, AuthenticationProvider, UserDetailsService, PasswordEncoder, SecurityContext and GrantedAuthority. It supports multiple authentication models including form login, HTTP Basic, JWT/resource-server security and OAuth2/OIDC integrations. In modern Spring Security, web security is commonly configured through a SecurityFilterChain bean. For REST and microservices applications, it can be used to implement stateless bearer-token security and fine-grained authorization.”**

---

# 21. 30-Second Hinglish Answer

> **“Spring Security Spring ecosystem ka security framework hai jo mainly authentication, authorization aur common web-security attacks se protection provide karta hai. Iska core request processing Security Filter Chain ke through hota hai. Authentication ke liye AuthenticationManager, AuthenticationProvider, UserDetailsService aur PasswordEncoder jaise components use hote hain, aur authenticated user ka context SecurityContext mein maintain hota hai. Modern Spring Security mein SecurityFilterChain bean ke through configuration commonly ki jaati hai. REST/microservices mein JWT aur OAuth2 jaise mechanisms ke saath bhi use kar sakte hain.”**

---

# 22. Interview Follow-up Questions

After this topic, interviewer may ask:

1. What is the difference between authentication and authorization?
2. What is SecurityFilterChain?
3. How does a request pass through Spring Security?
4. What is AuthenticationManager?
5. What is AuthenticationProvider?
6. What is UserDetailsService?
7. Why do we need PasswordEncoder?
8. What is SecurityContext?
9. What is the difference between role and authority?
10. How does JWT authentication work?
11. How is stateless authentication different from session-based authentication?
12. How does Spring Security work in microservices?
13. What is the difference between OAuth2 and JWT?
14. Why is `WebSecurityConfigurerAdapter` not used in modern Spring Security?
15. Where should authorization be enforced in a microservices architecture?

---

# 23. Memory Map ⭐⭐⭐⭐⭐

```text
SPRING SECURITY
│
├── Authentication
│   ├── Who are you?
│   ├── AuthenticationManager
│   ├── AuthenticationProvider
│   ├── UserDetailsService
│   └── PasswordEncoder
│
├── Authorization
│   ├── Roles
│   ├── Authorities
│   └── Method Security
│
├── Request Security
│   └── SecurityFilterChain
│
├── Security Context
│   ├── SecurityContext
│   └── SecurityContextHolder
│
├── Web Security
│   ├── CSRF
│   ├── CORS
│   ├── Sessions
│   └── Security Headers
│
├── Token Security
│   ├── JWT
│   ├── OAuth2
│   └── OIDC
│
└── Microservices Security
    ├── Resource Server
    ├── Gateway
    └── Service-to-Service Security
```

### One-line memory

> **Spring Security = Filter Chain + Authentication + Authorization + Security Context + Protection mechanisms.**

---

## Navigation

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.2 — Why Spring Security?**
