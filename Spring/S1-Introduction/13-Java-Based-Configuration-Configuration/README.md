# Spring S1 — Introduction

## 13. Java-Based Configuration (`@Configuration`)

> **Goal:** Understand how Spring uses Java classes as configuration metadata, how `@Configuration` and `@Bean` work, how dependencies are wired, and how Java configuration compares with XML and component scanning.

---

## 1. What is Java-Based Configuration?

Java-based configuration means defining Spring's application configuration using Java classes instead of XML.

The central annotations are:

```java
@Configuration
@Bean
```

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

### Hinglish

> **Java-based configuration mein hum XML ke badle Java class ke through Spring Beans define karte hain. `@Configuration` configuration class ko represent karta hai aur `@Bean` Spring container ko batata hai ki method ka returned object ek managed Bean hoga.**

---

# 2. Why Java-Based Configuration?

Traditional approach:

```text
XML
 ↓
applicationContext.xml
 ↓
<bean> definitions
```

Java configuration:

```text
Java class
 ↓
@Configuration
 ↓
@Bean
 ↓
Spring IoC Container
```

Main advantages:

- Type safety
- IDE support
- Refactoring support
- Less verbose than XML
- Configuration stays in Java
- Easy dependency wiring
- Good fit with modern Spring applications

---

# 3. `@Configuration`

Example:

```java
@Configuration
public class AppConfig {
}
```

It indicates that the class is a source of Bean definitions for the Spring container.

Conceptually:

```text
@Configuration
      ↓
Configuration metadata
      ↓
Spring Container
      ↓
Bean definitions
```

---

# 4. `@Bean`

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

The object returned by `paymentService()` is registered as a Spring-managed Bean.

By default, the Bean name is derived from the method name:

```text
paymentService()
      ↓
Bean name = paymentService
```

---

# 5. `@Configuration` vs `@Bean` ⭐

Very common interview question.

### `@Configuration`

Marks a class as a configuration source.

### `@Bean`

Marks a method whose return value should be registered as a Spring Bean.

Memory:

```text
@Configuration → Configuration class
@Bean         → Bean definition
```

---

# 6. Creating ApplicationContext from Java Configuration

Classic Spring:

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

Then:

```java
PaymentService service =
        context.getBean(PaymentService.class);
```

Flow:

```text
AppConfig.class
      ↓
AnnotationConfigApplicationContext
      ↓
Read @Configuration / @Bean
      ↓
Create Bean definitions
      ↓
Instantiate + wire Beans
      ↓
ApplicationContext
```

---

# 7. Dependency Injection with `@Bean`

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }

    @Bean
    public OrderService orderService(PaymentService paymentService) {
        return new OrderService(paymentService);
    }
}
```

Spring resolves `PaymentService` and supplies it to `orderService()`.

Object graph:

```text
PaymentService
      ↓
OrderService
```

---

# 8. Method-Parameter Injection in `@Bean`

This is one of the cleanest ways to express dependencies in Java configuration.

```java
@Bean
public OrderService orderService(PaymentService paymentService) {
    return new OrderService(paymentService);
}
```

The parameter is resolved from the Spring container.

You do not need to manually call:

```java
new PaymentService()
```

inside the dependent Bean method.

---

# 9. Multiple Dependencies

```java
@Bean
public OrderService orderService(
        PaymentService paymentService,
        InventoryService inventoryService) {

    return new OrderService(paymentService, inventoryService);
}
```

Spring resolves both dependencies before creating `OrderService`.

---

# 10. Custom Bean Name

By default:

```java
@Bean
public PaymentService paymentService() {
    return new PaymentService();
}
```

Bean name:

```text
paymentService
```

Custom name:

```java
@Bean("primaryPaymentService")
public PaymentService paymentService() {
    return new PaymentService();
}
```

Multiple names can also be declared:

```java
@Bean({"paymentService", "primaryPaymentService"})
public PaymentService paymentService() {
    return new PaymentService();
}
```

---

# 11. `@Bean` and Interfaces

```java
public interface PaymentGateway {
    void pay();
}
```

Implementation:

```java
public class StripePaymentGateway
        implements PaymentGateway {
}
```

Configuration:

```java
@Bean
public PaymentGateway paymentGateway() {
    return new StripePaymentGateway();
}
```

Consumer:

```java
@Bean
public OrderService orderService(PaymentGateway gateway) {
    return new OrderService(gateway);
}
```

This promotes loose coupling and supports Dependency Inversion.

---

# 12. `@Primary` with Java Configuration

If multiple Beans implement the same interface:

```java
@Bean
@Primary
public PaymentGateway stripeGateway() {
    return new StripePaymentGateway();
}

