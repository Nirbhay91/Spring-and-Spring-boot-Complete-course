# Spring — S1.2 Container Extension Points

> **Status:** ✅ Completed
>
> Har sub-topic ka separate deep-dive folder + README hai aur yahan se direct clickable link maintained hai.

## 2. Container Extension Points

Spring IoC Container sirf Beans create aur dependencies inject nahi karta. Spring container ko customize karne ke liye multiple **extension points** provide karta hai.

### Topics

| # | Topic | Status | Link |
|---|---|---|---|
| 2.1 | Bean Scope and Lifecycle | ✅ Completed | [Open](01-Bean-Scope-and-Lifecycle/README.md) |
| 2.2 | Singleton, Prototype, and Other Scopes | ✅ Completed | [Open](02-Singleton-Prototype-and-Other-Scopes/README.md) |
| 2.3 | Configuring Scope | ✅ Completed | [Open](03-Configuring-Scope/README.md) |
| 2.4 | Bean Lifecycle / Callbacks | ✅ Completed | [Open](04-Bean-Lifecycle-Callbacks/README.md) |

---

## What are Container Extension Points?

Spring IoC Container ke default behavior ko customize ya extend karne ke mechanisms ko broadly **extension points** kaha ja sakta hai.

Simplified flow:

```text
Application
    ↓
Spring IoC Container
    ↓
Bean Definition
    ↓
Bean Creation
    ↓
Dependency Injection
    ↓
Lifecycle / Callbacks / Post Processing
    ↓
Ready Bean
```

Interview mein important extension mechanisms:

- Bean scopes
- Bean lifecycle callbacks
- `BeanPostProcessor`
- `BeanFactoryPostProcessor`
- `ApplicationContextAware` and related `Aware` interfaces
- Event-based extension through Spring application events

## Interview Mental Model

```text
Container
   │
   ├── How long should Bean live?
   │       → Scope
   │
   ├── What happens during Bean creation/destruction?
   │       → Lifecycle / Callbacks
   │
   ├── Can Bean behavior be modified before/after initialization?
   │       → BeanPostProcessor
   │
   └── Can Bean definitions be modified before Bean creation?
           → BeanFactoryPostProcessor
```

## Navigation

[← Previous — S1.2.3 Configuring Scope](../S1-Container-Extension-Points/03-Configuring-Scope/README.md)

[🏠 Spring & Spring Boot Complete Course](https://github.com/Nirbhay91/Spring-and-Spring-boot-Complete-course)

---

## Progress

```text
Spring — S1
│
├── 1. Introduction
│   └── S1.1 → S1.13                         ✅
│
└── 2. Container Extension Points
    ├── 2.1 Bean Scope and Lifecycle          ✅
    ├── 2.2 Singleton, Prototype, Other       ✅
    ├── 2.3 Configuring Scope                 ✅
    └── 2.4 Bean Lifecycle / Callbacks        ✅
```

**Next:** S1.3 — Spring Expression Language (SpEL)
