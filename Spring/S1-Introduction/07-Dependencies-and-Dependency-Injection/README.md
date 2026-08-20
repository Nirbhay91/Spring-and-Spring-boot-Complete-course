# Spring S1 — Introduction

## 7. Dependencies and Dependency Injection (DI)

> **Goal:** Understand dependency, tight vs loose coupling, IoC, Dependency Injection, constructor/setter/field injection, interfaces, Spring resolution, and interview-level design decisions.

---

## 1. What is a Dependency?

A dependency exists when one class needs another class to perform its responsibility.

```java
class OrderService {
    private PaymentService paymentService;
}
```

Here:

```text
OrderService → depends on → PaymentService
```

### Hinglish

> **Agar ek class apna kaam karne ke liye kisi doosri class ke object/service par depend karti hai, to wo uski dependency hai.**

---

## 2. Simple Java Example

```java
class PaymentService {
    void pay() {
        System.out.println("Payment done");
    }
}

class OrderService {
    private PaymentService paymentService = new PaymentService();

    void placeOrder() {
        paymentService.pay();
    }
}
```

`OrderService` khud dependency create kar raha hai:

```java
new PaymentService()
```

This creates stronger coupling.

---

## 3. What is Tight Coupling?

When a class directly creates or strongly binds itself to a concrete dependency, replacing that dependency becomes difficult.

```java
class OrderService {
    private PaymentService paymentService = new PaymentService();
}
```

Problems:

```text
Harder testing
Harder replacement
Less flexibility
More coupling
```

For example, if you want:

```text
CardPaymentService
        OR
UPIPaymentService
```

you have to modify `OrderService`.

---

## 4. Loose Coupling

Instead of depending directly on a concrete implementation, depend on an abstraction.

```java
interface PaymentService {
    void pay();
}
```

```java
class CardPaymentService implements PaymentService {
    public void pay() {
        System.out.println("Card payment");
    }
}
```

```java
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Now:

```text
OrderService
     ↓
PaymentService interface
     ↓
CardPaymentService / UpiPaymentService
```

This is easier to replace and test.

---

## 5. What is Dependency Injection?

**Dependency Injection means supplying an object's dependencies from outside instead of making the object construct those dependencies itself.**

Without DI:

```java
class OrderService {
    private PaymentService paymentService = new PaymentService();
}
```

With DI:

```java
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

The dependency comes from outside.

### Hinglish

> **DI ka simple meaning: class apni dependency khud `new` karke create nahi karegi; dependency bahar se provide ki jayegi.**

---

## 6. Dependency Injection vs Dependency Inversion

Do not confuse these.

### Dependency Injection

A technique/mechanism for providing dependencies.

```text
Dependency → supplied from outside
```

### Dependency Inversion Principle

A SOLID design principle:

> High-level modules should not depend directly on low-level modules; both should depend on abstractions.

So:

```text
DIP = design principle
DI  = technique used to supply dependencies
```

---

## 7. Why Do We Need DI?

Main benefits:

```text
Loose coupling
Testability
Maintainability
Replaceability
Separation of responsibilities
Configuration flexibility
```

Example test:

```java
PaymentService mockPaymentService = ...;
OrderService service = new OrderService(mockPaymentService);
```

You can test `OrderService` without requiring the real payment implementation.

---

## 8. Spring's Role in DI

Spring IoC Container acts as the object assembly and management infrastructure.

Conceptually:

```text
Classes
  ↓
Bean Definitions
  ↓
Spring Container
  ↓
Create Beans
  ↓
Resolve Dependencies
  ↓
Inject Dependencies
  ↓
Ready Objects
```

Example:

```java
@Service
class PaymentService {
}
```

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring creates/manages the beans and supplies `PaymentService` to `OrderService`.

---

## 9. Types of Dependency Injection

The commonly discussed forms are:

```text
1. Constructor Injection
2. Setter Injection
3. Field Injection
```