@Bean
public PaymentGateway paypalGateway() {
    return new PaypalPaymentGateway();
}
```

When Spring needs a single `PaymentGateway`, the `@Primary` Bean is preferred unless a more specific qualifier is used.

---

# 13. `@Qualifier` with Java Configuration

```java
@Bean
public PaymentGateway stripeGateway() {
    return new StripePaymentGateway();
}

@Bean
public PaymentGateway paypalGateway() {
    return new PaypalPaymentGateway();
}
```

Consumer:

```java
@Bean
public OrderService orderService(
        @Qualifier("stripeGateway") PaymentGateway gateway) {
    return new OrderService(gateway);
}
```

Use:

```text
@Primary   → default preference
@Qualifier → explicit choice
```

---

# 14. `@Configuration` + Component Scanning

Java configuration can also enable component scanning:

```java
@Configuration
@ComponentScan("com.example")
public class AppConfig {
}
```

Then Spring can discover:

```java
@Service
@Repository
@Component
@Controller
```

This means Java configuration and stereotype annotations can work together.

---

# 15. `@Import`

Configuration can be split into multiple configuration classes.

```java
@Configuration
public class DatabaseConfig {
}
```

```java
@Configuration
@Import(DatabaseConfig.class)
public class AppConfig {
}
```

Useful for modular configuration.

Example structure:

```text
AppConfig
   ├── DatabaseConfig
   ├── SecurityConfig
   └── MessagingConfig
```

---

# 16. `@Import` with Multiple Configurations

```java
@Configuration
@Import({DatabaseConfig.class, MessagingConfig.class})
public class AppConfig {
}
```

This keeps large applications organized.

---

# 17. `@ImportResource`

Java configuration can import legacy XML:

```java
@Configuration
@ImportResource("classpath:legacy-context.xml")
public class AppConfig {
}
```

This is especially useful during migration:

```text
Legacy XML
    ↓
@ImportResource
    ↓
Java Configuration
    ↓
Spring Container
```

---

# 18. `@Configuration` vs `@Component`

Both are component candidates, but their intent differs.

### `@Component`

General-purpose component.

### `@Configuration`

Specialized configuration component intended to declare Bean definitions.

Example:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Interview point:

> `@Configuration` communicates configuration intent and receives special treatment for `@Bean` methods.

---

# 19. Full Example

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }

    @Bean
    public InventoryService inventoryService() {
        return new InventoryService();
    }

    @Bean
    public OrderService orderService(
            PaymentGateway paymentGateway,
            InventoryService inventoryService) {

        return new OrderService(
                paymentGateway,
                inventoryService);
    }
}
```

Startup:

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

Result:

```text
                 AppConfig
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
PaymentGateway  Inventory   OrderService
                    
                    ↑
            dependencies injected
```

---

# 20. The `@Bean` Method Call Trap ⭐

