# S1.2.4 — Bean Lifecycle / Callbacks

> **Status:** ✅ Completed

## 1. What is Bean Lifecycle?

Spring Bean lifecycle means the complete journey of a Bean managed by the IoC Container:

```text
Bean Definition
      ↓
Instantiation
      ↓
Dependency Injection
      ↓
Aware callbacks
      ↓
BeanPostProcessor — before initialization
      ↓
Initialization callbacks
      ↓
BeanPostProcessor — after initialization
      ↓
Bean is Ready
      ↓
Container shutdown
      ↓
Destruction callbacks
```

### Interview definition

> **Bean lifecycle describes how Spring creates, configures, initializes, uses and finally destroys a Bean.**

---

## 2. Why Bean Lifecycle Matters?

Lifecycle callbacks are useful when a Bean needs initialization or cleanup logic.

Examples:

```text
Initialization
→ open connection
→ load configuration
→ initialize cache
→ validate required state

Destruction
→ close resource
→ flush buffer
→ release resource
```

Business logic ko lifecycle callbacks mein unnecessarily nahi rakhna chahiye.

---

## 3. Basic Lifecycle Flow ⭐

Simplified interview flow:

```text
1. Bean Definition loaded
2. Bean instantiated
3. Dependencies injected
4. Aware callbacks
5. BeanPostProcessor.beforeInitialization()
6. @PostConstruct
7. InitializingBean.afterPropertiesSet()
8. Custom init method
9. BeanPostProcessor.afterInitialization()
10. Bean ready
11. Context close
12. @PreDestroy
13. DisposableBean.destroy()
14. Custom destroy method
```

> **Important:** Exact internal processing has additional container steps; this is the practical interview-oriented lifecycle order.

---

## 4. Bean Instantiation

Spring first creates the Bean instance.

Example:

```java
@Component
public class PaymentService {
}
```

Conceptually:

```text
new PaymentService()
```

Spring is responsible for creating the managed instance rather than application code manually constructing the dependency.

---

## 5. Dependency Injection

After instantiation, Spring populates the Bean's dependencies.

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
Inject dependency
```

---

## 6. Aware Interfaces

Spring can provide container-related information through `Aware` callbacks.

Examples:

```java
BeanNameAware
BeanFactoryAware
ApplicationContextAware
EnvironmentAware
```

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

`Aware` interfaces are useful when a Bean genuinely needs access to Spring infrastructure, but overusing them can increase coupling to Spring.

---

## 7. `BeanPostProcessor` ⭐⭐⭐

`BeanPostProcessor` is one of the most important Spring lifecycle extension points.

It allows custom processing **before and after initialization** of Beans.

```java
public interface BeanPostProcessor {

    Object postProcessBeforeInitialization(Object bean, String beanName);

    Object postProcessAfterInitialization(Object bean, String beanName);
}
```

Conceptual flow:

```text
Bean created
    ↓
Dependencies injected
    ↓
Before Initialization
    ↓
Initialization callbacks
    ↓
After Initialization
    ↓
Bean ready
```

---

## 8. Example — BeanPostProcessor

```java
@Component
public class LoggingBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(
            Object bean, String beanName) {
        System.out.println("Before: " + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(
            Object bean, String beanName) {
        System.out.println("After: " + beanName);
        return bean;
    }
}
```

It can inspect or wrap Beans as part of container processing.

### Important

A `BeanPostProcessor` can return the same Bean or another object, including a proxy.

This is a key mechanism behind several Spring features.

---

## 9. `@PostConstruct` ⭐

Modern Spring applications commonly use Jakarta's `@PostConstruct`.

```java
import jakarta.annotation.PostConstruct;

@Component
public class CacheService {

    @PostConstruct
    public void init() {
        System.out.println("Cache initialized");
    }
}
```

It is called after dependency injection and before the Bean is ready for normal use.

### Modern Spring note

For current Spring versions, use:

```java
jakarta.annotation.PostConstruct
```

not the old `javax.annotation.PostConstruct` namespace.

---

## 10. `InitializingBean`

Spring provides the callback interface:

```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```

Example:

```java
@Component
public class CacheService implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        System.out.println("Initialized");
    }
}
```

### Recommendation

For application code, annotations or explicit configuration are often preferred because implementing Spring-specific lifecycle interfaces couples the class to Spring.

---

## 11. Custom `initMethod`

With Java configuration:

```java
@Bean(initMethod = "init")
public CacheService cacheService() {
    return new CacheService();
}
```

Class:

```java
public class CacheService {

    public void init() {
        System.out.println("Init");
    }
}
```

This keeps the domain class free from a Spring lifecycle interface.

---

## 12. Initialization Callback Comparison ⭐

| Mechanism | Coupling | Typical use |
|---|---|---|
| `@PostConstruct` | Low | Common application initialization |
| `InitializingBean` | Higher | Spring-specific callback |
| `initMethod` | Low | Explicit Java configuration |
| `BeanPostProcessor` | Infrastructure-level | Cross-cutting/custom Bean processing |

Memory:

```text
@PostConstruct → simple
InitializingBean → Spring interface
initMethod → explicit configuration
BPP → container extension
```

---

## 13. `BeanPostProcessor` vs `BeanFactoryPostProcessor` ⭐⭐⭐

Very common interview question.

### BeanPostProcessor

Works on **Bean instances**.

```text
Bean definition
      ↓
