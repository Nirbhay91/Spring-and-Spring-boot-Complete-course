# S1.4.3 — Types of Advice

> **Status:** ✅ Completed

## 1. What is Advice? ⭐⭐⭐⭐⭐

**Advice** wo actual behavior/code hai jo selected join point par execute hota hai.

Simple memory:

```text
Pointcut = WHERE?
Advice   = WHAT?
```

Example:

```java
@Before("execution(* com.example.service..*(..))")
public void logBefore() {
    System.out.println("Before method execution");
}
```

Yahan:

```text
@Before       → Advice type
execution(...) → Pointcut
logBefore()   → Advice body
```

Spring AOP mein commonly five advice types discuss kiye jaate hain:

```text
@Before
@After
@AfterReturning
@AfterThrowing
@Around
```

---

# 2. Advice Types — Quick Comparison ⭐⭐⭐⭐⭐

| Advice | Kab execute hota hai? | Target method control? |
|---|---|---|
| `@Before` | Target method se pehle | ❌ No |
| `@After` | Completion ke baad, success/exception dono cases mein | ❌ No |
| `@AfterReturning` | Successful normal return ke baad | ❌ No |
| `@AfterThrowing` | Exception throw hone par | ❌ No |
| `@Around` | Target invocation ke around | ✅ Yes |

### Memory

```text
BEFORE          → pehle
AFTER           → completion ke baad
AFTER RETURNING → successful return ke baad
AFTER THROWING  → exception ke baad
AROUND          → complete control/wrapper
```

---

# 3. `@Before` Advice ⭐⭐⭐⭐⭐

`@Before` advice target method execute hone se **pehle** run hota hai.

Example:

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service..*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Calling: " + joinPoint.getSignature());
    }
}
```

### Flow

```text
Client
  ↓
Proxy
  ↓
@Before advice
  ↓
Target method
  ↓
Return
```

### Typical use cases

```text
Logging
Authorization checks
Audit preparation
Input inspection
Metrics start marker
```

### Important limitation

`@Before` advice target method ko directly control nahi karta.

Agar requirement hai:

> “Target method ko execute hi mat karo.”

then `@Around` is generally the more appropriate advice because it can control whether `proceed()` is called.

---

# 4. `@After` Advice ⭐⭐⭐⭐⭐

`@After` advice target method ke **completion ke baad** execute hota hai, chahe method normally return kare ya exception ke through complete ho.

Example:

```java
@After("execution(* com.example.service..*(..))")
public void cleanup() {
    System.out.println("Method completed");
}
```

### Flow — Success

```text
Target method
    ↓
Return
    ↓
@After
```

### Flow — Exception

```text
Target method
    ↓
Exception
    ↓
@After
```

### Typical use cases

```text
Cleanup
Resource-related final actions
Audit completion marker
Generic completion logging
```

### Important

`@After` ko `finally` ke conceptual equivalent ke roop mein yaad kar sakte ho, because it is intended for behavior that runs on both normal and exceptional completion.

---

# 5. `@AfterReturning` Advice ⭐⭐⭐⭐⭐

`@AfterReturning` sirf tab execute hota hai jab target method **normally return** karta hai.

Example:

```java
@AfterReturning(
    pointcut = "execution(* com.example.service..*(..))",
    returning = "result"
)
public void afterReturning(Object result) {
    System.out.println("Result = " + result);
}
```

### Flow — Success

```text
Target method
    ↓
Return result
    ↓
@AfterReturning
```

### Flow — Exception

```text
Target method
    ↓
Exception
    ↓
@AfterReturning ❌
```

### Typical use cases

```text
Successful operation logging
Returned-value auditing
Post-processing
Success metrics
```

---

# 6. Capturing Return Value ⭐⭐⭐⭐

`returning` attribute se returned value advice method mein capture kar sakte ho.

```java
@AfterReturning(
    pointcut = "execution(* com.example.service..*(..))",
    returning = "result"
)
public void afterReturning(Object result) {
    System.out.println(result);
}
```

Parameter name match hona chahiye:

```text
returning = "result"
              ↓
