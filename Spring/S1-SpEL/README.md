# S1.3 — Spring Expression Language (SpEL)

> **Status:** 🚧 In Progress
>
> SpEL ko topic-by-topic deep dive mein cover karenge. Har sub-topic ka separate README hoga aur yahan se clickable navigation maintain hogi.

## 3. Spring Expression Language (SpEL)

Spring Expression Language (**SpEL**) Spring ecosystem ki expression language hai jo runtime par objects, properties, methods, operators, collections aur Bean references ko evaluate karne ki capability deti hai.

### Topics

| # | Topic | Status | Link |
|---|---|---|---|
| 3.1 | Introduction To SpEL | ✅ Completed | [Open](01-Introduction-to-SpEL/README.md) |
| 3.2 | SpEL Features | ✅ Completed | [Open](02-SpEL-Features/README.md) |
| 3.3 | SpEL expression evaluation against a specific object instance | ⏳ Pending | — |
| 3.4 | SpEL in Bean Definition | ⏳ Pending | — |

## Where SpEL is used

Common Spring use cases:

```text
@Value expressions
Bean configuration
Conditional/configuration expressions
Property access
Method invocation
Collection selection/projection
Bean references
Runtime expression evaluation
```

Example:

```java
@Value("#{2 + 3}")
private int result;
```

`result` will receive `5` after expression evaluation.

Another example:

```java
@Value("${app.name}")
private String applicationName;
```

> **Important:** `${...}` is **property placeholder syntax**, while `#{...}` is commonly used for **SpEL expressions**. Interview mein dono ka difference important hai.

## Interview Mental Model

```text
SpEL
 │
 ├── Literals / Operators
 ├── Properties
 ├── Methods
 ├── Objects / Bean references
 ├── Collections
 ├── Selection / Projection
 └── Integration with Spring configuration
```

## Navigation

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

## Progress

```text
Spring — S1
│
├── 1. Introduction                         ✅
├── 2. Container Extension Points           ✅
└── 3. Spring Expression Language (SpEL)
    ├── 3.1 Introduction To SpEL             ✅
    ├── 3.2 SpEL Features                    ✅
    ├── 3.3 Expression evaluation            ⏳
    └── 3.4 SpEL in Bean Definition          ⏳
```

**Next:** S1.3.3 — SpEL expression evaluation against a specific object instance