Bean instance
      ↓
BeanPostProcessor
```

### BeanFactoryPostProcessor

Works on **Bean definitions / metadata before Beans are instantiated**.

```text
Bean definitions
      ↓
BeanFactoryPostProcessor
      ↓
Bean creation
```

### Memory

> **BPP → Bean instance**
>
> **BFPP → Bean definition**

---

## 14. `BeanFactoryPostProcessor`

Example concept:

```java
@Component
public class MyBeanFactoryPostProcessor
        implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(
            ConfigurableListableBeanFactory beanFactory) {
        // modify/inspect bean definitions or factory metadata
    }
}
```

Use it for container-level metadata/configuration processing, not normal Bean business logic.

---

## 15. Destruction Callbacks ⭐

When an ApplicationContext is closed normally, Spring can execute destruction callbacks for eligible singleton Beans.

Common mechanisms:

```text
@PreDestroy
DisposableBean.destroy()
@Bean(destroyMethod = "...")
```

---

## 16. `@PreDestroy`

Current Spring applications should use Jakarta annotation:

```java
import jakarta.annotation.PreDestroy;

@Component
public class ConnectionManager {

    @PreDestroy
    public void cleanup() {
        System.out.println("Closing resources");
    }
}
```

Useful for releasing resources owned by the Bean.

---

## 17. `DisposableBean`

Spring-specific destruction callback:

```java
@Component
public class ConnectionManager implements DisposableBean {

    @Override
    public void destroy() {
        System.out.println("Cleanup");
    }
}
```

Again, this couples the class directly to Spring.

---

## 18. Custom `destroyMethod`

Java configuration:

```java
@Bean(destroyMethod = "close")
public ConnectionManager connectionManager() {
    return new ConnectionManager();
}
```

Class:

```java
public class ConnectionManager {

    public void close() {
        // release resource
    }
}
```

---

## 19. Destruction Callback Comparison

| Mechanism | Coupling | Use |
|---|---|---|
| `@PreDestroy` | Low | Common cleanup callback |
| `DisposableBean` | Higher | Spring-specific cleanup |
| `destroyMethod` | Low | Explicit Java configuration |

---

## 20. Full Lifecycle Example ⭐⭐⭐

```java
@Component
public class PaymentClient {

    @PostConstruct
    public void init() {
        System.out.println("1. PostConstruct");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("2. PreDestroy");
    }
}
```

Conceptually:

```text
ApplicationContext starts
        ↓
PaymentClient created
        ↓
Dependencies injected
        ↓
@PostConstruct
        ↓
BeanPostProcessor.afterInitialization
        ↓
Bean ready
        ↓
ApplicationContext.close()
        ↓
@PreDestroy
```

---

## 21. Important Lifecycle Ordering ⭐⭐⭐

Interview-oriented ordering:

```text
Instantiation
      ↓
Dependency Injection
      ↓
Aware callbacks
      ↓
BeanPostProcessor.beforeInitialization
      ↓
@PostConstruct
      ↓
InitializingBean.afterPropertiesSet
      ↓
Custom init method
      ↓
BeanPostProcessor.afterInitialization
      ↓
READY
      ↓
@PreDestroy
      ↓
DisposableBean.destroy
      ↓
Custom destroy method
```

### Important nuance

Spring's actual lifecycle contains additional infrastructure-level callbacks and processing. For interviews, explain the main sequence above and mention that `BeanPostProcessor` surrounds initialization callbacks.

---

## 22. Does Spring Destroy Every Bean?

No.

Important distinction:

```text
Singleton Beans
→ Spring manages lifecycle and destruction callbacks

Prototype Beans
→ Spring creates/initializes them but does not automatically manage destruction callbacks
```

For prototype instances, application code needs an appropriate cleanup strategy when resources require explicit release.

---

## 23. What Happens If Context Is Not Closed?

Destruction callbacks depend on normal container shutdown.

For example, if an application is abruptly terminated, normal Spring destruction processing may not run.

Therefore external resources should also be managed using appropriate resource-management mechanisms rather than relying only on shutdown callbacks.

---

## 24. Lazy Beans and Lifecycle

With:

```java
@Lazy
@Component
public class ReportService {
}
```

The Bean may be created when first needed instead of during normal context startup.

Therefore its initialization lifecycle happens when the Bean is actually instantiated.

Memory:

```text
Eager singleton → startup creation
@Lazy singleton  → delayed creation
```

---

## 25. Prototype Lifecycle

Prototype lifecycle is different:

```text
getBean()
   ↓
Instantiate
   ↓
Inject dependencies
   ↓
Initialize
   ↓
Return object
```

Spring does not track the prototype instance for automatic destruction callbacks in the same way it does singleton Beans.

---

## 26. Lifecycle and AOP / Proxies ⭐

A Bean may be wrapped by a proxy during container processing.

Conceptually:

```text
Original Bean
     ↓
