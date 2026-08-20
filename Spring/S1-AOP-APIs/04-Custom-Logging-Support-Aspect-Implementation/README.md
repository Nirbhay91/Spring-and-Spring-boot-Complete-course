# S1.4.4 — Custom Logging Support Aspect Implementation

> **Status:** ✅ Completed

## 1. Goal

Is topic mein hum Spring AOP ka practical use case implement karenge: **custom logging aspect**.

Requirement:

> Service layer ke methods ko centrally log karna hai without har method ke andar manually logging code likhe.

Without AOP:

```java
public Order createOrder(Order order) {
    log.info("createOrder started");

    // business logic

    log.info("createOrder completed");
    return order;
}
```

Multiple methods ke saath ye repetitive ho jayega.

With AOP:

```text
Client
  ↓
Spring AOP Proxy
  ↓
Logging Aspect
  ↓
Target Service Method
```

Business classes mein logging concern duplicate nahi karna padega.

---

# 2. What We Will Build

```text
OrderController
      ↓
OrderService
      ↓
OrderRepository
```

Aur ek centralized aspect:

```text
LoggingAspect
      ↓
intercepts selected service methods
```

Expected log:

```text
START: OrderService.createOrder(..)
END: OrderService.createOrder(..)
```

Exception case:

```text
START: OrderService.createOrder(..)
ERROR: OrderService.createOrder(..) - Database unavailable
```

---

# 3. Dependencies ⭐⭐⭐

Spring Boot application mein AOP support ke liye commonly:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

Spring Boot starter AOP-related dependencies ko conveniently bring karta hai.

If using Spring Boot, usually separate AspectJ weaver setup ki requirement nahi hoti for ordinary Spring proxy-based AOP use cases.

---

# 4. Project Structure

```text
src/main/java/com/example/demo/
│
├── controller/
│   └── OrderController.java
│
├── service/
│   └── OrderService.java
│
├── aspect/
│   └── LoggingAspect.java
│
└── DemoApplication.java
```

Simple separation:

```text
Controller → request handling
Service    → business logic
Aspect     → cross-cutting logging
```

---

# 5. Service Class

```java
@Service
public class OrderService {

    public String createOrder() {
        System.out.println("Creating order...");
        return "ORDER_CREATED";
    }

    public String cancelOrder() {
        System.out.println("Cancelling order...");
        return "ORDER_CANCELLED";
    }
}
```

Notice:

```text
No logging code
No AOP code
Only business logic
```

This is the main benefit of separating the logging concern.

---

# 6. Basic Logging Aspect ⭐⭐⭐⭐⭐

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.demo.service..*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println(
            "START: " + joinPoint.getSignature()
        );
    }
}
```

Breakdown:

```text
@Aspect
   ↓
Class is an AOP aspect

@Component
   ↓
Spring manages the aspect bean

@Before
   ↓
Advice runs before target method

execution(...)
   ↓
Pointcut

JoinPoint
   ↓
Current invocation information
```

---

# 7. Pointcut Explained ⭐⭐⭐⭐⭐

Our expression:

```text
execution(* com.example.demo.service..*(..))
```

Break it down:

```text
execution(
    *
    com.example.demo.service..*
    (..)
)
```

Meaning:

```text
execution   → method execution pointcut
*           → any return type
service..*  → classes under service package/subpackages
(..)        → any arguments
```

So it can match methods such as:

```text
OrderService.createOrder()
OrderService.cancelOrder()
PaymentService.pay(Long, BigDecimal)
```

---

# 8. Full Logging Aspect — Before + After ⭐⭐⭐⭐⭐

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.demo.service..*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println(
            "START: " + joinPoint.getSignature()
        );
    }

    @After("execution(* com.example.demo.service..*(..))")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println(
            "END: " + joinPoint.getSignature()
        );
    }
}
```

Flow:

```text
Client
  ↓
Proxy
  ↓
@Before
  ↓
Target method
  ↓
@After
  ↓
Client
```

`@After` runs on normal and exceptional completion.

---

# 9. Better Approach — Reuse Pointcut ⭐⭐⭐⭐

Repeated expressions are difficult to maintain.

Instead:

```java
@Pointcut("execution(* com.example.demo.service..*(..))")
public void serviceMethods() {
}
```

