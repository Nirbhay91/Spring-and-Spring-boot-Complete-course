# Spring S1.2 — Container Extension Points

## 2.1 Bean Scope and Lifecycle

> **Status:** ✅ Completed

## 1. What is Bean Scope?

Bean scope defines **how many instances of a Spring Bean the IoC container creates and how long those instances are associated with the container/context**.

Simple:

```text
Bean Scope = Bean ki instance-creation / lifetime strategy
```

Most common scopes:

- Singleton
- Prototype
- Request
- Session
- Application
- WebSocket

The last four are web-aware scopes.

---

## 2. Singleton Scope ⭐

Singleton is the default Spring Bean scope.

```java
@Component
public class PaymentService {
}
```

Conceptually:

```text
ApplicationContext
      │
      └── PaymentService
              │
              └── one container-managed instance
```

If the same Bean is requested repeatedly from the same Spring container, the container returns the same singleton instance.

```java
PaymentService p1 = context.getBean(PaymentService.class);
PaymentService p2 = context.getBean(PaymentService.class);

System.out.println(p1 == p2); // true
```

### Interview trap

> Spring singleton does **not** mean one object for the entire JVM or one object across all application instances.

It means one instance **per Spring IoC container**.

If you have two independent application contexts, they can each have their own singleton instance.

---

## 3. Prototype Scope

Prototype creates a new Bean instance whenever the container is asked for that Bean.

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
| Default? | Yes | No |
| Instances | One per container | New instance per lookup/injection semantics |
| Typical use | Stateless services, repositories | Stateful/short-lived objects |
| Container creates object? | Yes | Yes |
| Container manages complete lifecycle? | Yes | Creation + initialization; destruction callbacks are not automatically invoked by container |

### Memory trick

```text
Singleton  → Same
Prototype  → New
```

---

## 5. How to Configure Scope

### Annotation

```java
@Scope("prototype")
@Component
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

## 6. Bean Lifecycle ⭐

Bean lifecycle means the sequence a Spring-managed Bean goes through from creation to destruction.

Simplified lifecycle:

```text
Bean Definition
      ↓
Instantiate Bean
      ↓
Populate Dependencies
      ↓
Aware callbacks
      ↓
BeanPostProcessor (before initialization)
      ↓
Initialization callbacks
      ↓
BeanPostProcessor (after initialization)
      ↓
Bean Ready for use
      ↓
ApplicationContext shutdown
      ↓
Destruction callbacks
```

This is the core mental model for interviews.

---

## 7. Bean Instantiation

Spring first creates the Bean instance according to its Bean definition.

For example:

```java
@Component
public class PaymentService {
}
```

The container manages construction instead of application code manually controlling the lifecycle.

---

## 8. Dependency Injection / Population

After instantiation, Spring resolves and injects dependencies.

```java
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Conceptually:

```text
Create OrderService
       ↓
Resolve PaymentService
       ↓
Inject PaymentService
```

---

## 9. Aware Interfaces

Spring can provide framework-level information to a Bean through `Aware` interfaces.

Examples:

- `BeanNameAware`
- `BeanFactoryAware`
- `ApplicationContextAware`
- `EnvironmentAware`
- `ResourceLoaderAware`

Example:

```java
@Component
public class MyBean implements ApplicationContextAware {

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext context) {
        this.context = context;
    }
}
```

### Interview point

Aware interfaces couple a Bean to Spring infrastructure. Use them only when that framework-level access is genuinely required.

---

## 10. BeanPostProcessor ⭐

`BeanPostProcessor` allows Spring or application code to process a Bean **before and after initialization**.

Methods:

```java
postProcessBeforeInitialization()
postProcessAfterInitialization()
```

Simplified:

```text
Bean created
   ↓
Dependencies injected
   ↓
BeforeInitialization
   ↓
Initialization
   ↓
AfterInitialization
   ↓
Ready Bean
```

Many Spring features rely on post-processing internally.

---

## 11. Initialization Callbacks

Spring supports several initialization mechanisms.

### `@PostConstruct`

```java
@PostConstruct
public void init() {
    System.out.println("Bean initialized");
}
```

Modern Spring applications commonly use Jakarta's annotation:

```java
import jakarta.annotation.PostConstruct;
```

### `InitializingBean`

```java
public class PaymentService implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // initialization logic
    }
}
```

### Custom `initMethod`

```java
@Bean(initMethod = "init")
public PaymentService paymentService() {
    return new PaymentService();
}
```

---

## 12. Destruction Callbacks

When the ApplicationContext shuts down, singleton Beans can receive destruction callbacks.

### `@PreDestroy`

```java
@PreDestroy
public void cleanup() {
    // cleanup
}
```

