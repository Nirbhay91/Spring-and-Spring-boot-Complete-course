# Spring S1 — Introduction

## 5. Declaring and Managing Beans

> **Goal:** Understand how a Spring object becomes a Bean, how Beans are declared, how the container creates and manages them, and how this connects to component scanning, Java configuration and dependency injection.

---

# 1. What is a Spring Bean?

A **Spring Bean** is an object that is instantiated, assembled, and managed by the Spring IoC container.

```text
Java Class
   ↓
Bean Registration / Definition
   ↓
Spring IoC Container
   ↓
Bean Instance
   ↓
Lifecycle + Dependencies Managed
```

### Hinglish

> **Spring Bean basically wo object hai jisko Spring IoC Container create aur manage karta hai.**

Important:

> Every Java object is NOT automatically a Spring Bean.

---

# 2. Why Do We Need Bean Management?

Without a container:

```java
PaymentService paymentService = new PaymentService();
OrderService orderService = new OrderService(paymentService);
```

Application code itself handles object creation and wiring.

As the application grows:

```text
Object Creation
Dependency Wiring
Lifecycle
Configuration
Implementation Selection
```

become harder to maintain.

Spring centralizes these responsibilities for managed objects.

---

# 3. Bean Definition vs Bean Instance

This distinction is important.

### Bean Definition

A Bean Definition is metadata describing how the container should create/configure/manage a bean.

Conceptually it can contain information such as:

```text
Bean class
Scope
Dependencies
Lazy initialization
Initialization behavior
Destruction behavior
Qualifiers / configuration metadata
```

### Bean Instance

The actual Java object created from that definition.

```text
Bean Definition
      ↓
Container
      ↓
Bean Instance
```

Interview line:

> **BeanDefinition is configuration/metadata; the bean is the actual managed object.**

---

# 4. Ways to Declare Beans

Common approaches include:

```text
1. Component scanning
2. @Bean method inside @Configuration
3. XML configuration
4. Programmatic registration / advanced configuration
```

Modern Spring applications commonly use component scanning and Java configuration.

---

# 5. Declaring a Bean with @Component

```java
@Component
public class PaymentService {
}
```

If component scanning discovers this class, Spring can register it as a bean.

Flow:

```text
@Component
   ↓
Component Scan
   ↓
Bean Definition
   ↓
Spring Container
   ↓
PaymentService Bean
```

---

# 6. Stereotype Annotations

Common specialized component annotations are:

```java
@Component
@Service
@Repository
@Controller
@RestController
```

Conceptually:

```text
@Component
   ├── @Service
   ├── @Repository
   └── @Controller
          └── @RestController
```

### `@Service`

Used to communicate that a component belongs to the service/business layer.

```java
@Service
public class OrderService {
}
```

### `@Repository`

Used for persistence/data-access components and participates in Spring's exception translation support.

```java
@Repository
public class OrderRepository {
}
```

### `@Controller`

Used for Spring MVC controllers.

### `@RestController`

Convenience annotation for REST-style controllers where handler return values are generally written to the HTTP response body.

---

# 7. Declaring a Bean with @Bean

Use `@Bean` when you want to explicitly declare an object in Java configuration.

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Conceptually:

```text
@Configuration
      ↓
@Bean method
      ↓
Bean Definition
      ↓
Container
      ↓
PaymentService
```

---

# 8. @Component vs @Bean

| `@Component` | `@Bean` |
|---|---|
| Usually placed on a class | Placed on a method |
| Discovered through component scanning | Explicitly declared in configuration |
| Good for application-owned components | Useful when explicit construction/configuration is needed |
| Less configuration | More control over creation |

### Interview Example

For your own service:

```java
@Service
public class PaymentService {
}
```

For a third-party class you cannot annotate:

```java
@Configuration
class Config {

    @Bean
    ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

The exact configuration for third-party libraries depends on the library and Spring Boot auto-configuration.

---

# 9. @Configuration

`@Configuration` marks a class as a source of bean definitions.

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Mental model:

```text
@Configuration
      ↓
Bean Definitions
      ↓
ApplicationContext
```

---

# 10. Component Scanning

Component scanning searches configured packages for component classes.

Example:

```java
@Configuration
@ComponentScan("com.example")
public class AppConfig {
}
```

Spring discovers eligible components such as:

```java
@Service
@Repository
@Component
@Controller
```

Then registers their Bean Definitions.

```text
Package
  ↓
Component Scan
  ↓
Candidate Components
  ↓
Bean Definitions
  ↓
Container
```

Important:

> Component scanning is a discovery mechanism; it is not the same thing as object instantiation itself.

---

# 11. Spring Boot and Component Scanning

In Spring Boot, `@SpringBootApplication` combines several important annotations, including component scanning behavior.

Conceptually:

```text
@SpringBootApplication
        │
        ├── @SpringBootConfiguration
        ├── @EnableAutoConfiguration
        └── @ComponentScan
