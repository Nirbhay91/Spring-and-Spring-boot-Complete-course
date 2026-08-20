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
| 10 | XML Configuration of DI | ✅ Completed | [Open](Spring/S1-Introduction/10-XML-Configuration-of-DI/README.md) |
| 11 | Spring Bean Autowiring | ✅ Completed | [Open](Spring/S1-Introduction/11-Spring-Bean-Autowiring/README.md) |
| 12 | Injection with @Autowired | ✅ Completed | [Open](Spring/S1-Introduction/12-Injection-with-Autowired/README.md) |
| 13 | Java Based Configuration (@Configuration) | ✅ Completed | [Open](Spring/S1-Introduction/13-Java-Based-Configuration-Configuration/README.md) |

---

### 2. Container Extension Points

| # | Topic | Status | Link |
|---|---|---|---|
| 2.1 | Bean Scope and Lifecycle | ✅ Completed | [Open](Spring/S1-Container-Extension-Points/01-Bean-Scope-and-Lifecycle/README.md) |
| 2.2 | Singleton, Prototype, and Other Scopes | ✅ Completed | [Open](Spring/S1-Container-Extension-Points/02-Singleton-Prototype-and-Other-Scopes/README.md) |
| 2.3 | Configuring Scope | ✅ Completed | [Open](Spring/S1-Container-Extension-Points/03-Configuring-Scope/README.md) |
| 2.4 | Bean Lifecycle / Callbacks | ✅ Completed | [Open](Spring/S1-Container-Extension-Points/04-Bean-Lifecycle-Callbacks/README.md) |

---

### 3. Spring Expression Language (SpEL)

| # | Topic | Status | Link |
|---|---|---|---|
| 3.1 | Introduction To SpEL | ✅ Completed | [Open](01-Introduction-to-SpEL/README.md) |
| 3.2 | SpEL Features | ✅ Completed | [Open](02-SpEL-Features/README.md) |
| 3.3 | SpEL expression evaluation against a specific object instance | ✅ Completed | [Open](03-Expression-Evaluation-Against-Specific-Object/README.md) |
| 3.4 | SpEL in Bean Definition | ✅ Completed | [Open](04-SpEL-in-Bean-Definition/README.md) |

[Open S1.3 SpEL Folder](Spring/S1-SpEL/README.md)

---

### 4. Spring AOP APIs

| # | Topic | Status | Link |
|---|---|---|---|
| 4.1 | Introduction of Aspect Oriented Programming (The Proxy Pattern) | ✅ Completed | [Open](Spring/S1-AOP-APIs/01-Introduction-of-AOP-The-Proxy-Pattern/README.md) |
| 4.2 | AOP Terminology | ✅ Completed | [Open](Spring/S1-AOP-APIs/02-AOP-Terminology/README.md) |
| 4.3 | Types of Advice | ✅ Completed | [Open](Spring/S1-AOP-APIs/03-Types-of-Advice/README.md) |
| 4.4 | Custom Logging support Aspect Implementation | ✅ Completed | [Open](Spring/S1-AOP-APIs/04-Custom-Logging-Support-Aspect-Implementation/README.md) |

---

## Spring-S2

### 1. DAO Introduction

| # | Topic | Status | Link |
|---|---|---|---|
| 1.1 | Introduction | ✅ Completed | [Open](Spring-S2/01-DAO-Introduction/README.md) |
| 1.2 | Plain JDBC limitations | ✅ Completed | [Open](Spring-S2/01-DAO-Introduction/02-Plain-JDBC-Limitations/README.md) |
| 1.3 | Spring JDBC/DAO Advantages | ✅ Completed | [Open](Spring-S2/01-DAO-Introduction/03-Spring-JDBC-DAO-Advantages/README.md) |
| 1.4 | Working with different Data Sources | ⏳ Pending | — |
| 1.5 | JdbcTemplate | ⏳ Pending | — |
| 1.6 | NamedParameterJdbcTemplate | ⏳ Pending | — |
| 1.7 | Spring JDBC/DAO with Annotations | ⏳ Pending | — |

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

---

# Spring - S4

## 1. Spring Security

> **Study rule:** Spring Security ko interview + production level par sequence mein cover karenge. Har topic/subtopic ka separate folder hoga aur completion ke baad yahan clickable link + status update hoga.

