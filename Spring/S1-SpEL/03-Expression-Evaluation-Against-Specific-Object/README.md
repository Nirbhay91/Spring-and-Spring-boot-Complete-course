# S1.3.3 — SpEL Expression Evaluation Against a Specific Object Instance

> **Status:** ✅ Completed

## 1. What does this topic mean?

SpEL expression ko kisi **specific Java object ke against evaluate** karna means expression ke andar properties/methods ko ek particular object ke context mein resolve karna.

Simple example:

```java
Employee employee = new Employee("Nirbhay", 30);
```

Expression:

```text
name
```

Agar `employee` root object hai, to SpEL:

```text
name → employee.getName()
```

ke equivalent property access resolve kar sakta hai.

---

## 2. Core Classes ⭐⭐⭐

Programmatic evaluation ke liye commonly ye classes/interfaces important hain:

```text
ExpressionParser
SpelExpressionParser
Expression
EvaluationContext
StandardEvaluationContext
```

Flow:

```text
Java Object
    ↓
EvaluationContext
    ↓
SpEL Expression
    ↓
ExpressionParser
    ↓
Expression.getValue(...)
    ↓
Result
```

---

## 3. Example Domain Object

```java
public class Employee {

    private String name;
    private int age;
    private double salary;

    public Employee(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public double getSalary() {
        return salary;
    }

    public String getDisplayName() {
        return name + "-Employee";
    }
}
```

---

## 4. Creating the Specific Object

```java
Employee employee =
        new Employee("Nirbhay", 30, 100000);
```

Ab hum chahte hain ki SpEL expressions isi `employee` object ke against evaluate hon.

---

## 5. Creating `SpelExpressionParser`

```java
ExpressionParser parser =
        new SpelExpressionParser();
```

Parser expression string ko SpEL expression representation mein parse karta hai.

---

## 6. Parsing the Expression

Example:

```java
Expression expression =
        parser.parseExpression("name");
```

Yahan:

```text
"name"
   ↓
SpEL parser
   ↓
Expression object
```

Expression abhi result nahi hai. Ye parsed expression hai jise context/object ke against evaluate kiya ja sakta hai.

---

## 7. Direct Evaluation Against an Object ⭐⭐⭐

Agar specific object directly available hai, `getValue` ke through root object provide kar sakte hain.

```java
String name =
        expression.getValue(employee, String.class);
```

Result:

```text
Nirbhay
```

### Mental model

```text
expression = "name"
rootObject = employee

getValue(employee, String.class)
        ↓
employee.name
        ↓
"Nirbhay"
```

---

## 8. Why is `employee` called the Root Object? ⭐

Jab hum likhte hain:

```java
expression.getValue(employee, String.class);
```

`employee` expression evaluation ka **root object** ban jaata hai.

Therefore expression:

```text
name
```

ka meaning hai:

```text
employee.name
```

Similarly:

```text
age
```

means:

```text
employee.age
```

---

## 9. Evaluating Multiple Properties

```java
Expression nameExpression =
        parser.parseExpression("name");

Expression ageExpression =
        parser.parseExpression("age");

Expression salaryExpression =
        parser.parseExpression("salary");

String name =
        nameExpression.getValue(employee, String.class);

Integer age =
        ageExpression.getValue(employee, Integer.class);

Double salary =
        salaryExpression.getValue(employee, Double.class);
```

Result:

```text
name   = Nirbhay
age    = 30
salary = 100000.0
```

---

## 10. Property Access vs Getter ⭐⭐⭐

SpEL expression:

```text
name
```

Agar `Employee` mein readable JavaBean property `getName()` available hai, Spring's property access infrastructure property ko resolve kar sakta hai.

So conceptually:

```text
name
 ↓
getName()
 ↓
Nirbhay
```

### Important

SpEL expression ko directly Java field access ke same assume mat karo. Property resolution evaluation context ke accessors aur object model par depend karta hai.

---

