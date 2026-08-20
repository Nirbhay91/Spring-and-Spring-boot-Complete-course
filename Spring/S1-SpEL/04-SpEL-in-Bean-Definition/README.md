# S1.3.4 — SpEL in Bean Definition

> **Status:** ✅ Completed

## 1. What does SpEL in Bean Definition mean?

Spring Bean Definition mein SpEL ka use karke bean properties, constructor arguments aur configuration values ko expression ke through calculate ya resolve kar sakte hain.

Simple idea:

```text
Bean Definition
      ↓
SpEL Expression
      ↓
Evaluate at container processing time
      ↓
Bean gets resolved value
```

SpEL ka common syntax:

```text
#{expression}
```

Example:

```xml
<property name="maxUsers" value="#{10 * 100}"/>
```

Result:

```text
maxUsers = 1000
```

---

## 2. Why use SpEL in Bean Definitions? ⭐⭐⭐

Agar bean configuration mein value simple literal nahi hai aur kisi expression se derive karni hai, SpEL useful ho sakta hai.

Common use cases:

```text
Arithmetic calculation
Bean property reference
Method invocation
System properties / environment-related values
Conditional values
Collection expressions
Static type references
Configuration composition
```

### Important interview point

> **SpEL is useful for expression-based configuration, but complex business logic should remain in Java code.**

---

## 3. XML Bean Configuration Example

Traditional Spring XML configuration:

```xml
<bean id="pricingConfig"
      class="com.example.PricingConfig">

    <property name="maxUsers" value="#{10 * 100}"/>

</bean>
```

The expression:

```text
#{10 * 100}
```

is evaluated and the resulting value is supplied to the bean property.

---

## 4. `#{...}` vs `${...}` ⭐⭐⭐

This is one of the most important interview questions.

### `${...}` — Property Placeholder

Used to resolve a configuration property.

```xml
<property name="url" value="${db.url}"/>
```

Conceptually:

```text
application.properties
        ↓
   db.url=...
        ↓
     ${db.url}
        ↓
   property value
```

### `#{...}` — SpEL

Used to evaluate an expression.

```xml
<property name="maxUsers" value="#{10 * 100}"/>
```

Conceptually:

```text
#{10 * 100}
      ↓
   SpEL evaluation
      ↓
     1000
```

### Memory

```text
${...} → LOOK UP PROPERTY
#{...} → EVALUATE EXPRESSION
```

---

## 5. Combining `${}` and `#{}` ⭐⭐⭐

They can be combined when a property supplies an input to a SpEL expression.

Suppose:

```properties
app.limit=100
```

Expression:

```text
#{${app.limit} * 2}
```

Conceptually:

```text
${app.limit}
      ↓
     100
      ↓
#{100 * 2}
      ↓
     200
```

So the final bean property becomes:

```text
200
```

### Interview answer

> **“`${}` resolves a property placeholder, while `#{}` evaluates a SpEL expression. They can be composed when a configuration property needs to participate in an expression.”**

---

## 6. SpEL in Constructor Arguments ⭐⭐

SpEL can also be used while defining constructor arguments.

Example:

```xml
<bean id="employee"
      class="com.example.Employee">

    <constructor-arg value="#{10 + 20}"/>

</bean>
```

The constructor receives:

```text
30
```

This is useful when the constructor input is derived from configuration or another expression.

---

## 7. SpEL in Bean Property Values ⭐⭐⭐

Example:

```xml
<bean id="applicationConfig"
      class="com.example.ApplicationConfig">

    <property name="timeout" value="#{30 * 1000}"/>

</bean>
```

Result:

```text
timeout = 30000
```

This can be useful when the bean expects milliseconds but configuration is conceptually maintained in seconds.

---

## 8. Referencing Another Bean ⭐⭐⭐

SpEL can reference another Spring Bean using:

```text
@beanName
```

Example:

```xml
<bean id="taxService"
      class="com.example.TaxService"/>

<bean id="orderService"
      class="com.example.OrderService">

    <property name="taxRate"
              value="#{@taxService.getTaxRate()}"/>

</bean>
```

