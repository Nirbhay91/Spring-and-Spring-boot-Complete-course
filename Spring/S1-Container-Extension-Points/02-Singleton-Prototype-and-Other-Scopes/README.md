# Spring S1.2 — Container Extension Points

## 2.2 Singleton, Prototype, and Other Scopes

> **Status:** ✅ Completed

## 1. Bean Scope

Bean scope defines how Spring manages the **instances and lifecycle boundary** of a Bean.

Main scopes:

```text
Singleton   → one instance per Spring container
Prototype   → new instance on container lookup
Request     → one instance per HTTP request
Session     → one instance per HTTP session
Application → web application scope
WebSocket   → WebSocket scope
```

---

## 2. Singleton Scope ⭐

Singleton is the **default Spring Bean scope**.

```java
@Component
public class PaymentService {
}
```

```java
PaymentService p1 = context.getBean(PaymentService.class);
PaymentService p2 = context.getBean(PaymentService.class);

System.out.println(p1 == p2); // true
```

### Important

Spring singleton means:

> **One instance per Spring IoC container.**

It does NOT mean one instance for the complete JVM.

If two independent ApplicationContexts exist, each context can have its own singleton instance.

```text
Context A → PaymentService A
Context B → PaymentService B
```

---

## 3. Prototype Scope

Prototype creates a new instance whenever the container is asked for the Bean.

```java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

```java
ReportGenerator r1 = context.getBean(ReportGenerator.class);
ReportGenerator r2 = context.getBean(ReportGenerator.class);

System.out.println(r1 == r2); // false
```

Mental model:

```text
getBean() #1 → Object A
getBean() #2 → Object B
getBean() #3 → Object C
```

---

## 4. Singleton vs Prototype ⭐

| Feature | Singleton | Prototype |
|---|---|---|
| Default | Yes | No |
| Instances | One per container | New per container lookup |
| Typical use | Stateless services/repositories | Stateful/short-lived objects |
| Creation | Managed by Spring | Managed by Spring at creation |
| Destruction | Spring manages eligible destruction callbacks | Container does not automatically invoke destruction callbacks |

Memory:

```text
Singleton → Same
Prototype → New
```

---

## 5. Other Spring Scopes

### Request Scope

One Bean instance for each HTTP request.

```java
@RequestScope
@Component
public class RequestContext {
}
```

Conceptually:

```text
HTTP Request 1 → RequestContext A
HTTP Request 2 → RequestContext B
```

Useful for request-specific state.

---

### Session Scope

One Bean instance per HTTP session.

```java
@SessionScope
@Component
public class UserSession {
}
```

Useful for session-specific state.

---

### Application Scope

One Bean instance associated with the web application's `ServletContext`.

```java
@ApplicationScope
@Component
public class ApplicationState {
}
```

Important distinction:

> Application scope is a web-aware scope; it is not simply another name for ordinary Spring singleton semantics.

---

### WebSocket Scope

One Bean instance associated with a WebSocket lifecycle.

```text
WebSocket connection
        ↓
WebSocket-scoped Bean
```

Useful when state belongs to a WebSocket interaction.

---

## 6. Scope Configuration

### `@Scope`

```java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

### `@Bean`

```java
@Bean
@Scope("prototype")
public ReportGenerator reportGenerator() {
    return new ReportGenerator();
}
```

### XML

```xml
<bean id="reportGenerator"
      class="com.example.ReportGenerator"
      scope="prototype"/>
```

---

## 7. `@RequestScope`, `@SessionScope`, etc.

Spring provides convenient annotations for web scopes:

```java
@RequestScope
@Component
class RequestContext {}
```

```java
@SessionScope
@Component
class UserSession {}
```

These are more expressive than manually writing the corresponding scope string.

---

## 8. Prototype Injected into Singleton ⭐

This is one of the most common interview traps.

```java
@Component
@Scope("prototype")
class ReportGenerator {
}
```

```java
@Component
class ReportService {

    private final ReportGenerator generator;

    ReportService(ReportGenerator generator) {
        this.generator = generator;
    }
}
```

