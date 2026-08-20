# S4.1.3 — Spring Security Architecture

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. Big Picture

Spring Security architecture ko samajhne ka easiest way hai request ka end-to-end flow samajhna:

```text
Client
  ↓
HTTP Request
  ↓
Servlet Container
  ↓
DelegatingFilterProxy
  ↓
FilterChainProxy
  ↓
SecurityFilterChain
  ↓
Security Filters
  ↓
Authentication / Authorization
  ↓
DispatcherServlet
  ↓
Controller
```

**Core idea:** security processing normally controller tak pahunchne se pehle hoti hai.

---

# 2. Main Building Blocks ⭐⭐⭐⭐⭐

Spring Security architecture mein important building blocks:

```text
DelegatingFilterProxy
        ↓
FilterChainProxy
        ↓
SecurityFilterChain
        ↓
Security Filters
        ↓
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
UserDetailsService / other identity source
        ↓
SecurityContext
        ↓
Authorization
```

Har component ka specific responsibility hai. Inhe ek hi component samajhna common interview mistake hai.

---

# 3. DelegatingFilterProxy ⭐⭐⭐⭐⭐

Servlet container filters ko manage karta hai, jabki Spring beans Spring ApplicationContext mein managed hote hain.

`DelegatingFilterProxy` in dono worlds ke beech bridge ka kaam karta hai.

Conceptually:

```text
Servlet Container
       ↓
DelegatingFilterProxy
       ↓
Spring ApplicationContext
       ↓
Spring Security bean
```

It delegates filter processing to a Spring-managed bean instead of putting all security logic directly inside a raw servlet filter.

### Interview line

> **DelegatingFilterProxy is a servlet Filter that delegates processing to a Spring-managed bean.**

---

# 4. FilterChainProxy ⭐⭐⭐⭐⭐

Spring Security ka central filter infrastructure component `FilterChainProxy` hai.

Conceptually:

```text
DelegatingFilterProxy
        ↓
FilterChainProxy
        ↓
Select matching SecurityFilterChain
        ↓
Run security filters
```

`FilterChainProxy` multiple security filter chains ko manage kar sakta hai.

Important distinction:

```text
DelegatingFilterProxy
        ↓
Delegates into Spring

FilterChainProxy
        ↓
Finds and invokes appropriate security filter chain
```

---

# 5. SecurityFilterChain ⭐⭐⭐⭐⭐

`SecurityFilterChain` define karta hai ki particular request ke liye kaunse security filters/rules apply honge.

Example modern configuration:

```java
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
```

Conceptually:

```text
HTTP Request
     ↓
Matching SecurityFilterChain
     ↓
Configured Security Filters
     ↓
Security Decision
```

### Important

A Spring application can have **multiple `SecurityFilterChain` beans** with different request matchers. The matching chain is selected for a request.

---

# 6. Security Filters ⭐⭐⭐⭐⭐

Security filters are responsible for different parts of the security process.

Depending on configuration, filters can handle things such as:

```text
Security context handling
Authentication mechanisms
Bearer token processing
CSRF
Logout
Request authorization
Exception translation
Headers
Session-related security
```

Do not memorize one fixed filter list and assume it is identical for every application.

The actual chain depends on enabled security features and configuration.

---

# 7. Request Flow — Anonymous Request

Suppose user calls:

```http
GET /employees
```

Flow:

```text
Client
  ↓
GET /employees
  ↓
Servlet Container
  ↓
DelegatingFilterProxy
  ↓
FilterChainProxy
  ↓
SecurityFilterChain
  ↓
Security filters
  ↓
Authorization decision
```

If the endpoint requires authentication and no valid authentication exists:

```text
Authentication missing
        ↓
AuthenticationEntryPoint
        ↓
401 / authentication response
```

Exact response behavior depends on the configured authentication mechanism and application configuration.

---

# 8. Request Flow — Authenticated Request ⭐⭐⭐⭐⭐

Suppose request contains:

```http
Authorization: Bearer <access-token>
```

Conceptually:

```text
Request
  ↓
Security Filter Chain
  ↓
Authentication mechanism
  ↓
Token validation
  ↓
Authentication object
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
```

If authorization succeeds:

```text
Controller
    ↓
Service
    ↓
Repository
```

---

# 9. Authentication Architecture ⭐⭐⭐⭐⭐

Authentication is responsible for establishing the identity of the caller.

A simplified architecture is:

```text
Request
  ↓
Authentication mechanism
  ↓
AuthenticationManager
  ↓
AuthenticationProvider
  ↓
Identity source
  ↓
Successful Authentication
  ↓
SecurityContext
```

Identity source may be:

```text
Database
LDAP
In-memory users
External Identity Provider
JWT claims / token validation
Custom provider
```

The exact flow differs by authentication mechanism.

---

# 10. AuthenticationManager ⭐⭐⭐⭐⭐

`AuthenticationManager` is the main authentication API.

Its core responsibility is to attempt authentication.

Conceptually:

```java
Authentication authenticate(Authentication authentication)
```

Flow:

```text
Authentication request
        ↓
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
Authentication result
```

A common implementation is `ProviderManager`.

---

# 11. AuthenticationProvider ⭐⭐⭐⭐⭐

`AuthenticationProvider` contains the logic for a particular authentication mechanism.

Examples:

```text
Username/password provider
JWT/bearer-token authentication support
Custom authentication provider
```

Conceptually:

```text
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
Validate credentials / token
        ↓
Return authenticated Authentication
```

Important:

> **AuthenticationManager coordinates authentication; AuthenticationProvider performs a specific authentication strategy.**

---

# 12. UserDetailsService ⭐⭐⭐⭐⭐

For username/password authentication, `UserDetailsService` is commonly used to load user information.

Conceptually:

```text
username
   ↓
UserDetailsService
   ↓
Database / identity store
   ↓
UserDetails
```

Example:

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username);
}
```

It is important to understand that `UserDetailsService` **loads user data**; it is not itself the complete authentication mechanism.

---

# 13. PasswordEncoder ⭐⭐⭐⭐⭐

When password-based authentication is used:

```text
Submitted password
       ↓
PasswordEncoder matching
       ↓
Stored password hash
```

Example:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

Passwords should not be stored in plaintext.

---

# 14. Authentication Object ⭐⭐⭐⭐⭐

After successful authentication, Spring Security represents the authenticated caller using an `Authentication` object.

Conceptually:

```text
Authentication
├── Principal
├── Credentials
└── Authorities
```

For example:

```text
Principal: nirbhay
Authorities:
  ROLE_ADMIN
  READ_EMPLOYEE
```

Credentials may be cleared or otherwise handled securely after authentication depending on the mechanism.

---

# 15. SecurityContext ⭐⭐⭐⭐⭐

The authenticated `Authentication` is associated with the `SecurityContext`.

```text
Authentication
      ↓
SecurityContext
      ↓
Current security identity
```

Application code can access it through:

```java
SecurityContextHolder
        .getContext()
        .getAuthentication();
```

The exact lifecycle/storage strategy can depend on the execution model and Spring Security configuration.

---

# 16. SecurityContextHolder ⭐⭐⭐⭐⭐

`SecurityContextHolder` is the main API used to access the current `SecurityContext`.

Example:

```java
Authentication authentication =
        SecurityContextHolder
            .getContext()
            .getAuthentication();
```

Conceptually:

```text
Current execution
      ↓
SecurityContextHolder
      ↓
SecurityContext
      ↓
Authentication
```

In traditional servlet applications, the context is commonly associated with the current thread during request processing.

Do not assume this exact model for every reactive/non-servlet execution model.

---

# 17. Authorization Architecture ⭐⭐⭐⭐⭐

After authentication, Spring Security needs to answer:

> **Is this authenticated identity allowed to access the requested resource?**

Conceptually:

```text
Authenticated User
       ↓
Authorities / Roles
       ↓
Authorization Rules
       ↓
Allow / Deny
```

Example:

```text
ROLE_ADMIN
     ↓