Consider:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }

    @Bean
    OrderService orderService() {
        return new OrderService(paymentService());
    }
}
```

Many candidates think:

> “`paymentService()` means a new object is always created.”

In a full `@Configuration` class, Spring can intercept `@Bean` method calls so that a call to another `@Bean` method is resolved through the container and respects the Bean's scope semantics rather than blindly creating an unmanaged duplicate.

This is tied to Spring's configuration class enhancement/proxy mechanism.

---

# 21. `proxyBeanMethods`

`@Configuration` supports:

```java
@Configuration(proxyBeanMethods = true)
```

and:

```java
@Configuration(proxyBeanMethods = false)
```

### `true`

Full configuration behavior is enabled. Inter-`@Bean` method calls can be intercepted to preserve container-managed semantics.

### `false`

Configuration is treated as a lightweight configuration candidate. Direct calls between `@Bean` methods are normal Java calls, so you should use method-parameter injection when one Bean depends on another.

Preferred clear style:

```java
@Bean
OrderService orderService(PaymentService paymentService) {
    return new OrderService(paymentService);
}
```

---

# 22. Why Parameter Injection Is Preferred

Instead of:

```java
@Bean
OrderService orderService() {
    return new OrderService(paymentService());
}
```

prefer:

```java
@Bean
OrderService orderService(PaymentService paymentService) {
    return new OrderService(paymentService);
}
```

Advantages:

- Explicit dependency
- Easier to read
- Works naturally with `proxyBeanMethods = false`
- Better testability
- Less reliance on configuration method interception

---

# 23. Bean Scope with `@Bean`

Example:

```java
@Bean
@Scope("prototype")
public ReportGenerator reportGenerator() {
    return new ReportGenerator();
}
```

Default Bean scope is singleton.

Other scopes depend on application context/environment.

---

# 24. Lazy Bean

```java
@Bean
@Lazy
public HeavyService heavyService() {
    return new HeavyService();
}
```

The Bean can be initialized lazily rather than eagerly at context startup.

---

# 25. Bean Lifecycle with Java Configuration

```java
@Bean(initMethod = "init", destroyMethod = "cleanup")
public PaymentService paymentService() {
    return new PaymentService();
}
```

This connects Java configuration with Spring Bean lifecycle callbacks.

---

# 26. Java Configuration vs XML

### XML

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>
```

### Java

```java
@Bean
public PaymentService paymentService() {
    return new PaymentService();
}
```

Java configuration is usually more readable and type-safe, while XML remains useful in legacy systems.

---

# 27. Java Configuration vs Component Scanning

### `@Bean`

Best when you want explicit control over object creation/configuration.

```java
@Bean
public PaymentGateway paymentGateway() {
    return new StripePaymentGateway();
}
```

### `@Component` / `@Service`

Useful when the class itself can be discovered by component scanning.

```java
@Service
public class PaymentService {
}
```

Rule of thumb:

```text
Third-party class / explicit construction / custom configuration
        → @Bean

Your application component discovered by scanning
        → @Component / @Service / @Repository
```

---

# 28. Java Configuration in Spring Boot

Spring Boot commonly uses Java configuration.

```java
@SpringBootApplication
public class Application {
}
```

`@SpringBootApplication` includes configuration-related behavior and component scanning/auto-configuration through its composed annotations.

Custom Beans can be declared with:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }
}
```

Spring Boot discovers configuration classes in the application context.

---

# 29. External Configuration vs Bean Configuration

Do not confuse:

```text
@Configuration
```

with external properties such as:

```text
application.properties
application.yml
```

`@Configuration` defines/configures Beans and application object graph.

External configuration stores environment-specific values.

For example:

```java
@ConfigurationProperties(prefix = "payment")
```

can bind external configuration into a typed object.

---

# 30. Real-World Payment Example

Suppose an application supports multiple payment providers.

```java
@Configuration
public class PaymentConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway();
    }

    @Bean
    public PaymentService paymentService(
            PaymentGateway gateway) {
        return new PaymentService(gateway);
    }
}
```

Business service depends on the interface:

```text
PaymentService
      ↓
PaymentGateway
      ↓
StripePaymentGateway
```

If the provider changes:

```text
Stripe → Razorpay
```

configuration can change without changing the business service's dependency type.

---

# 31. Common Mistake — Returning `new` Outside Spring

This bypasses Spring management:

```java
PaymentService service = new PaymentService();
```

If `PaymentService` has Spring-injected dependencies, those dependencies will not automatically be injected into an object created with ordinary `new`.

Prefer obtaining managed dependencies from Spring or, better, allowing Spring to construct the object graph.

---

# 32. Configuration Class Is Not the Application Logic

Avoid putting business logic inside configuration methods.

Bad:

```java
@Bean
public OrderService orderService() {
    // business logic here
    // database calls here
    // network calls here
    return ...;
}
```

Configuration should primarily describe how application components are constructed and wired.

---

# 33. Interview: Why `@Bean` Instead of `@Component`?

Use `@Bean` when:

- You don't own the class, such as a third-party library class.
- You need explicit construction logic.
- You need custom initialization/configuration.
- You want to choose a particular implementation of an interface.
- You want configuration to be centralized.

Use `@Component`/`@Service` when component scanning is a natural fit for your application class.

---

# 34. Interview: What Happens at Startup?

Simplified flow:

```text
Application starts
       ↓