Then:

```java
@Before("serviceMethods()")
public void logBefore(JoinPoint joinPoint) {
    // logging
}

@After("serviceMethods()")
public void logAfter(JoinPoint joinPoint) {
    // logging
}
```

Benefits:

```text
Centralized pointcut
Readable advice
Easy maintenance
Less duplication
```

---

# 10. Logging Method Arguments ⭐⭐⭐⭐⭐

`JoinPoint` se arguments retrieve kar sakte ho:

```java
@Before("serviceMethods()")
public void logBefore(JoinPoint joinPoint) {

    System.out.println(
        "Method: " + joinPoint.getSignature().getName()
    );

    System.out.println(
        "Args: " + Arrays.toString(joinPoint.getArgs())
    );
}
```

Possible output:

```text
Method: createOrder
Args: [Order{id=10, amount=500}]
```

### Security warning ⭐⭐⭐⭐⭐

Production logs mein blindly arguments log mat karo.

Avoid logging:

```text
Passwords
Access tokens
Refresh tokens
Card numbers
CVV
Secrets
PII where not required
```

Logging aspect ko sensitive-data masking policy follow karni chahiye.

---

# 11. Logging Return Value — `@AfterReturning` ⭐⭐⭐⭐

```java
@AfterReturning(
    pointcut = "serviceMethods()",
    returning = "result"
)
public void logResult(JoinPoint joinPoint, Object result) {

    System.out.println(
        "Method: " + joinPoint.getSignature().getName()
    );

    System.out.println("Result: " + result);
}
```

Flow:

```text
Target executes successfully
        ↓
Result generated
        ↓
@AfterReturning
        ↓
Result logged
```

Again, sensitive response data ko blindly log nahi karna chahiye.

---

# 12. Logging Exceptions — `@AfterThrowing` ⭐⭐⭐⭐⭐

```java
@AfterThrowing(
    pointcut = "serviceMethods()",
    throwing = "ex"
)
public void logException(
        JoinPoint joinPoint,
        Throwable ex) {

    System.err.println(
        "ERROR: "
        + joinPoint.getSignature()
        + " - "
        + ex.getMessage()
    );
}
```

Flow:

```text
Target method
      ↓
Exception
      ↓
@AfterThrowing
      ↓
Log exception
      ↓
Exception continues according to invocation flow
```

Important:

> `@AfterThrowing` ka purpose exception ko automatically swallow karna nahi hai.

---

# 13. Best Practical Version — `@Around` ⭐⭐⭐⭐⭐

A single `@Around` advice can log start, completion, duration and exceptions in one place.

```java
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    @Around("execution(* com.example.demo.service..*(..))")
    public Object logExecution(ProceedingJoinPoint joinPoint)
            throws Throwable {

        String method = joinPoint.getSignature().toShortString();
        long start = System.nanoTime();

        log.info("START: {}", method);

        try {
            Object result = joinPoint.proceed();

            long duration = System.nanoTime() - start;
            log.info("END: {} | duration={} ns", method, duration);

            return result;

        } catch (Throwable ex) {
            long duration = System.nanoTime() - start;

            log.error(
                "ERROR: {} | duration={} ns | message={}",
                method,
                duration,
                ex.getMessage()
            );

            throw ex;
        }
    }
}
```

### Why `@Around`?

Because requirement is:

```text
START
 ↓
Target
 ↓
END + duration
```

and on failure:

```text
START
 ↓
Target
 ↓
ERROR + duration
 ↓
rethrow
```

`@Around` gives us the complete invocation boundary.

---

# 14. Why `throw ex`? ⭐⭐⭐⭐⭐

A common mistake:

```java
catch (Throwable ex) {
    log.error("Error", ex);
}
```

and then method ends.

This can accidentally swallow the exception.

Better:

```java
catch (Throwable ex) {
    log.error("Error", ex);
    throw ex;
}
```

Memory:

```text
Log exception
     ↓
Don't accidentally hide exception
     ↓
Rethrow when propagation is expected
```

---

# 15. Use `@Pointcut` for Clean Design ⭐⭐⭐⭐⭐