DELETE /employees/**
     ↓
Allowed
```

If authenticated but not authorized:

```text
Access denied
     ↓
AccessDeniedHandler
     ↓
403 Forbidden
```

---

# 18. AuthenticationEntryPoint vs AccessDeniedHandler ⭐⭐⭐⭐⭐

Very common interview question.

## AuthenticationEntryPoint

Used when the request requires authentication but the caller is not authenticated.

Conceptually:

```text
Not authenticated
      ↓
AuthenticationEntryPoint
      ↓
401 / authentication challenge or response
```

## AccessDeniedHandler

Used when the caller is authenticated but does not have sufficient permission.

```text
Authenticated
      ↓
Insufficient authority
      ↓
AccessDeniedHandler
      ↓
403 Forbidden
```

### Memory trick

```text
401 → Who are you? → Authentication required
403 → I know you, but you can't do this
```

---

# 19. ExceptionTranslationFilter — Conceptual Role ⭐⭐⭐⭐

Spring Security has infrastructure that translates certain security exceptions into the appropriate security response flow.

Conceptually:

```text
Security exception
      ↓
Exception translation infrastructure
      ├── AuthenticationException
      │       ↓
      │   AuthenticationEntryPoint
      │
      └── AccessDeniedException
              ↓
          AccessDeniedHandler
```

This is one reason application code does not need to manually convert every security failure into an HTTP response.

---

# 20. Where Does DispatcherServlet Fit?

This distinction is important for Spring MVC interviews.

```text
Client
  ↓
Servlet Container
  ↓
Spring Security Filter Chain
  ↓
DispatcherServlet
  ↓
Controller
```

### Security filters vs DispatcherServlet

```text
Security Filter Chain
→ Security concerns

DispatcherServlet
→ Spring MVC request dispatching
```

Security normally processes the request before Spring MVC invokes the controller.

---

# 21. Complete Username/Password Architecture ⭐⭐⭐⭐⭐

```text
                    Client
                      │
                      ↓
               Login Request
                      │
                      ↓
             Security Filter Chain
                      │
                      ↓
             AuthenticationManager
                      │
                      ↓
             AuthenticationProvider
                      │
             ┌────────┴────────┐
             ↓                 ↓
     UserDetailsService   PasswordEncoder
             ↓                 ↓
         Database          Hash Match
             └────────┬────────┘
                      ↓
          Successful Authentication
                      ↓
              SecurityContext
                      ↓
                Authorization
                      ↓
                 Controller
```

This is the architecture you should be able to explain on a whiteboard.

---

# 22. JWT / Bearer Token Architecture ⭐⭐⭐⭐⭐

For a resource server, a simplified flow is:

```text
Client
  ↓
Authorization: Bearer <token>
  ↓
Security Filter Chain
  ↓
Bearer token authentication mechanism
  ↓
Token validation
  ↓
Authenticated principal
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
```

Important:

> **JWT processing is not the same thing as the username/password `UserDetailsService` flow.**

A resource server validates the bearer token according to its configured authentication mechanism.

---

# 23. Multiple SecurityFilterChains ⭐⭐⭐⭐⭐

A real application may need different security rules for different request groups.

Example concept:

```text
/api/**
    ↓
JWT SecurityFilterChain

/web/**
    ↓
Form Login SecurityFilterChain

/public/**
    ↓
Different / permissive rules
```

Spring Security can define multiple chains with request matchers and an order.

This is useful when one application hosts different security models.

---

# 24. Modern Spring Security Configuration ⭐⭐⭐⭐⭐

Modern Spring Security commonly uses bean-based configuration:

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

Older tutorials may use:

```java
WebSecurityConfigurerAdapter
```

For modern Spring Security, do not build new applications around that obsolete configuration style.

---

# 25. Architecture — One-Line Responsibility Map ⭐⭐⭐⭐⭐

| Component | Responsibility |
|---|---|
| DelegatingFilterProxy | Bridges servlet container filter processing to a Spring-managed bean |
| FilterChainProxy | Selects/invokes the applicable security filter chain |
| SecurityFilterChain | Defines security filters for matching requests |
| Security Filters | Perform security processing |
| AuthenticationManager | Main authentication API / coordinator |
| ProviderManager | Common AuthenticationManager implementation using providers |
| AuthenticationProvider | Handles a specific authentication mechanism |
| UserDetailsService | Loads user details for username/password-style authentication |
| PasswordEncoder | Encodes and verifies password hashes |
| Authentication | Represents authentication state/identity |
| SecurityContext | Holds current Authentication |
| SecurityContextHolder | Access point for SecurityContext |
| GrantedAuthority | Represents granted permission/authority |
| AuthenticationEntryPoint | Handles unauthenticated access |
| AccessDeniedHandler | Handles authenticated-but-forbidden access |
| DispatcherServlet | Spring MVC request dispatching after filter processing |

---

# 26. Common Interview Traps ⚠️

### Trap 1

> “SecurityFilterChain is one filter.”

❌ Incorrect.

It represents a configured chain of security filters.

### Trap 2

> “UserDetailsService authenticates the user.”

⚠️ Incomplete.

It primarily loads user details. Authentication is performed through the authentication infrastructure, commonly involving an `AuthenticationProvider`.

### Trap 3

> “AuthenticationManager directly queries the database.”

❌ Not necessarily.

It delegates authentication to appropriate providers.

### Trap 4

> “JWT always uses UserDetailsService.”

❌ Not necessarily.

JWT resource-server authentication can validate the token and derive the authenticated principal without using the traditional username/password `UserDetailsService` flow.

### Trap 5

> “401 means insufficient permission.”

❌ Usually wrong.

```text
401 → Authentication required / not successfully authenticated
403 → Authenticated but access denied
```

### Trap 6

> “Gateway authentication means downstream services don't need security.”

❌ Not a safe production assumption.

Each service should enforce the security boundary appropriate to its responsibilities.

---

# 27. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“Spring Security architecture is filter-chain based. An HTTP request first enters the servlet container and is delegated through DelegatingFilterProxy into Spring Security's FilterChainProxy. FilterChainProxy selects the applicable SecurityFilterChain, which contains security filters responsible for context handling, authentication, authorization and other configured protections. During authentication, the authentication mechanism creates an Authentication request that is processed through AuthenticationManager and appropriate AuthenticationProviders. For username/password authentication, a provider may use UserDetailsService to load the user and PasswordEncoder to verify the password. After successful authentication, the Authentication is associated with the SecurityContext. Authorization rules then decide whether the authenticated user can access the resource. If the user is unauthenticated, AuthenticationEntryPoint handles the failure; if authenticated but unauthorized, AccessDeniedHandler handles it. Only after the security processing succeeds does the request normally reach DispatcherServlet and the controller.”**

---

# 28. 30-Second Hinglish Answer

> **“Spring Security ka architecture mainly filter-chain based hai. Request servlet container se DelegatingFilterProxy ke through FilterChainProxy tak jaati hai, jo matching SecurityFilterChain select karta hai. Security filters authentication aur authorization process karte hain. Authentication ke liye AuthenticationManager appropriate AuthenticationProvider ko delegate karta hai. Username-password case mein provider UserDetailsService se user details load aur PasswordEncoder se password verify kar sakta hai. Successful authentication ke baad Authentication SecurityContext mein store hoti hai, phir authorization decide hota hai. Security pass hone ke baad request DispatcherServlet aur controller tak jaati hai.”**

---

# 29. Whiteboard Memory Map ⭐⭐⭐⭐⭐

```text
                 HTTP REQUEST
                      │
                      ↓
             Servlet Container
                      │
                      ↓
           DelegatingFilterProxy
                      │
                      ↓
              FilterChainProxy
                      │
                      ↓
            SecurityFilterChain
                      │
             ┌────────┴────────┐
             ↓                 ↓
       Authentication     Authorization
             │                 │
             ↓                 ↓
    AuthenticationManager   Rules / Authorities
             │                 │
             ↓                 ↓
    AuthenticationProvider   Allow / Deny
             │
       ┌─────┴─────┐
       ↓           ↓
 UserDetails   PasswordEncoder
       │
       ↓
 Authentication
       │
       ↓
 SecurityContext
       │
       └──────────────┐
                      ↓
              DispatcherServlet
                      ↓
                 Controller
```

### One-line memory

> **Request → Proxy → FilterChainProxy → SecurityFilterChain → Authentication → SecurityContext → Authorization → DispatcherServlet → Controller.**

---

# 30. Interview Follow-up Questions

1. What is DelegatingFilterProxy?
2. What is FilterChainProxy?
3. What is SecurityFilterChain?
4. Can an application have multiple SecurityFilterChains?
5. How does Spring Security select a SecurityFilterChain?
6. What is AuthenticationManager?
7. What is ProviderManager?
8. What is AuthenticationProvider?
9. What is UserDetailsService?
10. Does UserDetailsService authenticate the user?
11. What is SecurityContext?
12. What is SecurityContextHolder?
13. Difference between AuthenticationEntryPoint and AccessDeniedHandler?
14. Difference between 401 and 403?
15. Where does DispatcherServlet fit in the security flow?
16. How does JWT authentication fit into the filter chain?
17. Why is `WebSecurityConfigurerAdapter` not used in modern Spring Security?
18. What happens if multiple SecurityFilterChains can match a request?
19. How would you explain Spring Security architecture on a whiteboard?
20. How would you secure a microservice if an API Gateway already authenticates the request?

---

## Navigation

[← S4.1.2 — Why Spring Security?](../02-Why-Spring-Security/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.4 — Security Filter Chain**
