# Spring-and-Spring-boot-Complete-course

# Spring and Spring Boot

## Spring - S1

### 1. Introduction

> **Study rule:** S1 Introduction ke topics ko sequence mein complete karenge. Har topic ka separate deep-dive folder hoga aur yahan se uska direct clickable link rahega.

| # | Topic | Status | Link |
|---|---|---|---|
| 1 | Overview of Spring Technology | ✅ Completed | [Open](Spring/S1-Introduction/01-Overview-of-Spring-Technology/README.md) |
| 2 | The Motivation for Spring, Spring Architecture | ✅ Completed | [Open](Spring/S1-Introduction/02-The-Motivation-for-Spring-and-Spring-Architecture/README.md) |
| 3 | The Spring Framework | ✅ Completed | [Open](Spring/S1-Introduction/03-The-Spring-Framework/README.md) |
| 4 | Spring Introduction | ✅ Completed | [Open](Spring/S1-Introduction/04-Spring-Introduction/README.md) |
| 5 | Declaring and Managing Beans | ✅ Completed | [Open](Spring/S1-Introduction/05-Declaring-and-Managing-Beans/README.md) |
| 6 | BeanFactory vs ApplicationContext | ✅ Completed | [Open](Spring/S1-Introduction/06-BeanFactory-vs-ApplicationContext/README.md) |
| 7 | Dependencies and Dependency Injection (DI) | ✅ Completed | [Open](Spring/S1-Introduction/07-Dependencies-and-Dependency-Injection/README.md) |
| 8 | Examining Dependencies | ✅ Completed | [Open](Spring/S1-Introduction/08-Examining-Dependencies/README.md) |
| 9 | Dependency Inversion / Dependency Injection (DI) | ✅ Completed | [Open](Spring/S1-Introduction/09-Dependency-Inversion-and-Dependency-Injection/README.md) |
| 10 | XML Configuration of DI | ⏳ Pending | — |
| 11 | Spring Bean Autowiring | ⏳ Pending | — |
| 12 | Injection with @Autowired | ⏳ Pending | — |
| 13 | Java Based Configuration (@Configuration) | ⏳ Pending | — |

### Introduction Progress

```text
S1 — Introduction
│
├── 1. Overview of Spring Technology                         ✅
├── 2. The Motivation for Spring, Spring Architecture       ✅
├── 3. The Spring Framework                                  ✅
├── 4. Spring Introduction                                   ✅
├── 5. Declaring and Managing Beans                           ✅
├── 6. BeanFactory vs ApplicationContext                      ✅
├── 7. Dependencies and Dependency Injection (DI)             ✅
├── 8. Examining Dependencies                                 ✅
├── 9. Dependency Inversion / Dependency Injection (DI)       ✅
├── 10. XML Configuration of DI                               ⏳
├── 11. Spring Bean Autowiring                                ⏳
├── 12. Injection with @Autowired                              ⏳
└── 13. Java Based Configuration (@Configuration)             ⏳
```

---

### 2. Container Extensions Points
- Bean Scope and Lifecycle
- Singleton, Prototype, and Other Scopes
- Configuring Scope
- Bean Lifecycle / Callbacks

### 3. Spring Expression Language (SpEL)
- Introduction To SpEL
- SpEL Features
- SpEL expression evaluation against a specific object instance
- SpEL in Bean Definition

### 4. Spring AOP APIs
- Introduction of Aspect Oriented Programming (The Proxy Pattern)
- AOP Terminology
- Types of Advice
- Custom Logging support Aspect Implementation

## Spring-S2

### 1. DAO Introduction
- Introduction
- Plain JDBC limitations
- Spring JDBC/DAO Advantages
- Working with different Data Sources
- JdbcTemplate
- NamedParameterJdbcTemplate
- Spring JDBC/DAO with Annotations

### 2. Spring Transaction Management
- Introduction to Transaction Management
- Local Transaction Vs Distributed Transaction
- Need of Spring Transaction Management
- Implementing Spring Transaction management using Annotation Driven Approach
- Transaction Attributes

### 3. Spring with MongoDB
- Install and Launch MongoDB
- SpringDAO with NoSQL DB using MongoDB

### 4. Spring Web Integration
- Introduction To MVC
- Understanding MVC1, MVC2 Architectures
- Front Controller Design Pattern
- Spring MVC Basics
- Configuration and the DispatcherServlet
- @Controller, @RequestMapping (Handlers)
- @RequestParam and Parameter Binding
- View Resolvers
- Controller Details - @RequestParam, @PathVariable
- Model Data and @ModelAttribute

### 5. RESTful Services with Spring
- RESTful Services with Spring
- REST Overview, URI Templates
- REST and Spring MVC
- Spring support for REST
- @RequestMapping/@PathVariable, @RequestBody, @ResponseBody
- URI Templates and @PathVariable
- Controllers with @RestController

## Spring - S3

### 1. Introduction
- A brief Overview of Maven
- Intro to Spring Boot - What is Spring Boot and What It Does
- Spring Boot Hello World / SpringApplication
- SpringBootApplication / CommandLineRunner / ApplicationRunner

### 2. Configuration & Customization
- Working with Properties - YAML and .properties
- Logging and its Configuration
- Auto-configuration Overview
- Customization

### 3. Spring Boot Database Support
- Basic Auto-configuration - Datasource and Pooling
- Configuration Properties
- Spring Boot's JPA Support - spring-boot-starter-data-jpa
- Spring Boot Data (with Data-JPA in Detail)
- Using Spring Boot Data - CrudRepository/JpaRepository
- Configuring Spring Boot with NoSQL vendor MongoDB
- Configuring Spring Boot with Redis Cache

### 4. Spring Boot Web/REST
- DispatcherServlet Review
- Web Starters and Configuration spring-boot-starter-web
- Using Embedded Servers (Tomcat, Netty)
- Usage of template thymeLeaf

### 5. Actuator and Devtools
- Actuator Overview and Capabilities
- Actuator Endpoints
- Custom Actuators and Health Checks
- Devtools Overview