Conceptually:

```text
orderService
     ↓
SpEL
     ↓
@taxService
     ↓
getTaxRate()
     ↓
value injected into orderService
```

### Important

The referenced bean must be resolvable in the evaluation context used by Spring.

---

## 9. Referencing a Bean Property ⭐⭐⭐

Suppose:

```java
class PricingConfig {
    private double taxRate;

    public double getTaxRate() {
        return taxRate;
    }
}
```

Another bean can use:

```text
#{@pricingConfig.taxRate}
```

Conceptually:

```text
@pricingConfig
      ↓
 taxRate property
      ↓
 resolved value
```

---

## 10. Calling a Bean Method ⭐⭐⭐

Example:

```text
#{@pricingService.calculateDiscount()}
```

SpEL resolves the Bean and invokes the method.

This demonstrates that SpEL can do more than static arithmetic.

### Caution

Calling methods from configuration is powerful, but keep such expressions simple and deterministic. Configuration should not become hidden business logic.

---

## 11. Type References — `T(...)`

Bean definitions can use SpEL type references.

Example:

```text
#{T(java.lang.Math).PI}
```

Or:

```text
#{T(java.lang.Math).max(10, 20)}
```

Result:

```text
20
```

Memory:

```text
T(Type) → access type/static members
```

---

## 12. Static Constants in Bean Definition

Suppose a bean needs a standard Java constant.

Example:

```text
#{T(java.lang.Integer).MAX_VALUE}
```

This can resolve the static constant rather than hard-coding the number.

Use this only when it improves readability; a simple literal is often clearer.

---

## 13. Conditional / Ternary Configuration ⭐⭐

SpEL supports conditional expressions.

Example:

```text
#{systemProperties['app.mode'] == 'prod' ? 60 : 10}
```

Conceptually:

```text
if app.mode == prod
    timeout = 60
else
    timeout = 10
```

This can be useful for small configuration decisions.

For large conditional configuration, prefer Spring profiles, conditional configuration and normal Java configuration instead of creating complex SpEL expressions.

---

## 14. Collection Expressions in Bean Definition

SpEL supports collections.

Example:

```text
#{ {'JAVA', 'SPRING', 'KAFKA'} }
```

This can be used where a bean property expects a collection.

Map example:

```text
#{ {'java':'backend', 'react':'frontend'} }
```

The exact target type and conversion depend on the bean property and conversion infrastructure.

---

## 15. Property Resolution with SpEL

A bean property can combine configuration and expressions.

Example:

```properties
app.timeout=30
```

Bean configuration concept:

```text
#{${app.timeout} * 1000}
```

Result:

```text
30000
```

This pattern is useful when external configuration should remain simple while the bean receives a derived value.

---

## 16. `@Value` and Bean Definition Metadata ⭐⭐⭐

Modern Spring applications commonly use SpEL through `@Value` instead of XML.

Example:

```java
@Component
public class AppConfig {

    @Value("#{10 * 100}")
    private int maxUsers;
}
```

Result:

```text
maxUsers = 1000
```

Property placeholder:

```java
@Value("${app.name}")
private String appName;
```

Combined:

```java
@Value("#{${app.limit} * 2}")
private int limit;
```

### Interview connection

XML `<property>` value and annotation-based `@Value` are two different configuration styles, but both can participate in Spring's expression/property resolution infrastructure.

---

## 17. Java `@Bean` Configuration ⭐⭐⭐

SpEL can also appear in annotation-based configuration.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public AppSettings appSettings(
            @Value("#{10 * 100}") int maxUsers) {

        return new AppSettings(maxUsers);
    }
}
```

Here Spring evaluates the `@Value` expression and supplies the resolved value to the `@Bean` method parameter.

### Important

For ordinary dependency injection, prefer declaring the dependency directly rather than hiding it inside a SpEL expression.

---

## 18. How Does Spring Process It? ⭐⭐⭐

High-level flow:

```text
Bean Definition / @Value
          ↓