## 11. Evaluating a Method ⭐⭐⭐

SpEL object ke methods invoke kar sakta hai, subject to the configured evaluation context/resolvers.

Expression:

```text
getDisplayName()
```

Evaluation:

```java
String result =
        parser.parseExpression("getDisplayName()")
              .getValue(employee, String.class);
```

Result:

```text
Nirbhay-Employee
```

---

## 12. Nested Object Example

Suppose:

```java
public class Address {
    private String city;

    public String getCity() {
        return city;
    }
}
```

Employee:

```java
public Address getAddress() {
    return address;
}
```

Expression:

```text
address.city
```

SpEL navigation:

```text
employee
   ↓
address
   ↓
city
   ↓
Bangalore
```

---

## 13. Using `StandardEvaluationContext` ⭐⭐⭐

Instead of directly passing the root object to `getValue`, we can explicitly create an evaluation context.

```java
StandardEvaluationContext context =
        new StandardEvaluationContext(employee);
```

Then:

```java
Expression expression =
        parser.parseExpression("name");

String name =
        expression.getValue(context, String.class);
```

Result:

```text
Nirbhay
```

### Why use a context?

Because `EvaluationContext` can provide more than just a root object:

```text
Root object
Variables
Property accessors
Method resolvers
Bean resolver
Type locator
Type converter
```

---

## 14. Direct Object vs EvaluationContext ⭐⭐⭐

### Approach 1 — Direct root object

```java
expression.getValue(employee, String.class);
```

Good for simple evaluation.

### Approach 2 — EvaluationContext

```java
EvaluationContext context =
        new StandardEvaluationContext(employee);

expression.getValue(context, String.class);
```

Better when evaluation needs variables, custom resolvers, Bean references or other context configuration.

### Interview answer

> **“For a simple expression I can directly pass the root object to `getValue()`. When I need richer evaluation configuration such as variables, property accessors, method resolvers or Bean resolution, I use an `EvaluationContext`.”**

---

## 15. Using Variables with the Object ⭐⭐⭐

Suppose:

```java
StandardEvaluationContext context =
        new StandardEvaluationContext(employee);

context.setVariable("bonus", 10000);
```

Expression:

```text
salary + #bonus
```

Evaluate:

```java
Double total =
        parser.parseExpression("salary + #bonus")
              .getValue(context, Double.class);
```

Result:

```text
110000.0
```

Important:

```text
employee.salary → root object's property
#bonus          → context variable
```

---

## 16. Root Object vs Variable ⭐⭐⭐

Ye interview mein confuse hota hai.

### Root object

```text
name
age
salary
```

Direct property access root object se resolve hota hai.

### Variable

```text
#bonus
#department
#user
```

Variables `EvaluationContext` mein set kiye jaate hain.

Memory:

```text
name      → Root Object
#bonus    → Variable
```

---

## 17. Method + Variable Example

Expression:

```text
salary + #bonus
```

Or:

```text
getSalary() + #bonus
```

Conceptually:

```text
Root Object      → employee
Context Variable → bonus
Expression       → salary + #bonus
```

---

## 18. Bean Reference with Context

Bean references require appropriate Bean resolution support in the evaluation context.

Conceptually:

```text
@taxService.calculateTax()
```

For a standalone `StandardEvaluationContext`, a BeanResolver may need to be configured before `@taxService` can be resolved.

This is an important interview nuance:

> **Creating `StandardEvaluationContext(employee)` alone does not automatically make every Spring ApplicationContext Bean available.**

---

## 19. Type-Safe `getValue()` ⭐

Prefer the typed overload when the expected result type is known.

```java
String name =
        expression.getValue(employee, String.class);
```

Instead of:

```java
Object result =
        expression.getValue(employee);
```

Typed evaluation improves readability and lets the evaluation infrastructure perform the requested type conversion where supported.

---

## 20. Complete Example ⭐⭐⭐