method parameter = result
```

Type ko more specific bhi rakh sakte ho when appropriate:

```java
public void afterReturning(Order result) {
}
```

---

# 7. `@AfterThrowing` Advice ⭐⭐⭐⭐⭐

`@AfterThrowing` tab execute hota hai jab target method exception throw karke exit karta hai.

Example:

```java
@AfterThrowing(
    pointcut = "execution(* com.example.service..*(..))",
    throwing = "ex"
)
public void afterThrowing(Exception ex) {
    System.out.println("Exception: " + ex.getMessage());
}
```

### Flow

```text
Target method
    ↓
Exception
    ↓
@AfterThrowing
```

Normal return case:

```text
Target method
    ↓
Success
    ↓
@AfterThrowing ❌
```

### Typical use cases

```text
Error logging
Alerting
Failure metrics
Audit failure events
```

---

# 8. Capturing the Exception ⭐⭐⭐⭐

`throwing` attribute exception ko advice method mein bind karta hai.

```java
@AfterThrowing(
    pointcut = "execution(* com.example.service..*(..))",
    throwing = "ex"
)
public void handleException(Exception ex) {
    System.out.println(ex.getMessage());
}
```

Memory:

```text
returning → returned value
throwing  → thrown exception
```

---

# 9. `@Around` Advice ⭐⭐⭐⭐⭐

`@Around` sabse powerful advice type hai because ye target invocation ko **surround** karta hai aur `proceed()` ke through target execution ko control kar sakta hai.

Example:

```java
@Around("execution(* com.example.service..*(..))")
public Object around(ProceedingJoinPoint pjp) throws Throwable {

    System.out.println("Before");

    Object result = pjp.proceed();

    System.out.println("After");

    return result;
}
```

### Flow

```text
Client
  ↓
Proxy
  ↓
@Around - before
  ↓
proceed()
  ↓
Target method
  ↓
@Around - after
  ↓
Return
```

---

# 10. Why `ProceedingJoinPoint`? ⭐⭐⭐⭐⭐

`@Around` advice mein:

```java
ProceedingJoinPoint
```

use hota hai.

Target method execute karne ke liye:

```java
pjp.proceed();
```

### Important

`@Around` advice target method ko:

```text
Execute once
Execute conditionally
Execute later
Skip
Potentially invoke more than once
```

karne ki capability deta hai, although invoking more than once is generally dangerous unless deliberately designed.

Example skip:

```java
@Around("execution(* com.example.service..*(..))")
public Object around(ProceedingJoinPoint pjp) throws Throwable {

    if (!isAuthorized()) {
        throw new SecurityException("Access denied");
    }

    return pjp.proceed();
}
```

Agar authorization fail hua:

```text
proceed() call nahi hua
        ↓
Target method execute nahi hua
```

---

# 11. `@Around` Can Modify Return Value ⭐⭐⭐⭐

Example:

```java
@Around("execution(* com.example.service..*(..))")
public Object around(ProceedingJoinPoint pjp) throws Throwable {

    Object result = pjp.proceed();

    return "Modified: " + result;
}
```

Conceptually:

```text
Target returns A
      ↓
Around receives A
      ↓
Around returns Modified A
```

Return type compatibility ka dhyan rakhna important hai.

---

# 12. `@Around` Can Modify Arguments ⭐⭐⭐⭐

`ProceedingJoinPoint` ke through invocation arguments inspect kiye ja sakte hain, and suitable use cases mein altered arguments ke saath proceed kiya ja sakta hai.

Example concept:

```java
Object[] args = pjp.getArgs();
```

Then invocation can be controlled using appropriate `proceed(...)` semantics.

### Caution

Argument mutation ko blindly use nahi karna chahiye. It can make code difficult to reason about.

---

# 13. `@Around` Can Handle Exceptions ⭐⭐⭐⭐

```java
@Around("execution(* com.example.service..*(..))")
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    try {
        return pjp.proceed();
    } catch (Exception ex) {
        System.out.println("Logging exception");
        throw ex;
    }
}
```

Important:

> Logging ke liye exception swallow karna generally wrong hai. Agar exception application flow mein propagate karna chahiye, usko rethrow karo.

---

# 14. Complete Advice Execution Order ⭐⭐⭐⭐⭐

Suppose same join point par multiple advice configured hain.

Conceptual execution:

```text
@Around before
      ↓
