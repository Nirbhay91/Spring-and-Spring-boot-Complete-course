# Spring S1 — Introduction

## 12. Injection with `@Autowired`

> **Goal:** Understand exactly how Spring's `@Autowired` performs dependency injection, how candidate resolution works, where it can be used, what happens with multiple Beans, and why constructor injection is preferred in production code.

---

## 1. What is `@Autowired`?

`@Autowired` is a Spring annotation used to tell the IoC container to resolve and inject a dependency at an injection point.

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Hinglish:

> **`@Autowired` Spring ko batata hai ki is injection point par required dependency ko IoC container se resolve karke inject karo.**

Important distinction:

```text
DI       = dependency bahar se supply hoti hai
@Autowired = Spring ko injection point identify karne ka instruction
Autowiring = suitable candidate ko automatically resolve karna
```

---

# 2. Why Do We Use `@Autowired`?

Without dependency injection:

```java
public class OrderService {
    private PaymentService paymentService = new PaymentService();
}
```

Problems:

- Tight coupling
- Harder unit testing
- Implementation directly selected
- Object creation business class ke andar
- Configuration less flexible

With Spring:

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Spring manages the dependency.

---

# 3. Basic Constructor Injection

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Flow:

```text
ApplicationContext
      ↓
Find OrderService
      ↓
See PaymentService dependency
      ↓
Find matching Bean
      ↓
Call constructor
      ↓
OrderService Bean ready
```

---

# 4. `@Autowired` on a Single Constructor ⭐

Modern Spring does **not require `@Autowired`** when the class has only one constructor.

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

This is usually preferred over:

```java
@Autowired
public OrderService(PaymentService paymentService) {
}
```

Interview point:

> **If there is only one constructor, Spring can use it for dependency injection without explicitly adding `@Autowired`.**

---

# 5. Multiple Constructors

Suppose:

```java
public OrderService() {
}

public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

If you want to explicitly indicate the constructor to use:

```java
@Autowired
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

The annotation makes the intended injection constructor explicit.

---

# 6. Field Injection

`@Autowired` can be placed directly on a field.

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring injects the dependency into the field.

### Why it works

```text
Create OrderService
        ↓
Locate @Autowired field
        ↓
Resolve PaymentService
        ↓
Inject it into field
```

### Why it is generally not preferred

- Dependencies are hidden
- Field cannot naturally be `final`
- Unit testing is less convenient without Spring/reflection
- Object can appear constructible while required dependency is not explicit
- Design intent is less visible

---

# 7. Setter Injection

`@Autowired` can also be placed on a setter.

```java
@Service
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Use setter injection when a dependency is genuinely optional or can be configured after object construction.

For mandatory dependencies, constructor injection is normally clearer.

---

# 8. Method Injection with `@Autowired`

Spring can autowire a method as well:

```java
@Autowired
public void configure(PaymentService paymentService,
                      InventoryService inventoryService) {
    this.paymentService = paymentService;
    this.inventoryService = inventoryService;
}
```

Spring resolves the method parameters and invokes the method.

This is less common than constructor injection but useful to know for interviews.

---

# 9. Constructor vs Field vs Setter

| Injection | Best use | Main concern |
|---|---|---|
| Constructor | Required dependencies | Best default choice |
| Setter | Optional/changeable dependency | Mutable state |
| Field | Legacy/simple cases | Hidden dependency, harder testing |
| Method | Multiple related dependencies/configuration | Less common |

Memory:

```text
Mandatory → Constructor
Optional/changeable → Setter
Legacy/avoid for new code → Field
```

---

# 10. How Does `@Autowired` Find the Bean?

Suppose:

```java
public OrderService(PaymentGateway gateway) {
}
```

Spring first needs a candidate compatible with `PaymentGateway`.

Conceptually:

```text
Injection point
      ↓
Required type = PaymentGateway
      ↓
Find candidate Beans
      ↓
Filter compatible candidates
      ↓
Resolve ambiguity
      ↓
Inject selected Bean
```

Candidate resolution can involve:

- Type compatibility
- `@Qualifier`
- `@Primary`
- Bean name/dependency-name matching in applicable resolution scenarios
- Other Spring candidate metadata

---

# 11. `@Autowired` with Interface ⭐

```java
public interface PaymentGateway {
    void pay(double amount);
}
```

Implementations:

```java
@Service
public class StripeGateway implements PaymentGateway {
    public void pay(double amount) {
    }
}
```

Consumer:

```java
@Service
public class OrderService {