| # | Topic / Subtopic | Status | Link |
|---|---|---|---|
| 1.1 | Introduction to Spring Security | ✅ Completed | [Open](Spring-S4/01-Spring-Security/01-Introduction-to-Spring-Security/README.md) |
| 1.2 | Why Spring Security? | ✅ Completed | [Open](Spring-S4/01-Spring-Security/02-Why-Spring-Security/README.md) |
| 1.3 | Spring Security Architecture | ✅ Completed | [Open](Spring-S4/01-Spring-Security/03-Spring-Security-Architecture/README.md) |
| 1.4 | Security Filter Chain | ⏳ Pending | — |
| 1.5 | DelegatingFilterProxy | ⏳ Pending | — |
| 1.6 | Authentication vs Authorization | ⏳ Pending | — |
| 1.7 | Principal, Credentials and Authorities | ⏳ Pending | — |
| 1.8 | SecurityContext and SecurityContextHolder | ⏳ Pending | — |
| 1.9 | AuthenticationManager and ProviderManager | ⏳ Pending | — |
| 1.10 | AuthenticationProvider | ⏳ Pending | — |
| 1.11 | UserDetails and UserDetailsService | ⏳ Pending | — |
| 1.12 | PasswordEncoder and BCrypt | ⏳ Pending | — |
| 1.13 | In-Memory Authentication | ⏳ Pending | — |
| 1.14 | JDBC/Database-backed Authentication | ⏳ Pending | — |
| 1.15 | Custom UserDetailsService | ⏳ Pending | — |
| 1.16 | SecurityFilterChain Configuration | ⏳ Pending | — |
| 1.17 | Authorization Rules and RequestMatchers | ⏳ Pending | — |
| 1.18 | Role vs Authority | ⏳ Pending | — |
| 1.19 | Method-Level Security | ⏳ Pending | — |
| 1.20 | `@PreAuthorize`, `@PostAuthorize`, `@Secured` | ⏳ Pending | — |
| 1.21 | Form Login | ⏳ Pending | — |
| 1.22 | HTTP Basic Authentication | ⏳ Pending | — |
| 1.23 | Session Management | ⏳ Pending | — |
| 1.24 | CSRF Protection | ⏳ Pending | — |
| 1.25 | CORS with Spring Security | ⏳ Pending | — |
| 1.26 | Exception Handling: AuthenticationEntryPoint & AccessDeniedHandler | ⏳ Pending | — |
| 1.27 | Stateless vs Stateful Security | ⏳ Pending | — |
| 1.28 | JWT Authentication Architecture | ⏳ Pending | — |
| 1.29 | JWT Generation, Validation and Claims | ⏳ Pending | — |
| 1.30 | JWT Authentication Filter | ⏳ Pending | — |
| 1.31 | JWT Refresh Token Strategy | ⏳ Pending | — |
| 1.32 | JWT Revocation / Logout Strategy | ⏳ Pending | — |
| 1.33 | OAuth 2.0 Fundamentals | ⏳ Pending | — |
| 1.34 | OAuth 2.0 Roles and Grant Types | ⏳ Pending | — |
| 1.35 | OpenID Connect (OIDC) | ⏳ Pending | — |
| 1.36 | OAuth2 Login | ⏳ Pending | — |
| 1.37 | Resource Server | ⏳ Pending | — |
| 1.38 | JWT Resource Server | ⏳ Pending | — |
| 1.39 | Opaque Token Introspection | ⏳ Pending | — |
| 1.40 | Authorization Server Concepts | ⏳ Pending | — |
| 1.41 | Security Headers | ⏳ Pending | — |
| 1.42 | Password Storage & Credential Security | ⏳ Pending | — |
| 1.43 | Session Fixation, Brute Force and Common Attacks | ⏳ Pending | — |
| 1.44 | Security Testing with Spring Security Test | ⏳ Pending | — |
| 1.45 | MockMvc Security Testing | ⏳ Pending | — |
| 1.46 | Spring Security in Microservices | ⏳ Pending | — |
| 1.47 | API Gateway + Authentication | ⏳ Pending | — |
| 1.48 | Service-to-Service Authentication | ⏳ Pending | — |
| 1.49 | OAuth2 Client Credentials Flow | ⏳ Pending | — |
| 1.50 | Security Best Practices & Production Checklist | ⏳ Pending | — |
| 1.51 | Spring Security 6+ / Spring Boot 3+ Migration Notes | ⏳ Pending | — |
| 1.52 | Spring Security Interview Questions & Scenarios | ⏳ Pending | — |