```java
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    @Pointcut("execution(* com.example.demo.service..*(..))")
    public void serviceMethods() {
    }

    @Around("serviceMethods()")
    public Object logExecution(ProceedingJoinPoint joinPoint)
            throws Throwable {

        String method = joinPoint.getSignature().toShortString();
        long start = System.nanoTime();

        log.info("START: {}", method);

        try {
            Object result = joinPoint.proceed();

            log.info(
                "END: {} | duration={} ns",
                method,
                System.nanoTime() - start
            );

            return result;

        } catch (Throwable ex) {
            log.error(
                "ERROR: {} | duration={} ns",
                method,
                System.nanoTime() - start,
                ex
            );
            throw ex;
        }
    }
}
```

This is a clean baseline implementation.

---

# 16. Custom Annotation-Based Logging ⭐⭐⭐⭐⭐

Large projects mein package-based pointcut sometimes too broad ho sakta hai.

Custom annotation create kar sakte ho:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecution {
}
```

Then:

```java
@Service
public class PaymentService {

    @LogExecution
    public void processPayment() {
        // business logic
    }
}
```

Aspect:

```java
@Around("@annotation(com.example.demo.aspect.LogExecution)")
public Object logExecution(ProceedingJoinPoint joinPoint)
        throws Throwable {

    log.info("START: {}", joinPoint.getSignature());

    try {
        return joinPoint.proceed();
    } finally {
        log.info("END: {}", joinPoint.getSignature());
    }
}
```

### Advantage

```text
Only explicitly annotated methods
        ↓
Logging
```

This is useful when only selected methods need detailed logging.

---

# 17. Package-Based vs Annotation-Based Pointcut ⭐⭐⭐⭐⭐

| Package-based | Annotation-based |
|---|---|
| Broad coverage | Targeted coverage |
| Less annotation in business code | Requires annotation |
| Good for all service methods | Good for selected methods |
| Can produce noisy logs | Better control |

Interview answer:

> **“If every service method needs consistent logging, I can use a package-based pointcut. If only selected operations need detailed audit/logging, I prefer a custom annotation-based pointcut.”**

---

# 18. Logging Proxy Flow ⭐⭐⭐⭐⭐

```text
Controller
    ↓
Injected OrderService reference
    ↓
Spring AOP Proxy
    ↓
LoggingAspect
    ↓
proceed()
    ↓
OrderService target
    ↓
Business logic
    ↓
return / exception
    ↓
LoggingAspect
    ↓
Controller
```

Important:

> Injected service reference may be a proxy rather than the raw target object.

---

# 19. Self-Invocation Problem ⭐⭐⭐⭐⭐

Example:

```java
@Service
public class OrderService {

    public void createOrder() {
        processPayment();
    }

    @LogExecution
    public void processPayment() {
        // payment logic
    }
}
```

If `createOrder()` internally calls:

```java
processPayment();
```

the call stays inside the target object.

```text
External caller
     ↓
Proxy
     ↓
createOrder()
     ↓
processPayment()
     ↑
No second proxy crossing
```

So annotation-based proxy advice on `processPayment()` may not run.

### Better design

Move the operation to another Spring bean:

```text
OrderService
     ↓
PaymentService Proxy
     ↓
