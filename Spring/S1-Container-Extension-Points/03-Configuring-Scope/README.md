# Spring S1.2 — Container Extension Points

## 2.3 Configuring Scope

> **Status:** ✅ Completed

## 1. What does Configuring Scope mean?

Spring Bean scope decide karta hai ki Bean instance **kitni baar create hogi aur kis lifecycle boundary ke andar manage hogi**.

Configuring scope ka matlab hai Spring ko explicitly batana ki Bean ko kaunse scope mein manage karna hai.

Default:

```text
@Component → Singleton
```

Agar different behavior chahiye, scope configure karte hain.

---

## 2. Ways to Configure Scope ⭐

Spring mein commonly scope configure karne ke ways:

```text
1. @Scope annotation
2. @RequestScope / @SessionScope etc.
3. @Bean + @Scope
4. XML configuration
5. Scoped Proxy
6. Custom scope (advanced)
```

---

## 3. `@Scope` with `@Component`

Prototype configure karne ka common example:

```java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

Now:

```java
ReportGenerator r1 = context.getBean(ReportGenerator.class);
ReportGenerator r2 = context.getBean(ReportGenerator.class);

System.out.println(r1 == r2); // false
```

### Memory

```text
No @Scope → Singleton
@Scope("prototype") → New instance on lookup
```

---

## 4. Singleton Explicitly Configure Karna

Although singleton default hai, explicitly bhi likh sakte ho:

```java
@Component
@Scope("singleton")
public class PaymentService {
}
```

Usually unnecessary because singleton already default hai.

Interview point:

> `@Scope("singleton")` is explicit configuration; it is not required for the default behavior.

---

## 5. `@Bean` + `@Scope`

Java-based configuration mein:

```java
@Configuration
public class AppConfig {

    @Bean
    @Scope("prototype")
    public ReportGenerator reportGenerator() {
        return new ReportGenerator();
    }
}
```

Now container lookup par new instance milega.

```java
context.getBean(ReportGenerator.class);
context.getBean(ReportGenerator.class);
```

→ Different instances.

---

## 6. XML Configuration

Legacy Spring applications mein XML use hota tha.

```xml
<bean id="reportGenerator"
      class="com.example.ReportGenerator"
      scope="prototype"/>
```

Singleton:

```xml
<bean id="paymentService"
      class="com.example.PaymentService"
      scope="singleton"/>
```

### Interview comparison

```text
Modern Spring → annotations / Java configuration
Legacy Spring → XML configuration
```

---

## 7. Web Scope Annotations ⭐

Web applications mein scope strings manually likhne ke bajay dedicated annotations use kar sakte hain.

### Request Scope

```java
@Component
@RequestScope
public class RequestContext {
}
```

Equivalent concept:

```java
@Scope("request")
```

### Session Scope

```java
@Component
@SessionScope
public class UserSession {
}
```

### Application Scope

```java
@Component
@ApplicationScope
public class ApplicationState {
}
```

These are intended for web-aware application contexts.

---

## 8. Request-Scoped Bean Inside Singleton ⭐

Suppose:

```java
@RequestScope
@Component
public class RequestContext {
}
```

and:

```java
@Component
public class OrderService {

    private final RequestContext requestContext;

    public OrderService(RequestContext requestContext) {
        this.requestContext = requestContext;
    }
}
```

Problem:

```text
OrderService → Singleton
RequestContext → Request scoped
```

Different lifetimes hain.

Spring needs a mechanism that resolves the correct request-scoped instance when the singleton uses it.

This is where **scoped proxy** can be useful.

---

## 9. Scoped Proxy ⭐

Scoped proxy ek proxy object provide karta hai jo actual scoped Bean ko current scope ke according resolve karta hai.

Example:

```java
@Component
@RequestScope
public class RequestContext {
}
```

Conceptually:

```text
Singleton OrderService
        ↓
   Proxy reference
        ↓
Current HTTP Request
        ↓
