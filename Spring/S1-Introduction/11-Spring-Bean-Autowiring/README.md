# Spring S1 — Introduction

## 11. Spring Bean Autowiring

> **Goal:** Understand how Spring automatically resolves and injects dependencies, the different autowiring approaches, how ambiguity is handled, and what actually happens inside the IoC container.

---

## 1. What is Autowiring?

**Autowiring** means Spring automatically identifies a dependency and injects a suitable Spring-managed Bean into another Bean.

Without autowiring:

```java
OrderService service = new OrderService(paymentService);
```

With Spring DI/autowiring, the container builds the object graph and supplies the dependency.

```text
Spring IoC Container
        ↓
Find dependency
        ↓
Resolve candidate Bean
        ↓
Inject dependency
        ↓
Create fully configured Bean
```

### Hinglish

> **Autowiring ka simple meaning hai: Spring khud dependency ko identify karke suitable Bean consumer ke andar inject kar deta hai.**

---

# 2. Why Do We Need Autowiring?

Suppose:

```text
OrderService
     ↓
PaymentService
```

If the application has hundreds of Beans, manually wiring every dependency becomes difficult.

Autowiring reduces explicit wiring configuration.

Benefits:

```text
Less boilerplate
Automatic dependency resolution
Cleaner configuration
Works naturally with constructor injection
Useful with component scanning
```

---

# 3. Autowiring Is a Form of Dependency Injection

Remember the distinction:

```text
Dependency Injection
        ↓
Dependency is supplied from outside

Autowiring
        ↓
Spring automatically resolves which dependency to supply
```

So:

> **Autowiring is not a replacement for DI. It is one way Spring performs dependency resolution/injection.**

---

# 4. Example Without Explicit Autowiring

```java
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring can inspect the constructor and resolve the required Bean when using modern constructor injection conventions.

The dependency contract is explicit:

```text
OrderService
     ↓
PaymentService
```

---

# 5. XML Autowiring

Classic XML configuration supports the `autowire` attribute.

Example:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>

<bean id="orderService"
      class="com.example.OrderService"
      autowire="byType"/>
```

Spring attempts to resolve dependencies according to the selected autowiring mode.

This is important for understanding legacy Spring applications.

---

# 6. XML Autowiring Modes

Classic XML supports:

```text
no
byName
byType
constructor
autodetect (historical/obsolete)
```

### `no`

No automatic autowiring.

### `byName`

Matches a property name with a Bean name.

### `byType`

Matches a property type with a Bean of that type.

### `constructor`

Attempts to resolve constructor arguments by type.

### `autodetect`

An old mode that is no longer used in modern Spring configuration; it should be treated as historical knowledge for interviews/legacy code.

---

# 7. `byName` Autowiring

Suppose:

```java
public class OrderService {

    private PaymentService paymentService;

    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

XML:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>

<bean id="orderService"
      class="com.example.OrderService"
      autowire="byName"/>
```

Spring looks for a Bean whose name matches the property name:

```text
property name = paymentService
Bean name     = paymentService
```

Then it injects it.

---

# 8. `byType` Autowiring ⭐

Suppose:

```java
public class OrderService {

    private PaymentService paymentService;

    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

XML:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>

<bean id="orderService"
      class="com.example.OrderService"
      autowire="byType"/>
```

Spring searches for a Bean compatible with `PaymentService`.

Important:

```text
byName → Bean name matters
byType → Bean type matters
```

---

# 9. `byType` Ambiguity

Suppose there are two Beans:

```xml
<bean id="stripePaymentService"
      class="com.example.StripePaymentService"/>

<bean id="razorpayPaymentService"
      class="com.example.RazorpayPaymentService"/>
```

Both implement:

```java
PaymentService
```

If a dependency requires:

```java
PaymentService paymentService
```

Spring may find multiple candidates.

```text
PaymentService
    ↑             ↑
Stripe        Razorpay
```

Now the container cannot choose based only on type.

This leads to an ambiguity problem and typically results in an exception such as `NoUniqueBeanDefinitionException` in annotation-based dependency resolution.

---

# 10. How Does Spring Resolve Multiple Candidates?

Modern Spring provides several mechanisms.

### `@Primary`

Marks one candidate as the preferred default.

```java
@Primary
@Service
class StripePaymentService implements PaymentService {
}
```

### `@Qualifier`

Explicitly selects a candidate.

```java
@Service("razorpayPaymentService")
class RazorpayPaymentService implements PaymentService {
}
```

Consumer:

```java
public OrderService(
        @Qualifier("razorpayPaymentService") PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

### Exact bean name / parameter matching

Depending on the injection scenario and metadata available, Spring can also use dependency-name matching as part of candidate resolution.

---

# 11. Autowiring with `@Autowired`

Annotation-based autowiring is much more common in modern Spring applications.

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

However, field injection is generally not preferred for new code.

Preferred:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

With a single constructor, modern Spring can use that constructor without requiring `@Autowired` explicitly.

---

# 12. Constructor Autowiring ⭐

Best-practice example:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring sees the constructor dependency and resolves the `PaymentService` Bean.

Why this approach is preferred:

```text
Dependency is mandatory
Field can be final
Object cannot exist in an invalid partially initialized state
Easy unit testing
No reflection-based field assignment needed
```

---

# 13. Multiple Constructors

If a class has multiple constructors, Spring's constructor selection rules matter.

Example:

```java
public OrderService() {
}

public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

When there are multiple constructors, explicitly indicating the intended injection constructor with `@Autowired` can remove ambiguity.

```java
@Autowired
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

---

# 14. `@Autowired(required = false)`

Historically/where supported, an optional autowired dependency can be expressed as:

```java
@Autowired(required = false)
private PaymentService paymentService;
```

But for modern code, prefer explicit optional dependency designs such as:

```java
Optional<PaymentService>
```

or a suitable default implementation/strategy.

Do not make mandatory business dependencies optional just to hide configuration errors.

---

# 15. `@Qualifier`

When multiple implementations exist:

```java
public interface PaymentGateway {
    void pay();
}
```

```java
@Service("stripe")
class StripeGateway implements PaymentGateway {
}
```

```java
@Service("razorpay")
class RazorpayGateway implements PaymentGateway {
}
```

Consumer:

```java
public OrderService(
        @Qualifier("stripe") PaymentGateway gateway) {
    this.gateway = gateway;
}
```

Memory:

```text
@Primary   → default candidate
@Qualifier → explicit candidate
```

---

# 16. `@Primary`

```java
@Primary
@Service
class StripeGateway implements PaymentGateway {
}
```

If multiple candidates match and no qualifier selects one, Spring can prefer the primary candidate.

Important:

> `@Primary` does not mean there can only be one implementation. It means this candidate should be preferred when otherwise ambiguous.

---

# 17. Collection Autowiring

Spring can inject all matching Beans.

```java
public OrderService(List<PaymentGateway> gateways) {
    this.gateways = gateways;
}
```

If the container has:

```text
StripeGateway
RazorpayGateway
PayPalGateway
```

the list can contain all matching implementations.

This is useful for:

```text
Strategy pattern
Plugin systems
Chain processing
Multiple notification channels
Payment providers
```

---

# 18. `Map<String, BeanType>` Injection

Spring can also inject a map of matching Beans:

```java
public PaymentRouter(
        Map<String, PaymentGateway> gateways) {
    this.gateways = gateways;
}
```

Conceptually:

```text
stripe   → StripeGateway
razorpay → RazorpayGateway
paypal   → PayPalGateway
```

This is useful when selecting implementations dynamically by a key.

---

# 19. Optional Dependencies

Modern Spring supports several clean approaches.

### Optional wrapper

```java
public OrderService(Optional<PaymentGateway> gateway) {
    this.gateway = gateway;
}
```

### `ObjectProvider`

```java
public OrderService(ObjectProvider<PaymentGateway> provider) {
    this.provider = provider;
}
```

This can be useful when the dependency is optional, lazily resolved, or potentially has multiple candidates.

---

# 20. `@Autowired` vs `@Resource` vs `@Inject`

These annotations have different semantics.

### `@Autowired`

Spring-specific.

Primarily type-based resolution with Spring's candidate-selection rules.

### `@Qualifier`

Refines candidate selection.

### `@Resource`

Jakarta annotation commonly associated with name-oriented injection semantics, with fallback behavior depending on the resolution context.

### `@Inject`

Jakarta/JSR-style dependency injection annotation; candidate resolution is type-oriented with qualifier mechanisms.

Interview memory:

```text
@Autowired → Spring
@Inject    → Jakarta DI
@Resource  → Jakarta resource injection / name-oriented semantics
```

---

# 21. Autowiring by Type Is Not the Same as `@Autowired`

Important interview distinction:

```text
XML autowire="byType"
```

is an XML autowiring mode.

Whereas:

```java
@Autowired
```

is an annotation-based dependency injection mechanism.

Both involve automatic dependency resolution, but their configuration model and candidate-resolution details differ.

---

# 22. Bean Name vs Type

Example:

```text
Bean name: stripeGateway
Type: StripeGateway
Interfaces: PaymentGateway
```

A dependency may be declared as:

```java
PaymentGateway gateway
```

Spring considers type compatibility and then candidate-selection rules.

Do not assume that the variable name alone determines the Bean in every injection scenario.

---

# 23. Autowiring and DIP

Autowiring works especially well with DIP.

```text
OrderService
     ↓