PaymentService.processPayment()
```

This makes the invocation cross the proxy boundary.

---

# 20. Logging Level Strategy ⭐⭐⭐⭐

Production application mein appropriate log levels use karo:

```text
TRACE → very detailed diagnostics
DEBUG → developer troubleshooting
INFO  → important application flow
WARN  → suspicious/unexpected situation
ERROR → failures requiring attention
```

Typical service logging:

```java
log.info("Order created: {}", orderId);
```

Exception:

```java
log.error("Order creation failed: {}", orderId, ex);
```

Avoid:

```java
System.out.println(...)
```

for production application logging.

---

# 21. Structured Logging ⭐⭐⭐⭐

Instead of giant string concatenation:

```java
log.info("Order {} created for user {}", orderId, userId);
```

Prefer structured/parameterized logging supported by your logging stack.

In distributed microservices, useful contextual fields include:

```text
traceId
spanId
requestId
userId (if appropriate and safe)
service
operation
```

Sensitive information should be excluded or masked.

---

# 22. Logging vs Auditing ⭐⭐⭐⭐⭐

Don't confuse:

```text
Technical Logging
vs
Business Audit
```

### Logging

Used for:

```text
Debugging
Monitoring
Troubleshooting
Operational visibility
```

### Auditing

Used for:

```text
Who performed an action?
What changed?
When did it change?
What was the business impact?
```

AOP can support both, but audit requirements usually need stricter data-retention, integrity and security controls.

---

# 23. Logging Aspect Should Not Become Business Logic ⭐⭐⭐⭐

Bad design:

```java
@Around("serviceMethods()")
public Object log(ProceedingJoinPoint pjp) throws Throwable {
    // logging
    // database business update
    // payment calculation
    // notification
    // logging
}
```

Aspect ka responsibility focused rakho.

Better:

```text
LoggingAspect → logging/observability
PaymentService → payment business logic
OrderService   → order business logic
```

---

# 24. Performance Considerations ⭐⭐⭐⭐

AOP logging har method par enable karne se:

```text
More interception
More log statements
More formatting/serialization
More I/O
```

ho sakta hai.

Therefore:

- Avoid excessive DEBUG/INFO logs in hot paths.
- Don't log huge objects unnecessarily.
- Avoid sensitive payloads.
- Use parameterized logging.
- Use sampling where appropriate for high-volume observability systems.
- Keep pointcuts as narrow as practical.

---

# 25. Important Production Consideration — Async Logging

High-throughput applications mein synchronous logging overhead reduce karne ke liye logging framework/appender configuration mein asynchronous mechanisms use kiye ja sakte hain.

But:

```text
Async logging ≠ AOP
```

AOP decides **where/how logging behavior is injected**.

Logging framework decides **how log events are processed/written**.

---

# 26. Custom Logging Aspect — Recommended Baseline ⭐⭐⭐⭐⭐

```java
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    @Pointcut("execution(* com.example.demo.service..*(..))")
    public void serviceMethods() {
    }

    @Around("serviceMethods()")
    public Object logExecution(ProceedingJoinPoint pjp)
            throws Throwable {

        String method = pjp.getSignature().toShortString();
        long start = System.nanoTime();

        log.info("START: {}", method);

        try {
            Object result = pjp.proceed();

            log.info(
                "END: {} | duration={} ns",
                method,
                System.nanoTime() - start
            );

            return result;

        } catch (Throwable ex) {
            log.error(
                "ERROR: {} | duration={} ns",
                method,
                System.nanoTime() - start,
                ex
            );

            throw ex;
        }
    }
}
```

### Why this design?

```text
@Aspect
→ cross-cutting module

@Component
→ Spring-managed aspect

@Pointcut
→ centralized selection

@Around
→ complete invocation boundary

ProceedingJoinPoint
→ target invocation control

try/catch
→ success + failure logging

throw ex
→ preserve failure semantics
```

---

# 27. Interview Scenario ⭐⭐⭐⭐⭐

### Question

> “How would you implement centralized logging for all service methods without modifying each service?”

### Answer

```text
1. Add spring-boot-starter-aop.
2. Create @Aspect + @Component.
3. Define a service-layer pointcut.
4. Use @Around advice.
5. Log method name and start time.
6. Call proceed().
7. Log duration on success.
8. Log exception on failure.
9. Rethrow the exception.
10. Avoid logging secrets/sensitive payloads.
```

Example:

```java
@Around("serviceMethods()")
public Object log(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.nanoTime();
    try {
        return pjp.proceed();
    } finally {
        log.info(
            "{} took {} ns",
            pjp.getSignature().toShortString(),
            System.nanoTime() - start
        );
    }
}
```

If exception-specific logging is required, catch and rethrow as shown earlier.

---

# 28. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1
**“AOP logging means putting logger in every service.”**

❌ No. Centralized aspect ka purpose hi duplication remove karna hai.

### Trap 2
**“`@After` only runs after successful execution.”**

❌ No. It also applies on exceptional completion.

### Trap 3
**“`@AfterThrowing` handles the exception automatically.”**

❌ It observes/intercepts exceptional completion; it does not automatically remove the exception from the flow.

### Trap 4
**“`@Around` always has to call `proceed()`.”**

❌ It can intentionally skip target invocation.

### Trap 5
**“Logging all method arguments is safe.”**

❌ Never blindly log passwords, tokens, card data or secrets.

### Trap 6
**“AOP can intercept self-invocation normally.”**

❌ Normal proxy-based Spring AOP self-invocation bypasses the proxy.

### Trap 7
**“AOP and application logging framework are the same.”**

❌ AOP provides interception/injection; the logging framework writes/processes log events.

---

# 29. Interview Follow-Up Questions

1. How do you implement centralized logging using Spring AOP?
2. Why use `@Around` for logging?
3. Why use `ProceedingJoinPoint`?
4. What happens if `proceed()` is not called?
5. How do you log execution time?
6. How do you log method arguments?
7. How do you capture return values?
8. How do you capture exceptions?
9. Why should an aspect rethrow exceptions?
10. How do you define a reusable pointcut?
11. Package-based vs annotation-based pointcut?
12. How do you prevent sensitive data from appearing in logs?
13. What is the self-invocation problem?
14. How do you solve self-invocation for AOP?
15. How does Spring create the logging proxy?
16. How would you reduce logging overhead?
17. Logging vs auditing?
18. Why avoid `System.out.println` in production?
19. How would you add traceId/requestId to logs?
20. How would you implement logging for only selected methods?
21. What happens if the logging aspect itself throws an exception?
22. How would you test a logging aspect?
23. How would you avoid logging large response objects?
24. Explain custom logging AOP implementation in 2 minutes.

---

# 30. Testing Strategy ⭐⭐⭐⭐

Aspect ko sirf manually verify karne ke bajay integration test se verify karna better hai.

Test cases:

```text
1. Successful method
   → START + END log

