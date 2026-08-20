# S1.3.1 — Introduction to Spring Expression Language (SpEL)

> **Status:** ✅ Completed

## 1. What is SpEL?

**SpEL (Spring Expression Language)** Spring Framework ki expression language hai jo runtime par expressions ko evaluate karne ke liye use hoti hai.

Simple words mein:

> SpEL Spring ko runtime par values, properties, methods, objects aur Bean references ko expression ke through evaluate/access karne ki capability deta hai.

Basic example:

```java
@Value("#{2 + 3}")
private int result;
```

Result:

```text
result = 5
```

---

## 2. Why do we need SpEL?

Normally configuration mein static values hoti hain:

```java
@Value("100")
private int limit;
```

SpEL se expression evaluate kar sakte hain:

```java
@Value("#{50 * 2}")
private int limit;
```

SpEL ka purpose sirf arithmetic nahi hai. It can work with:

```text
Properties
Objects
Methods
Operators
Collections
Bean references
Conditional expressions
Type references
```

---

## 3. Basic Syntax ⭐

SpEL expressions commonly `#{...}` syntax mein likhe jaate hain.

```java
@Value("#{2 + 3}")
private int value;
```

```text
#{ expression }
     ↑
    SpEL
```

### Important interview distinction

```text
#{...} → SpEL expression
${...} → Property placeholder
```

Example:

```java
@Value("${server.port}")
private int port;
```

Here Spring property resolve karta hai.

```java
@Value("#{2 + 3}")
private int value;
```

Here SpEL expression evaluate hota hai.

---

## 4. SpEL vs Property Placeholder ⭐⭐⭐

| Feature | SpEL | Property Placeholder |
|---|---|---|
| Syntax | `#{...}` | `${...}` |
| Purpose | Expression evaluation | Configuration property lookup |
| Arithmetic | ✅ | ❌ |
| Method invocation | ✅ | ❌ |
| Bean reference | ✅ | ❌ |
| Property lookup | Can access properties | Main purpose |
| Example | `#{2 + 3}` | `${server.port}` |

### Interview answer

> **“`${...}` is mainly used for resolving externalized configuration properties, while `#{...}` is used for evaluating SpEL expressions.”**

---

## 5. Where can SpEL be used?

SpEL is integrated into several Spring features.

Common examples:

```text
@Value
Bean definitions
Spring configuration
Programmatic expression evaluation
Conditional/configuration scenarios
Security expressions
Data-related Spring modules
```

For this chapter, focus first on core expression evaluation and its use in Bean configuration.

---

## 6. SpEL in `@Value` ⭐

One of the most common real-world uses is `@Value`.

```java
@Component
public class AppConfig {

    @Value("#{10 + 20}")
    private int maxUsers;
}
```

Spring evaluates the expression and injects:

```text
maxUsers = 30
```

---

## 7. Literal Values

SpEL supports literals such as:

```text
Numbers
Strings
Boolean
null
```

Examples:

```java
#{100}
#{true}
#{'Spring'}
```

String literals are commonly written using single quotes.

---

## 8. Arithmetic Operators

SpEL supports arithmetic operations.

```java
#{10 + 5}
#{10 - 5}
#{10 * 5}
#{10 / 5}
#{10 % 3}
```

Example:

```java
@Value("#{100 * 2}")
private int amount;
```

Result:

```text
200
```

---

## 9. Relational Operators

SpEL supports comparisons such as:

```text
==
!=
<
>
<=
>=
```

Example:

```java
@Value("#{10 > 5}")
private boolean valid;
```

Result:

```text
true
```

---

## 10. Logical Operators

Common logical operators include:

```text
and
or
not
```

Example:

```java
#{10 > 5 and 20 > 10}
```

Result:

```text
true
```

---

## 11. Property Access

SpEL can access properties of an object.

Conceptually:

```text
object.property
```

Example object:

```java
public class Employee {
    private String name;

    public String getName() {
        return name;
    }
}
```

An expression can navigate the property:

```text
employee.name
```

The underlying accessor mechanisms resolve the property through the object's readable property.

---

## 12. Method Invocation

SpEL can invoke methods on supported objects.

Conceptual example:

```text
'hello'.toUpperCase()
```

Result:

```text
HELLO
```

This is one reason SpEL is more powerful than simple property substitution.

---

## 13. Bean References ⭐

SpEL can refer to Spring Beans.

Example:

```java
@Component("discountService")
public class DiscountService {

    public int getDiscount() {
        return 10;
    }
}
```

Another Bean can reference it through SpEL:

```java
@Value("#{@discountService.getDiscount()}")
private int discount;
```

Important syntax:

```text
@beanName
```

Example:

```text
#{@discountService.getDiscount()}
```

---

## 14. Type References

SpEL can refer to Java types using `T(...)` syntax.

Example:

```text
T(java.lang.Math).PI
```

Another example:

```text
T(java.lang.Math).max(10, 20)
```

Result:

```text
20
```

### Interview point

```text
T(TypeName)
```

is used to reference a type from an expression.

---

## 15. Collections

SpEL supports collection-oriented operations.

Examples of capabilities:

```text
List access
Map access
Selection
Projection
Collection literals
```

These are important advanced SpEL features and will be covered in **S1.3.2 — SpEL Features**.

---

## 16. Programmatic SpEL Evaluation ⭐

SpEL is not limited to annotations.

Spring provides an expression parser API.

Core classes include:

```java
ExpressionParser
Expression
SpelExpressionParser
```

Basic example:

```java
ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression("2 + 3");

Integer result = expression.getValue(Integer.class);
```