Spring Container
          ↓
Property / Expression Resolution
          ↓
SpEL Parser
          ↓
Evaluation Context
          ↓
Evaluate #{...}
          ↓
Type Conversion if required
          ↓
Resolved Constructor Argument / Property
          ↓
Bean Creation
```

The exact internal path varies depending on the configuration mechanism, but this is the correct interview-level mental model.

---

## 19. Does SpEL Run During `new` Object Creation?

Conceptually, the expression is resolved as part of Spring's processing of bean configuration/dependency injection before the relevant value is supplied to the bean.

Do not describe it as:

> “The Java constructor itself executes SpEL.”

Instead say:

> **“Spring resolves the expression while processing the bean definition/injection metadata and then supplies the resulting value to the bean.”**

---

## 20. SpEL and Dependency Injection ⭐⭐⭐

SpEL can participate in value injection, but it is not itself the dependency injection mechanism.

Example:

```java
@Value("#{10 * 2}")
private int timeout;
```

This is value resolution/injection.

Whereas:

```java
@Autowired
private PaymentService paymentService;
```

is dependency injection of another bean.

### Memory

```text
@Autowired → inject dependency
@Value + SpEL → resolve/inject value
```

---

## 21. SpEL vs `@Autowired` ⭐⭐⭐

| Feature | `@Autowired` | SpEL / `@Value` |
|---|---|---|
| Main purpose | Dependency injection | Value/expression resolution |
| Typical target | Bean dependency | Primitive/String/config value or expression result |
| Bean reference | Directly injects bean | Can reference bean through SpEL |
| Example | `PaymentService service` | `@Value("#{10 * 2}")` |

### Interview line

> **“I use `@Autowired`/constructor injection for dependencies and SpEL with `@Value` for expression-based values.”**

---

## 22. Should We Use SpEL Everywhere? ❌

No.

Avoid using SpEL for:

```text
Complex business logic
Large conditional trees
Multi-step workflows
Database operations
Complex transformations
Security-sensitive dynamic execution
```

Prefer:

```text
Java code
@Configuration
Spring Profiles
@ConfigurationProperties
Conditional configuration
Dedicated services
```

### Senior-level principle

> **Keep configuration declarative and simple; keep business logic explicit and testable in Java.**

---

## 23. Security Considerations ⭐⭐⭐

SpEL is powerful because expressions can potentially navigate objects, invoke methods and access types depending on the evaluation context.

Therefore:

```text
Untrusted Input
      ↓
Dynamic SpEL
      ↓
Potentially dangerous evaluation
```

Avoid evaluating arbitrary user-controlled expressions.

If dynamic evaluation is genuinely required, restrict the evaluation capabilities and expose only what is necessary.

---

## 24. Modern Spring Boot Recommendation ⭐⭐⭐

For modern Spring Boot applications, don't automatically choose SpEL just because it can solve a configuration problem.

For strongly typed application configuration, prefer:

```java
@ConfigurationProperties
```

Example concept:

```java
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private int timeout;
    private int maxUsers;
}
```

Benefits:

```text
Type safety
IDE support
Validation support
Clear configuration model
Easier testing
```

Use SpEL when an actual expression is useful, not as a replacement for normal configuration binding.

---

## 25. Real-World Example

Suppose:

```properties
app.base-timeout=30
app.multiplier=2
```

A derived timeout could conceptually be expressed as:

```text
#{${app.base-timeout} * ${app.multiplier} * 1000}
```

Result:

```text
30 * 2 * 1000
= 60000 ms
```

This demonstrates:

```text
External configuration
        ↓
Property placeholders
        ↓
SpEL expression
        ↓
