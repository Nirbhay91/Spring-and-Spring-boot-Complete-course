# Spring S1 — Introduction

## 9. Dependency Inversion & Dependency Injection (DI)

> **Goal:** Clearly distinguish the **Dependency Inversion Principle (DIP)** from **Dependency Injection (DI)** and understand how Spring uses DI to build loosely coupled applications.

---

## 1. The Core Problem

Consider this design:

```java
class OrderService {
    private final MySqlPaymentRepository repository = new MySqlPaymentRepository();
}
```

`OrderService` directly controls the creation of a concrete implementation.

```text
OrderService
     ↓
MySqlPaymentRepository
```

Problems:

```text
Tight coupling
Harder testing
Harder replacement
Business logic knows infrastructure details
```

The first design question is:

> **Should a high-level business class know exactly which low-level implementation to create?**

Usually, no.

---

# 2. Dependency Inversion Principle (DIP)

DIP is the **D** in SOLID.

It has two key ideas:

1. High-level modules should not depend directly on low-level modules. Both should depend on abstractions.
2. Abstractions should not depend on details; details should depend on abstractions.

Instead of:

```text
OrderService
     ↓
MySqlPaymentRepository
```

Prefer:

```text
OrderService
     ↓
PaymentRepository  ← abstraction
     ↑
MySqlPaymentRepository
```

### Hinglish

> **High-level business logic ko directly low-level implementation par depend nahi karna chahiye. Dono abstraction par depend karein.**

---

# 3. Dependency Inversion ≠ Dependency Injection

This is one of the most important interview distinctions.

### Dependency Inversion Principle

A **design principle**.

```text
How should dependencies be structured?
```

### Dependency Injection

A **technique/mechanism** for supplying dependencies from outside.

```text
How is the dependency supplied?
```

Memory:

```text
DIP → design principle
DI  → implementation technique
```

DI can help implement DIP, but they are not the same thing.

---

# 4. Before DIP

```java
class OrderService {

    private final MySqlPaymentRepository repository =
            new MySqlPaymentRepository();

    public void placeOrder() {
        repository.save();
    }
}
```

Dependency direction:

```text
High-level
OrderService
     ↓
Low-level
MySqlPaymentRepository
```

The high-level module is coupled to the implementation detail.

---

# 5. Apply an Abstraction

Create a contract:

```java
public interface PaymentRepository {
    void save();
}
```

Implementation:

```java
public class MySqlPaymentRepository
        implements PaymentRepository {

    @Override
    public void save() {
        // MySQL-specific implementation
    }
}
```

Now the business service depends on the abstraction:

```java
class OrderService {

    private final PaymentRepository repository;

    OrderService(PaymentRepository repository) {
        this.repository = repository;
    }
}
```

Architecture:

```text
             PaymentRepository
                    ↑
                    │
        ┌───────────┴───────────┐
        │                       │
MySqlPaymentRepository   MongoPaymentRepository
```

---

# 6. Where Does DI Come In?

The constructor tells us what `OrderService` needs:

```java
OrderService(PaymentRepository repository)
```

But `OrderService` does not create the implementation.

Spring can provide it:

```text
Spring Container
      ↓
PaymentRepository
      ↓
MySqlPaymentRepository
      ↓
OrderService
```

Example:

```java
@Repository
class MySqlPaymentRepository implements PaymentRepository {
}
```

```java
@Service
class OrderService {

    private final PaymentRepository repository;

    OrderService(PaymentRepository repository) {
        this.repository = repository;
    }
}
```

Spring manages the object graph and injects the dependency.

---

# 7. DIP + DI Together

This is the complete picture:

```text
                 DESIGN
                   │
                   ▼
          Dependency Inversion
                   │
                   ▼
         Depend on abstraction
                   │
                   ▼
             IMPLEMENTATION
                   │
                   ▼
        Dependency Injection
                   │
                   ▼
       Supply implementation externally
```

In a Spring application:

```text
OrderService
      ↓
PaymentRepository   ← abstraction
      ↑
MySqlPaymentRepository
      ↑
Spring Container injects it
```

---

# 8. Why DIP Matters in Real Projects

Suppose today the application uses MySQL:

```text
OrderService → PaymentRepository → MySQL
```

Tomorrow we move to PostgreSQL:

```text
OrderService → PaymentRepository → PostgreSQL
```

The business service does not need to know the storage implementation.

This improves:

```text
Maintainability
Testability
Replaceability
Extensibility
Separation of concerns
```

---

# 9. Real-World Example — Payment Gateway

Suppose an application supports:

```text
Razorpay
Stripe
PayPal
```

Bad design:

```java
class OrderService {
    private final RazorpayClient client = new RazorpayClient();
}
```

Now `OrderService` is tied to Razorpay.

Better abstraction:

```java
interface PaymentGateway {
    PaymentResult pay(PaymentRequest request);
}
```

Implementations:

```text
RazorpayGateway
StripeGateway
PayPalGateway
```

Business layer:

```java
class OrderService {

    private final PaymentGateway gateway;

    OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Spring decides which implementation should be injected.

---

# 10. @Primary with DIP

If one implementation is the default:

```java
@Service
@Primary
class StripeGateway implements PaymentGateway {
}
```

Then:

```java
@Service
class OrderService {

    private final PaymentGateway gateway;

    OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Spring can select the primary candidate when multiple candidates exist.

---

# 11. @Qualifier with DIP

If the application explicitly needs Stripe:

```java
@Service("stripeGateway")
class StripeGateway implements PaymentGateway {
}
```

Then:

```java
OrderService(
    @Qualifier("stripeGateway") PaymentGateway gateway) {
    this.gateway = gateway;
}
```

Memory:

```text
DIP        → depend on PaymentGateway
@Qualifier → choose which implementation
DI         → inject it into OrderService
```

---

# 12. DI Without Spring

Dependency Injection is not a Spring-only concept.

Plain Java can perform DI:

```java
PaymentGateway gateway = new StripeGateway();

OrderService service = new OrderService(gateway);
```

This is still Dependency Injection because the dependency is supplied from outside.

Spring mainly provides a powerful container to automate object creation, configuration, lifecycle management, and dependency resolution.

---

# 13. Spring DI

Spring version:

```java
@Service
class OrderService {

    private final PaymentGateway gateway;

    OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

```java
@Service
class StripeGateway implements PaymentGateway {
}
```

The container effectively builds:

```text
StripeGateway
      ↓
PaymentGateway
      ↓
OrderService
```

---

# 14. Constructor Injection and DIP

Constructor injection makes the abstraction dependency explicit:

```java
class OrderService {

    private final PaymentGateway gateway;

    OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

The class says:

> **“I need a PaymentGateway.”**

It does not say:

> **“I need StripeGateway and I will create it myself.”**

That separation is the key design benefit.

---

# 15. DIP and Unit Testing

Production:

```text
OrderService
     ↓
StripeGateway
```

Test:

```text
OrderService
     ↓
MockPaymentGateway
```

Example:

```java
PaymentGateway mockGateway = mock(PaymentGateway.class);
OrderService service = new OrderService(mockGateway);
```

The business class can be tested without the real external payment provider.

---

# 16. DIP and Hexagonal/Clean Architecture

The same idea appears in larger architectures.

```text
        Infrastructure
   ┌──────────────────────┐
   │ Stripe / DB / Kafka  │
   └──────────┬───────────┘
              │ implements
              ▼
      Application Ports
              ▲
              │ depends on
       Domain / Use Case
```

The business core depends on abstractions, while infrastructure provides implementations.

This is closely related to:

```text
Ports and Adapters
Hexagonal Architecture
Clean Architecture
Onion Architecture
```

---

# 17. Dependency Direction in Clean Design

Traditional tightly coupled design:

```text
Business Logic
      ↓
Infrastructure
```

DIP-oriented design:

```text
Business Logic
      ↓
Abstraction
      ↑
Infrastructure
```

The dependency direction is inverted relative to the concrete implementation dependency.

That is the essence of **Dependency Inversion**.

---

# 18. High-Level vs Low-Level Modules

Example:

### High-level module

```text
OrderService
PaymentService
CheckoutService
```

Contains business rules.

### Low-level module

```text
MySqlRepository
KafkaProducer
HttpClient
RedisClient
```

Contains technical details.

DIP says the high-level module should not be tightly coupled to those low-level details.

---

# 19. Important Nuance — Not Every Interface Means DIP

Simply creating an interface does not automatically mean good Dependency Inversion.

Bad abstraction:

```text
IUserServiceForMySqlOnly
```

If the abstraction is designed around one implementation's details, the dependency may still be poorly inverted.

Good abstraction:

```text
UserRepository
PaymentGateway
NotificationSender
```

The abstraction should represent the capability/contract needed by the higher-level code.

---

# 20. Don't Over-Abstraction

DIP does not mean:

```text
Every class → interface
Every interface → 1 implementation
```

Creating unnecessary abstractions can increase complexity.

Use abstractions where they provide meaningful boundaries, variability, testability, or architectural decoupling.

Interview point:

> **Dependency Inversion is about controlling dependency direction, not blindly adding interfaces everywhere.**

---

# 21. DIP and Microservices

The same principle applies at service boundaries.

For example:

```text
Order Service
      ↓
Payment API contract
      ↑
Payment Service implementation
```

The Order Service should depend on a stable contract/API rather than internal implementation details of Payment Service.

Within each service, DI can wire collaborators.

---

# 22. DIP + Event-Driven Architecture

Instead of directly depending on another service implementation:

```text
OrderService → PaymentService implementation
```

A decoupled design may use an event contract:

```text
OrderService
     ↓
OrderPlaced event
     ↓
Message Broker
     ↓
Payment Service
```

The services depend on the event contract rather than direct implementation details.

This is architectural decoupling, although events introduce their own consistency and reliability considerations.

---

# 23. Common Interview Trap — “Spring implements DIP”

Better answer:

> **Spring provides dependency injection capabilities that can be used to implement a DIP-oriented design. DIP itself is a software design principle, not a Spring feature.**

Spring does not automatically make poorly designed code follow DIP.

Example:

```java
@Service
class OrderService {
    private final StripeGateway gateway = new StripeGateway();
}
```

Even inside Spring, this still tightly couples the class to `StripeGateway`.

---

# 24. Common Interview Trap — “Interface = Loose Coupling”

Not automatically.

```java
interface PaymentGateway {}
```

If the whole application still assumes Stripe-specific behavior everywhere, the abstraction has not meaningfully decoupled the design.

Loose coupling depends on:

```text
Good abstraction
Clear boundaries
Externalized construction
Limited implementation knowledge
Stable contracts
```

---

# 25. DIP vs IoC vs DI

| Concept | Meaning |
|---|---|
| DIP | SOLID design principle about dependency direction |
| IoC | Broad concept where control is transferred to an external mechanism/framework |
| DI | Technique for supplying dependencies externally |
| Spring IoC Container | Spring infrastructure that creates/manages Beans and resolves dependencies |

Memory:

```text
DIP → Design
IoC → Control
DI  → Injection
Spring → Container
```

---

# 26. Interview Scenario

### Interviewer:

> “Why don't you directly create your repository inside the service?”

### Strong answer:

> “Because that couples the business service to a concrete infrastructure implementation. I prefer depending on a repository abstraction and injecting the implementation from outside. This follows the Dependency Inversion Principle and makes the service easier to test and replace. In Spring, the IoC container can manage and inject the appropriate implementation.”

---

# 27. Practical Spring Example

```java
public interface NotificationSender {
    void send(String message);
}
```

Implementation:

```java
@Service
public class EmailNotificationSender
        implements NotificationSender {

    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}
```

Consumer:

```java
@Service
public class OrderService {

    private final NotificationSender sender;

    public OrderService(NotificationSender sender) {
        this.sender = sender;
    }

    public void placeOrder() {
        // business logic
        sender.send("Order placed");
    }
}
```

Architecture:

```text
                 Spring Container
                       │
                       ▼
OrderService ───────► NotificationSender
                           ▲
                           │
                  EmailNotificationSender
```

---

# 28. Step-by-Step Mental Model

When designing a Spring component:

```text
Step 1 → Identify what the class needs
Step 2 → Define the required contract
Step 3 → Avoid creating implementation directly
Step 4 → Create implementation(s)
Step 5 → Inject dependency externally
Step 6 → Let Spring manage the object graph
Step 7 → Test using a fake/mock implementation
```

---

# 29. 2-Minute Interview Answer

> **“Dependency Inversion Principle is the D in SOLID. It says high-level modules should not depend directly on low-level modules; both should depend on abstractions, and implementation details should depend on those abstractions. Dependency Injection is different: it is a technique for supplying a dependency from outside instead of the consumer creating it. In Spring, I normally define an abstraction such as PaymentGateway, provide implementations such as StripeGateway, and inject the abstraction through constructor injection. Spring's IoC container resolves and manages the object graph. This gives us loose coupling, easier testing, and the ability to replace implementations. I also avoid creating interfaces mechanically; the abstraction should represent a meaningful contract. So, DIP is the design principle, DI is the mechanism, and Spring provides the container infrastructure to perform DI.”**

---

# 30. 30-Second Hinglish Answer

> **“DIP SOLID ka principle hai jisme high-level business class ko directly low-level implementation par depend nahi karna chahiye. Dono abstraction par depend karein. DI ek technique hai jisme dependency bahar se provide ki jaati hai. Spring IoC Container implementation ko resolve karke inject karta hai. Example mein OrderService directly StripeGateway ko `new` nahi karega; wo PaymentGateway abstraction par depend karega aur Spring appropriate implementation inject karega. Isse coupling kam, testing easy aur implementation replace karna simple ho jaata hai.”**

---

# 31. 🧠 Memory Trick

```text
DIP
 ↓
WHO SHOULD I DEPEND ON?
 ↓
ABSTRACTION

DI
 ↓
HOW DO I GET IT?
 ↓
FROM OUTSIDE

SPRING
 ↓
WHO WIRES IT?
 ↓
IOC CONTAINER
```

### One-line memory

> **“DIP decides the dependency direction; DI supplies the dependency.”**

---

# 32. Interview Follow-Up Questions

1. What is Dependency Inversion Principle?
2. What are the two parts of DIP?
3. DIP vs Dependency Injection?
4. DIP vs IoC?
5. Why should high-level modules depend on abstractions?
6. Give a real-world DIP example.
7. How does Spring help implement DIP?
8. Does using Spring automatically guarantee DIP?
9. Does DI require an interface?
10. Can DI exist without Spring?
11. Why is constructor injection useful for DIP?
12. What is tight coupling?
13. What is loose coupling?
14. Why shouldn't services directly create repositories?
15. Does every class need an interface?
16. What makes a good abstraction?
17. How does DIP improve testing?
18. How does DIP help replace implementations?
19. What is the relationship between DIP and Clean Architecture?
20. What is the relationship between DIP and Hexagonal Architecture?
21. How does DIP apply to microservices?
22. Can events help decouple services?
23. What are high-level and low-level modules?
24. Explain `@Primary` in a DIP-oriented design.
25. Explain `@Qualifier` in a DIP-oriented design.
26. Explain DIP, DI and IoC in one example.
27. Why is `new` inside a service sometimes a design problem?
28. Is interface alone enough for loose coupling?
29. How would you refactor tightly coupled payment code?
30. Explain DIP in 2 minutes.

---

# 🔗 Navigation

[← Previous — Examining Dependencies](../08-Examining-Dependencies/README.md)

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
└── 9. Dependency Inversion / DI                ✅
```

**Next:** S1.10 — XML Configuration of DI