RequestContext instance
```

The proxy allows a longer-lived Bean to refer to a shorter-lived scoped object.

---

## 10. `@Scope` with Proxy Mode

Example:

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestContext {
}
```

Common proxy modes:

```java
ScopedProxyMode.NO
ScopedProxyMode.INTERFACES
ScopedProxyMode.TARGET_CLASS
```

### `TARGET_CLASS`

Creates a class-based proxy, commonly using CGLIB-based proxying in Spring's infrastructure.

### `INTERFACES`

Uses an interface-based proxy and therefore requires the Bean to expose suitable interfaces.

### `NO`

No scoped proxy is created.

---

## 11. `@RequestScope` and Proxy Mode

Dedicated web scope annotations can also be configured with proxy behavior where needed.

Conceptually:

```java
@RequestScope
@Component
public class RequestContext {
}
```

If proxy behavior is needed for injection into a longer-lived Bean, configure the appropriate scoped proxy strategy.

---

## 12. Scope vs Proxy — Important Difference

Don't confuse:

```text
Scope       → Defines lifecycle/instance boundary
Proxy       → Helps access a scoped instance from another lifecycle
```

Example:

```text
Request Scope
     ↓
One object per request

Scoped Proxy
     ↓
Allows singleton to safely refer to current request object
```

---

## 13. `ObjectProvider` as Alternative ⭐

Instead of relying on a scoped proxy, explicit lookup can be useful.

```java
@Component
public class OrderService {

    private final ObjectProvider<RequestContext> provider;

    public OrderService(ObjectProvider<RequestContext> provider) {
        this.provider = provider;
    }

    public void process() {
        RequestContext context = provider.getObject();
    }
}
```

For prototype scope, this is also useful:

```java
ObjectProvider<ReportGenerator> provider;

ReportGenerator generator = provider.getObject();
```

### When to prefer it?

When explicit, programmatic lookup makes the lifecycle dependency clearer.

---

## 14. `@Lookup` as Another Option

Spring can dynamically resolve a shorter-lived Bean using `@Lookup`.

```java
@Component
public abstract class OrderService {

    public void process() {
        RequestContext context = getRequestContext();
    }

    @Lookup
    protected abstract RequestContext getRequestContext();
}
```

This delegates lookup to the Spring container.

---

## 15. Custom Scope ⭐

Spring supports custom scopes through the `Scope` interface.

Conceptually:

```java
public interface Scope {
    Object get(String name, ObjectFactory<?> objectFactory);
    Object remove(String name);
    void registerDestructionCallback(String name, Runnable callback);
    Object resolveContextualObject(String key);
    String getConversationId();
}
```

A custom scope can define its own instance/lifecycle boundary.

Examples of conceptual custom boundaries:

```text
Tenant scope
Job scope
Conversation scope
```

Custom scopes are advanced and should be introduced only when a real application requirement exists.

---

## 16. Registering a Custom Scope

A custom scope can be registered with the BeanFactory's scope registry.

Conceptually:

```java
ConfigurableBeanFactory beanFactory = ...;
beanFactory.registerScope("tenant", new TenantScope());
```

Then:

```java
@Scope("tenant")
@Component
public class TenantContext {
}
```

The actual registration mechanism depends on application architecture and startup configuration.

---

## 17. Choosing the Right Configuration

### Simple application service

```java
@Component
class PaymentService {}
```

→ Singleton by default.

### New object per lookup

```java
@Scope("prototype")
```

→ Prototype.

### Per HTTP request

```java
@RequestScope
```

→ Request scope.

### Per HTTP session

```java
@SessionScope
```

→ Session scope.

### Short-lived Bean required by singleton

```text
ObjectProvider / @Lookup / suitable proxy
```

→ Explicitly solve the lifecycle mismatch.

---

## 18. Scope Configuration in Spring Boot ⭐

Spring Boot does not change the basic Spring Bean scope rules.

Example:

```java
@Service
@Scope("prototype")
public class ReportService {
}
```

