# S4.1.4 — Security Filter Chain

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is Security Filter Chain?

`SecurityFilterChain` Spring Security ka core request-processing mechanism hai.

Simple words mein:

> **SecurityFilterChain = security filters ki configured chain jo incoming HTTP request ko controller tak pahunchne se pehle security processing deti hai.**

High-level flow:

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
DispatcherServlet
  ↓
Controller
```

---

# 2. SecurityFilterChain vs FilterChainProxy ⭐⭐⭐⭐⭐

Interview mein dono ko confuse mat karna.

### SecurityFilterChain

Defines the security filter chain for matching requests.

```text
SecurityFilterChain
   ├── Filter A
   ├── Filter B
   ├── Filter C
   └── Filter D
```

### FilterChainProxy

Spring Security ka central filter jo available `SecurityFilterChain` configurations mein se request ke liye appropriate chain select karta hai.

```text
                    FilterChainProxy
                          │
              ┌───────────┼───────────┐
              ↓           ↓           ↓
          Chain #1     Chain #2     Chain #3
          /api/**       /web/**      /admin/**
```

**Memory:**

```text
FilterChainProxy → Which chain?
SecurityFilterChain → Which security filters?
```

---

# 3. Why Do We Need a Filter Chain?

A request ko controller tak bhejne se pehle multiple security checks karne pad sakte hain.

For example:

```text
Request
  ↓
Security Context
  ↓
Authentication
  ↓
CSRF
  ↓
Request Authorization
  ↓
Security Headers
  ↓
Exception Handling
  ↓
Controller
```

Agar har controller mein ye checks manually likhe jaayen:

```java
if (!authenticated) { ... }
if (!hasRole) { ... }
if (!validToken) { ... }
```

to code duplicated aur difficult to maintain ho jayega.

Filter chain security concerns ko request-processing pipeline mein centralized rakhti hai.

---

# 4. Modern SecurityFilterChain Configuration ⭐⭐⭐⭐⭐

Modern Spring Security applications generally define a `SecurityFilterChain` bean.

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
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

Important:

```text
HttpSecurity
    ↓
configure security
    ↓
http.build()
    ↓
SecurityFilterChain bean
```

---

# 5. What Does HttpSecurity Do?

`HttpSecurity` is the configuration API used to configure web-based security.

Through it we can configure areas such as:

```text
Authorization
Form Login
HTTP Basic
CSRF
CORS
Logout
Session management
Exception handling
OAuth2 Resource Server
Headers
```

Example:

```java
http
    .csrf(csrf -> csrf.disable())
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/public/**").permitAll()
        .anyRequest().authenticated()
    );
```

### Important production note

CSRF ko blindly disable nahi karna chahiye. Decision application architecture aur authentication mechanism par depend karta hai.

For example, browser-based session/cookie authentication commonly needs CSRF protection, while a stateless bearer-token API may have a different CSRF threat model.

---

# 6. Request Matching ⭐⭐⭐⭐⭐

SecurityFilterChain request matching ka concept important hai.

Example:

```java
@Bean
SecurityFilterChain apiChain(HttpSecurity http)
        throws Exception {

    http.securityMatcher("/api/**")
        .authorizeHttpRequests(auth -> auth
            .anyRequest().authenticated()
        );

    return http.build();
}
```

Here:

```text
/api/employees
/api/orders
/api/payments
```

is chain ke scope mein aa sakte hain.

Whereas:

```text
/web/home
```

is particular chain ke scope mein nahi aayega.

---

# 7. Multiple SecurityFilterChains ⭐⭐⭐⭐⭐

Ek application mein multiple `SecurityFilterChain` beans ho sakte hain.

Real-world example:

```text
                    Incoming Request
                           ↓
                    FilterChainProxy
                           ↓
              ┌────────────┴────────────┐
              ↓                         ↓
        /api/** chain              /web/** chain
        JWT / Bearer               Form Login
              ↓                         ↓
          API rules                 Web rules
```

Example:

```java
@Bean
@Order(1)
SecurityFilterChain apiChain(HttpSecurity http)
        throws Exception {

    http
        .securityMatcher("/api/**")
        .authorizeHttpRequests(auth -> auth
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2.jwt());

    return http.build();
}

@Bean
SecurityFilterChain webChain(HttpSecurity http)
        throws Exception {

    http
        .authorizeHttpRequests(auth -> auth
            .anyRequest().authenticated()
        )
        .formLogin();

    return http.build();
}
```

Conceptually:

```text
/api/** → API chain → Bearer token
other   → Web chain → Form Login
```

The chain-selection order matters when multiple chains could match a request.

---

# 8. How Is a Chain Selected? ⭐⭐⭐⭐⭐

Think of it like this:

```text
Request
  ↓
FilterChainProxy
  ↓
Check configured SecurityFilterChains
  ↓
Find first applicable matching chain
  ↓
Execute that chain
```

Therefore, overlapping matchers must be designed carefully.

### Interview point

> **When multiple security chains are configured, their matchers and ordering determine which chain handles a request.**

---

# 9. Security Filters

SecurityFilterChain ke andar multiple filters ho sakte hain.

Depending on configuration, the chain can contain filters responsible for:

```text
Security context handling
Authentication
Bearer token processing
Username/password login
CSRF
Logout
Request authorization
Exception translation
Security headers
Session-related security
```

### Important

There is **no single universal filter list** that is identical for every Spring Security application.

Enabled features and configuration affect the actual chain.

---

# 10. Example: Bearer Token Request ⭐⭐⭐⭐⭐

Suppose:

```http
GET /api/orders
Authorization: Bearer eyJ...
```

Conceptual flow:

```text
HTTP Request
      ↓
FilterChainProxy
      ↓
API SecurityFilterChain
      ↓
Bearer Token Authentication
      ↓
Token Validation
      ↓
Authentication
      ↓
SecurityContext
      ↓
Authorization
      ↓
Controller
```

The exact filter implementation and order depend on the configured Spring Security features.

---

# 11. Example: Username + Password Request ⭐⭐⭐⭐⭐

Conceptual flow:

```text
Login Request
      ↓
SecurityFilterChain
      ↓
Username/password authentication mechanism
      ↓
AuthenticationManager
      ↓
AuthenticationProvider
      ↓
UserDetailsService
      ↓
PasswordEncoder
      ↓
Authentication
      ↓
SecurityContext
      ↓
Authenticated request
```

Again, the exact filter sequence depends on the configuration.

---

# 12. Filter Order — Important Interview Concept ⭐⭐⭐⭐⭐

Security filters are not simply executed in random order.

Their ordering matters because one filter can prepare information required by another filter.

Conceptually:

```text
Earlier filter
     ↓
Creates / loads security information
     ↓
Later filter
     ↓
Uses that information
```

For example, authentication must be established before authorization can make a decision based on the authenticated identity.

### Don't memorize only numbers

Interview mein better answer:

> “Filter ordering is important, and the actual order depends on the configured security features. I understand the responsibilities and relative flow rather than relying only on hard-coded filter positions.”

---

# 13. Adding a Custom Filter ⭐⭐⭐⭐⭐

Custom filters can be added when application-specific processing is genuinely required.

Example concept:

```java
http.addFilterBefore(
    customFilter,
    UsernamePasswordAuthenticationFilter.class
);
```

Or other appropriate placement APIs can be used depending on the requirement.

### Why `addFilterBefore()`?

Because sometimes custom logic needs to execute before a known security filter.

Conceptually:

```text
Existing Filter A
      ↓
Custom Filter
      ↓
Existing Filter B
```

### Production advice

Custom filter add karna easy hai, but unnecessary custom filters architecture ko complex bana sakte hain.

First check whether Spring Security already provides the required mechanism.

---

# 14. Custom JWT Filter — Common Legacy Pattern ⚠️

Many tutorials show:

```java
OncePerRequestFilter
        ↓
Read Authorization header
        ↓
Extract JWT
        ↓
Validate JWT
        ↓
Set SecurityContext
```

This pattern can be valid for custom authentication designs, but modern Spring Security applications should first consider the built-in **OAuth2 Resource Server JWT support** rather than writing JWT parsing/validation infrastructure from scratch.

### Interview-level answer

> “If the application is a standard JWT resource server, I prefer Spring Security's built-in resource-server support. I would write a custom filter only when the authentication requirement is genuinely custom.”

---

# 15. OncePerRequestFilter

`OncePerRequestFilter` is a convenient base class for custom servlet filters when you want a filter implementation to execute once per request dispatch under its documented semantics.

Example:

```java
public class CustomFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        // custom processing
        filterChain.doFilter(request, response);
    }
}
```

Important:

```java
filterChain.doFilter(request, response);
```

is what allows processing to continue to the next filter.

If you forget it for a request path, you may stop the downstream chain.

---

# 16. What Happens When a Filter Rejects a Request?

Suppose authentication fails.

Conceptually:

```text
Request
  ↓
Security Filter
  ↓
Authentication failure
  ↓
Exception handling / authentication entry point
  ↓
HTTP response
```

If authorization fails:

```text
Authenticated user
      ↓
Authorization decision
      ↓
Denied
      ↓
AccessDeniedHandler
      ↓
HTTP response
```

The controller may never execute.

---

# 17. SecurityFilterChain and DispatcherServlet ⭐⭐⭐⭐⭐

This is frequently asked.

```text
HTTP Request
     ↓
Servlet Filters
     ↓
Spring Security Filter Chain
     ↓
DispatcherServlet
     ↓
Handler Mapping
     ↓
Controller
```

### Difference

| SecurityFilterChain | DispatcherServlet |
|---|---|
| Security processing | MVC request dispatching |
| Authentication | Finds controller/handler |
| Authorization | Handles MVC processing |
| CSRF/security headers | Calls controller |
| Security exceptions | MVC exceptions/handlers |

Security filtering normally happens before controller invocation.

---

# 18. `permitAll()` vs Security Chain ⭐⭐⭐⭐⭐

A common misconception is:

> “If I use `permitAll()`, Spring Security doesn't process the request.”

Not necessarily.

`permitAll()` is an **authorization rule** saying that the request is allowed without requiring authorization.

The request can still pass through the configured security infrastructure.

Example:

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/public/**").permitAll()
    .anyRequest().authenticated()
)
```

Think:

```text
Security processing
      ↓
Authorization rule
      ↓
permitAll → allow
```

Not:

```text
permitAll → bypass all security filters
```

---

# 19. `web.ignoring()` vs `permitAll()` ⚠️

These are not the same.

### `permitAll()`

Request security infrastructure ke through process ho sakti hai, but authorization allows access.

### `web.ignoring()`

Certain requests ko Spring Security filter chain se completely ignore karne ke liye use kiya ja sakta hai.

For normal application endpoints, `permitAll()` is generally the safer/default choice when you simply want public access while retaining security infrastructure.

Static resources may be a specific use case for ignoring, but the choice should be deliberate.

---

# 20. Security Context and Filter Chain ⭐⭐⭐⭐⭐

Security filters help establish or retrieve the security context during request processing.

Conceptually:

```text
Request
  ↓
Security Context handling
  ↓
Authentication mechanism
  ↓
Authentication
  ↓
SecurityContext
  ↓
Authorization
```

At the end of request processing, Spring Security manages the context lifecycle according to the configured execution model.

---

# 21. Exception Handling in the Chain ⭐⭐⭐⭐⭐

Security exceptions need to be translated into the correct response behavior.

Conceptual flow:

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

Result commonly maps to:

```text
Unauthenticated → 401
Forbidden        → 403
```

Exact response format depends on configuration.

---

# 22. Security Headers in the Chain

Spring Security can add security-related HTTP response headers.

Conceptual flow:

```text
Request
  ↓
Security processing
  ↓
Response
  ↓
Security headers
  ↓
Client
```

Examples of security-header concerns include:

```text
Content Security Policy
Frame protection
Content-Type options
Referrer policy
HSTS
```

Exact defaults depend on Spring Security version and configuration.

---

# 23. CSRF and the Filter Chain

For applications where CSRF protection is enabled:

```text
Request
  ↓
CSRF validation
  ↓
Valid → continue
Invalid → reject
```

CSRF protection is particularly important for browser-based applications using ambient credentials such as cookies.

For stateless bearer-token APIs, the threat model is different; do not simply copy `csrf().disable()` from a tutorial without understanding why it is appropriate.

---

# 24. CORS and Security Filter Chain

CORS is a browser cross-origin policy concern.

If the application needs CORS support, Spring Security should be configured consistently with the application's CORS configuration.

Conceptually:

```text
Browser
  ↓
Cross-origin request
  ↓
CORS processing
  ↓
Security processing
  ↓
Application
```

A correct CORS configuration is different from simply allowing every origin.

---

# 25. Debugging the Security Filter Chain ⭐⭐⭐⭐⭐

When debugging security problems, knowing the chain is extremely useful.

Useful questions:

```text
Which SecurityFilterChain matched?
Which filters are active?
Was authentication created?
Was SecurityContext populated?
Which authorization rule matched?
Why was request rejected?
```

Spring Security supports debug/logging facilities that can help inspect the configured security behavior.

For development/debugging, logs can reveal security filter processing and configuration details.

Avoid enabling overly verbose security logs permanently in production without considering sensitive information and operational cost.

---

# 26. Complete API Request Example ⭐⭐⭐⭐⭐

Suppose:

```http
GET /api/employees/10
Authorization: Bearer <JWT>
```

Flow:

```text
1. Client sends request
          ↓
2. Servlet container receives request
          ↓
3. DelegatingFilterProxy
          ↓
4. FilterChainProxy
          ↓
5. /api/** SecurityFilterChain selected
          ↓
6. Bearer token authentication
          ↓
7. JWT validated
          ↓
8. Authentication created
          ↓
9. SecurityContext updated
          ↓
10. Authorization rule evaluated
          ↓
11. Request allowed
          ↓
12. DispatcherServlet
          ↓
13. EmployeeController
          ↓
14. EmployeeService
          ↓
15. Repository
          ↓
16. Response
```

This is a strong architecture explanation for a 5+ years interview.

---

# 27. Common Interview Traps ⚠️

### Trap 1

> “SecurityFilterChain means one filter.”

❌ Wrong.

It is a chain of security filters.

### Trap 2

> “FilterChainProxy and SecurityFilterChain are the same.”

❌ Wrong.

```text
FilterChainProxy → selects/invokes chain
SecurityFilterChain → configured security filter chain
```

### Trap 3

> “permitAll() bypasses Spring Security.”

❌ Wrong.

It is an authorization decision, not a blanket filter bypass.

### Trap 4

> “Every JWT application needs a custom JWT filter.”

❌ Not necessarily.

Use built-in resource-server JWT support where appropriate.

### Trap 5

> “There is one fixed list of Spring Security filters.”

❌ Wrong.

The actual chain depends on enabled features and configuration.

### Trap 6

> “If Gateway authenticates, downstream services don't need security.”

❌ Unsafe assumption.

Security boundaries should be designed according to the architecture and trust model.

---

# 28. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“SecurityFilterChain is the core request-processing mechanism of Spring Security. A request enters the servlet container and is delegated through DelegatingFilterProxy to FilterChainProxy. FilterChainProxy selects the applicable SecurityFilterChain based on the configured matchers and ordering. That chain contains security filters responsible for tasks such as security-context handling, authentication, authorization, CSRF, logout, headers and other configured protections. For example, in a JWT resource server, a bearer-token authentication mechanism can validate the token, create an Authentication and associate it with the SecurityContext. Authorization rules then decide whether the request is allowed. If the request is rejected, Spring Security's exception-handling infrastructure produces the appropriate authentication or access-denied response. If security processing succeeds, the request continues to DispatcherServlet and eventually the controller. Multiple SecurityFilterChains can be configured when different URL groups need different security models.”**

---

# 29. 30-Second Hinglish Answer

> **“SecurityFilterChain Spring Security ki core request-processing chain hai. Request servlet container se DelegatingFilterProxy aur FilterChainProxy ke through matching SecurityFilterChain tak jaati hai. Is chain mein multiple security filters hote hain jo authentication, authorization, CSRF, security context aur other security concerns handle karte hain. Multiple chains bhi bana sakte hain, jaise `/api/**` ke liye JWT aur web pages ke liye form login. Security successful hone ke baad request DispatcherServlet aur controller tak jaati hai.”**

---

# 30. Whiteboard Memory Map

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
             ┌───────────┴───────────┐
             ↓                       ↓
        /api/** chain             /web/** chain
             ↓                       ↓
      Bearer/JWT flow            Form Login
             │                       │
             └───────────┬───────────┘
                         ↓
                  Security Filters
                         ↓
                  Authentication
                         ↓
                  SecurityContext
                         ↓
                    Authorization
                         ↓
              ┌──────────┴──────────┐
              ↓                     ↓
            Allow                  Deny
              ↓                     ↓
       DispatcherServlet       401 / 403 flow
              ↓
          Controller
```

### One-line memory

> **Request → FilterChainProxy → matching SecurityFilterChain → Security Filters → Authentication → SecurityContext → Authorization → Controller.**

---

# 31. Interview Follow-up Questions

1. What is SecurityFilterChain?
2. Difference between FilterChainProxy and SecurityFilterChain?
3. What is DelegatingFilterProxy?
4. How does Spring Security select a filter chain?
5. Can we configure multiple SecurityFilterChains?
6. Why does filter ordering matter?
7. How do you add a custom filter?
8. What is `OncePerRequestFilter`?
9. What happens if a filter doesn't call `filterChain.doFilter()`?
10. What is the difference between `permitAll()` and `web.ignoring()`?
11. Does `permitAll()` bypass all security filters?
12. How does JWT authentication fit into the chain?
13. How does CSRF protection fit into the filter chain?
14. Where does DispatcherServlet fit?
15. What happens when authentication fails?
16. What happens when authorization fails?
17. Why shouldn't every application implement its own JWT filter?
18. How would you debug a Spring Security filter-chain problem?
19. How would you configure API and web security differently?
20. Explain SecurityFilterChain on a whiteboard.

---

## Navigation

[← S4.1.3 — Spring Security Architecture](../03-Spring-Security-Architecture/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.5 — DelegatingFilterProxy**
