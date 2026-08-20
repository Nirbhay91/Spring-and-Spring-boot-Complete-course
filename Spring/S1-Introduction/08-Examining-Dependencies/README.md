# Spring S1 — Introduction

## 8. Examining Dependencies

> **Goal:** Understand how to identify, model, resolve, and manage dependencies in a Spring application before choosing an injection mechanism.

---

## 1. What Does “Examining Dependencies” Mean?

Before Spring can inject a dependency, we need to understand:

```text
Who depends on whom?
What type of dependency is required?
Is it mandatory or optional?
Which implementation should be selected?
Can multiple candidates exist?
Is there a dependency cycle?
```

Example:

```text
OrderController
      ↓
OrderService
   ┌──┴──────────────┐
   ↓                 ↓
PaymentService   InventoryService
```

This is the **dependency graph** of the application.

### Hinglish

> **Examining dependencies ka matlab hai application ke object graph ko samajhna — kaunsi class ko kis dependency ki need hai, dependency required hai ya optional hai, aur Spring ko kaunsa Bean inject karna hai.**

---

## 2. Dependency Graph

A dependency graph represents relationships between components.

```text
A → B
B → C
A → C (indirectly)
```

For an order system:

```text
OrderController
       ↓
OrderService
       ↓
PaymentService
       ↓
PaymentClient
```

The graph helps us reason about:

- Coupling
- Object creation order
- Testability
- Circular dependencies
- Bean resolution
- Architecture boundaries

---

## 3. Direct vs Indirect Dependency

### Direct dependency

```java
class OrderService {
    OrderService(PaymentService paymentService) {
    }
}
```

`OrderService` directly depends on `PaymentService`.

### Indirect dependency

```text
OrderService
    ↓
PaymentService
    ↓
PaymentClient
```

`OrderService` indirectly depends on `PaymentClient` through `PaymentService`.

This distinction matters when analyzing architecture and coupling.

---

## 4. Dependency vs Collaboration

A class may collaborate with another object without owning its creation.

For example:

```text
OrderService → PaymentService
```

The important question is not simply:

> “Does this class use another class?”

but:

> **“What contract does it need, and who should supply that collaborator?”**

This is where Dependency Injection becomes useful.

---

## 5. Concrete Dependency vs Abstraction

### Concrete dependency

```java
class OrderService {
    private final CardPaymentService paymentService;
}
```

The class is coupled to one implementation.

### Abstraction dependency

```java
class OrderService {
    private final PaymentService paymentService;
}
```

Now the class depends on a contract.

```text
OrderService
      ↓
PaymentService
   ↙       ↘
Card       UPI
```

This makes implementation replacement easier.

---

## 6. Required vs Optional Dependency

Not every dependency has the same importance.

### Required dependency

The object cannot perform its responsibility correctly without it.

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Constructor injection is usually appropriate.

### Optional dependency

The application can still operate when the dependency is absent.

Possible Spring approaches include:

```text
Optional<T>
@Nullable
ObjectProvider<T>
Conditional configuration
```

Design question:

> **If the dependency is missing, can the service still behave correctly?**

If the answer is no, model it as a required dependency rather than hiding the problem.

---

## 7. One Dependency vs Multiple Dependencies

Example:

```java
class OrderService {

    OrderService(
        PaymentService paymentService,
        InventoryService inventoryService,
        NotificationService notificationService) {
    }
}
```

Multiple dependencies are not automatically bad.

But a very large constructor can indicate:

```text
Too many responsibilities
Low cohesion
Missing abstraction
God class
Poor separation of concerns
```

### Interview point

> **Constructor size is a design signal, not a hard rule.**

---

## 8. Multiple Implementations

Suppose:

```java
interface PaymentService {
    void pay();
}
```

Implementations:

```text
CardPaymentService
UpiPaymentService
WalletPaymentService
```

If a class requests:

```java
PaymentService paymentService
```

