# Spring - S5 — Advanced & Missing Areas

> This module covers the important areas not sufficiently covered in S1–S4 and completes the Spring/Spring Boot interview-level roadmap.

## 1. Spring Boot Auto-Configuration Deep Dive
- Auto-configuration internals
- `@SpringBootApplication` internals
- `@EnableAutoConfiguration`
- Conditional annotations
- Auto-configuration ordering
- Starter dependencies
- Custom auto-configuration
- Auto-configuration debugging

## 2. Spring Boot Configuration & Profiles
- Externalized configuration
- `application.properties` / YAML
- Profiles
- `@Profile`
- `@Value`
- `@ConfigurationProperties`
- Property precedence
- Environment and command-line properties
- Configuration validation
- Secrets and secure configuration

## 3. Spring Boot Bean & Conditional Configuration
- `@Bean` and `@Configuration`
- Conditional beans
- `@ConditionalOnBean`
- `@ConditionalOnMissingBean`
- `@ConditionalOnProperty`
- `@Primary` and `@Qualifier`
- Lazy initialization
- Circular dependency handling

## 4. Spring Boot REST API Advanced
- Request validation
- Bean Validation
- Custom validators
- `@Valid` / `@Validated`
- Global exception handling
- `@ControllerAdvice` / `@RestControllerAdvice`
- `ProblemDetail` and REST error design
- Content negotiation
- API versioning
- Pagination and sorting
- Idempotency
- API security boundaries

## 5. Spring Data JPA Advanced
- Entity lifecycle
- Persistence context
- First-level cache
- Entity relationships
- Fetch strategies
- Cascade types
- Orphan removal
- JPQL
- Native queries
- Projections
- Specifications
- Pagination
- N+1 query problem
- Entity graphs
- Optimistic locking
- Pessimistic locking

## 6. Spring Transaction Management Advanced
- `@Transactional` internals
- Transaction propagation
- Isolation levels
- Read-only transactions
- Rollback rules
- Transaction proxy limitations
- Self-invocation problem
- Transaction boundaries
- Local vs distributed transactions
- Saga pattern overview

## 7. Spring Boot Caching
- Spring Cache abstraction
- `@Cacheable`
- `@CachePut`
- `@CacheEvict`
- Cache keys
- Cache invalidation
- Redis caching
- TTL
- Distributed cache considerations
- Cache consistency

## 8. Spring Boot Testing
- Unit vs integration testing
- `@SpringBootTest`
- `@WebMvcTest`
- `@DataJpaTest`
- MockMvc
- Mockito
- Test slices
- Test configuration
- Testcontainers
- Database integration testing
- Security testing
- Contract testing overview

## 9. Spring Boot Observability
- Logging architecture
- Structured logging
- Correlation IDs
- Micrometer
- Metrics
- Custom metrics
- Health indicators
- Liveness/readiness
- Distributed tracing
- OpenTelemetry concepts
- Production monitoring

## 10. Spring Boot Async & Scheduling
- `@Async`
- Executor configuration
- Thread pool sizing
- `CompletableFuture` integration
- Exception handling in async tasks
- Context propagation
- `@Scheduled`
- Fixed rate vs fixed delay
- Cron scheduling
- Distributed scheduling considerations

## 11. Spring Boot Messaging & Kafka
- Spring Kafka fundamentals
- Producer configuration
- Consumer configuration
- Consumer groups
- Offset management
- Serialization/deserialization
- Error handling
- Retry and dead-letter topics
- Idempotent consumers
- Kafka transactions overview
- Event-driven architecture

## 12. Spring Boot Microservices Integration
- OpenFeign
- REST service-to-service communication
- Service discovery integration
- API Gateway integration
- Config Server concepts
- Circuit Breaker
- Retry
- Timeout
- Bulkhead
- Rate limiting
- Resilience4j

## 13. Spring Boot Security Integration
- Spring Boot + Spring Security architecture
- JWT Resource Server integration
- OAuth2 Client integration
- OAuth2 Login
- Method security
- Service-to-service authentication
- Security configuration best practices
- Secure actuator endpoints
- CORS/CSRF production decisions

## 14. Production Readiness
- Graceful shutdown
- Connection pool tuning
- HikariCP
- JVM/application configuration
- Secure headers
- Secrets management
- TLS/HTTPS
- Error sanitization
- Dependency vulnerability scanning
- SBOM concepts
- Audit logging
- Rate limiting

## 15. Docker & Deployment
- Containerizing Spring Boot
- Dockerfile best practices
- Layered JARs
- Container memory configuration
- Environment variables
- Health checks
- Kubernetes basics
- ConfigMap / Secret concepts
- Readiness/liveness probes
- Rolling deployment concepts

## 16. Spring Boot Performance
- Startup performance
- Lazy initialization
- Database query optimization
- Connection pool tuning
- Caching strategy
- Thread pool tuning
- Memory considerations
- Actuator metrics analysis
- Bottleneck identification
- Load testing

## 17. Architecture & Design Scenarios
- Monolith vs modular monolith vs microservices
- Layered architecture
- Hexagonal architecture
- Clean architecture
- Domain-driven design basics
- Transaction boundary design
- Distributed consistency
- Idempotency
- Resilience patterns
- Security boundaries

## 18. Spring Boot Interview Revision
- Internal flow questions
- Auto-configuration questions
- Configuration questions
- JPA questions
- Transaction questions
- REST questions
- Testing questions
- Production troubleshooting
- Performance scenarios
- Architecture scenarios

## 19. Spring Boot Final Assessment
- 30-second answers
- 60-second answers
- Scenario-based questions
- Production debugging questions
- Architecture design questions
- 5+ year interview mock assessment

## Status

| Section | Status |
|---|---|
| S1 Spring Fundamentals | Existing / Reviewed |
| S2 Spring JDBC / DAO / Web | Existing / Reviewed |
| S3 Spring Boot Core | Existing but gaps identified |
| S4 Spring Security | Existing / Reviewed |
| S5 Advanced & Missing Areas | 🚧 Started |

**Goal:** Close the major gaps identified after reviewing Spring S1–S4 without removing or duplicating the existing material.
