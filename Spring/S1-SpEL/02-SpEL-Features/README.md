# S1.3.2 — SpEL Features

> **Status:** ✅ Completed

## 1. What are SpEL Features?

SpEL is more than simple value substitution. It provides a rich expression language for evaluating values and navigating objects at runtime.

Core capabilities:

```text
Literals
Arithmetic operators
Relational operators
Logical operators
String operations
Properties
Methods
Bean references
Type references
Constructors
Collections
Lists / Maps
Selection
Projection
Variables
Assignment
Ternary / Elvis operators
Safe navigation
Expression templates
```

---

## 2. Literals

SpEL supports common literal types.

```text
Numbers
Strings
Booleans
null
```

Examples:

```text
100
3.14
true
'Spring'
null
```

String literals normally use single quotes.

---

## 3. Arithmetic Operators ⭐

SpEL supports:

```text
+   addition
-   subtraction
*   multiplication
/   division
%   remainder
^   power
```

Example:

```java
@Value("#{10 * 5}")
private int result;
```

Result:

```text
50
```

---

## 4. Relational Operators

Common operators:

```text
==
!=
<
>
<=
>=
lt
le
gt
ge
eq
ne
```

Example:

```text
#{10 > 5}
```

Result:

```text
true
```

SpEL also provides textual forms such as `lt`, `gt`, `eq`, etc., which can be useful where symbolic operators are inconvenient.

---

## 5. Logical Operators

SpEL supports:

```text
and
or
not
&&
||
!
```

Example:

```text
#{10 > 5 and 20 > 10}
```

Result:

```text
true
```

---

## 6. String Operations

SpEL can work with String methods.

Example:

```text
'Spring'.toUpperCase()
```

Result:

```text
SPRING
```

You can also access String properties/methods available to the underlying object.

---

## 7. Property Navigation ⭐

SpEL can navigate an object's properties.

Example concept:

```text
employee.name
```

Nested navigation is possible:

```text
employee.address.city
```

This makes SpEL useful when an expression needs to traverse an object graph.

---

## 8. Method Invocation ⭐

Methods can be invoked from expressions.

Example:

```text
'hello'.toUpperCase()
```

Another example conceptually:

```text
employee.getName()
```

This is a major difference from simple property placeholder resolution.

---

## 9. Bean References ⭐⭐⭐

Spring Beans can be referenced using `@beanName` when the evaluation context supports Bean resolution.

Example:

```java
@Component("taxService")
public class TaxService {

    public double calculateTax() {
        return 18.0;
    }
}
```

Expression:

```text
#{@taxService.calculateTax()}
```

Important syntax:

```text
@beanName
```

### Interview point

`@` in this context means a Spring Bean reference, not a Java annotation.

---

## 10. Type References — `T(...)` ⭐

SpEL supports type references using `T(...)`.

Example:

```text
T(java.lang.Math).PI
```

Method example:

```text
T(java.lang.Math).max(10, 20)
```

Result:

```text
20
```

Static fields and methods can therefore be accessed through type references.

---

## 11. Constructor Invocation

SpEL can create objects using `new`.

Conceptual example:

```text
new java.util.Date()
```

This is useful in expression evaluation, but object creation should be used carefully in application configuration.

---

## 12. Lists

SpEL supports inline list literals.

Example:

```text
{1, 2, 3, 4}
```

A list can also contain expressions/objects.

Conceptually:

```text
{'Java', 'Spring', 'Boot'}
```

---

## 13. Maps

SpEL supports inline map literals.

Example:

```text
{key1:'value1', key2:'value2'}
```

This can be useful when an expression needs a small map-like structure.

---

## 14. Indexing Collections and Maps

SpEL supports indexing.

List/array example:

```text
employees[0]
```

Map example:

```text
employees['EMP001']
```

Conceptually:

```text
collection[index]
map[key]
```

---

## 15. Selection — `.?[]` ⭐⭐⭐

Selection filters elements from a collection based on a condition.

Syntax:

```text
collection.?[condition]
```

Example:

```text
employees.?[salary > 50000]
```

Meaning:

> Return employees whose salary is greater than 50,000.

Memory:

```text
.?[] → FILTER
```

---

## 16. First Selection — `.^[]`

`.^[]` selects the **first** element that matches the condition.

Example:

```text
employees.^[salary > 50000]
```

Meaning:

> Find the first employee whose salary is greater than 50,000.

Memory:

```text
.^[] → FIRST MATCH
```

---

## 17. Last Selection — `.$[]`

`.$[]` selects the **last** element that matches the condition.

Example:

```text
employees.$[salary > 50000]
```

Memory:

```text
.$[] → LAST MATCH
```

---

## 18. Projection — `.![...]` ⭐⭐⭐

Projection transforms each element of a collection into another value.

Syntax:

```text
collection.![expression]
```

Example:

```text
employees.![name]
```

Meaning:

> Create a collection containing the `name` of every employee.

Memory:

```text
.?[]  → FILTER
.![ ] → MAP / TRANSFORM
```

This distinction is a common interview question.

---

## 19. Selection vs Projection ⭐⭐⭐

| Feature | Syntax | Meaning |
|---|---|---|
| Selection | `.?[condition]` | Filter elements |
| First selection | `.^[condition]` | First matching element |
| Last selection | `.$[condition]` | Last matching element |
| Projection | `.![expression]` | Transform each element |

### Memory

```text
.?[]  = WHERE / FILTER
.![ ] = SELECT / MAP
.^[]  = FIRST
.$[]  = LAST
```

---

## 20. Variables ⭐

SpEL expressions can use variables through an `EvaluationContext`.

Example:

```java
StandardEvaluationContext context =
        new StandardEvaluationContext();

context.setVariable("name", "Nirbhay");
```

Expression:

```text
#name
```

Result:

```text
Nirbhay
```

Important syntax:

```text
#variableName
```

---

## 21. Special Variables

Spring's evaluation infrastructure can expose predefined variables in particular integrations.

One important example is the `systemProperties` / environment-related access available through Spring expression evaluation contexts depending on configuration.

Do not assume every predefined variable exists in every `EvaluationContext`; available variables depend on the context and integration being used.

---

## 22. Assignment

SpEL can assign values in contexts where write access is allowed.

Conceptually:

```text
name = 'Spring'
```

Whether assignment is permitted depends on the evaluation context and property access configuration.

For normal application configuration, prefer explicit Java configuration rather than using expressions for complex state mutation.

---

## 23. Ternary Operator ⭐

SpEL supports conditional expressions.

Syntax:

```text
condition ? valueIfTrue : valueIfFalse
```

Example:

```text
#{10 > 5 ? 'YES' : 'NO'}
```

Result:

```text
YES
```

---

## 24. Elvis Operator ⭐⭐

SpEL supports the Elvis operator `?:` for null/default-value style expressions.

Example:

```text
name ?: 'Unknown'
```

Meaning:

> If `name` has a usable value, use it; otherwise use `Unknown`.

Memory:

```text
?: → DEFAULT VALUE
```

---

## 25. Safe Navigation Operator ⭐⭐

The safe navigation operator `?.` helps avoid null-related navigation failures.

Example:

```text
employee?.address?.city
```

If an intermediate value is `null`, the navigation can yield `null` rather than blindly dereferencing the null object.

### Important

Use safe navigation only where null is an expected possibility; it should not hide invalid application state.

---

## 26. Expression Templates

SpEL supports templates where literal text and expressions are combined.

Conceptually:

```text
'Hello #{name}'
```

Template parsing requires a suitable `ParserContext` when evaluating programmatically.

Example concept:

```java
ParserContext templateContext = new TemplateParserContext();
```

This is useful for generating dynamic text from expressions.

---

## 27. EvaluationContext and Features ⭐⭐⭐

Many advanced SpEL features depend on the evaluation context.

```text
Expression
   ↓
EvaluationContext
   ├── Root object
   ├── Variables
   ├── Bean resolver
   ├── Property accessors
   ├── Method resolvers
   └── Type locator / type access
```

Therefore an expression's capabilities are not determined by the expression string alone.

---

## 28. `StandardEvaluationContext` vs `SimpleEvaluationContext`

### StandardEvaluationContext

Provides a broad set of SpEL capabilities and is intended for general-purpose expression evaluation.

It can support features such as:

```text
Properties
Methods
Variables
Bean resolution (when configured)
Type references
Custom resolvers/accessors
```

### SimpleEvaluationContext

Designed to expose only a restricted subset of SpEL features and is useful when full language power is unnecessary.

### Interview memory

```text
Standard → powerful / general
Simple   → restricted / safer subset
```

---

## 29. Customizing Evaluation

SpEL can be customized through evaluation infrastructure such as:

```text
PropertyAccessor
MethodResolver
ConstructorResolver
BeanResolver
TypeLocator
TypeConverter
OperatorOverloader
```

This is an advanced area but useful for senior-level Spring interviews because it demonstrates that SpEL is an extensible expression engine rather than just annotation syntax.

---

## 30. SpEL and `@Value` — Practical Examples ⭐⭐⭐

### Arithmetic

```java
@Value("#{10 + 20}")
private int total;
```

