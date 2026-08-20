# S4.1.5 — DelegatingFilterProxy

> **Status:** ✅ Completed  
> **Level:** 5+ Years / Interview + Production

## 1. What is DelegatingFilterProxy? ⭐⭐⭐⭐⭐

`DelegatingFilterProxy` Spring Framework ka servlet `Filter` implementation hai jo **Servlet container ke filter lifecycle ko Spring-managed bean ke saath bridge** karta hai.

Simple words:

> **DelegatingFilterProxy khud business/security logic implement karne ke bajay request processing ko Spring ApplicationContext mein registered target bean ko delegate karta hai.**

High-level flow:

```text
HTTP Request
     ↓
Servlet Container
     ↓
DelegatingFilterProxy
     ↓
Spring ApplicationContext
     ↓
Target Spring-managed Filter
```

Spring Security ke case mein commonly:

```text
Servlet Container
      ↓
DelegatingFilterProxy
      ↓
FilterChainProxy
      ↓
SecurityFilterChain
      ↓
Security Filters
```

---

# 2. Why Do We Need It? ⭐⭐⭐⭐⭐

Servlet container aur Spring ApplicationContext alag management systems hain.

```text
Servlet Container
├── Servlet
├── Servlet Filter
└── Listener

Spring ApplicationContext
├── @Bean
├── @Component
├── Service
└── Security beans
```

Problem:

> Servlet container ko directly Spring-managed security infrastructure ka knowledge nahi hota.

`DelegatingFilterProxy` bridge provide karta hai.

```text
Servlet Filter API
       ↕
DelegatingFilterProxy
       ↕
Spring Bean
```

---

# 3. DelegatingFilterProxy vs Normal Filter ⭐⭐⭐⭐⭐

### Normal Servlet Filter

```java
public class MyFilter implements Filter {

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain) {

        // filtering logic
        chain.doFilter(request, response);
    }
}
```

Container is filter ko servlet filter infrastructure ke through manage kar sakta hai.

### DelegatingFilterProxy

Iska main purpose delegation hai:

```text
Container
   ↓
DelegatingFilterProxy
   ↓
Spring Bean
```

It lets the actual target filter be managed by Spring.

---

# 4. Spring Security mein iska Role ⭐⭐⭐⭐⭐

Spring Security architecture ko yaad rakho:

```text
Client
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

Yahan `DelegatingFilterProxy` aur `FilterChainProxy` ko confuse nahi karna.

### DelegatingFilterProxy

> **Container se Spring-managed filter infrastructure tak delegation.**

### FilterChainProxy

> **Applicable Spring Security filter chain ko select/invoke karta hai.**

Memory:

```text
DelegatingFilterProxy
       ↓
"Spring ke filter bean ko call karo"

FilterChainProxy
       ↓
"Kaunsi SecurityFilterChain apply hogi?"
```

---

# 5. Complete Spring Security Flow ⭐⭐⭐⭐⭐

Suppose client calls:

```http
GET /api/employees
Authorization: Bearer <JWT>
```

Flow:

```text
1. Client
       ↓
2. Servlet Container
       ↓
3. DelegatingFilterProxy
       ↓
4. FilterChainProxy
       ↓
5. Matching SecurityFilterChain
       ↓
6. Security Filters
       ↓
7. Authentication
       ↓
8. SecurityContext
       ↓
9. Authorization
       ↓
10. DispatcherServlet
       ↓
11. Controller
```

### Key point

`DelegatingFilterProxy` **authentication itself perform nahi karta**.

It is the bridge/delegation layer.

---

# 6. How DelegatingFilterProxy Finds the Target ⭐⭐⭐⭐⭐

Conceptually, `DelegatingFilterProxy` receives a target bean name and looks up the corresponding Spring-managed bean from the WebApplicationContext.

For example, the well-known Spring Security integration uses:

```text
springSecurityFilterChain
```

Conceptually:

```text
DelegatingFilterProxy
        ↓
Find target bean
        ↓
springSecurityFilterChain
        ↓