A singleton `ReportService` normally receives its dependency when the singleton is created.

Therefore:

```text
Singleton ReportService
        ↓
one injected ReportGenerator instance
```

It does **not** automatically get a new prototype instance on every method call.

---

## 9. How to Get a Fresh Prototype from a Singleton

Use `ObjectProvider`:

```java
@Component
class ReportService {

    private final ObjectProvider<ReportGenerator> provider;

    ReportService(ObjectProvider<ReportGenerator> provider) {
        this.provider = provider;
    }

    ReportGenerator createGenerator() {
        return provider.getObject();
    }
}
```

Now each call can request a new prototype instance:

```text
createGenerator()
       ↓
provider.getObject()
       ↓
new prototype instance
```

Other options include:

- `ObjectFactory`
- `jakarta.inject.Provider`
- `@Lookup`
- Scoped proxy, where appropriate

---

## 10. `@Lookup`

Spring can override a lookup method to obtain a Bean from the container.

```java
@Component
abstract class ReportService {

    public void generate() {
        ReportGenerator generator = getGenerator();
        // use generator
    }

    @Lookup
    protected abstract ReportGenerator getGenerator();
}
```

This is useful when a singleton needs a fresh shorter-lived Bean.

---

## 11. Scoped Proxy ⭐

A shorter-lived scoped Bean can be injected into a longer-lived Bean using a scoped proxy when the required web scope/proxy infrastructure is configured.

Conceptually:

```text
Singleton
   ↓
Proxy
   ↓
Current request/session scoped object
```

The proxy resolves the actual scoped target according to the active scope.

This is especially useful for request/session scoped Beans used by singleton services.

---

## 12. Why Singleton Services Are Usually Stateless

Singleton Beans are shared.

Bad design:

```java
@Component
class OrderService {
    private String currentUser;
}
```

Multiple threads can access the same instance.

Better:

```java
public OrderResponse createOrder(String userId) {
    // request-specific data stays local
}
```

Rule:

> **Singleton + shared mutable state = potential concurrency problem.**

Scope does not automatically make an object thread-safe.

---

## 13. Prototype Does Not Mean Thread-Safe

Prototype gives separate instances to separate container lookups, but that does not automatically make each instance thread-safe.

Example:

```text
Thread A → Prototype Object A
Thread B → Prototype Object B
```

If Thread A and Thread B share Object A themselves, concurrency issues can still occur.

Thread safety depends on object design, not merely scope.

---

## 14. Prototype Destruction ⭐

Spring manages creation and initialization of prototype Beans, but it does not automatically invoke destruction callbacks for prototype instances when the context shuts down.

Interview answer:

> **“Spring manages the creation and initialization of prototype Beans, but the container does not automatically manage their destruction callbacks.”**

If a prototype owns an external resource, application code needs an appropriate cleanup strategy.

---

## 15. Singleton Destruction

For singleton Beans managed by the ApplicationContext, Spring can invoke destruction callbacks when the context is closed normally.

Examples:

```java
@PreDestroy
public void cleanup() {
}
```

or:

```java
@Bean(destroyMethod = "cleanup")
public ResourceManager resourceManager() {
    return new ResourceManager();
}
```

---

## 16. Scope and Lifecycle Difference

### Scope

Answers:

> How many instances and what scope boundary?

### Lifecycle

Answers:

> What happens to a particular Bean instance during creation, initialization, use and destruction?

Memory:

```text
Scope    → How many / which boundary?
Lifecycle → What happens during lifetime?
```

---

## 17. Scope Selection in Real Projects

### Stateless Service

```text
PaymentService
OrderService
UserService
        ↓
Singleton
```

### Per-operation state

```text
ReportGenerator
        ↓
Prototype or explicit factory/provider
```

### Per HTTP request

```text
RequestContext
        ↓
Request scope
```

### Per user session

```text
ShoppingCartSession
        ↓
Session scope
```

### WebSocket interaction

```text
ChatSocketState
        ↓
WebSocket scope
```