PaymentGateway   ← abstraction
     ↑
StripeGateway    ← implementation
```

Spring can inject the implementation without `OrderService` creating it.

So:

```text
DIP → design principle
DI  → dependency supply
Autowiring → automatic dependency resolution
```

---

# 24. Autowiring and IoC Container

The IoC container maintains the Bean definitions and object graph.

```text
Bean Definitions
       ↓
Dependency Metadata
       ↓
Candidate Resolution
       ↓
Instantiation
       ↓
Dependency Injection
       ↓
Initialization
       ↓
Ready Bean
```

Autowiring is therefore part of Spring's broader dependency-resolution process.

---

# 25. What Happens When No Candidate Exists?

Suppose:

```java
public OrderService(PaymentGateway gateway) {
}
```

but no matching Bean exists.

For a required dependency, application context creation can fail with an exception such as:

```text
NoSuchBeanDefinitionException
```

The application may fail to start because Spring cannot construct the required object graph.

This is generally good: a missing mandatory dependency should be detected early.

---

# 26. What Happens When Multiple Candidates Exist?

Suppose:

```text
PaymentGateway
   ↑          ↑
Stripe      Razorpay
```

and the consumer requests only:

```java
PaymentGateway gateway
```

without a resolution rule.

Spring may throw:

```text
NoUniqueBeanDefinitionException
```

Typical solutions:

```text
@Primary
@Qualifier
More specific injection point
Collection injection
```

---

# 27. Autowiring and Circular Dependencies

Example:

```text
ServiceA → ServiceB
ServiceB → ServiceA
```

This is a circular dependency.

Constructor injection makes such cycles fail clearly rather than allowing an incompletely constructed object graph.

Best solution:

> **Redesign the dependency graph instead of using lazy/proxy-based workarounds merely to hide the cycle.**

Possible redesigns:

```text
Extract shared responsibility
Introduce a third service
Move common logic
Use event-driven communication where appropriate
```

---

# 28. Autowiring + `@Lazy`

A dependency can sometimes be lazily resolved using `@Lazy`.

```java
public OrderService(@Lazy PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

This can defer initialization, but it should not be used as a default solution for poor dependency design or circular dependencies.

---

# 29. Field Injection vs Constructor Autowiring

### Field

```java
@Autowired
private PaymentService paymentService;
```

### Constructor

```java
private final PaymentService paymentService;

public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Constructor injection is generally preferred because:

```text
Explicit dependencies
Immutability
Testability
Fail-fast construction
Better design clarity
```

---

# 30. Setter Injection Use Case

Setter injection can be appropriate when a dependency is genuinely optional or can be changed after construction.

```java
@Autowired
public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

But mandatory dependencies should normally be constructor dependencies.

---

# 31. Autowiring in Spring Boot

Spring Boot applications commonly use component scanning:

```java
@Service
class OrderService {
}
```

```java
@Repository
class OrderRepository {
}
```

Spring Boot creates the application context, discovers configuration/components, and resolves their dependencies.

Typical flow:

```text
@SpringBootApplication
        ↓
Component scanning + configuration
        ↓
Bean definitions
        ↓
Dependency resolution
        ↓
Autowiring / injection
        ↓
Application ready
```

---

# 32. Practical Payment Example

```java
public interface PaymentGateway {
    void pay(double amount);
}
```

```java
@Service("stripe")
class StripeGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("Stripe: " + amount);
    }
}
```

```java
@Service("razorpay")
class RazorpayGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("Razorpay: " + amount);
    }
}
```

Consumer:

```java
@Service
class OrderService {

    private final PaymentGateway gateway;

    OrderService(@Qualifier("stripe") PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Flow:

```text
Spring Container
      ↓
Find PaymentGateway candidates
      ↓
Stripe + Razorpay
      ↓
@Qualifier("stripe")
      ↓
StripeGateway
      ↓
OrderService
```

---

# 33. Autowiring vs Manual Wiring

| Manual Wiring | Autowiring |
|---|---|
| Developer explicitly supplies dependency | Container resolves dependency |
| More configuration | Less boilerplate |
| Useful for precise construction | Useful for conventional DI |
| Can be verbose at scale | Can become ambiguous with many candidates |

Autowiring does not remove the need to understand the dependency graph.

---

# 34. Common Interview Trap — “Autowiring Always Uses Bean Name”

Incorrect.

Modern annotation-based injection is primarily type-oriented and then applies candidate-selection rules.

XML `byName` is specifically name-based.

Remember:

```text
XML byName → name
XML byType → type
@Autowired → type + candidate resolution
```

---

# 35. Common Interview Trap — “@Autowired Creates the Object”

Incorrect.

`@Autowired` tells Spring about an injection point.

The IoC container is responsible for managing Bean creation/configuration according to its configuration and lifecycle rules.

Conceptually:

```text
@Bean / @Component / @Service / XML
             ↓
       Bean definition
             ↓
        IoC Container
             ↓
     Dependency resolution
             ↓
          Injection
```

---

# 36. Common Interview Trap — “@Autowired Means Singleton”

Incorrect.

Autowiring does not determine scope.

A dependency can be injected from a Bean with a configured scope.

```text
Injection mechanism ≠ Bean scope
```

Scope is a separate concept.

---

# 37. Common Interview Trap — “Spring Always Creates One Object”

Not necessarily.

Bean scope determines instance behavior.

Default Spring Bean scope is singleton within a container, but prototype and web-aware scopes also exist.

Therefore:

```text
Autowiring → how dependency is resolved/injected
Scope      → lifecycle/instance semantics
```

---

# 38. 2-Minute Interview Answer

> **“Spring Bean autowiring means the IoC container automatically resolves and injects a suitable Bean into a dependency injection point. In classic XML configuration, we had modes such as byName, byType and constructor. In modern Spring, constructor injection is preferred, and Spring can resolve a single constructor dependency without requiring `@Autowired` explicitly. When multiple Beans match the same type, we can use `@Primary` for the default candidate or `@Qualifier` for explicit selection. Spring can also inject collections or maps when we need all implementations. If no required candidate exists, the context can fail with `NoSuchBeanDefinitionException`; if multiple candidates are ambiguous, it can fail with `NoUniqueBeanDefinitionException`. Autowiring is not the same as DIP: DIP is a design principle, DI is the mechanism, and autowiring is Spring's automatic dependency-resolution approach. Constructor injection is generally preferred because dependencies are explicit, immutable and easier to test.”**

---

# 39. 30-Second Hinglish Answer

> **“Spring autowiring ka matlab hai IoC container dependency ko automatically resolve karke inject karta hai. Old XML mein `byName`, `byType` aur constructor jaise modes the. Modern Spring mein constructor injection preferred hai. Agar same type ke multiple Beans hain to `@Primary` default select kar sakta hai aur `@Qualifier` specific implementation select karta hai. Agar required Bean nahi mile to `NoSuchBeanDefinitionException`, aur multiple ambiguous Beans ho to `NoUniqueBeanDefinitionException` aa sakta hai. Autowiring DI ka automatic resolution mechanism hai; DIP ek design principle hai.”**

---

# 40. 🧠 Memory Trick

```text
AUTOWIRING
    ↓
Spring finds dependency
    ↓
TYPE / NAME / CONSTRUCTOR
    ↓
Candidate resolution
    ↓
@Primary / @Qualifier
    ↓
Injection
```

### One-line memory

> **“Autowiring = Spring khud dependency ka suitable candidate find karke inject karta hai.”**

---

# 41. Interview Follow-Up Questions

1. What is Spring Bean autowiring?
2. Is autowiring the same as Dependency Injection?
3. What are XML autowiring modes?
4. Explain `byName`.
5. Explain `byType`.
6. Explain constructor autowiring.
7. What happens if two Beans have the same type?
8. How does `@Primary` work?
9. How does `@Qualifier` work?
10. `@Primary` vs `@Qualifier`?
11. What happens if no matching Bean exists?
12. What is `NoSuchBeanDefinitionException`?
13. What is `NoUniqueBeanDefinitionException`?
14. Why is constructor injection preferred?
15. Why is field injection generally discouraged?
16. Can Spring inject a List of Beans?
17. Can Spring inject a Map of Beans?
18. Can a dependency be optional?
19. What is `ObjectProvider`?
20. `@Autowired` vs `@Inject`?
21. `@Autowired` vs `@Resource`?
22. Is `@Autowired` required on a single constructor?
23. What happens with multiple constructors?
24. How does autowiring work with interfaces?
25. Does autowiring depend on Bean scope?
26. Does `@Autowired` create the Bean?
27. How does autowiring relate to IoC?
28. How does autowiring relate to DIP?
29. What is circular dependency?
30. How can circular dependency be redesigned?
31. What is `@Lazy` and when can it help?
32. XML `byType` vs `@Autowired`?
33. How does Spring choose among multiple candidates?
34. Can XML and annotation configuration coexist?
35. Explain autowiring in a Spring Boot application.
36. Explain autowiring in 2 minutes.

---

# 🔗 Navigation

[← Previous — XML Configuration of DI](../10-XML-Configuration-of-DI/README.md)

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
└── 11. Spring Bean Autowiring                  ✅
```

**Next:** S1.12 — Injection with `@Autowired`