FilterChainProxy
```

The exact registration mechanism can differ depending on whether the application uses traditional servlet configuration or Spring Boot auto-configuration.

---

# 7. `springSecurityFilterChain` ⭐⭐⭐⭐⭐

Spring Security commonly exposes the security filter infrastructure under the bean name:

```text
springSecurityFilterChain
```

Conceptually:

```text
DelegatingFilterProxy
        ↓
"springSecurityFilterChain"
        ↓
FilterChainProxy
        ↓
SecurityFilterChain(s)
```

Do not interpret the bean name as meaning that it is itself one `SecurityFilterChain` object in every conceptual layer. The Spring Security infrastructure uses `FilterChainProxy` to manage the configured chains.

---

# 8. Delegation Concept ⭐⭐⭐⭐⭐

Imagine a receptionist:

```text
Client arrives
     ↓
Receptionist
     ↓
Find correct department
     ↓
Forward request
```

Similarly:

```text
HTTP Request
     ↓
DelegatingFilterProxy
     ↓
Spring-managed target Filter
```

It does not need to contain the complete business/security implementation.

---

# 9. Lifecycle Bridge

`DelegatingFilterProxy` also addresses an important lifecycle boundary.

```text
Servlet Container lifecycle
          ↓
DelegatingFilterProxy
          ↓
Spring ApplicationContext lifecycle
          ↓
Target Spring bean
```

The target object can therefore participate in Spring's dependency injection and bean management instead of being an isolated raw servlet filter.

### Important nuance

The exact lifecycle delegation behavior is configurable, and `DelegatingFilterProxy` has options related to target bean lookup and lifecycle handling. Do not simplify this into “Spring always controls the servlet Filter lifecycle exactly like every other bean.”

---

# 10. Dependency Injection Advantage ⭐⭐⭐⭐⭐

Without Spring-managed target filter:

```text
Filter
 ├── manually create dependency
 ├── manually configure dependency
 └── manually manage dependency
```

With Spring-managed target:

```text
DelegatingFilterProxy
       ↓
Spring Bean
       ↓
Dependency Injection
       ↓
Other Spring Beans
```

Example conceptual target filter:

```java
@Component
public class AuditFilter implements Filter {

    private final AuditService auditService;

    public AuditFilter(AuditService auditService) {
        this.auditService = auditService;
    }

    // filter logic
}
```

The target filter can use normal Spring dependency injection when it is a Spring-managed bean.

---

# 11. `DelegatingFilterProxy` Does Not Mean “All Filters Are Spring Beans”

Important interview nuance.

The proxy delegates to a **target Spring-managed Filter bean**.

It does not mean that every servlet filter in the entire application automatically becomes a Spring bean just because `DelegatingFilterProxy` exists.

Think:

```text
Container filter registration
        ↓
DelegatingFilterProxy
        ↓
One configured Spring target
```

---

# 12. Registration Concept ⭐⭐⭐⭐⭐

In traditional servlet applications, a filter can be registered using servlet-container mechanisms.

Conceptually:

```text
Servlet Container
       ↓
Filter registration
       ↓
DelegatingFilterProxy
       ↓
Spring target bean
```

Modern Spring Boot applications commonly rely on auto-configuration to integrate Spring Security with the servlet environment.

You generally don't need to manually register `DelegatingFilterProxy` in a standard Spring Boot application using Spring Security's normal setup.

---

# 13. Spring Boot Perspective ⭐⭐⭐⭐⭐

In Spring Boot + Spring Security, the integration is largely auto-configured.

Conceptually:

```text
Spring Boot
    ↓
Security auto-configuration
    ↓
Security infrastructure
    ↓
Servlet filter integration
    ↓
DelegatingFilterProxy / registered security filter
    ↓
FilterChainProxy
```

Therefore, as a developer, you usually focus on:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {
    // security rules
    return http.build();
}
```

rather than manually creating the entire servlet filter bridge.

---

# 14. DelegatingFilterProxy vs FilterChainProxy ⭐⭐⭐⭐⭐