Spring may have multiple candidates.

Now dependency examination asks:

```text
Which implementation should be injected?
```

Common tools:

```text
@Primary
@Qualifier
```

---

## 9. @Primary

```java
@Service
@Primary
class CardPaymentService implements PaymentService {
}
```

`@Primary` marks a preferred candidate when multiple beans match a single dependency.

Use it when one implementation should be the normal/default choice.

---

## 10. @Qualifier

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

`@Qualifier` makes the desired candidate explicit.

### Memory

```text
@Primary   → default choice
@Qualifier → explicit choice
```

---

## 11. Dependency Direction

A healthy dependency graph should have a clear direction.

Example:

```text
Controller
   ↓
Service
   ↓
Repository
```

Avoid accidental reverse dependencies such as:

```text
Controller
   ↓
Service
   ↓
Repository
   ↑
Service-specific controller logic
```

Layer boundaries should remain intentional.

---

## 12. Circular Dependencies

A circular dependency occurs when the graph contains a cycle.

```text
A → B
↑   ↓
└───┘
```

Example:

```java
class OrderService {
    OrderService(PaymentService paymentService) {}
}

class PaymentService {
    PaymentService(OrderService orderService) {}
}
```

With constructor injection, neither object can be constructed first.

### Better approach

Do not immediately switch injection style just to hide the cycle.

Instead ask:

```text
Should responsibilities be split?
Should a third service own shared logic?
Should communication become event-based?
Is one dependency actually unnecessary?
```

---

## 13. Dependency Chain

A dependency can itself have dependencies.

```text
OrderService
     ↓
PaymentService
     ↓
PaymentClient
     ↓
HttpClient
```

Spring resolves the object graph recursively through its container infrastructure.

Simplified flow:

```text
Request OrderService
        ↓
Resolve PaymentService
        ↓
Resolve PaymentClient
        ↓
Resolve HttpClient
        ↓
Create / obtain dependencies
        ↓
Create OrderService
```

---

## 14. Bean Definition vs Dependency

A **Bean Definition** describes how Spring should manage a Bean.

A **Dependency** describes what another Bean requires.

Conceptually:

```text
Bean Definition
      ↓
How should PaymentService be managed?

Dependency
      ↓
OrderService needs PaymentService
```

These are related but different concepts.

---

## 15. Constructor Dependency Examination

Given:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

When examining the class, identify:

```text
Class: OrderService
Required dependency: PaymentService
Injection point: constructor
Candidate type: PaymentService
Potential ambiguity: multiple implementations?
```

This is exactly the kind of reasoning Spring's dependency resolution infrastructure performs.

---

## 16. Setter Dependency Examination

```java
@Service
class NotificationService {

    private EmailClient emailClient;

    @Autowired
    void setEmailClient(EmailClient emailClient) {
        this.emailClient = emailClient;
    }
}
```

Dependency characteristics:

```text
Type: EmailClient
Injection point: setter
Can be changed after construction
```

This can be useful for optional/reconfigurable dependencies, but required dependencies are generally clearer through constructors.

---

## 17. Field Dependency Examination

```java
@Autowired
private PaymentService paymentService;
```

The dependency is:

```text
Type → PaymentService
Injection point → field
```

But the dependency is less visible in the public construction API.

This is one reason constructor injection is generally preferred.

---

## 18. Dependency Ownership

A useful architecture question is:

> **Who should own the responsibility of creating and configuring this object?**

If an object is a Spring-managed Bean, the container normally owns its lifecycle.

Business classes should generally focus on behavior rather than manually assembling the complete application object graph.

---

## 19. Dependency Lifecycle

Dependencies can have different scopes/lifecycles.

Examples:

```text
Singleton
Prototype
Request
Session
Application
WebSocket
```

When examining dependencies, ask whether the lifecycle of the dependency is compatible with the lifecycle of the consuming Bean.

Example concern:

```text
Long-lived singleton
        ↓
Short-lived contextual object
```

Spring may require a scoped proxy or another appropriate mechanism for certain scoped dependencies.

---

## 20. Dependency Configuration Sources

Spring dependencies can be discovered/configured through mechanisms such as:

```text
@Component
@Service
@Repository
@Controller
@Bean
@Configuration
Component scanning
XML configuration (legacy/common in older systems)
```

The exact source does not change the fundamental question:

> **What dependency does this Bean require, and how can the container resolve it?**

---

## 21. Examining Dependencies in Spring Boot

Typical Spring Boot flow:

```text
@SpringBootApplication
        ↓
Component Scanning
        ↓
Bean Definitions
        ↓
Dependency Metadata
        ↓
Bean Creation
        ↓
Dependency Resolution
        ↓
Injection
        ↓
Application Ready
```

For example:

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

Spring identifies both Beans and wires the dependency.

---

## 22. What Happens When No Candidate Exists?

For a required dependency:

```text
OrderService
    ↓
PaymentService required
    ↓
No matching Bean
    ↓
Dependency resolution fails
```

For a normal eager singleton during application startup, this can prevent the ApplicationContext from starting successfully.

Interview wording:

> **Spring fails fast when a required dependency cannot be resolved during bean creation.**

The exact timing can vary with lazy initialization and other configuration.

---

## 23. What Happens When Multiple Candidates Exist?

```text
PaymentService
   ↙    ↓     ↘
Card  UPI   Wallet
```

If Spring cannot determine a unique candidate, dependency resolution can fail due to ambiguity.

Resolve with:

```text
@Primary
@Qualifier
More specific injection metadata
```

---

## 24. Collection Injection

Sometimes you intentionally need **all implementations**, not one.

```java
public PaymentRouter(List<PaymentService> paymentServices) {
    this.paymentServices = paymentServices;
}
```

Spring can inject multiple matching Beans into collections/maps depending on the injection point.

This is useful for plugin/strategy-style designs.

Example:

```text
PaymentService implementations
        ↓
List<PaymentService>
        ↓
PaymentRouter
```

---

## 25. Dependency Examination for Testing

Before writing a unit test, identify:

```text
Class under test
      ↓
Required collaborators
      ↓
Which collaborators should be mocked?
      ↓
Which are real?
```

Example:

```text
OrderService
├── PaymentService → mock
├── InventoryService → mock
└── NotificationService → mock
```

This makes the unit test boundary explicit.

---

## 26. Dependency Examination and SOLID

Dependency examination connects directly to SOLID principles.

Especially:

### Single Responsibility Principle

Too many dependencies may indicate too many responsibilities.

### Dependency Inversion Principle

Depend on abstractions rather than unstable concrete implementations.

### Open/Closed Principle

Abstractions and DI can make adding implementations easier without modifying consumers.

---

## 27. Practical Example — Payment System

Suppose an order service supports:

```text
Card
UPI
Wallet
```

Bad design:

```java
class OrderService {
    private CardPaymentService card = new CardPaymentService();
    private UpiPaymentService upi = new UpiPaymentService();
}
```

Better:

```java
interface PaymentService {
    void pay(double amount);
}
```

Then:

```java
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Now the dependency can be selected by configuration/container metadata.

---

## 28. Dependency Examination Checklist

When you see a Spring class in an interview, ask these questions:

```text
1. What does this class depend on?
2. Are dependencies direct or indirect?
3. Are they required or optional?
4. Constructor / setter / field injection?
5. Is the dependency an abstraction?
6. Are multiple implementations available?
7. If yes, @Primary or @Qualifier?
8. Is there a circular dependency?
9. Is the dependency lifecycle compatible?
10. Is the class taking too many dependencies?
11. Can dependencies be mocked in unit tests?
12. Who owns object creation and lifecycle?
```

---

## 29. Common Interview Traps

### ❌ “Examining dependencies just means checking `@Autowired`.”

Wrong. Dependency analysis is broader than one annotation.

### ❌ “If there are multiple implementations, Spring always picks the first one.”

Wrong. Ambiguity must be resolved through appropriate candidate selection metadata/configuration.

### ❌ “More dependencies always mean bad code.”

Not necessarily. The domain may genuinely require several collaborators. A very large dependency set is a design signal.

### ❌ “Circular dependencies should be solved by changing constructor injection to setter injection.”

Not as a first choice. Prefer redesigning the dependency graph.

### ❌ “A dependency must always be an interface.”

Wrong. Concrete classes can also be injected.

### ❌ “Bean definition and dependency are the same thing.”

Wrong. A Bean Definition describes container management; a dependency describes a required collaborator.

---

# 30. 2-Minute Interview Answer

> **“When examining dependencies in a Spring application, I first identify the dependency graph — which component depends on which collaborator. Then I determine whether each dependency is required or optional, whether the consumer depends on an abstraction or concrete implementation, and whether there are multiple candidates. For multiple implementations I can use @Primary or @Qualifier, or inject a collection when I intentionally need all implementations. I also check for circular dependencies, lifecycle compatibility, and whether a class has too many collaborators, which can indicate poor separation of responsibilities. In Spring, the IoC container resolves these dependencies from Bean definitions and wires the object graph. Constructor injection is generally my preferred approach for mandatory dependencies because the dependency is explicit and the object can be created in a valid state.”**

---

# 31. 30-Second Hinglish Answer

> **“Examining dependencies ka matlab hai pehle dependency graph samajhna — kaunsi class ko kis service ki requirement hai. Phir check karte hain dependency required hai ya optional, abstraction hai ya concrete class, multiple implementations hain ya nahi, circular dependency to nahi hai aur lifecycle compatible hai ya nahi. Multiple implementations mein @Primary ya @Qualifier use kar sakte hain. Spring IoC container Bean definitions ke basis par dependency resolve karke object graph wire karta hai. Required dependencies ke liye constructor injection preferred hai.”**

---

# 32. 🧠 Memory Trick

```text
EXAMINE DEPENDENCY

WHO?
  ↓
WHAT?
  ↓
REQUIRED / OPTIONAL?
  ↓
ABSTRACTION / CONCRETE?
  ↓
ONE / MANY CANDIDATES?
  ↓
CYCLE?
  ↓
LIFECYCLE?
  ↓
TESTABLE?
```

One-line memory:

> **“Before injecting a dependency, understand the dependency graph.”**

---

# 33. Interview Follow-Up Questions

1. What does examining dependencies mean in Spring?
2. What is a dependency graph?
3. What is a direct dependency?
4. What is an indirect dependency?
5. Required vs optional dependency?
6. Why prefer constructor injection for required dependencies?
7. What happens if a required Bean is missing?
8. What happens if multiple Beans match?
9. What is `@Primary`?
10. What is `@Qualifier`?
11. When would you inject a `List<Interface>`?
12. What is a circular dependency?
13. Why are constructor circular dependencies problematic?
14. How should circular dependencies be redesigned?
15. What is dependency direction?
16. What is dependency lifecycle compatibility?
17. Can a dependency be a concrete class?
18. Does DI require an interface?
19. How does Spring resolve dependencies at a high level?
20. Bean Definition vs Dependency?
21. How does dependency examination help unit testing?
22. How can too many dependencies indicate a design problem?
23. How does Dependency Inversion relate to dependency examination?
24. How does Spring Boot discover dependencies?
25. Explain dependency examination with an e-commerce example.

---

# 🔗 Navigation

[← Previous — Dependencies and Dependency Injection](../07-Dependencies-and-Dependency-Injection/README.md)

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
└── 8. Examining Dependencies                   ✅
```

**Next:** S1.9 — Dependency Inversion / Dependency Injection (DI)