---

## 18. Interview Comparison Table

| Scope | Boundary | Typical use | Web-aware? |
|---|---|---|---|
| Singleton | Spring container | Stateless shared service | No |
| Prototype | Container lookup | New short-lived instance | No |
| Request | HTTP request | Request-specific state | Yes |
| Session | HTTP session | Session state | Yes |
| Application | Web application | Web application state | Yes |
| WebSocket | WebSocket lifecycle | Socket-specific state | Yes |

---

## 19. Common Interview Traps ⭐

### Trap 1
**Singleton = one object in JVM.**

❌ Wrong.

✅ One instance per Spring container.

### Trap 2
**Prototype means a new object on every method call.**

❌ Wrong.

✅ New instance on container lookup; ordinary injected reference does not magically change per method call.

### Trap 3
**Prototype injected into singleton remains prototype dynamically.**

❌ Not automatically.

✅ The singleton normally keeps the resolved dependency reference.

### Trap 4
**Scope makes Bean thread-safe.**

❌ Wrong.

### Trap 5
**Application scope is exactly the same as singleton.**

❌ Don't say this.

They have different semantics and boundaries.

---

## 20. Interview Follow-Up Questions

1. What are Spring Bean scopes?
2. What is the default scope?
3. What exactly does singleton mean in Spring?
4. Is a Spring singleton one object per JVM?
5. What is prototype scope?
6. Singleton vs prototype?
7. What happens if prototype is injected into singleton?
8. How can a singleton obtain a fresh prototype instance?
9. What is `ObjectProvider`?
10. What is `ObjectFactory`?
11. What is `Provider`?
12. What is `@Lookup`?
13. What is scoped proxy?
14. What is request scope?
15. What is session scope?
16. What is application scope?
17. What is WebSocket scope?
18. How do you configure a Bean scope?
19. `@Scope` vs `@RequestScope`?
20. Does scope guarantee thread safety?
21. Why should singleton services generally be stateless?
22. Are prototype destruction callbacks automatically called?
23. Singleton scope vs application scope?
24. How do you choose a Bean scope in production?
25. Explain Spring Bean scopes in 2 minutes.

---

## 21. 2-Minute Interview Answer ⭐

> **“Spring supports multiple Bean scopes. Singleton is the default and means one instance per Spring IoC container. Prototype creates a new instance whenever the container is asked for that Bean. For web applications, Spring also provides request, session, application and WebSocket scopes. A common interview case is a prototype Bean injected into a singleton. The singleton normally receives the prototype instance when the singleton is created, so it does not get a new instance on every method call. If a fresh instance is required, we can use `ObjectProvider`, `ObjectFactory`, `Provider`, `@Lookup`, or an appropriate scoped proxy. Also, scope does not make a Bean thread-safe. Singleton services should generally be stateless, and Spring does not automatically invoke destruction callbacks for prototype Beans.”**

---

## 22. 30-Second Hinglish Answer

> **“Spring mein Bean scope decide karta hai ki Bean instances kis boundary ke according manage hongi. Singleton default hai aur ek Spring container mein ek instance hota hai. Prototype mein container se lookup par new instance milta hai. Web applications mein request, session, application aur WebSocket scopes bhi available hain. Important interview point ye hai ki prototype ko singleton mein inject karne se har method call par new object nahi milta. Fresh object ke liye `ObjectProvider`, `@Lookup` ya suitable scoped proxy use kar sakte hain.”**

---

## 🧠 Memory Trick

```text
Singleton  → SAME
Prototype  → NEW
Request    → HTTP REQUEST
Session    → USER SESSION
Application→ WEB APP
WebSocket  → SOCKET
```

### One-line memory

> **“Singleton shares, Prototype creates, Request follows request, Session follows user session.”**

---

## Navigation

[← S1.2.1 Bean Scope and Lifecycle](../01-Bean-Scope-and-Lifecycle/README.md)

[↗ S1.2 Container Extension Points](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** S1.2.3 — Configuring Scope