| Aspect | DelegatingFilterProxy | FilterChainProxy |
|---|---|---|
| Main role | Bridge/delegation | Security filter-chain infrastructure |
| Layer | Servlet ↔ Spring | Spring Security |
| Knows target bean | Yes | Manages security chains |
| Selects SecurityFilterChain | No | Yes |
| Authentication | No | Invokes configured security filters that perform it |
| Typical Spring Security flow | First integration layer | Central security filter infrastructure |

### Memory trick

```text
DelegatingFilterProxy
= Bridge

FilterChainProxy
= Security chain manager
```

---

# 15. DelegatingFilterProxy vs DispatcherServlet ⭐⭐⭐⭐⭐

Another common interview question.

### DelegatingFilterProxy

Works in the **Servlet Filter layer**.

```text
Request
  ↓
Filter
  ↓
DelegatingFilterProxy
```

### DispatcherServlet

Works in the **Spring MVC Servlet layer** and dispatches requests to controllers/handlers.

```text
Security Filters
      ↓
DispatcherServlet
      ↓
Controller
```

Memory:

```text
Filter layer → DelegatingFilterProxy
MVC layer    → DispatcherServlet
```

---

# 16. What If Target Bean Is Missing? ⚠️

If `DelegatingFilterProxy` cannot resolve the configured target according to its configuration/lifecycle rules, request processing can fail with a configuration/runtime error.

This is one reason the target bean name and application context configuration must be correct.

In standard Spring Security Boot setup, this infrastructure is normally auto-configured, so manually reproducing it is rarely necessary.

---

# 17. Lazy Target Lookup ⭐⭐⭐⭐

`DelegatingFilterProxy` supports target-bean lookup behavior that can defer resolving the target until it is needed, depending on configuration.

Why useful?

```text
Servlet container starts
        ↓
Spring context may still initialize
        ↓
Target lookup can be resolved according to proxy configuration
```

Interview answer:

> **DelegatingFilterProxy supports target bean lookup and can defer target resolution depending on its configuration.**

Avoid claiming that every application always uses lazy lookup.

---

# 18. Request Delegation ⭐⭐⭐⭐⭐

At request time, the proxy delegates filtering to the target.

Conceptually:

```text
public void doFilter(...)
        ↓
Find target Filter
        ↓
Target Filter.doFilter(...)
```

Therefore:

```text
DelegatingFilterProxy
        ↓
Target Filter
        ↓
Next filter
```

The target is responsible for continuing the chain where appropriate.

---

# 19. Exception Flow

Suppose Spring Security rejects a request.

```text
Request
   ↓
DelegatingFilterProxy
   ↓
FilterChainProxy
   ↓
Security Filter
   ↓
Security exception
   ↓
Security exception handling
   ↓
HTTP response
```

The proxy itself is not the component deciding whether a JWT is valid or whether the user has a role.

---

# 20. Common Misconceptions ⚠️

### Misconception 1

> “DelegatingFilterProxy is the security filter.”

⚠️ Incomplete.

It is the proxy/bridge that delegates to the target Spring-managed filter infrastructure.

### Misconception 2

> “DelegatingFilterProxy authenticates users.”

❌ No.

Authentication is handled by configured Spring Security authentication mechanisms/providers/filters.

### Misconception 3

> “DelegatingFilterProxy and FilterChainProxy are the same.”

❌ No.

```text
DelegatingFilterProxy → delegation bridge
FilterChainProxy      → Spring Security filter-chain infrastructure
```

### Misconception 4

> “We always need to manually configure DelegatingFilterProxy in Spring Boot.”

❌ Usually no.

Standard Spring Boot + Spring Security setup provides the required integration automatically.

### Misconception 5

> “It delegates directly to the controller.”

❌ No.

It delegates to the configured target filter, commonly the Spring Security filter infrastructure.

---

# 21. Production Example ⭐⭐⭐⭐⭐

Suppose we have:

```text
Spring Boot REST API
       ↓
Spring Security
       ↓
JWT Resource Server
```