Modern Spring applications commonly use:

```java
import jakarta.annotation.PreDestroy;
```

### `DisposableBean`

```java
public class PaymentService implements DisposableBean {

    @Override
    public void destroy() {
        // cleanup
    }
}
```

### Custom `destroyMethod`

```java
@Bean(destroyMethod = "cleanup")
public PaymentService paymentService() {
    return new PaymentService();
}
```

---

## 13. Complete Lifecycle Example

```java
@Component
public class PaymentService {

    @PostConstruct
    public void init() {
        System.out.println("init");
    }

    public void pay() {
        System.out.println("pay");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("cleanup");
    }
}
```

Simplified sequence:

```text
Constructor
   ↓
Dependency Injection
   ↓
@PostConstruct
   ↓
BeanPostProcessor after initialization
   ↓
Bean available
   ↓
Context shutdown
   ↓
@PreDestroy
```

---

## 14. Important Ordering Concept ⭐

At interview level, remember:

```text
Instantiation
   ↓
Populate properties / Dependency Injection
   ↓
Aware callbacks
   ↓
BeanPostProcessor before initialization
   ↓
@PostConstruct / initialization callbacks
   ↓
BeanPostProcessor after initialization
   ↓
Bean ready
```

Exact internal details can vary by callback mechanism and Spring infrastructure, so avoid claiming that every Bean follows one simplistic callback list without qualification.

---

## 15. Singleton Destruction vs Prototype Destruction ⭐

Important interview question.

### Singleton

Spring manages the full lifecycle including destruction callbacks when the context is closed normally.

### Prototype

Spring creates and initializes the object, but it does **not** automatically invoke destruction callbacks for prototype Beans.

Why?

Because after handing the prototype instance to the client, the container does not keep full lifecycle ownership of that instance in the same way it does for singleton Beans.

Interview answer:

> **“Spring manages creation and initialization of prototype Beans, but it does not automatically manage their destruction callbacks.”**

---

## 16. What Happens if a Prototype is Injected into a Singleton?

Example:

```java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

```java
@Component
public class ReportService {

    private final ReportGenerator generator;

    public ReportService(ReportGenerator generator) {
        this.generator = generator;
    }
}
```

If `ReportService` is singleton, its dependency is normally resolved when the singleton is created.

That means the singleton can keep the same injected prototype instance rather than receiving a new prototype every time a method is called.

### Need a fresh instance each time?

Use mechanisms such as:

- `ObjectProvider<T>`
- `ObjectFactory<T>`
- `Provider<T>`
- `@Lookup`
- scoped proxy where appropriate

Example:

```java
@Component
public class ReportService {

    private final ObjectProvider<ReportGenerator> provider;

    public ReportService(ObjectProvider<ReportGenerator> provider) {
        this.provider = provider;
    }

    public ReportGenerator createReportGenerator() {
        return provider.getObject();
    }
}
```

---

## 17. Why Singleton Beans Should Usually Be Stateless

Singleton means one shared instance per container.

If a singleton stores request-specific mutable state:

```java
@Component
public class OrderService {

    private String currentUser;
}
```

multiple requests/threads can interact with the same object.

Better:

```java
public OrderResponse createOrder(String userId) {
    // use local/request data
}
```

### Rule

> **Singleton + shared mutable state = concurrency risk.**

Keep singleton services generally stateless and thread-safe.

---

## 18. Web Scopes

Spring web applications support additional scopes when a suitable web-aware ApplicationContext is used.

### Request scope

One Bean instance per HTTP request.

```java
@RequestScope
@Component
public class RequestContext {
}
```

### Session scope

One Bean instance per HTTP session.

```java
@SessionScope
@Component
public class UserSession {
}
```

### Application scope

One Bean instance associated with the web application's `ServletContext`.

### WebSocket scope

One Bean instance associated with a WebSocket lifecycle.

---

## 19. Scope Comparison

| Scope | Instance concept | Typical use |
|---|---|---|
| Singleton | One per container | Stateless service, repository |
| Prototype | New per container lookup | Stateful short-lived object |
| Request | One per HTTP request | Request-specific state |
| Session | One per HTTP session | User session state |
| Application | One per web application context/ServletContext semantics | Application-level web state |
| WebSocket | One per WebSocket | WebSocket-specific state |

---

## 20. Thread Safety and Scope

Scope does not automatically make a Bean thread-safe.

For example:

```text
Singleton
  ↓