```

So Boot applications commonly discover application components automatically based on package structure.

---

# 12. Bean Naming

Beans have names/identifiers in the container.

For component scanning, a default bean name is commonly derived from the class name.

Example:

```java
@Service
public class PaymentService {
}
```

A conventional default name is:

```text
paymentService
```

You can explicitly specify a component name:

```java
@Component("paymentProcessor")
public class PaymentService {
}
```

With `@Bean`, the method name commonly becomes the bean name unless explicitly configured otherwise.

```java
@Bean
public PaymentService paymentService() {
    return new PaymentService();
}
```

---

# 13. Managing Dependencies Between Beans

Suppose:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring sees that `OrderService` requires `PaymentService` and resolves the dependency if a suitable bean is available.

```text
PaymentService Bean
       ↓
Spring Container
       ↓
OrderService Bean
```

This is dependency injection.

Detailed DI mechanisms are covered in later S1 topics.

---

# 14. Constructor Injection

Preferred style for required dependencies:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

If a class has a single constructor, modern Spring can use it without requiring `@Autowired` on that constructor.

Benefits:

```text
Explicit dependencies
Immutability
Easier unit testing
Required dependency enforcement
```

---

# 15. Multiple Beans of the Same Type

Suppose:

```java
interface PaymentService {
}

@Service
class CardPaymentService implements PaymentService {
}

@Service
class UpiPaymentService implements PaymentService {
}
```

Now this is ambiguous:

```java
public OrderService(PaymentService paymentService) {
}
```

Spring may find multiple candidates.

Solutions include:

```text
@Primary
@Qualifier
Explicit configuration
```

Example:

```java
@Service
@Primary
class CardPaymentService implements PaymentService {
}
```

or:

```java
public OrderService(
        @Qualifier("upiPaymentService") PaymentService paymentService) {
}
```

---

# 16. Bean Scope

Bean scope determines how the container creates/uses bean instances.

Common scopes:

```text
singleton
prototype
request
session
application
websocket
```

Default for standard Spring applications:

```text
singleton
```

Important:

> A Spring singleton means one instance per Spring IoC container for that bean, not one JVM-wide instance in every possible context.

---

# 17. Singleton Bean

```java
@Service
public class PaymentService {
}
```

By default, Spring creates one shared instance of that bean within the container.

```text
OrderService ─┐
              ├──> PaymentService Bean
CartService ──┘
```

This is why singleton beans should generally be designed to be thread-safe, especially when handling concurrent web requests.

---

# 18. Prototype Bean

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class RequestProcessor {
}
```

A prototype request tells the container to create a new instance when the bean is requested from the container.

Important nuance:

> Spring does not manage the complete destruction lifecycle of prototype instances in the same way it manages singleton beans.

---

# 19. Bean Lifecycle Management

Spring manages more than just object creation.

High-level lifecycle:

```text
Bean Definition
      ↓
Instantiation
      ↓
Dependency Injection
      ↓
Aware callbacks / post-processors
      ↓
Initialization
      ↓
Bean Ready
      ↓
Usage
      ↓
Destruction callbacks
```

Common lifecycle hooks include:

```java
@PostConstruct
@PreDestroy
```

Exact lifecycle ordering contains additional extension points and depends on the bean/configuration.

---

# 20. Lazy Bean Initialization

By default, singleton beans are commonly initialized eagerly when the application context is created.

A bean can be marked lazy:

```java
@Component
@Lazy
public class HeavyService {
}
```

Conceptually:

```text
Eager
Application startup
      ↓
Bean created

Lazy
Application startup
      ↓
Bean definition available
      ↓
First required access
      ↓
Bean created
```

Trade-off:

```text
Pros: potentially faster startup
Cons: first-use initialization cost and failures may move to runtime
```

---

# 21. Conditional / Profile-Based Bean Management

Spring can conditionally register beans based on environment/configuration.

Example:

```java
@Configuration
@Profile("dev")
class DevConfig {

    @Bean
    PaymentService paymentService() {
        return new MockPaymentService();
    }
}
```

This is useful for environment-specific behavior.

Spring Boot adds many conditional configuration facilities through auto-configuration.

---

# 22. Retrieving Beans from the Container

You can explicitly retrieve a bean:

```java
ApplicationContext context = ...;

PaymentService service =
        context.getBean(PaymentService.class);
```

However, application business classes generally should prefer dependency injection rather than repeatedly calling `getBean()`.

### Avoid this style in business code

```java
PaymentService service =
        context.getBean(PaymentService.class);
```

### Prefer

```java
public OrderService(PaymentService service) {
    this.service = service;
}
```

Why?

```text
Explicit dependency
Better testability
Less container coupling
Cleaner design
```

---

# 23. Programmatic Registration — Advanced Awareness

Spring also supports programmatic bean registration through container extension APIs.

One example is `BeanDefinitionRegistry` / related configuration mechanisms.

This is usually framework/infrastructure-level code rather than normal application code.

Interview takeaway:

> **Most application developers use annotations and Java configuration; programmatic registration is useful when dynamic or framework-level registration is required.**

---

# 24. Bean Management Mental Model

Remember:

```text
DECLARE
  ↓
DISCOVER / REGISTER
  ↓
CREATE
  ↓
INJECT
  ↓
INITIALIZE
  ↓
MANAGE
  ↓
DESTROY
```