Request:

```http
GET /api/payment/123
Authorization: Bearer <JWT>
```

Flow:

```text
Client
  ↓
Servlet Container
  ↓
DelegatingFilterProxy
  ↓
FilterChainProxy
  ↓
API SecurityFilterChain
  ↓
Bearer Token Authentication
  ↓
JWT validation
  ↓
SecurityContext
  ↓
Authorization
  ↓
PaymentController
```

The key role of `DelegatingFilterProxy` is the bridge at the beginning of the Spring Security filter processing path.

---

# 22. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“DelegatingFilterProxy is a Spring-provided servlet Filter that acts as a bridge between the servlet container and a Spring-managed Filter bean. Instead of putting the complete security logic inside the container-managed filter, it delegates the request to a target bean in the Spring ApplicationContext. In Spring Security, this commonly leads to the `FilterChainProxy`, which manages the configured `SecurityFilterChain` instances. So the high-level flow is Servlet Container → DelegatingFilterProxy → FilterChainProxy → SecurityFilterChain → security filters → DispatcherServlet → Controller. The important distinction is that DelegatingFilterProxy does not itself authenticate the user or select authorization rules; its main responsibility is delegation between the servlet filter environment and Spring-managed security infrastructure.”**

---

# 23. 30-Second Hinglish Answer

> **“DelegatingFilterProxy Spring aur Servlet Container ke beech bridge hai. Servlet container request ko DelegatingFilterProxy ko deta hai, aur ye request ko Spring ApplicationContext ke target filter bean ko delegate karta hai. Spring Security mein target commonly FilterChainProxy hota hai, jo matching SecurityFilterChain ko execute karta hai. Isliye flow hai: Container → DelegatingFilterProxy → FilterChainProxy → SecurityFilterChain → Security Filters → DispatcherServlet → Controller. DelegatingFilterProxy khud authentication nahi karta.”**

---

# 24. Whiteboard Memory Map ⭐⭐⭐⭐⭐

```text
                    HTTP REQUEST
                         │
                         ↓
                 Servlet Container
                         │
                         ↓
              DelegatingFilterProxy
                  (Bridge Layer)
                         │
                         ↓
                  FilterChainProxy
                (Security Infrastructure)
                         │
                         ↓
               SecurityFilterChain
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
       Authentication          Authorization
              │                     │
              └──────────┬──────────┘
                         ↓
                  SecurityContext
                         │
                         ↓
                  DispatcherServlet
                         │
                         ↓
                     Controller
```

### One-line memory

> **DelegatingFilterProxy = Servlet Container → Spring Security bridge.**

---

# 25. Interview Follow-up Questions

1. What is DelegatingFilterProxy?
2. Why is DelegatingFilterProxy needed?
3. How does DelegatingFilterProxy work internally?
4. What is the target bean in Spring Security?
5. What is `springSecurityFilterChain`?
6. Difference between DelegatingFilterProxy and FilterChainProxy?
7. Difference between DelegatingFilterProxy and DispatcherServlet?
8. Does DelegatingFilterProxy authenticate the user?
9. How does DelegatingFilterProxy support dependency injection?
10. How is DelegatingFilterProxy registered in Spring Boot?
11. Do we manually configure it in a normal Spring Boot application?
12. What happens if the target bean cannot be found?
13. What is lazy target lookup?
14. How does a request flow through DelegatingFilterProxy?
15. Where does SecurityFilterChain fit after DelegatingFilterProxy?
16. Explain DelegatingFilterProxy on a whiteboard.
17. How does it bridge the servlet and Spring lifecycles?
18. What is the role of `FilterChainProxy` after delegation?
19. How would you debug a DelegatingFilterProxy configuration problem?
20. Explain the complete request flow from browser to controller.

---

## Navigation

[← S4.1.4 — Security Filter Chain](../04-Security-Filter-Chain/README.md)

[🏠 Spring Security — S4](../README.md)

[🏠 Master README](../../../README.md)

**Next source sequence → S4.1.6 — Authentication vs Authorization**
