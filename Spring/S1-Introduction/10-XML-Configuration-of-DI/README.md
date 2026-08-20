# Spring S1 — Introduction

## 10. XML Configuration of Dependency Injection (DI)

> **Goal:** Understand how Spring's classic XML-based configuration declares Beans and wires dependencies, why it was widely used, how `constructor-arg` and `property` work, and how it compares with modern Java/annotation configuration.

---

## 1. What is XML Configuration?

Before annotation-based and Java-based configuration became common, Spring applications frequently defined Beans and their dependencies in an XML file such as `applicationContext.xml`.

Example:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
         http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="paymentService"
          class="com.example.PaymentService"/>

</beans>
```

### Hinglish

> **XML configuration mein Spring Beans aur unki dependencies XML file ke andar declare/configure karte hain. Container is metadata ko read karke object graph create aur wire karta hai.**

---

# 2. Why Was XML Configuration Used?

XML was historically useful because configuration was externalized from Java classes.

Benefits:

```text
Centralized configuration
No annotations required
Configuration could be changed without modifying component classes
Explicit dependency wiring
Useful for older/legacy Spring applications
```

However, large XML files can become verbose and harder to maintain.

---

# 3. Basic Bean Definition

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>
```

Meaning:

```text
id    → Bean name
class → implementation class
```

Conceptually:

```text
XML Bean Definition
        ↓
Spring IoC Container
        ↓
PaymentService object
        ↓
Managed Bean
```

---

# 4. Bean ID vs Class

Example:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>
```

`paymentService` is the **Bean name/identifier**.

`com.example.PaymentService` is the Java class used to create the Bean.

They are different concepts.

---

# 5. Loading XML Configuration

Classic Spring code:

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");
```

Then:

```java
PaymentService service =
        context.getBean("paymentService", PaymentService.class);
```

Flow:

```text
applicationContext.xml
        ↓
ClassPathXmlApplicationContext
        ↓
Parse Bean definitions
        ↓
Create / configure Beans
        ↓
Resolve dependencies
        ↓
ApplicationContext ready
```

---

# 6. Constructor Injection Using XML

Java class:

```java
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

XML:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>

<bean id="orderService"
      class="com.example.OrderService">
    <constructor-arg ref="paymentService"/>
</bean>
```

Meaning:

```text
Spring creates PaymentService
        ↓
passes it to OrderService constructor
        ↓
OrderService becomes fully initialized
```

---

# 7. Constructor Injection with `name`

If constructor parameters are named and metadata is available, XML can express a named argument:

```xml
<constructor-arg name="paymentService"
                 ref="paymentService"/>
```

For older/ambiguous configurations, index-based or type-based constructor arguments may be useful.

---

# 8. Constructor Injection with `index`

```xml
<constructor-arg index="0" ref="paymentService"/>
```

Example with multiple dependencies:

```java
public OrderService(PaymentService paymentService,
                    InventoryService inventoryService) {
}
```

XML:

```xml
<bean id="orderService"
      class="com.example.OrderService">
    <constructor-arg index="0" ref="paymentService"/>
    <constructor-arg index="1" ref="inventoryService"/>
</bean>
```

---

# 9. Constructor Injection with `type`

```xml
<constructor-arg type="com.example.PaymentService"
                 ref="paymentService"/>
```

This can help Spring distinguish overloaded constructors/arguments in XML configuration.

---

# 10. Injecting a Literal Value

XML can inject simple values:

```xml
<bean id="paymentService"
      class="com.example.PaymentService">
    <constructor-arg value="ONLINE"/>
</bean>
```

The value is not another Spring Bean reference.

Memory:

```text
ref   → another Bean
value → literal value
```

---

# 11. `ref` vs `value` ⭐

This is a common interview question.

### `ref`

```xml
<constructor-arg ref="paymentService"/>
```

Means:

> Inject another Spring-managed Bean.

### `value`

```xml
<constructor-arg value="100"/>
```

Means:

> Inject a literal value.

Memory:

```text
ref   = Bean reference
value = actual value
```

---

# 12. Setter Injection Using XML

Java:

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
<bean id="orderService"
      class="com.example.OrderService">
    <property name="paymentService"
              ref="paymentService"/>
</bean>
```

Flow:

```text
Create OrderService
       ↓
Call setPaymentService(...)
       ↓
Dependency injected
```

---

# 13. `property` with a Literal Value

```xml
<bean id="emailService"
      class="com.example.EmailService">
    <property name="host"
              value="smtp.example.com"/>
</bean>
```

Here `host` maps to a JavaBean setter such as:

```java
setHost(String host)
```

---

# 14. Constructor Injection vs Setter Injection in XML

| Constructor | Setter |
|---|---|
| `constructor-arg` | `property` |
| Good for required dependencies | Useful for optional/configurable properties |
| Object can be fully initialized at construction | Dependency supplied after object creation |
| Supports immutable fields naturally | Allows later mutation |

Preferred modern approach for mandatory dependencies:

```text
Constructor Injection
```

---

# 15. Injecting Collections

Spring XML can configure collections such as:

```text
list
set
map
props
```

Example:

```xml
<property name="supportedMethods">
    <list>
        <value>CARD</value>
        <value>UPI</value>
        <value>WALLET</value>
    </list>