2. Exception method
   → START + ERROR log
   → exception still propagates

3. Return value
   → expected result returned

4. Pointcut boundary
   → selected service method intercepted
   → non-selected method not intercepted

5. Sensitive data
   → secrets are masked/not logged
```

For log verification, test setup ke according logging appender/test logger approach use kiya ja sakta hai.

---

# 31. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“For centralized logging in Spring Boot, I would use Spring AOP. First I add `spring-boot-starter-aop`, then create an `@Aspect` and make it a Spring bean. I define a pointcut for the service layer and use `@Around` advice because I need to log both before and after the method and also measure execution time. Inside the advice I capture the start time, log the method name, call `proceed()`, log successful completion and duration, and catch exceptions to log the failure before rethrowing the exception so the original application behavior is preserved. For larger applications, I may use a custom annotation instead of a broad package pointcut when only selected operations should be logged. I also avoid logging passwords, tokens, card data or other sensitive information. Since Spring AOP is proxy-based, I also consider self-invocation limitations and keep the logging aspect focused on observability rather than business logic.”**

---

# 32. 30-Second Hinglish Answer

> **“Centralized logging ke liye Spring AOP mein `@Aspect` banayenge aur service methods ke liye pointcut define karenge. Logging ke liye `@Around` use karna best hai kyunki method ke before aur after dono side ka control milta hai. `ProceedingJoinPoint.proceed()` se actual method execute karenge, execution time calculate karenge aur exception aane par log karke exception ko rethrow karenge. Production mein password, token aur sensitive data log nahi karna chahiye. Self-invocation ka bhi dhyan rakhna hai kyunki proxy bypass ho sakta hai.”**

---

# 🧠 Complete Memory Map

```text
Custom Logging AOP
│
├── Dependency
│   └── spring-boot-starter-aop
│
├── Aspect
│   └── @Aspect + @Component
│
├── Pointcut
│   ├── service package
│   └── OR custom annotation
│
├── Advice
│   └── @Around
│
├── Invocation
│   └── ProceedingJoinPoint.proceed()
│
├── Success
│   └── END + duration
│
├── Failure
│   └── ERROR + rethrow
│
└── Production Safety
    ├── No secrets
    ├── No sensitive payloads
    ├── Parameterized logging
    └── Avoid excessive logs
```

### One-line memory

> **“Custom Logging Aspect = Pointcut se methods select karo → Around advice mein START log karo → `proceed()` → duration/result log karo → exception par ERROR log karke rethrow karo.”**

---

## Navigation

[← S1.4.3 — Types of Advice](../03-Types-of-Advice/README.md)

[↗ S1.4 — Spring AOP APIs](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

**Status: ✅ Completed**

**Next:** Continue with the next topic in the source sequence.