Configuration classes discovered
       ↓
@Bean definitions registered
       ↓
Dependencies resolved
       ↓
Beans instantiated according to scope/lifecycle
       ↓
ApplicationContext ready
```

Exact internal processing is more detailed, but this is the correct interview-level mental model.

---

# 35. 2-Minute Interview Answer ⭐

> **“Java-based configuration is a Spring approach where we define application configuration using Java classes instead of XML. We mark the configuration class with `@Configuration` and declare Spring Beans using `@Bean`. The returned objects become managed Beans in the IoC container. Dependencies can be expressed directly as method parameters, for example an `OrderService` Bean method can accept `PaymentService` and Spring resolves that dependency from the container. We can also use `@Primary` and `@Qualifier` when multiple implementations exist. Java configuration can be combined with component scanning, `@Import`, and even legacy XML through `@ImportResource`. Compared with XML, Java configuration is type-safe and refactoring-friendly. In modern Spring Boot applications, Java configuration, annotations and auto-configuration are commonly used, while XML is still relevant for legacy integration.”**

---

# 36. 30-Second Hinglish Answer

> **“Java-based configuration mein hum XML ki jagah Java class se Spring Beans configure karte hain. `@Configuration` configuration class ko mark karta hai aur `@Bean` method ke returned object ko Spring-managed Bean banata hai. Dependencies method parameters ke through inject kar sakte hain. Multiple implementations ke case mein `@Primary` ya `@Qualifier` use kar sakte hain. Ye XML se zyada type-safe aur refactoring-friendly hai aur Spring Boot mein commonly use hota hai.”**

---

# 37. 🧠 Memory Trick

```text
@Configuration
      ↓
@Bean
      ↓
Bean Definition
      ↓
Dependency as method parameter
      ↓
IoC Container
      ↓
Managed Object Graph
```

### One-line memory

> **“Configuration class mein Bean define karo, dependency parameter mein lo, Spring object graph wire karega.”**

---

# 38. Interview Follow-Up Questions

1. What is Java-based configuration in Spring?
2. What does `@Configuration` do?
3. What does `@Bean` do?
4. `@Configuration` vs `@Bean`?
5. How do you load a Java configuration class?
6. What is `AnnotationConfigApplicationContext`?
7. How does dependency injection work between `@Bean` methods?
8. Why is method-parameter injection preferred?
9. What is the default Bean name for an `@Bean` method?
10. How do you define a custom Bean name?
11. What is `@Primary`?
12. What is `@Qualifier`?
13. `@Primary` vs `@Qualifier`?
14. Can `@Configuration` work with component scanning?
15. What is `@ComponentScan`?
16. What is `@Import`?
17. What is `@ImportResource`?
18. How do you import XML into Java configuration?
19. `@Configuration` vs `@Component`?
20. Why use `@Bean` instead of `@Component`?
21. How do Bean scopes work with `@Bean`?
22. How do you configure lifecycle callbacks with `@Bean`?
23. What is `proxyBeanMethods`?
24. What is the difference between `proxyBeanMethods = true` and `false`?
25. What happens when one `@Bean` method directly calls another?
26. How does Java configuration support DIP?
27. Java configuration vs XML?
28. Java configuration vs component scanning?
29. How is Java configuration used in Spring Boot?
30. What is the difference between `@Configuration` and `application.properties`?
31. Why doesn't `new` automatically perform Spring injection?
32. Explain Java-based configuration in 2 minutes.

---

# 🔗 Navigation

[← Previous — Injection with @Autowired](../12-Injection-with-Autowired/README.md)

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
├── 5. Declaring and Managing Beans              ✅
├── 6. BeanFactory vs ApplicationContext         ✅
├── 7. Dependencies and Dependency Injection    ✅
├── 8. Examining Dependencies                   ✅
├── 9. Dependency Inversion / DI                ✅
├── 10. XML Configuration of DI                 ✅
├── 11. Spring Bean Autowiring                  ✅
├── 12. Injection with @Autowired               ✅
└── 13. Java Based Configuration (@Configuration) ✅
```

**Next:** S1.14 — continue with the next topic in Spring S1 Introduction.