Constructor injection is generally preferred for required dependencies.

---

# 10. Constructor Injection

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### Why preferred?

```text
Dependency is explicit
Can use final fields
Object can be created in valid state
Easy unit testing
No reflection-based field setup required by the application code
```

If there is only one constructor, Spring can use it without `@Autowired`.

---

# 11. Setter Injection

```java
@Service
public class NotificationService {

    private EmailClient emailClient;

    @Autowired
    public void setEmailClient(EmailClient emailClient) {
        this.emailClient = emailClient;
    }
}
```

Useful when a dependency is optional or can be configured after object construction.

Conceptually:

```text
Object created
     ↓
Dependency set later
```

For mandatory dependencies, constructor injection is generally clearer.

---

# 12. Field Injection

Example:

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

It is concise, but generally less preferred in modern application design.

Reasons:

```text
Dependencies are less explicit
Harder plain unit testing
Fields cannot naturally be final
More framework coupling
```

Interview answer:

> **For required dependencies, prefer constructor injection.**

---

# 13. Optional Dependencies

Sometimes a dependency is not mandatory.

Possible approaches include:

```text
Optional<T>
@Nullable
ObjectProvider<T>
Conditional configuration
```

Example:

```java
public ReportService(Optional<AuditService> auditService) {
    this.auditService = auditService;
}
```

The exact choice depends on the use case.

---

# 14. Multiple Implementations

Suppose:

```java
public interface PaymentService {
    void pay();
}
```

Two implementations:

```java
@Service
class CardPaymentService implements PaymentService {
}
```

```java
@Service
class UpiPaymentService implements PaymentService {
}
```

Now this is ambiguous:

```java
public OrderService(PaymentService paymentService) {
}
```

Spring needs a way to choose a candidate.

---

# 15. Resolve Ambiguity with @Primary

```java
@Service
@Primary
class CardPaymentService implements PaymentService {
}
```

Now when Spring needs a single `PaymentService`, the primary candidate can be selected.

Use when one implementation should be the default.

---

# 16. Resolve Ambiguity with @Qualifier

```java
@Service("upiPaymentService")
class UpiPaymentService implements PaymentService {
}
```

Then:

```java
public OrderService(
        @Qualifier("upiPaymentService") PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Use when you want to explicitly select a particular implementation.

---

# 17. Constructor Injection with Multiple Dependencies

```java
@Service
public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;

    public OrderService(
            PaymentService paymentService,
            InventoryService inventoryService,
            NotificationService notificationService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
    }
}
```

This is explicit, but a constructor with too many dependencies can be a design smell.

If a class needs many collaborators, ask:

```text
Does this class have too many responsibilities?
Can responsibilities be split?
Is there a missing abstraction?
```

---

# 18. Dependency Resolution — High-Level Flow

Suppose:

```java
@Service
class OrderService {
    OrderService(PaymentService paymentService) { }
}
```

Spring conceptually performs:

```text
1. Discover/Register OrderService
2. Inspect its constructor
3. Identify PaymentService dependency
4. Find matching bean
5. Create/obtain PaymentService
6. Inject it into OrderService
7. Complete OrderService initialization
```

The actual container processing includes BeanDefinitions, dependency resolution and post-processors, so this is a simplified mental model.

---

# 19. What if Dependency is Missing?

Suppose:

```java
public OrderService(PaymentService paymentService) {
}
```

but no suitable `PaymentService` bean exists.

The application context generally fails during bean creation/startup for a required eager singleton dependency.

Conceptually:

```text
OrderService
     ↓
PaymentService required
     ↓
No candidate found
     ↓
Bean creation failure
     ↓
Application context startup failure
```

For lazy beans or specific configurations, the timing of the failure can differ.

---

# 20. Circular Dependencies

Example:

```text
A → B
B → A
```

```java
class A {
    A(B b) {}
}