@Before
      ↓
Target method
      ↓
@AfterReturning / @AfterThrowing
      ↓
@After
      ↓
@Around after
```

### Success path

```text
Around before
    ↓
Before
    ↓
Target
    ↓
AfterReturning
    ↓
After
    ↓
Around after
```

### Exception path

```text
Around before
    ↓
Before
    ↓
Target throws
    ↓
AfterThrowing
    ↓
After
    ↓
Exception propagates
```

**Note:** Actual ordering among multiple aspects/advisors depends on their ordering configuration (`@Order`, `Ordered`, etc.). Do not assume declaration order is universally the execution order.

---

# 15. `@After` vs `@AfterReturning` vs `@AfterThrowing` ⭐⭐⭐⭐⭐

| Situation | `@After` | `@AfterReturning` | `@AfterThrowing` |
|---|---:|---:|---:|
| Normal return | ✅ | ✅ | ❌ |
| Exception | ✅ | ❌ | ✅ |
| Get return value | ❌ | ✅ | ❌ |
| Get exception | ❌ | ❌ | ✅ |

### Memory

```text
SUCCESS
  ├── @AfterReturning
  └── @After

EXCEPTION
  ├── @AfterThrowing
  └── @After
```

---

# 16. `@Before` vs `@Around` ⭐⭐⭐⭐⭐

| `@Before` | `@Around` |
|---|---|
| Runs before target | Wraps target invocation |
| Simpler | More powerful |
| Cannot decide whether target runs | Can control `proceed()` |
| Cannot directly replace return value | Can modify return value |
| Good for simple pre-processing | Good for timing, transactions, authorization, etc. |

### Interview line

> **“Use `@Before` when you only need pre-processing. Use `@Around` when you need control over the target invocation.”**

---

# 17. `@After` vs `finally` ⭐⭐⭐⭐

Conceptually:

```java
try {
    target();
} finally {
    cleanup();
}
```

is similar to the intention of:

```java
@After
```

Both are associated with completion regardless of success or exception.

But don't claim they are literally the same implementation mechanism.

---

# 18. Real-World Example — Logging ⭐⭐⭐

Requirement:

> Every service method ka start log karo.

Use:

```java
@Before("execution(* com.example.service..*(..))")
```

Flow:

```text
Request
 ↓
@Before log
 ↓
Service method
```

---

# 19. Real-World Example — Execution Time ⭐⭐⭐⭐⭐

Requirement:

> Service method ka execution time measure karna hai.

Use:

```java
@Around("execution(* com.example.service..*(..))")
public Object measure(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.nanoTime();

    Object result = pjp.proceed();

    long duration = System.nanoTime() - start;
    System.out.println("Time = " + duration);

    return result;
}
```

Why `@Around`?

Because we need:

```text
start timer
   ↓
execute target
   ↓
stop timer
```

---

# 20. Real-World Example — Error Logging ⭐⭐⭐

Requirement:

> Service exception ko centrally log karna hai.

Use:

```java
@AfterThrowing(
    pointcut = "execution(* com.example.service..*(..))",
    throwing = "ex"
)
public void logError(Exception ex) {
    System.err.println(ex.getMessage());
}
```

---

# 21. Real-World Example — Successful Response Audit ⭐⭐⭐

Requirement:

> Successful operation ka result audit karna hai.

Use:

```java
@AfterReturning(
    pointcut = "execution(* com.example.service..*(..))",
    returning = "result"
)
public void audit(Object result) {
    System.out.println("Successful result: " + result);
}
```

---

# 22. Real-World Example — Authorization ⭐⭐⭐⭐⭐

Requirement:

> Unauthorized user ke liye target method execute hi nahi hona chahiye.

`@Around` useful hai:

```java
@Around("execution(* com.example.service.secure..*(..))")
public Object checkAccess(ProceedingJoinPoint pjp) throws Throwable {

    if (!hasAccess()) {
        throw new SecurityException("Access denied");
    }

    return pjp.proceed();
}
```

Flow:

```text
Client
 ↓