    private final PaymentGateway gateway;

    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Spring sees that `StripeGateway` is a compatible Bean and can inject it if it is the unique candidate.

Important:

> **`@Autowired` does not mean “inject an interface.” Spring injects a concrete Bean that is compatible with the required type.**

---

# 12. Multiple Implementations Problem

Suppose:

```text
PaymentGateway
   ↑          ↑
Stripe      Razorpay
```

Both are Spring Beans.

Consumer asks for:

```java
PaymentGateway gateway
```

Now Spring has multiple candidates.

Without a resolution rule, this can produce:

```text
NoUniqueBeanDefinitionException
```

---

# 13. Solving Ambiguity with `@Primary`

```java
@Primary
@Service
public class StripeGateway implements PaymentGateway {
}
```

Now Stripe becomes the preferred candidate when no more specific selection is supplied.

Memory:

```text
Multiple candidates
       ↓
@Primary
       ↓
Preferred default candidate
```

Important:

> `@Primary` does not remove the other Beans.

---

# 14. Solving Ambiguity with `@Qualifier` ⭐

```java
@Service("stripeGateway")
public class StripeGateway implements PaymentGateway {
}
```

```java
@Service("razorpayGateway")
public class RazorpayGateway implements PaymentGateway {
}
```

Consumer:

```java
public OrderService(
        @Qualifier("razorpayGateway") PaymentGateway gateway) {
    this.gateway = gateway;
}
```

Here the injection point explicitly selects Razorpay.

Memory:

```text
@Primary   → default choice
@Qualifier → explicit choice
```

---

# 15. `@Autowired` + `@Qualifier`

They work together:

```java
@Autowired
public OrderService(
        @Qualifier("stripeGateway") PaymentGateway gateway) {
    this.gateway = gateway;
}
```

Meaning:

```text
@Autowired  → this constructor is an injection point
@Qualifier  → choose this particular candidate
```

---

# 16. Collection Injection

Instead of selecting one implementation, inject all matching Beans.

```java
public NotificationService(
        List<NotificationSender> senders) {
    this.senders = senders;
}
```

If Spring has:

```text
EmailSender
SmsSender
PushSender
```

all matching Beans can be injected.

Useful for:

- Strategy pattern
- Plugin architecture
- Notification systems
- Payment providers
- Processing chains

---

# 17. `Map<String, BeanType>` Injection

```java
public PaymentRouter(
        Map<String, PaymentGateway> gateways) {
    this.gateways = gateways;
}
```

Conceptually:

```text
stripeGateway   → StripeGateway
razorpayGateway → RazorpayGateway
```

The Bean names can be used as keys.

---

# 18. Optional Dependency

Spring supports optional dependency designs.

Example:

```java
public OrderService(Optional<PaymentGateway> gateway) {
    this.gateway = gateway;
}
```

Another option:

```java
public OrderService(ObjectProvider<PaymentGateway> provider) {
    this.provider = provider;
}
```

For mandatory business dependencies, do not make them optional merely to suppress startup errors.

---

# 19. `@Autowired(required = false)`

Spring also supports:

```java
@Autowired(required = false)
private PaymentGateway gateway;
```

This makes the injection point optional.

However, for modern application design, prefer explicit optionality through constructor parameters such as `Optional<T>` or an appropriate provider/strategy design.

---

# 20. `@Autowired` and Bean Lifecycle

Dependency injection happens as part of Bean creation/configuration.

Simplified lifecycle:

```text
Instantiate Bean
      ↓
Populate dependencies
      ↓
Aware callbacks / post-processors
      ↓
Initialization callbacks
      ↓
Bean ready
```

`@Autowired` injection is handled by Spring's dependency-injection infrastructure, including Bean post-processing mechanisms.

Interview point:

> **`@Autowired` is not magic inside the Java compiler. The Spring container processes the annotation and performs dependency resolution/injection.**

---

# 21. `AutowiredAnnotationBeanPostProcessor`

A deeper interview question:

> Which Spring component processes `@Autowired`?

Spring uses the `AutowiredAnnotationBeanPostProcessor` to detect and process injection points annotated with `@Autowired` (and related supported annotations).

High-level flow:

```text
Bean created
    ↓
BeanPostProcessor infrastructure
    ↓
AutowiredAnnotationBeanPostProcessor
    ↓
Find @Autowired injection points
    ↓
Resolve dependencies
    ↓
Inject dependencies
```

This is a useful 5-year-experience-level internal concept.

---

# 22. `@Autowired` Is Runtime Container Metadata

The Java compiler does not perform dependency injection because of `@Autowired`.

Instead:

```text
Java class
   ↓
Annotation metadata
   ↓
Spring ApplicationContext
   ↓
BeanPostProcessor / container infrastructure
   ↓
Dependency resolution
   ↓
Injection
```

This distinction helps explain why `new OrderService(...)` outside the Spring container does not automatically trigger field injection.

---

# 23. What If You Use `new` Yourself?

Example:

```java
OrderService service = new OrderService();
```

If `OrderService` is not created/managed by Spring, Spring's container is not managing that object.

Therefore, annotations such as `@Autowired` on its fields are not automatically processed by the Spring container.

Memory:

> **Spring annotations need the Spring-managed lifecycle to have their container behavior.**

---

# 24. `@Autowired` vs Constructor Injection

This is a common interview trap.

Question:

> “Is constructor injection possible without `@Autowired`?”

Answer:

**Yes.** If the class has a single constructor, modern Spring can use it automatically.

```java
@Service
class OrderService {

    private final PaymentGateway gateway;

    OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

So `@Autowired` is not synonymous with constructor injection.

---

# 25. `@Autowired` vs `@Qualifier` vs `@Primary`

| Annotation | Purpose |
|---|---|
| `@Autowired` | Marks dependency injection point |
| `@Qualifier` | Selects a specific candidate |
| `@Primary` | Marks preferred candidate |

Typical combination:

```java
@Autowired
OrderService(@Qualifier("stripe") PaymentGateway gateway) {
}
```

---

# 26. `@Autowired` vs `@Resource` vs `@Inject`

### `@Autowired`

Spring-specific annotation.

### `@Inject`

Jakarta Dependency Injection standard annotation.

### `@Resource`

Jakarta annotation commonly used for resource injection and name-oriented resolution semantics.

Interview-safe summary:

```text
@Autowired → Spring
@Inject    → Jakarta DI
@Resource  → Jakarta resource injection
```

Exact candidate resolution can differ, so do not say all three are identical.

---

# 27. Constructor Injection and Immutability

```java
private final PaymentGateway gateway;
```

Constructor:

```java
public OrderService(PaymentGateway gateway) {
    this.gateway = gateway;
}
```

Advantages:

```text
final field
Mandatory dependency explicit
Object fully initialized
Easy unit testing
Clear dependency graph
```

This is one of the strongest reasons modern Spring teams prefer constructor injection.

---

# 28. Unit Testing Constructor Injection

Without starting Spring:

```java
PaymentGateway gateway = mock(PaymentGateway.class);
OrderService service = new OrderService(gateway);
```

This is simple because the dependency is explicit in the constructor.

With field injection, tests often require reflection or a Spring test context.

---

# 29. Circular Dependency

Example:

```text
OrderService
     ↓
PaymentService
     ↓
OrderService
```

Constructor injection exposes this cycle clearly and Spring cannot construct the circular object graph normally.

Best solution:

```text
Redesign responsibilities
Extract common service
Introduce abstraction carefully
Use events where business semantics permit
```

Do not use `@Lazy` blindly just to hide architectural problems.

---

# 30. Real-World Payment Example

```java
public interface PaymentGateway {
    void pay(double amount);
}
```

```java
@Service("stripe")
class StripeGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("Stripe payment");
    }
}
```

```java
@Service("razorpay")
class RazorpayGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("Razorpay payment");
    }
}
```

Consumer:

```java
@Service
class OrderService {