Or:

```text
Class
 ↓
Bean Definition
 ↓
Spring Container
 ↓
Bean Instance
 ↓
Dependencies + Lifecycle + Scope
```

---

# 25. Practical Example

```java
public interface PaymentService {
    void pay();
}
```

```java
@Service
public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("UPI payment");
    }
}
```

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.pay();
    }
}
```

Spring conceptually does:

```text
Scan UpiPaymentService
       ↓
Register PaymentService implementation
       ↓
Create OrderService Bean
       ↓
Resolve PaymentService dependency
       ↓
Inject dependency
       ↓
OrderService ready
```

---

# 26. @Bean Example with Third-Party Class

Suppose a library provides:

```java
public class ExternalClient {
    public ExternalClient(String url) {
    }
}
```

You cannot modify it to add `@Component`.

Use Java configuration:

```java
@Configuration
public class ClientConfig {

    @Bean
    public ExternalClient externalClient() {
        return new ExternalClient("https://example.com");
    }
}
```

Now Spring can manage the returned object as a bean.

This is one of the most practical reasons to understand `@Bean`.

---

# 27. Common Interview Traps

### ❌ Every class is a Bean

Wrong. It must be registered/configured for container management.

### ❌ `@Component` and `@Bean` are identical

Wrong. They represent different declaration mechanisms.

### ❌ Singleton means thread-safe

Wrong. Singleton controls instance creation; thread safety is a separate design concern.

### ❌ Prototype means Spring manages destruction automatically

Not in the same complete lifecycle manner as singleton beans.

### ❌ Always use `ApplicationContext.getBean()`

No. Prefer dependency injection in application code.

### ❌ `@Autowired` is mandatory on every constructor

No. A single constructor can be used without `@Autowired`.

### ❌ `@Service` is fundamentally a different container mechanism from `@Component`

`@Service` is a specialized stereotype of `@Component`; it primarily communicates application intent/role.

---

# 28. 2-Minute Interview Answer

> **"A Spring Bean is an object managed by the Spring IoC container. We can declare beans through component scanning using annotations such as @Component, @Service and @Repository, or explicitly through @Bean methods inside configuration classes. Spring creates bean definitions, instantiates beans, resolves their dependencies, applies lifecycle processing and manages their scope. The default scope is singleton per container. For required dependencies, constructor injection is generally preferred. When multiple beans of the same type exist, we can use @Primary or @Qualifier. We can also use profiles, lazy initialization and other configuration mechanisms to control bean registration and creation."**

---

# 29. 30-Second Hinglish Answer

> **"Spring Bean wo object hai jo Spring IoC Container create aur manage karta hai. Bean declare karne ke common ways @Component/@Service/@Repository ke through component scanning aur @Configuration ke andar @Bean method hain. Container bean definition banata hai, object create karta hai, dependency inject karta hai aur lifecycle manage karta hai. Default scope singleton hota hai. Multiple same-type beans ho to @Primary ya @Qualifier use kar sakte hain."**

---

# 30. 🧠 Memory Trick

```text
BEAN =

Declare
  ↓
Register
  ↓
Create
  ↓
Inject
  ↓
Initialize
  ↓
Use
  ↓
Destroy
```

### Declaration shortcut

```text
@Component → class discovered
@Bean      → method explicitly creates/declares
```

---

# 31. Interview Follow-Up Questions

1. What is a Spring Bean?
2. How do you declare a Bean?
3. `@Component` vs `@Bean`?
4. Why do we use `@Configuration`?
5. What is component scanning?
6. How does Spring discover a component?
7. What is a BeanDefinition?
8. BeanDefinition vs Bean instance?
9. What is the default bean scope?
10. What is singleton scope?
11. Is a singleton Spring Bean thread-safe?
12. What is prototype scope?
13. How does Spring manage prototype beans?
14. What is lazy initialization?
15. What is `@Primary`?
16. What is `@Qualifier`?
17. Why is constructor injection preferred?
18. Can a third-party class become a Spring Bean?
19. Why use `@Bean` for third-party classes?
20. What happens when the application context starts?
21. How are dependencies resolved between beans?
22. What is the role of `ApplicationContext`?
23. Why should we avoid `getBean()` in business code?
24. What is the Spring Bean lifecycle?
25. What are `@PostConstruct` and `@PreDestroy`?
26. What is `@Lazy`?
27. What are Spring profiles?
28. What is programmatic bean registration?
29. Is `@Service` different from `@Component` internally?
30. Does Spring create a new object every time a singleton bean is injected?

---

# 🔗 Navigation

[← Previous — Spring Introduction](../04-Spring-Introduction/README.md)

[↗ S1 — Introduction](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

## Progress

```text
Spring — S1 Introduction
│
├── 1. Overview of Spring Technology             ✅
├── 2. Motivation + Spring Architecture         ✅
├── 3. The Spring Framework                      ✅
├── 4. Spring Introduction                       ✅
└── 5. Declaring and Managing Beans             ✅
```

**Next:** S1.6 — BeanFactory vs ApplicationContext