Proxy
 ↓
Around advice
 ↓
Access check
 ├── fail → exception
 └── pass → proceed()
                ↓
             Target
```

---

# 23. Around Advice — Important `proceed()` Rules ⭐⭐⭐⭐⭐

### Rule 1

If `proceed()` is never called:

```text
Target method does not execute
```

### Rule 2

If `proceed()` is called once:

```text
Target normally executes once
```

### Rule 3

Calling `proceed()` multiple times can invoke the target multiple times and can cause duplicate side effects.

Example dangerous scenario:

```text
DB insert
 ↓
proceed()
 ↓
DB insert again
```

So normally:

```java
Object result = pjp.proceed();
```

once call karna expected pattern hai.

---

# 24. Checked Exception in `@Around` ⭐⭐⭐⭐

`ProceedingJoinPoint.proceed()` throws `Throwable`, so common implementation is:

```java
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    return pjp.proceed();
}
```

This is why many `@Around` examples use:

```java
throws Throwable
```

---

# 25. Advice Ordering ⭐⭐⭐⭐⭐

AOP application mein multiple aspects ho sakte hain:

```text
SecurityAspect
LoggingAspect
TransactionAspect
PerformanceAspect
```

Question:

> “Kaunsa pehle execute hoga?”

Ordering configure ki ja sakti hai using mechanisms such as:

```java
@Order(1)
```

or:

```java
implements Ordered
```

Example:

```java
@Aspect
@Order(1)
@Component
public class SecurityAspect {
}
```

```java
@Aspect
@Order(2)
@Component
public class LoggingAspect {
}
```

Lower order value generally means higher precedence for ordering purposes.

For around advice, the resulting nesting means the higher-precedence advice can surround lower-precedence advice.

---

# 26. Important Interview Scenario — Transaction + Logging ⭐⭐⭐⭐⭐

Suppose:

```text
Security
Logging
Transaction
Business method
```

Interviewer asks:

> “How do you guarantee the exact execution order?”

Answer:

> **“I explicitly define aspect ordering using `@Order` or `Ordered` rather than relying on accidental ordering.”**

Example:

```text
Security @Order(1)
   ↓
Logging @Order(2)
   ↓
Transaction @Order(3)
   ↓