Derived bean value
```

For larger configuration models, however, `@ConfigurationProperties` is usually cleaner.

---

## 26. Common Interview Traps ⭐⭐⭐

### Trap 1 — `${}` is SpEL

❌ Wrong.

```text
${...} → property placeholder
#{...} → SpEL
```

---

### Trap 2 — SpEL is the same as DI

❌ Wrong.

SpEL can participate in value resolution and can reference beans, but dependency injection is a broader container mechanism.

---

### Trap 3 — Complex business logic should go into SpEL

❌ Wrong.

Keep business logic in Java.

---

### Trap 4 — Every `@Value` expression must use SpEL

❌ Wrong.

```java
@Value("${app.name}")
```

is property placeholder resolution.

```java
@Value("#{10 * 2}")
```

is SpEL.

---

### Trap 5 — SpEL is always preferred over `@ConfigurationProperties`

❌ Wrong.

For structured, strongly typed configuration, `@ConfigurationProperties` is often the better choice.

---

## 27. Interview Follow-Up Questions

1. What is SpEL in a Bean Definition?
2. Why would you use SpEL in bean configuration?
3. Difference between `${}` and `#{}`?
4. Can SpEL be used in XML bean definitions?
5. Can SpEL be used in constructor arguments?
6. Can SpEL be used in bean properties?
7. How do you reference another Bean using SpEL?
8. How do you call a Bean method from SpEL?
9. What does `T(...)` mean?
10. Can `${}` and `#{}` be combined?
11. How does `@Value` use SpEL?
12. Is SpEL the same as dependency injection?
13. When would you prefer `@ConfigurationProperties` over SpEL?
14. Does SpEL execute business logic?
15. What happens when Spring processes a SpEL-based bean value?
16. Is SpEL safe for arbitrary user input?
17. How does SpEL access another Bean?
18. Can SpEL be used with `@Bean` methods?
19. Where would you avoid SpEL?
20. Explain SpEL in Bean Definition in 2 minutes.

---

## 28. 2-Minute Interview Answer ⭐

> **“SpEL can be used in Spring Bean Definitions to calculate or resolve constructor arguments and property values dynamically. The usual syntax is `#{...}`. For example, `#{10 * 100}` can provide 1000 as a bean property value. SpEL can also reference other Spring Beans using `@beanName`, invoke methods, access properties, use type references with `T(...)`, and work with values coming from configuration. `${...}` should be distinguished from SpEL because it resolves a property placeholder, while `#{...}` evaluates a SpEL expression. They can also be combined, for example `#{${app.limit} * 2}`. In modern Spring Boot applications, I would use SpEL for small expression-based configuration and prefer `@ConfigurationProperties` for structured, type-safe configuration. I would keep complex business logic out of SpEL and never evaluate untrusted user input as an unrestricted expression.”**

---

## 29. 30-Second Hinglish Answer

> **“SpEL Bean Definition mein tab use karte hain jab bean ke constructor argument ya property ki value expression se calculate karni ho. `#{...}` SpEL hota hai, jabki `${...}` property placeholder hota hai. SpEL se doosre bean ko `@beanName` se reference, method call aur static type access `T(...)` kar sakte hain. Modern Spring Boot mein simple expression ke liye SpEL aur structured configuration ke liye `@ConfigurationProperties` prefer karunga.”**

---

## 🧠 Memory Map

```text
SpEL in Bean Definition
│
├── Property Value
│   └── #{10 * 100}
│
├── Constructor Argument
│   └── #{expression}
│
├── Property Placeholder
│   └── ${app.value}
│
├── Combination
│   └── #{${app.value} * 2}
│
├── Bean Reference
│   └── #{@beanName}
│
├── Method
│   └── #{@beanName.method()}
│
├── Type Reference
│   └── #{T(java.lang.Math).PI}
│
└── Modern Alternative
    └── @ConfigurationProperties
```

### One-line memory

> **“`${}` value lookup karta hai, `#{}` expression evaluate karta hai, aur SpEL ko configuration tak limited rakhna better hai.”**

---

## Navigation

[← S1.3.3 Expression Evaluation Against Specific Object](../03-Expression-Evaluation-Against-Specific-Object/README.md)

[↗ S1.3 Spring Expression Language](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** Continue with the next topic in the source sequence.