class B {
    B(A a) {}
}
```

This creates a circular dependency.

Constructor-based circular dependencies cannot be resolved by simply constructing either object first.

Best solution:

> **Redesign the dependency graph rather than trying to hide the cycle.**

Possible design fixes:

```text
Extract common responsibility
Introduce an intermediate service
Use event-driven interaction where appropriate
Review class responsibilities
```

---

# 21. Dependency Injection and Testing

One of the biggest practical benefits is testability.

Production:

```text
OrderService
     ↓
RealPaymentService
```

Unit test:

```text
OrderService
     ↓
MockPaymentService
```

Example:

```java
PaymentService paymentService = mock(PaymentService.class);
OrderService orderService = new OrderService(paymentService);
```

The class does not need to know how the dependency was created.

---

# 22. Dependency Injection and Immutability

Constructor injection allows:

```java
private final PaymentService paymentService;
```

Once the object is constructed, the reference cannot be reassigned.

This improves clarity and helps create objects with required dependencies available from construction time.

Important nuance:

> `final` makes the reference non-reassignable; it does not automatically make the referenced object immutable.

---

# 23. Dependency Injection and Interfaces

DI does not require interfaces.

This works too:

```java
@Service
class OrderService {
    private final TaxService taxService;

    OrderService(TaxService taxService) {
        this.taxService = taxService;
    }
}
```

However, abstractions/interfaces are often useful when you need multiple implementations or want to separate contracts from implementations.

Interview trap:

> **DI and interfaces are related in design, but DI does not technically require an interface.**

---

# 24. Dependency Injection vs Service Locator

### Dependency Injection

```text
Dependency comes into the class
```

```java
OrderService(PaymentService paymentService)
```

### Service Locator

```text
Class asks a container/registry for dependency
```

Conceptually:

```java
context.getBean(PaymentService.class);
```

DI keeps dependencies explicit in the class API, while Service Locator hides them behind lookup.

---

# 25. Dependency Injection vs `new`

### Direct construction

```java
PaymentService service = new PaymentService();
```

The class controls creation.

### DI

```java
OrderService(PaymentService service) {
    this.service = service;
}
```

The class receives the dependency.

The important design shift is:

```text
Who creates the dependency?

Direct construction → dependent class
DI                  → external composition/container
```

---

# 26. Composition Root

The place where application dependencies are assembled is often called the **composition root**.

In a Spring application, the container and configuration infrastructure effectively perform this composition.

```text
Configuration
     ↓
Container
     ↓
Object Graph
     ↓
Application
```

This keeps business classes focused on business behavior rather than object construction.

---

# 27. Object Graph

A real application forms a dependency graph.

Example:

```text
OrderController
      ↓
OrderService
   ┌──┴──────────────┐
   ↓                 ↓
PaymentService   InventoryService
   ↓