Business method
```

Exact nesting should be validated based on the configured advisors and advice types, but explicit ordering is the key principle.

---

# 27. Advice Selection Cheat Sheet ⭐⭐⭐⭐⭐

| Requirement | Best starting point |
|---|---|
| Log before method | `@Before` |
| Cleanup after success/failure | `@After` |
| Process successful result | `@AfterReturning` |
| Log exceptions | `@AfterThrowing` |
| Measure execution time | `@Around` |
| Block method execution | `@Around` |
| Modify arguments | `@Around` |
| Modify return value | `@Around` |
| Control target invocation | `@Around` |

### One-line memory

> **“Simple before → Before, completion → After, success → AfterReturning, exception → AfterThrowing, control → Around.”**

---

# 28. Common Interview Traps ⭐⭐⭐⭐⭐

### Trap 1
**`@After` means only successful completion.**

❌ Wrong.

`@After` applies after completion whether normal or exceptional.

### Trap 2
**`@AfterReturning` runs when an exception occurs.**

❌ Wrong.

It is for normal successful return.

### Trap 3
**`@AfterThrowing` catches and removes the exception.**

❌ Not automatically.

It is notification/interception behavior; exception propagation still follows the method/advice semantics unless explicitly handled elsewhere.

### Trap 4
**`@Before` can skip the target method.**

❌ Not by simply returning from the advice.

For conditional execution control, use `@Around` or another appropriate mechanism.

### Trap 5
**`@Around` must call `proceed()`.**

❌ Not strictly.

It can intentionally skip the target.

### Trap 6
**`@Around` should always call `proceed()` multiple times.**

❌ Dangerous.

Multiple calls can execute side effects multiple times.

### Trap 7
**Advice order is based on source-code order.**

❌ Don't rely on it.

Use explicit ordering.

---

# 29. Interview Follow-Up Questions

1. What is Advice in Spring AOP?
2. What are the five types of advice?
3. Explain `@Before`.
4. Explain `@After`.
5. Explain `@AfterReturning`.
6. Explain `@AfterThrowing`.
7. Explain `@Around`.
8. Difference between `@After` and `@AfterReturning`?
9. Difference between `@After` and `@AfterThrowing`?
10. Difference between `@Before` and `@Around`?
11. Why does `@Around` use `ProceedingJoinPoint`?
12. What happens if `proceed()` is not called?
13. Can `@Around` modify the return value?
14. Can `@Around` modify arguments?
15. Can `@AfterReturning` access the return value?
16. Can `@AfterThrowing` access the exception?
17. What happens when target method throws an exception?
18. What is the execution order of different advice types?
19. How do you order multiple aspects?
20. What is `@Order`?
21. Why is multiple `proceed()` dangerous?
22. Which advice would you choose for performance monitoring and why?
23. Which advice would you choose to block unauthorized calls and why?
24. Explain all advice types in 2 minutes.

---

# 30. 2-Minute Interview Answer ⭐⭐⭐⭐⭐

> **“Spring AOP commonly provides five types of advice: Before, After, AfterReturning, AfterThrowing and Around. `@Before` runs before the target method and is useful for logging or pre-processing. `@After` runs after method completion regardless of whether it completed normally or with an exception, so it is useful for cleanup. `@AfterReturning` runs only after successful normal completion and can capture the returned value. `@AfterThrowing` runs when the target method exits by throwing an exception and can capture that exception. `@Around` is the most powerful because it surrounds the target invocation using `ProceedingJoinPoint`. It can execute code before and after the target, control whether `proceed()` is called, and potentially modify arguments, return values or exception handling. For simple pre-processing I prefer `@Before`, while for timing, authorization or invocation control I use `@Around`. If multiple aspects exist, I use explicit ordering such as `@Order` rather than relying on accidental order.”**

---

# 31. 30-Second Hinglish Answer

> **“Spring AOP mein five main advice types hain. `@Before` method se pehle, `@After` method complete hone ke baad success ya exception dono cases mein, `@AfterReturning` successful return ke baad, aur `@AfterThrowing` exception ke time execute hota hai. `@Around` sabse powerful hai kyunki `ProceedingJoinPoint` ke through target method ko control kar sakta hai. Agar `proceed()` nahi call karenge to target execute nahi hoga. Simple logging ke liye Before, exception logging ke liye AfterThrowing, aur execution timing ya authorization jaise control ke liye Around use karenge.”**

---

# 🧠 Memory Map

```text
Advice
│
├── @Before
│   └── Before target
│
├── @After
│   └── Completion (success/exception)
│
├── @AfterReturning
│   └── Successful return
│
├── @AfterThrowing
│   └── Exception
│
└── @Around
    ├── Before
    ├── proceed()
    ├── Target
    ├── After
    └── Full invocation control
```

### One-line memory

> **Before = pehle | After = completion | AfterReturning = success | AfterThrowing = exception | Around = full control**

---

## Navigation

[← S1.4.2 — AOP Terminology](../02-AOP-Terminology/README.md)

[↗ S1.4 — Spring AOP APIs](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

**Status: ✅ Completed**

**Next:** S1.4.4 — Custom Logging support Aspect Implementation
