# Spring-S2.1 — DAO Introduction

> **Status:** 🚧 In Progress / Introduction started

## 1. What is DAO?

**DAO = Data Access Object**.

DAO is a design pattern used to separate **data-access logic** from business logic.

```text
Controller
    ↓
Service
    ↓
DAO / Repository
    ↓
Database
```

The Service should focus on business rules, while DAO handles interaction with the persistence mechanism.

---

## 2. Why do we need DAO?

Without separation, a service class may contain:

```text
Business Logic
      +
JDBC Connection
      +
SQL Queries
      +
Statement/PreparedStatement
      +
ResultSet handling
      +
Exception handling
      +
Connection closing
```

This creates tight coupling and makes code difficult to test and maintain.

DAO separates these responsibilities.

```text
Business Layer
      ↓
DAO Interface
      ↓
DAO Implementation
      ↓
Data Source
```

---

## 3. DAO Example

### DAO Interface

```java
public interface EmployeeDao {

    Employee findById(Long id);

    void save(Employee employee);
}
```

### DAO Implementation

```java
@Repository
public class EmployeeDaoImpl implements EmployeeDao {

    @Override
    public Employee findById(Long id) {
        // JDBC / JPA / other persistence logic
        return null;
    }

    @Override
    public void save(Employee employee) {
        // database interaction
    }
}
```

### Service

```java
@Service
public class EmployeeService {

    private final EmployeeDao employeeDao;

    public EmployeeService(EmployeeDao employeeDao) {
        this.employeeDao = employeeDao;
    }

    public Employee getEmployee(Long id) {
        return employeeDao.findById(id);
    }
}
```

The service does not need to know whether the DAO internally uses JDBC, JPA, MongoDB, etc.

---

## 4. DAO and Repository

In modern Spring applications, you will frequently see the term **Repository** instead of DAO.

Conceptually:

```text
DAO
→ focuses on data-access abstraction

Repository
→ commonly represents persistence/data-access abstraction
```

Spring Data provides repository abstractions such as:

```java
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {
}
```

For interview purposes, remember:

> DAO is the general data-access design pattern; Repository is a commonly used abstraction for accessing domain data, especially in Spring Data applications.

---

## 5. DAO's Main Responsibility

DAO should primarily handle:

- Create
- Read
- Update
- Delete
- Query execution
- Mapping database results to objects
- Persistence-related exceptions/operations

DAO should **not** contain business rules such as:

```text
if employee is eligible for bonus
if customer qualifies for discount
if payment should be approved
```

Those belong in the service/business layer.

---

## 6. DAO with JDBC

Traditional JDBC data access often looks like:

```java
public Employee findById(Long id) {

    String sql = "SELECT id, name FROM employee WHERE id = ?";

    // obtain connection
    // create PreparedStatement
    // set parameter
    // execute query
    // process ResultSet
    // close resources

    return employee;
}
```

This shows why Spring JDBC abstractions become useful: they reduce repetitive JDBC infrastructure code.

---

## 7. Spring's DAO Support

Spring provides support for data access through technologies such as:

```text
JDBC
JPA
Hibernate
Transactions
ORM integrations
```

A major goal is to simplify infrastructure concerns while keeping application code focused on its responsibilities.

For Spring JDBC, a common abstraction is:

```java
JdbcTemplate
```

Instead of manually managing much of the JDBC boilerplate, the application can delegate common operations to `JdbcTemplate`.

---

## 8. DAO Exception Handling

Traditional JDBC has checked exceptions such as:

```java
SQLException
```

Spring's data-access support provides a consistent exception abstraction through its **DataAccessException** hierarchy.

Important interview point:

> Spring translates technology-specific data-access exceptions into a consistent unchecked exception hierarchy.

This reduces coupling between application code and a specific persistence technology's exception model.

---

## 9. DAO Architecture

```text
                    ┌──────────────┐
                    │  Controller  │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │    Service   │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │ DAO/Repository│
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │ Data Source  │
                    └──────────────┘
```

This separation improves maintainability and makes responsibilities clearer.

---

## 10. Interview Answer — What is DAO?

### 30-second answer

> **“DAO stands for Data Access Object. It is a design pattern used to separate database or persistence-related operations from business logic. The service layer handles business rules, while the DAO handles operations such as querying, inserting, updating and deleting data. In Spring applications, DAO-style data access can be implemented using JDBC, JPA, Hibernate or Spring Data repositories.”**

### Hinglish memory

> **“DAO ka kaam database se baat karna hai; Service ka kaam business logic handle karna hai. DAO separation se business code database implementation se loosely coupled rehta hai.”**

---

## 11. Interview Follow-ups

1. What is DAO?
2. Why do we need DAO?
3. DAO vs Repository?
4. DAO vs Service?
5. Where should business logic reside?
6. How does Spring support DAO?
7. What is `JdbcTemplate`?
8. What is Spring's `DataAccessException`?
9. Why is exception translation useful?
10. Can a DAO use JPA or Hibernate?
11. Why should a service not contain raw JDBC code?
12. How would you test a DAO?

---

## 12. Memory Map

```text
DAO
│
├── Data Access Object
│
├── Separates persistence from business logic
│
├── CRUD / queries
│
├── JDBC / JPA / Hibernate / other persistence
│
├── Spring support
│   ├── JdbcTemplate
│   └── DataAccessException
│
└── Service remains focused on business logic
```

### One-line memory

> **DAO = database/persistence access ko business logic se separate karne wali abstraction.**

---

## Navigation

[🏠 Master README](../../README.md)

**Next source sequence:** DAO Introduction ke next subtopic par continue karenge.