### Property from configuration

```java
@Value("${app.name}")
private String appName;
```

### Combining property and expression

```java
@Value("#{${app.limit} * 2}")
private int doubledLimit;
```

The exact property must resolve to a value that forms a valid expression.

### Bean reference

```java
@Value("#{@taxService.calculateTax()}")
private double tax;
```

---

## 31. SpEL vs Java Code

SpEL is useful for configuration and controlled runtime expressions, but it should not replace normal Java application logic.

### Good

```text
Simple configuration expression
Property transformation
Conditional configuration value
Small collection filtering
```

### Avoid

```text
Complex business rules
Large calculations
Multi-step workflows
Core domain logic
```

Memory:

> **SpEL = configuration/expression power, not a replacement for service-layer Java code.**

---

## 32. Security and Untrusted Expressions ⭐⭐⭐

SpEL can invoke methods and access objects depending on the evaluation context.

Therefore evaluating attacker-controlled expressions can be dangerous.

Avoid patterns like:

```text
user input
   ↓
construct SpEL string
   ↓
evaluate dynamically
```

unless the expression language and evaluation context are intentionally restricted and the input is trusted/validated.

For untrusted scenarios, prefer restricted evaluation capabilities such as `SimpleEvaluationContext` where appropriate and avoid exposing unnecessary resolvers/accessors.

---

## 33. Common Interview Questions

1. What are the major features of SpEL?
2. What is the syntax for Bean references?
3. What does `T(...)` mean?
4. What does `#variable` mean?
5. What is selection in SpEL?
6. What is projection in SpEL?
7. Difference between `.?[]` and `.![...]`?
8. What do `.^[]` and `.$[]` do?
9. What is the Elvis operator?
10. What is the safe navigation operator?
11. What is a ternary expression?
12. Can SpEL invoke methods?
13. Can SpEL create objects?
14. Can SpEL access collections?
15. How do you access a Map value?
16. What is `EvaluationContext`?
17. Difference between `StandardEvaluationContext` and `SimpleEvaluationContext`?
18. How do Bean references work in SpEL?
19. What is expression templating?
20. What are `PropertyAccessor` and `MethodResolver`?
21. Why is evaluating untrusted SpEL dangerous?
22. Where would you use SpEL in a real Spring Boot application?
23. Should business logic be written in SpEL?
24. Explain SpEL features in 2 minutes.

---

## 34. 2-Minute Interview Answer ⭐

> **“SpEL provides runtime expression evaluation inside the Spring ecosystem. Its major features include literals, arithmetic and logical operators, property navigation, method invocation, Spring Bean references using `@beanName`, type references using `T(...)`, collection and map access, selection using `.?[]`, projection using `.![...]`, variables using `#variable`, ternary and Elvis operators, safe navigation, and expression templates. SpEL capabilities are controlled by the EvaluationContext, which can provide a root object, variables, property accessors, method resolvers and other infrastructure. `StandardEvaluationContext` provides broad capabilities, while `SimpleEvaluationContext` exposes a restricted subset. Because SpEL can be powerful, complex business logic should remain in Java and untrusted expressions should not be evaluated with unrestricted capabilities.”**

---

## 35. 30-Second Hinglish Answer

> **“SpEL ke major features mein operators, property access, method invocation, Bean reference `@beanName`, type reference `T(...)`, variables `#variable`, collections, selection `.?[]`, projection `.![...]`, ternary, Elvis `?:` aur safe navigation `?.` aate hain. Advanced evaluation ke liye `EvaluationContext` use hota hai. Interview mein especially selection vs projection, Standard vs Simple EvaluationContext aur `${}` vs `#{}` ka difference yaad rakho.”**

---

## 🧠 Memory Map

```text
SpEL Features
│
├── Values
│   ├── Literals
│   └── Operators
│
├── Objects
│   ├── Properties
│   ├── Methods
│   ├── @Bean
│   └── T(Type)
│
├── Collections
│   ├── [index]
│   ├── ?[]  → Filter
│   ├── ^[]  → First
│   ├── $[]  → Last
│   └── ![]  → Map/Transform
│
├── Runtime
│   ├── #variables
│   └── EvaluationContext
│
└── Conditional
    ├── ?: ternary
    ├── Elvis
    └── ?. safe navigation
```

### One-line memory

> **“SpEL can read, navigate, call, filter, transform and conditionally evaluate objects at runtime.”**

---

## Navigation

[← S1.3.1 Introduction To SpEL](../01-Introduction-to-SpEL/README.md)

[↗ S1.3 Spring Expression Language](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** S1.3.3 — SpEL expression evaluation against a specific object instance