    private final PaymentGateway gateway;

    public OrderService(
            @Qualifier("stripe") PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

Architecture:

```text
                PaymentGateway
                     ↑
          ┌──────────┴──────────┐
          │                     │
       Stripe                Razorpay
          ↑
          │
     OrderService
```

`OrderService` depends on the abstraction, while Spring chooses the implementation.

---

# 31. Common Interview Traps ⭐

### Trap 1
**“`@Autowired` creates the object.”**

Not exactly.

Spring manages Bean creation and dependency resolution; `@Autowired` marks an injection point.

### Trap 2
**“`@Autowired` always works by Bean name.”**

No. Type compatibility and candidate-resolution rules are central; qualifiers/primary/name matching can refine selection.

### Trap 3
**“`@Primary` means only one implementation can exist.”**

No. Multiple candidates can exist; one can be preferred.

### Trap 4
**“Constructor always needs `@Autowired`.”**

No. A single constructor can be automatically used by modern Spring.

### Trap 5
**“If I create a Bean with `new`, Spring will inject `@Autowired` fields.”**

No. The object must be managed/processed by Spring's container infrastructure.

### Trap 6
**“Field injection is always best because it is shorter.”**

Shorter code does not mean better dependency design. Constructor injection is generally preferred for mandatory dependencies.

---

# 32. Interview: What Happens Internally?

A good high-level answer:

```text
Application starts
      ↓
Spring builds ApplicationContext
      ↓
Bean definitions discovered
      ↓
Bean instance created
      ↓
@Autowired injection metadata detected
      ↓
Candidate dependency resolved
      ↓
Dependency injected
      ↓
Bean initialization continues
      ↓
Bean becomes ready
```

For deeper discussion:

```text
AutowiredAnnotationBeanPostProcessor
```

is a key internal component.

---

# 33. `@Autowired` and Spring Boot

In Spring Boot:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Component scanning and configuration create the application context.

Then:

```text
@Service
@Repository
@Component
@Configuration
@Bean
       ↓
Bean definitions
       ↓
Dependency resolution
       ↓
@Autowired / constructor injection
```

---

# 34. Best Practice for 5-Year Experience

When asked:

> “How do you use `@Autowired` in production?”

Answer:

```text
1. Prefer constructor injection for mandatory dependencies.
2. Use @Qualifier when multiple implementations exist.
3. Use @Primary for a sensible default implementation.
4. Avoid field injection in new production code.
5. Keep dependency graph simple and acyclic.
6. Use Optional/ObjectProvider for genuinely optional dependencies.
7. Do not hide configuration problems with required=false unnecessarily.
```

---

# 35. 2-Minute Interview Answer

> **“`@Autowired` is a Spring annotation that marks an injection point where the IoC container should resolve and inject a dependency. It can be used on constructors, fields, setters, and methods. In modern Spring, if a class has a single constructor, we normally don't need `@Autowired`; Spring can use that constructor automatically. When multiple Beans match the required type, we can use `@Qualifier` to select a specific Bean or `@Primary` to define a preferred default. Spring can also inject collections or maps when multiple implementations are required. Internally, Spring's dependency-injection infrastructure, including `AutowiredAnnotationBeanPostProcessor`, processes the annotation and resolves the dependency from the ApplicationContext. For production code, I prefer constructor injection because dependencies are explicit, fields can remain final, the object is fully initialized, and unit testing is easier. I avoid field injection for mandatory dependencies and use optional/provider-based approaches only when the dependency is genuinely optional.”**

---

# 36. 30-Second Hinglish Answer

> **“`@Autowired` Spring ko batata hai ki kisi injection point par dependency IoC container se resolve karke inject karni hai. Ye constructor, field, setter ya method par use ho sakta hai. Agar single constructor hai to modern Spring mein `@Autowired` likhna mandatory nahi hai. Agar multiple implementations hain to `@Qualifier` specific Bean select karta hai aur `@Primary` default candidate choose karta hai. Production mein main mandatory dependencies ke liye constructor injection prefer karta hoon kyunki dependency explicit, immutable aur testable rehti hai.”**

---

# 37. 🧠 Memory Trick

```text
@Autowired
    ↓
Injection Point
    ↓
Required Type
    ↓
Find Candidates
    ↓
@Qualifier / @Primary
    ↓
Resolve
    ↓
Inject
```

One line:

> **“Autowired dependency ko request karta hai, Qualifier choice batata hai, Primary default choice batata hai.”**

---

# 38. Interview Follow-Up Questions

1. What is `@Autowired`?
2. Where can `@Autowired` be used?
3. Constructor vs field injection?
4. Does a single constructor need `@Autowired`?
5. What happens when there are multiple constructors?
6. What is `@Qualifier`?
7. What is `@Primary`?
8. What happens when multiple Beans match a type?
9. What is `NoUniqueBeanDefinitionException`?
10. What happens when no candidate exists?
11. What is `NoSuchBeanDefinitionException`?
12. Can an interface be autowired?
13. How does Spring choose an implementation of an interface?
14. Can Spring inject a `List<T>`?
15. Can Spring inject a `Map<String,T>`?
16. What is optional dependency injection?
17. What is `ObjectProvider`?
18. What is `@Autowired(required=false)`?
19. Why is field injection discouraged?
20. Why is constructor injection preferred?
21. What happens if an object is created using `new`?
22. Does `@Autowired` create the object?
23. What is `AutowiredAnnotationBeanPostProcessor`?
24. What is the role of BeanPostProcessor?
25. `@Autowired` vs `@Inject`?
26. `@Autowired` vs `@Resource`?
27. `@Qualifier` vs `@Primary`?
28. How does `@Autowired` work in Spring Boot?
29. How do you solve circular dependencies?
30. When would you use setter injection?
31. Can a method be annotated with `@Autowired`?
32. How does Spring resolve collection injection?
33. How do you inject one specific implementation?
34. Why should mandatory dependencies not be optional?
35. Explain `@Autowired` internally.
36. Explain `@Autowired` in 2 minutes.

---

# 🔗 Navigation

[← Previous — Spring Bean Autowiring](../11-Spring-Bean-Autowiring/README.md)

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
└── 12. Injection with @Autowired               ✅
```

**Next:** S1.13 — Java Based Configuration (`@Configuration`)