</property>
```

Java:

```java
public void setSupportedMethods(List<String> methods) {
    this.supportedMethods = methods;
}
```

---

# 16. Injecting Bean References into Collections

```xml
<property name="paymentServices">
    <list>
        <ref bean="cardPaymentService"/>
        <ref bean="upiPaymentService"/>
    </list>
</property>
```

This is useful for strategy/plugin-style designs.

---

# 17. Nested Bean Definition

A Bean can be defined inline when it does not need to be shared independently:

```xml
<property name="paymentService">
    <bean class="com.example.PaymentService"/>
</property>
```

This is different from referencing a separately named Bean:

```xml
<property name="paymentService"
          ref="paymentService"/>
```

---

# 18. Bean Aliases

A Bean can have aliases:

```xml
<alias name="paymentService"
       alias="primaryPaymentService"/>
```

This allows the same Bean definition to be referenced through another name.

---

# 19. Multiple Bean Definitions of the Same Class

```xml
<bean id="cardPaymentService"
      class="com.example.PaymentService"/>

<bean id="upiPaymentService"
      class="com.example.PaymentService"/>
```

Same Java class, different Bean definitions.

The important distinction is:

```text
Class ≠ Bean definition
```

Each definition can represent a separately configured Bean instance, subject to its scope/configuration.

---

# 20. Bean Scope in XML

Example:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"
      scope="prototype"/>
```

Common scopes include:

```text
singleton
prototype
request
session
application
websocket
```

Web-aware scopes require the appropriate web application context/environment.

---

# 21. XML and Dependency Resolution

Consider:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>

<bean id="orderService"
      class="com.example.OrderService">
    <constructor-arg ref="paymentService"/>
</bean>
```

Dependency graph:

```text
paymentService
      ↓
orderService
```

Spring reads the metadata and resolves the reference while creating/configuring the Beans.

---

# 22. XML Configuration and Interfaces

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

XML:

```xml
<bean id="paymentGateway"
      class="com.example.StripePaymentGateway"/>
```

Consumer:

```java
public class OrderService {

    private final PaymentGateway gateway;

    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

XML:

```xml
<bean id="orderService"
      class="com.example.OrderService">
    <constructor-arg ref="paymentGateway"/>
</bean>
```

This is XML-based DI while still following a DIP-oriented design.

---

# 23. XML `bean` vs Component Scanning

### XML explicit Bean

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>
```

### Component scanning

```xml
<context:component-scan base-package="com.example"/>
```

With component scanning, Spring discovers stereotype-annotated classes such as:

```java
@Service
@Repository
@Component
@Controller
```

XML can therefore also be used to enable annotation-based discovery.

---

# 24. XML Configuration with `context`

Example namespace:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
         http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.example"/>

</beans>
```

This demonstrates an important point:

> **XML configuration and annotation-based configuration are not mutually exclusive.**

---

# 25. XML vs Java Configuration

### XML

```xml
<bean id="orderService"
      class="com.example.OrderService">
    <constructor-arg ref="paymentService"/>
</bean>
```

### Java Configuration

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }

    @Bean
    OrderService orderService(PaymentService paymentService) {
        return new OrderService(paymentService);
    }
}
```

Both describe the application object graph, but Java configuration provides type-safe, refactor-friendly configuration in Java source.

---

# 26. XML vs Annotation Configuration

### XML

```xml
<bean id="paymentService"
      class="com.example.PaymentService"/>
```

### Annotation

```java
@Service
class PaymentService {
}
```

### Java configuration

```java
@Bean
PaymentService paymentService() {
    return new PaymentService();
}
```

All can participate in Spring's IoC container.

---

# 27. Why XML Is Still Important for Interviews

You may work on a modern Spring Boot project where XML is rarely used, but interviewers can ask about XML because:

```text
Legacy Spring applications
Older enterprise applications
Migration projects
Historical Spring configuration
Understanding IoC fundamentals
```

The important skill is not memorizing XML syntax blindly.

Understand:

```text
Bean definition
Dependency reference
Constructor injection
Setter injection
Container loading
Scope
Lifecycle
```

---

# 28. XML in Legacy Applications

A common legacy structure may look like:

```text
applicationContext.xml
       ↓
service beans
       ↓
dao beans
       ↓
data source
       ↓
transaction configuration
```

Modernization can gradually move toward:

```text
XML
 ↓
Java Config
 ↓
Annotations
 ↓
Spring Boot Auto-configuration
```

But migration should be incremental and validated rather than assuming XML is automatically wrong.

---

# 29. XML Configuration and Spring Boot

Spring Boot strongly favors convention, annotations, Java configuration, and auto-configuration.

However, Boot can still integrate XML configuration when needed.

For example, XML configuration can be imported into the application context using appropriate Spring mechanisms such as `@ImportResource`.

Example:

```java
@SpringBootApplication
@ImportResource("classpath:legacy-context.xml")
public class Application {
}
```

This is useful during legacy migration.

---

# 30. Important: XML Does Not Mean No IoC

A common misunderstanding is:

> “XML is old, so IoC is different.”

No.

The configuration format changes, but the underlying concept remains:

```text
Configuration metadata
        ↓