```java
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;

public class SpELDemo {

    public static void main(String[] args) {

        Employee employee =
                new Employee("Nirbhay", 30, 100000);

        ExpressionParser parser =
                new SpelExpressionParser();

        Expression nameExpression =
                parser.parseExpression("name");

        String name =
                nameExpression.getValue(employee, String.class);

        System.out.println(name);

        StandardEvaluationContext context =
                new StandardEvaluationContext(employee);

        context.setVariable("bonus", 10000);

        Expression salaryExpression =
                parser.parseExpression("salary + #bonus");

        Double totalSalary =
                salaryExpression.getValue(context, Double.class);

        System.out.println(totalSalary);
    }
}
```

Output:

```text
Nirbhay
110000.0
```

---

## 21. Complete Execution Flow ⭐⭐⭐

```text
Employee Object
      │
      ▼
SpelExpressionParser
      │
      ▼
parseExpression("salary + #bonus")
      │
      ▼
Expression Object
      │
      ▼
StandardEvaluationContext(employee)
      │
      ├── Root Object = employee
      └── Variable bonus = 10000
      │
      ▼
expression.getValue(context)
      │
      ▼
salary + bonus
      │
      ▼
110000.0
```

---

## 22. What Happens Internally? ⭐⭐⭐

High-level flow:

```text
Expression String
       ↓
SpEL Parser
       ↓
Parsed Expression / AST
       ↓
EvaluationContext + Root Object
       ↓
Property / Method / Variable Resolution
       ↓
Type Conversion (if required)
       ↓
Final Result
```

### Important

SpEL parsing and evaluation are conceptually separate:

```text
parseExpression() → parses expression
getValue()        → evaluates expression
```

This distinction is frequently asked in interviews.

---

## 23. Parsing vs Evaluation ⭐⭐⭐

| Step | API | Purpose |
|---|---|---|
| Parse | `parseExpression()` | Expression ko parse karta hai |
| Evaluate | `getValue()` | Parsed expression ko context/object ke against run karta hai |

Example:

```java
Expression expression =
        parser.parseExpression("salary + #bonus");
```

No final result yet.

Then:

```java
expression.getValue(context, Double.class);
```

Actual evaluation happens.

---

## 24. Null Handling

If an expression tries to navigate through a null value, evaluation can fail depending on the expression and configured behavior.

Example:

```text
employee.address.city
```

If:

```text
employee.address == null
```

then normal navigation can result in an evaluation exception.

Safe navigation can be used where appropriate:

```text
employee?.address?.city
```

But don't use safe navigation to silently hide invalid domain state.

---

## 25. Collection Evaluation Against an Object

Suppose root object contains:

```java
List<Employee> employees;
```

Expression can navigate to the collection and use SpEL collection features:

```text
employees.?[salary > 50000]
```

Or project names:

```text
employees.![name]
```

This connects directly with the previous topic **S1.3.2 — SpEL Features**.

---

## 26. `StandardEvaluationContext` Customization

For advanced scenarios:

```java
StandardEvaluationContext context =
        new StandardEvaluationContext(employee);
```

It can be configured with infrastructure such as:

```text
setVariable()
setBeanResolver()
PropertyAccessor
MethodResolver
TypeLocator
TypeConverter
```

This makes it possible to control how expressions resolve data and behavior.

---

## 27. Security ⭐⭐⭐

If expressions are built from untrusted user input:

```text
User Input
    ↓
SpEL Expression
    ↓
Evaluation
```

this can be dangerous because powerful evaluation contexts may expose object properties, methods, types or other capabilities.

### Best practice

```text
Don't evaluate arbitrary user-controlled SpEL.
        ↓
Use fixed/trusted expressions.
        ↓
Restrict evaluation capabilities when needed.
        ↓
Prefer SimpleEvaluationContext for suitable restricted use cases.
```

---

## 28. Real-World Spring Example

A common application pattern is using SpEL inside Spring configuration where the root object/context is managed by Spring.

Example:

```java
@Value("#{@pricingService.getDiscount()}")
private double discount;
```

Here the developer generally does not manually create the `EvaluationContext`; Spring's infrastructure processes the expression.

The programmatic API is especially useful when your application itself needs to evaluate expressions dynamically.

---

## 29. Common Interview Traps ⭐

### Trap 1 — Parser directly returns the final value

❌ Wrong.

```java
parseExpression()
```

returns an `Expression` representation.

Evaluation occurs through `getValue()`.

---

### Trap 2 — Root object means Spring Bean

❌ Wrong.

A root object can simply be any Java object:

```java
new Employee(...)
```

It does not have to be a Spring-managed Bean.

---

### Trap 3 — `#name` means root property

❌ Wrong.

```text
name   → root object property
#name  → variable
```

---

### Trap 4 — `StandardEvaluationContext` automatically knows ApplicationContext Beans

❌ Not necessarily.

Bean resolution requires appropriate BeanResolver/integration.

---

### Trap 5 — SpEL always accesses private fields directly

❌ Don't assume that.

Property access is performed through Spring's evaluation/property-access infrastructure.

---

## 30. Interview Follow-Up Questions

1. How do you evaluate a SpEL expression against a specific object?
2. What is a root object?
3. What is `SpelExpressionParser`?
4. What does `parseExpression()` return?
5. What does `getValue()` do?
6. Difference between parsing and evaluation?
7. How do you access a root object's property?
8. How does SpEL resolve JavaBean properties?
9. How do you invoke a method on the root object?
10. What is `EvaluationContext`?
11. Why use `StandardEvaluationContext`?
12. How do you define variables in an evaluation context?
13. Difference between root property and `#variable`?
14. How do you reference a Spring Bean in programmatic SpEL?
15. Does `StandardEvaluationContext` automatically resolve Spring Beans?
16. How do you evaluate an expression with a specific return type?
17. What happens if nested property is null?
18. How does safe navigation help?
19. What happens internally between parsing and evaluation?
20. Why should arbitrary user input not be evaluated as SpEL?
21. When would you use `SimpleEvaluationContext`?
22. Explain this flow in an interview: Parser → Expression → Context → getValue.

---

## 31. 2-Minute Interview Answer ⭐

> **“SpEL can evaluate an expression against a specific Java object by treating that object as the root object. Programmatically, we create a `SpelExpressionParser`, parse the expression using `parseExpression()`, and then call `getValue()` with the root object and optionally the expected result type. For example, if `employee` is the root object, the expression `name` resolves against that employee's property. For more advanced evaluation, we create a `StandardEvaluationContext(employee)`, where we can additionally define variables such as `#bonus`, configure property or method resolution, and provide Bean resolution when required. Parsing and evaluation are separate operations: `parseExpression()` creates the parsed `Expression`, while `getValue()` performs the actual evaluation.”**

---

## 32. 30-Second Hinglish Answer

> **“Agar mujhe kisi specific Java object ke against SpEL evaluate karna hai, to us object ko root object bana sakte hain. `SpelExpressionParser` se expression parse karte hain aur `getValue(employee, String.class)` se evaluate karte hain. Agar variables, Bean references ya custom evaluation rules chahiye, to `StandardEvaluationContext(employee)` use karte hain. Simple yaad rakho: `parseExpression()` = expression parse, `getValue()` = expression evaluate.”**

---

## 🧠 Memory Trick

```text
OBJECT
  ↓
ROOT OBJECT
  ↓
SpEL Parser
  ↓
Expression
  ↓
EvaluationContext
  ↓
getValue()
  ↓
RESULT
```

### One-line memory

> **“Root object deta hai context, parser expression banata hai, aur getValue() result nikalta hai.”**

---

## Navigation

[← S1.3.2 SpEL Features](../02-SpEL-Features/README.md)

[↗ S1.3 Spring Expression Language](../README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

**Status: ✅ Completed**

**Next:** S1.3.4 — SpEL in Bean Definition
