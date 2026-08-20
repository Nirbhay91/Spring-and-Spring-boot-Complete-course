# Spring — S1.2 Container Extension Points

> **Status:** 🚧 Started
>
> Is section ko topic-by-topic complete karenge. Har sub-topic ka separate folder + README hoga aur is page se clickable link maintain hoga.

## 2. Container Extension Points

Spring IoC Container sirf Beans create aur dependencies inject nahi karta. Spring container ko customize karne ke liye multiple **extension points** provide karta hai.

### Topics

| # | Topic | Status | Link |
|---|---|---|---|
| 2.1 | Bean Scope and Lifecycle | ⏳ Pending | — |
| 2.2 | Singleton, Prototype, and Other Scopes | ⏳ Pending | — |
| 2.3 | Configuring Scope | ⏳ Pending | — |
| 2.4 | Bean Lifecycle / Callbacks | ⏳ Pending | — |

---

## What are Container Extension Points?

Spring container ke default behavior ko customize ya extend karne ke mechanisms ko broadly **Container Extension Points** kaha ja sakta hai.

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

> **Important:** Is S1.2 section mein pehle source ke listed topics — scope and lifecycle — sequence mein complete karenge. Advanced extension mechanisms ko relevant topic ke saath connect karenge.

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

[← Previous — S1.13 Java Based Configuration](../S1-Introduction/13-Java-Based-Configuration-Configuration/README.md)

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
    ├── 2.1 Bean Scope and Lifecycle          ⏳
    ├── 2.2 Singleton, Prototype, Other       ⏳
    ├── 2.3 Configuring Scope                 ⏳
    └── 2.4 Bean Lifecycle / Callbacks        ⏳
```

**Next:** S1.2.1 — Bean Scope and Lifecycle