Result:

```text
5
```

### Flow

```text
Expression String
       ↓
SpelExpressionParser
       ↓
Expression
       ↓
getValue()
       ↓
Result
```

---

## 17. EvaluationContext ⭐

When expressions need access to an object, variables, methods, or Bean references, an `EvaluationContext` can provide the evaluation context.

Common implementations include:

```text
StandardEvaluationContext
SimpleEvaluationContext
```

Conceptually:

```java
ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression("name");

EvaluationContext context =
        new StandardEvaluationContext(employee);

String name = expression.getValue(context, String.class);
```

This topic connects directly with:

**S1.3.3 — SpEL expression evaluation against a specific object instance.**

---

## 18. Root Object

An evaluation context can have a root object.

Example:

```java
Employee employee = new Employee("Nirbhay");

EvaluationContext context =
        new StandardEvaluationContext(employee);
```

Expression:

```text
name
```

The expression is evaluated against the root `Employee` object.

Conceptually:

```text
Root Object
    ↓
EvaluationContext
    ↓
SpEL Expression
    ↓
Property / Method
```

---

## 19. Why SpEL is Powerful

SpEL combines:

```text
Expression evaluation
        +
Object navigation
        +
Method invocation
        +
Bean access
        +
Type references
        +
Collection operations
```

Therefore it can express dynamic behavior directly inside Spring configuration or programmatic expression evaluation.

---

## 20. SpEL and Spring Boot

Spring Boot does not replace SpEL.

SpEL is still part of the underlying Spring Framework.

Example:

```java
@Value("#{2 + 3}")
private int value;
```

Boot application context will process the expression as part of Spring's configuration/Bean processing.

### Practical advice

Use SpEL when expression-based configuration genuinely improves the design. Don't put complex business logic into annotations.

---

## 21. Security Consideration ⭐

SpEL is powerful because expressions can access objects and invoke methods depending on the evaluation context and configuration.

Therefore **never treat untrusted external input as a trusted SpEL expression without appropriate controls**.

For application code:

```text
Trusted expression → controlled use
Untrusted expression → security risk if evaluated dynamically
```

This is especially important when expressions are constructed from external/user-controlled data.

---

## 22. Common Interview Traps

### Trap 1
`${}` and `#{}` are the same.

❌ Wrong.

```text
${} → property placeholder
#{} → SpEL
```

### Trap 2
SpEL is only used with `@Value`.

❌ Wrong.

SpEL also has programmatic evaluation and is used across Spring features.

### Trap 3
SpEL is Java code.

❌ Wrong.

It is a separate expression language integrated with Spring.

### Trap 4
Every SpEL expression automatically has access to every Bean.

❌ Not necessarily.

Access depends on the evaluation context and Spring integration being used.

### Trap 5
SpEL should contain complex business logic.

❌ Avoid this. Keep business logic in Java/application services.

---

## 23. Interview Follow-Up Questions

1. What is SpEL?
2. Why do we use SpEL?
3. What is the syntax of SpEL?
4. Difference between `#{}` and `${}`?
5. Where is SpEL commonly used?
6. Can SpEL perform arithmetic operations?
7. Can SpEL invoke methods?
8. Can SpEL access object properties?
9. Can SpEL reference Spring Beans?
10. What is `@beanName` in SpEL?
11. What is `T(...)` in SpEL?
12. What is `ExpressionParser`?
13. What is `SpelExpressionParser`?
14. What is `EvaluationContext`?
15. Difference between `StandardEvaluationContext` and `SimpleEvaluationContext`?
16. What is a root object in SpEL?
17. Can SpEL work with collections?
18. Is SpEL part of Spring Framework or Spring Boot?
19. Can SpEL be dangerous with untrusted input?
20. Why should complex business logic not be written in SpEL?
21. Explain SpEL in 2 minutes.

---

## 24. 2-Minute Interview Answer ⭐

> **“SpEL stands for Spring Expression Language. It is an expression language provided by the Spring Framework for evaluating expressions at runtime. It can work with literals, arithmetic and logical operators, object properties, method invocation, Bean references, type references and collections. A common usage is `@Value`, where `#{...}` represents a SpEL expression. This is different from `${...}`, which is mainly used for resolving externalized configuration properties. SpEL can also be evaluated programmatically using `SpelExpressionParser`, `Expression`, and an `EvaluationContext`. Because SpEL is powerful, complex business logic should not be placed inside expressions, and untrusted external input should not be evaluated as SpEL without appropriate security controls.”**

---

## 25. 30-Second Hinglish Answer

> **“SpEL yani Spring Expression Language Spring Framework ki expression language hai. Isse runtime par expressions evaluate kar sakte hain, jaise arithmetic, object properties, methods, Bean references aur collections. Commonly `@Value("#{...}")` ke saath use hota hai. `#{...}` SpEL ke liye hota hai, jabki `${...}` configuration property resolve karne ke liye. Programmatically bhi `SpelExpressionParser` aur `EvaluationContext` ke through expression evaluate kar sakte hain.”**

---

## 🧠 Memory Trick

```text
SpEL
 │
 ├── #{...}        → Expression
 ├── ${...}        → Property
 ├── @bean         → Bean reference
 ├── T(Type)       → Type reference
 ├── object.prop   → Property access
 ├── method()      → Method invocation
 └── parser        → Programmatic evaluation
```

### One-line memory

> **“SpEL = Spring ke andar runtime expression evaluation.”**

---

## Navigation

[↗ S1.3 Spring Expression Language (SpEL)](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** S1.3.2 — SpEL Features