Potentially shared by many request threads
```

Therefore:

- Avoid mutable shared fields.
- Prefer local variables.
- Use thread-safe data structures when shared state is necessary.
- Don't assume `@Scope("prototype")` automatically solves all concurrency problems.

---

## 21. Lifecycle vs Scope ⭐

These concepts are related but different.

### Scope

Answers:

> **How many instances / what lifecycle boundary does this Bean have?**

### Lifecycle

Answers:

> **What stages does a particular Bean instance go through from creation to destruction?**

Memory:

```text
Scope    → How many / lifetime boundary?
Lifecycle → What happens during that lifetime?
```

---

## 22. Real-World Example

### PaymentService

```text
PaymentService
      ↓
Stateless business logic
      ↓
Singleton is usually appropriate
```

### RequestContext

```text
HTTP Request
      ↓
Request-scoped object
      ↓
Destroyed with request scope
```

### ReportGenerator

If it contains per-operation mutable state:

```text
Operation
   ↓
Fresh ReportGenerator
   ↓
Complete
```

Prototype or another explicitly designed creation mechanism may be appropriate.

---

## 23. Common Interview Traps

### Trap 1
**“Singleton means one object in the JVM.”**

❌ Not exactly.

✅ One instance per Spring container.

### Trap 2
**“Prototype gets destroyed automatically by Spring.”**

❌ Not in the same way as singleton destruction.

### Trap 3
**“Prototype injected into singleton gives a new object on every method call.”**

❌ Not by ordinary injection.

### Trap 4
**“Scope makes the Bean thread-safe.”**

❌ No.

### Trap 5
**“`@PostConstruct` is the only initialization mechanism.”**

❌ Spring supports multiple lifecycle mechanisms.

---

## 24. Interview Follow-Up Questions

1. What is Bean scope in Spring?
2. What is the default Spring Bean scope?
3. What does singleton scope mean?
4. Is Spring singleton one object per JVM?
5. Singleton vs prototype?
6. How do you configure prototype scope?
7. What happens when a prototype Bean is injected into a singleton?
8. How can you obtain a fresh prototype instance from a singleton?
9. What is `ObjectProvider`?
10. What is Bean lifecycle?
11. Explain the Spring Bean lifecycle.
12. When does dependency injection happen?
13. What is `BeanPostProcessor`?
14. What is the difference between before and after initialization processing?
15. What is `@PostConstruct`?
16. What is `InitializingBean`?
17. What is `initMethod`?
18. What is `@PreDestroy`?
19. What is `DisposableBean`?
20. What is `destroyMethod`?
21. Are prototype destruction callbacks automatically called?
22. What are Aware interfaces?
23. What is request scope?
24. What is session scope?
25. What is application scope?
26. What is WebSocket scope?
27. Scope vs lifecycle?
28. Why should singleton services generally be stateless?
29. Does scope guarantee thread safety?
30. Explain Bean lifecycle in 2 minutes.

---

## 25. 2-Minute Interview Answer ⭐

> **“Spring Bean scope defines how Spring manages the instances of a Bean. Singleton is the default scope, meaning one instance per Spring IoC container. Prototype creates a new instance whenever the container is asked for that Bean. In web applications, Spring also provides request, session, application and WebSocket scopes. Bean lifecycle describes the stages from instantiation and dependency injection through initialization and post-processing to destruction. During lifecycle processing, Spring can invoke Aware callbacks, BeanPostProcessors, `@PostConstruct` or other initialization callbacks, and destruction callbacks such as `@PreDestroy` for eligible Beans. An important point is that Spring does not automatically manage destruction callbacks for prototype Beans. Also, a prototype injected into a singleton does not automatically become a new instance on every method call; if we need that behavior, we can use `ObjectProvider`, `ObjectFactory`, `Provider`, `@Lookup`, or an appropriate scoped proxy.”**

---

## 26. 30-Second Hinglish Answer

> **“Bean scope decide karta hai ki Spring Bean ki instances kaise manage karega. Singleton default hai aur same Spring container mein ek instance hota hai, jabki prototype mein container se lookup par new instance milta hai. Web applications mein request aur session jaise scopes bhi hote hain. Bean lifecycle mein creation, dependency injection, initialization, post-processing aur destruction phases aate hain. Important point: prototype Bean ke destruction callbacks Spring automatically manage nahi karta.”**

---

## 🧠 Memory Trick

```text
SCOPE
 ↓
How many / kis boundary tak instance?

LIFECYCLE
 ↓
Create → Inject → Initialize → Use → Destroy
```

### One-line memory

> **“Scope tells Spring how the Bean instance is managed; lifecycle tells what happens to that instance.”**

---

## Navigation

[← S1.2 Container Extension Points](../README.md)

[Next → S1.2.2 Singleton, Prototype, and Other Scopes](../02-Singleton-Prototype-and-Other-Scopes/README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** S1.2.2 — Singleton, Prototype, and Other Scopes