BeanPostProcessor
     ↓
Proxy / Wrapped Bean
     ↓
Application uses resulting object
```

This is one reason `BeanPostProcessor` is important to understand for Spring AOP and other infrastructure features.

---

## 27. Lifecycle in Spring Boot

Spring Boot uses the same underlying Spring IoC lifecycle.

For example:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Application startup creates and initializes eligible Beans according to their scopes and lifecycle configuration.

On normal context shutdown, eligible destruction callbacks are invoked.

---

## 28. Best Practices ⭐

### Prefer simple lifecycle callbacks

```java
@PostConstruct
public void init() {}
```

### Keep initialization focused

Avoid long-running operations that unnecessarily delay startup.

### Keep Beans stateless where possible

Especially singleton services.

### Don't abuse `ApplicationContextAware`

It can increase coupling and turn dependency injection into service-location style code.

### Use explicit resource ownership

If a Bean owns a resource, make its cleanup responsibility clear.

---

## 29. Common Interview Traps ⭐

### Trap 1
`@PostConstruct` runs before dependency injection.

❌ Wrong.

It runs after required dependencies have been populated.

### Trap 2
`BeanPostProcessor` works only after initialization.

❌ Wrong.

It has before and after initialization hooks.

### Trap 3
`BeanFactoryPostProcessor` works on Bean instances.

❌ Wrong.

It operates on Bean factory metadata/definitions before normal Bean instantiation.

### Trap 4
Prototype destruction is automatically handled like singleton destruction.

❌ Wrong.

### Trap 5
Scope and lifecycle are the same thing.

❌ Wrong.

```text
Scope → instance boundary
Lifecycle → instance journey
```

### Trap 6
Spring will always call `@PreDestroy`.

❌ Not guaranteed on abrupt process termination.

---

## 30. Interview Follow-Up Questions

1. What is Spring Bean lifecycle?
2. Explain Bean lifecycle step by step.
3. What happens after Bean instantiation?
4. When does dependency injection happen?
5. What are `Aware` interfaces?
6. What is `BeanPostProcessor`?
7. Why is `BeanPostProcessor` important?
8. Difference between before and after initialization?
9. What is `@PostConstruct`?
10. What is `InitializingBean`?
11. What is `initMethod`?
12. Compare `@PostConstruct`, `InitializingBean` and `initMethod`.
13. What is `@PreDestroy`?
14. What is `DisposableBean`?
15. What is `destroyMethod`?
16. Compare destruction callbacks.
17. Difference between BeanPostProcessor and BeanFactoryPostProcessor?
18. What is the exact lifecycle order?
19. Are prototype destruction callbacks managed by Spring?
20. What happens during ApplicationContext shutdown?
21. What happens if context is not closed?
22. How does `@Lazy` affect lifecycle?
23. How does AOP relate to BeanPostProcessor?
24. Why avoid excessive `Aware` interfaces?
25. Explain Bean lifecycle in 2 minutes.

---

## 31. 2-Minute Interview Answer ⭐

> **“Spring Bean lifecycle is the journey of a Bean from creation to destruction. First Spring creates the Bean and injects its dependencies. Then Aware callbacks can provide container-related information. BeanPostProcessor can process the Bean before and after initialization. Common initialization callbacks include `@PostConstruct`, `InitializingBean.afterPropertiesSet()` and a configured `initMethod`. After these steps, the Bean is ready for application use. When the ApplicationContext is closed normally, destruction callbacks such as `@PreDestroy`, `DisposableBean.destroy()` and configured destroy methods can run for eligible Beans. A key distinction is that `BeanPostProcessor` works with Bean instances, while `BeanFactoryPostProcessor` works with Bean definitions or factory metadata before Beans are instantiated. Also, Spring does not automatically manage destruction callbacks for prototype Beans in the same way as singleton Beans.”**

---

## 32. 30-Second Hinglish Answer

> **“Spring Bean lifecycle ka matlab hai Bean ka creation se destruction tak ka complete journey. Pehle Bean instantiate hota hai, dependencies inject hoti hain, phir Aware callbacks aur BeanPostProcessor processing hoti hai. Initialization ke liye `@PostConstruct`, `InitializingBean` ya custom `initMethod` use kar sakte hain. Context close hone par `@PreDestroy`, `DisposableBean` ya destroy method cleanup ke liye use hote hain. Interview ka important point hai: BPP Bean instance par kaam karta hai, jabki BFPP Bean definitions par kaam karta hai.”**

---

## 🧠 Memory Trick

```text
CREATE
  ↓
INJECT
  ↓
AWARE
  ↓
BPP BEFORE
  ↓
INIT
  ↓
BPP AFTER
  ↓
READY
  ↓
DESTROY
```

### One-line memory

> **“Create → Inject → Aware → Before → Init → After → Ready → Destroy.”**

---

## Navigation

[← S1.2.3 Configuring Scope](../03-Configuring-Scope/README.md)

[↗ S1.2 Container Extension Points](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** S1.3 — Spring Expression Language (SpEL)