PaymentClient
```

Spring's DI container helps construct and wire this graph.

---

# 28. Constructor Injection Best-Practice Example

```java
@Service
public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    public OrderService(
            PaymentService paymentService,
            InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }

    public void placeOrder() {
        inventoryService.reserve();
        paymentService.pay();
    }
}
```

Benefits:

```text
Dependencies visible
Object valid after construction
Easy unit testing
Fields can be final
```

---

# 29. Common Interview Traps

### ❌ DI means Spring automatically creates every Java object

Wrong. Spring manages objects registered/configured as Beans.

### ❌ DI requires an interface

Wrong. DI can inject concrete classes too.

### ❌ Constructor injection requires @Autowired

Not when there is a single constructor.

### ❌ Field injection is always better because it is shorter

Shorter does not mean better; constructor injection makes required dependencies explicit and improves testability.

### ❌ `new` is never allowed in a Spring application

Wrong. You can create objects manually when appropriate. The key is whether an object should be container-managed and whether manual construction undermines the desired dependency/lifecycle management.

### ❌ `@Autowired` itself is Dependency Injection

`@Autowired` is an annotation used by Spring to facilitate dependency resolution/injection; DI is the broader design principle/technique.

### ❌ Dependency Injection and Dependency Inversion Principle are the same

Wrong. DI is a technique; DIP is a SOLID principle.

---

# 30. Interview Comparison

| Topic | Meaning |
|---|---|
| Dependency | Object/service another class needs |
| Tight Coupling | Strong direct dependency/creation relationship |
| Loose Coupling | Depend on abstractions and externalize creation |
| IoC | Control of object creation/configuration is moved to a container/framework |
| DI | Dependencies are supplied from outside |
| DIP | SOLID principle about depending on abstractions |
| Constructor Injection | Dependency supplied through constructor |
| Setter Injection | Dependency supplied through setter |
| Field Injection | Dependency injected directly into field |
| `@Primary` | Select default candidate among multiple beans |
| `@Qualifier` | Select a specific candidate |

---

# 31. 2-Minute Interview Answer

> **"A dependency is an object that another class needs to perform its responsibility. If a class creates its dependency directly using `new`, it becomes tightly coupled to that implementation. Dependency Injection solves this by supplying the dependency from outside. In Spring, the IoC container creates and manages Beans, resolves their dependencies, and injects them. The three commonly discussed injection styles are constructor, setter and field injection. For mandatory dependencies, I prefer constructor injection because dependencies are explicit, fields can be final, the object can be created in a valid state, and unit testing is easier. If multiple implementations exist, I can use @Primary or @Qualifier. DI should also be distinguished from the Dependency Inversion Principle: DI is a technique, while DIP is a SOLID design principle."**

---

# 32. 30-Second Hinglish Answer

> **"Dependency matlab ek class ko kaam karne ke liye kisi doosri class/service ki requirement. Agar class khud `new` karke dependency banati hai to coupling badh jaati hai. Dependency Injection mein dependency bahar se provide hoti hai aur Spring IoC Container us dependency ko resolve karke Bean mein inject karta hai. Constructor injection required dependencies ke liye preferred hai kyunki dependency explicit hoti hai, field final ho sakti hai aur testing easy hoti hai. Multiple implementations ke case mein @Primary ya @Qualifier use kar sakte hain."**

---

# 33. 🧠 Memory Trick

```text
DEPENDENCY
    ↓
Who needs whom?

DI
    ↓
Dependency comes from outside

SPRING
    ↓
Find → Create → Inject → Manage
```

### One-line memory

> **"Class should tell what it needs, not how to create it."**

---

# 34. Interview Follow-Up Questions

1. What is a dependency?
2. What is tight coupling?
3. What is loose coupling?
4. What is Dependency Injection?
5. Why do we need DI?
6. How does Spring perform DI?
7. What are the types of DI?
8. Constructor vs setter injection?
9. Why is constructor injection preferred?
10. Why is field injection discouraged?
11. Does DI require an interface?
12. DI vs Dependency Inversion Principle?
13. DI vs Service Locator?
14. What happens when a dependency is missing?
15. What happens when multiple beans match a dependency?
16. What is `@Primary`?
17. What is `@Qualifier`?
18. How does Spring resolve a constructor dependency?
19. What is a circular dependency?
20. How do you fix circular dependencies?
21. How does DI improve unit testing?
22. Why use `final` with constructor injection?
23. Can you inject a third-party class?
24. Can you use `new` inside a Spring application?
25. What is an object graph?
26. What is a composition root?
27. What is IoC?
28. IoC vs DI?
29. Is `@Autowired` the same as DI?
30. Explain DI in 2 minutes with a real project example.

---

# 🔗 Navigation

[← Previous — BeanFactory vs ApplicationContext](../06-BeanFactory-vs-ApplicationContext/README.md)

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
└── 7. Dependencies and Dependency Injection    ✅
```

**Next:** S1.8 — Examining Dependencies