`@Service` is a stereotype for a Spring-managed component, while `@Scope` controls its scope.

Default remains singleton unless configured otherwise.

---

## 19. Common Mistakes

### Mistake 1
Thinking every `@Component` is prototype.

❌ Wrong.

```text
@Component → singleton by default
```

### Mistake 2
Using request/session scope without a web-aware context.

❌ Scope needs the appropriate web environment.

### Mistake 3
Thinking proxy changes the Bean's actual scope.

❌ Proxy does not redefine the scope; it provides an access mechanism to the scoped target.

### Mistake 4
Using prototype simply for thread safety.

❌ Scope alone is not a thread-safety strategy.

### Mistake 5
Using custom scopes without a real requirement.

❌ Adds unnecessary infrastructure complexity.

---

## 20. Interview Follow-Up Questions

1. How do you configure Bean scope in Spring?
2. What is `@Scope`?
3. What is the default Bean scope?
4. How do you configure prototype scope?
5. How do you configure singleton scope?
6. How do you configure request scope?
7. How do you configure session scope?
8. What are `@RequestScope` and `@SessionScope`?
9. What happens when a request-scoped Bean is injected into a singleton?
10. What is scoped proxy?
11. What is `ScopedProxyMode.TARGET_CLASS`?
12. What is `ScopedProxyMode.INTERFACES`?
13. Difference between scope and proxy?
14. ObjectProvider vs scoped proxy?
15. What is `@Lookup`?
16. Can you define a custom Spring scope?
17. How do you register a custom scope?
18. Does Spring Boot change Bean scope defaults?
19. Does scope make a Bean thread-safe?
20. When should you use prototype scope?
21. Why are stateless services usually singleton?
22. XML scope configuration vs annotation configuration?
23. How would you solve a prototype-in-singleton lifecycle mismatch?
24. Explain scoped proxy in an interview.
25. Explain Bean scope configuration in 2 minutes.

---

## 21. 2-Minute Interview Answer ⭐

> **“Spring Bean scope can be configured using `@Scope`, dedicated web-scope annotations such as `@RequestScope` and `@SessionScope`, Java configuration with `@Bean`, or XML in legacy applications. Singleton is the default. For a prototype Bean we can use `@Scope("prototype")`. When a shorter-lived Bean such as a request-scoped Bean is needed by a longer-lived singleton, we can use a scoped proxy or explicit lookup mechanisms such as `ObjectProvider` or `@Lookup`. A scoped proxy doesn't change the actual scope; it provides a proxy that resolves the correct scoped target. Spring also supports custom scopes through the `Scope` interface, but they should be used only when there is a real lifecycle boundary that the standard scopes cannot model.”**

---

## 22. 30-Second Hinglish Answer

> **“Spring mein Bean scope configure karne ke liye mainly `@Scope` use karte hain. Default singleton hota hai, aur prototype ke liye `@Scope("prototype")` use kar sakte hain. Web applications mein `@RequestScope` aur `@SessionScope` jaise annotations available hain. Agar singleton ko request ya prototype jaise shorter-lived Bean ki zarurat ho, to scoped proxy, `ObjectProvider` ya `@Lookup` use kar sakte hain. Scope lifecycle boundary define karta hai, jabki proxy scoped object ko access karne ka mechanism deta hai.”**

---

## 🧠 Memory Trick

```text
@Scope("singleton")  → SAME
@Scope("prototype")  → NEW
@RequestScope         → REQUEST
@SessionScope         → SESSION
Proxy                 → CURRENT SCOPED OBJECT
ObjectProvider        → EXPLICIT LOOKUP
```

### One-line memory

> **“Scope defines the boundary; proxy/provider solves the access when lifetimes differ.”**

---

## Navigation

[← S1.2.2 Singleton, Prototype, and Other Scopes](../02-Singleton-Prototype-and-Other-Scopes/README.md)

[↗ S1.2 Container Extension Points](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** S1.2.4 — Bean Lifecycle / Callbacks