Spring IoC Container
        ↓
Bean creation + dependency resolution + lifecycle management
```

---

# 31. XML Bean Lifecycle Hooks

XML can also configure lifecycle callbacks.

Example:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"
      init-method="init"
      destroy-method="cleanup"/>
```

The container can invoke configured lifecycle methods at appropriate points.

This connects XML configuration with later topics around Bean lifecycle and callbacks.

---

# 32. Common Errors

### Missing referenced Bean

```xml
<constructor-arg ref="paymentService"/>
```

but no matching Bean definition exists.

Result: dependency resolution fails.

### Wrong class name

```xml
class="com.example.DoesNotExist"
```

Result: Bean creation/configuration cannot succeed.

### Wrong property name

```xml
<property name="wrongName" .../>
```

Spring cannot apply the property if the target Bean does not expose the corresponding writable property as expected.

### Constructor mismatch

XML specifies constructor arguments that do not match an available constructor.

---

# 33. `ref` Is Not a Java `new`

This:

```xml
<constructor-arg ref="paymentService"/>
```

does **not** mean:

```java
new PaymentService();
```

It means:

> **Use the Spring-managed Bean identified by `paymentService` as the argument.**

This distinction is very important.

---

# 34. XML DI and Object Graph

Example:

```text
                    Spring Container
                           │
             ┌─────────────┴─────────────┐
             ↓                           ↓
       PaymentService              InventoryService
             │                           │
             └─────────────┬─────────────┘
                           ↓
                      OrderService
```

XML describes enough metadata for Spring to assemble this graph.

---

# 35. 2-Minute Interview Answer

> **“XML configuration is the classic Spring way of defining Beans and their dependencies externally from Java classes. We can declare a Bean using the `<bean>` element with an id and class, and wire dependencies using `constructor-arg` for constructor injection or `property` for setter injection. With `ref` we reference another Spring-managed Bean, while `value` injects a literal value. The XML file is loaded into an ApplicationContext, which reads the Bean definitions and builds the object graph. Although modern Spring Boot applications usually prefer annotations, Java configuration and auto-configuration, XML is still important for legacy Spring applications and migration projects. Spring Boot can also integrate legacy XML when required. The underlying IoC and DI concepts remain the same regardless of whether the metadata comes from XML, annotations, or Java configuration.”**

---

# 36. 30-Second Hinglish Answer

> **“XML configuration Spring ka traditional configuration approach hai jisme `<bean>` ke through Bean define karte hain aur `constructor-arg` ya `property` ke through dependency inject karte hain. `ref` ka matlab hai kisi Spring Bean ko reference karna aur `value` literal value ke liye hota hai. XML ko ApplicationContext load karta hai aur uske basis par Bean object graph create aur wire karta hai. Aaj Spring Boot mein annotations aur Java configuration zyada common hain, lekin legacy applications aur migration ke liye XML samajhna important hai.”**

---

# 37. 🧠 Memory Trick

```text
XML DI
  ↓
<bean>
  ↓
constructor-arg → Constructor Injection
  ↓
property        → Setter Injection
  ↓
ref             → Another Bean
  ↓
value           → Literal Value
  ↓
ApplicationContext
  ↓
IoC Container
```

### One-line memory

> **“Bean define karo → dependency reference karo → container object graph wire karega.”**

---

# 38. Interview Follow-Up Questions

1. What is XML configuration in Spring?
2. Why was XML configuration popular historically?
3. What is a `<bean>` element?
4. What is the difference between Bean id and class?
5. How do you load XML configuration?
6. What is `ClassPathXmlApplicationContext`?
7. How do you perform constructor injection using XML?
8. What is `constructor-arg`?
9. What is `property`?
10. `ref` vs `value`?
11. How do you inject a literal value?
12. How do you inject another Bean?
13. How do you inject a collection using XML?
14. Can XML and annotations be used together?
15. What is `<context:component-scan>`?
16. XML vs Java configuration?
17. XML vs annotation configuration?
18. Why is XML still relevant in interviews?
19. Can Spring Boot use XML configuration?
20. What is `@ImportResource`?
21. How does XML configuration participate in IoC?
22. How do you configure Bean scope in XML?
23. How do you configure lifecycle callbacks in XML?
24. What happens if a referenced Bean is missing?
25. What happens if constructor arguments don't match?
26. What is a Bean alias?
27. Can multiple Bean definitions use the same class?
28. Does XML configuration change the concept of DI?
29. Why is constructor injection preferred for mandatory dependencies?
30. Explain XML DI in 2 minutes.

---

# 🔗 Navigation

[← Previous — Dependency Inversion & DI](../09-Dependency-Inversion-and-Dependency-Injection/README.md)

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
└── 10. XML Configuration of DI                 ✅
```

**Next:** S1.11 — Spring Bean Autowiring
