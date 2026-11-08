# Spring Boot Production Readiness Guide

A practical checklist for turning a Spring Boot API from a working local application into a service that is safer to operate in a real environment.

> This is independent learning material. It does not reproduce employer or client implementation details.

## 1. Configuration

Keep environment-specific values outside the source code.

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Prefer environment variables, a secrets manager or platform configuration for credentials. Do not commit passwords, tokens or production connection strings.

Useful configuration areas:
- Database connection settings
- Connection pool limits
- External service URLs
- Timeouts
- Feature flags
- Logging levels
- Active profile

## 2. Health and readiness

A service can be running while still being unable to serve traffic.

Think about two questions:

**Liveness:** Is the application process alive?

**Readiness:** Is the application ready to receive traffic and use required dependencies?

Example lifecycle:

```text
Container starts
     |
     v
Application boot
     |
     v
Dependencies/config loaded
     |
     v
Readiness = UP
     |
     v
Traffic allowed
```

Spring Boot Actuator can expose health information that can be integrated with container/orchestration platforms.

## 3. API error handling

Do not return stack traces to clients.

A consistent error response should contain enough information for the caller and support troubleshooting:

```json
{
  "code": "MERCHANT_NOT_FOUND",
  "message": "Merchant was not found",
  "timestamp": "2026-09-23T08:00:00Z",
  "correlationId": "8c21d1"
}
```

Keep internal exception details in server-side logs instead of exposing database or implementation details.

## 4. Timeouts

Every external network call should have an intentional timeout.

```text
Service A
   |
   | HTTP request
   v
Service B
   |
   +--> responds quickly -> continue
   |
   +--> slow/unavailable -> timeout -> controlled failure
```

Without a timeout, blocked calls can consume application threads/resources and create cascading failures.

## 5. Retry carefully

Retries are useful for transient failures, but they are not a universal error-handling solution.

Use:
- Limited retry count
- Exponential backoff
- Jitter where appropriate
- Timeouts
- Idempotency for retryable business operations

Avoid retrying permanent failures such as validation errors.

Example decision:

```text
HTTP failure
    |
    +--> 400/401/403 -> normally do not retry
    |
    +--> transient 5xx/network timeout -> retry only if safe
    |
    +--> non-idempotent operation -> require idempotency strategy
```

## 6. Database and transactions

Keep a transaction around a coherent business operation rather than around an unnecessarily large request flow.

Review:
- Transaction boundaries
- Isolation level
- Connection pool size
- Slow queries
- Missing/ineffective indexes
- N+1 queries
- Large result sets
- Pagination

For Hibernate/JPA, inspect generated SQL when diagnosing performance rather than assuming the entity code is efficient.

## 7. Logging

Good logs answer:
- What operation happened?
- Which request/user/business identifier was involved?
- Did it succeed or fail?
- How long did it take?
- What dependency failed?

Prefer structured logging and consistent correlation/trace identifiers.

Never log passwords, tokens, card data, private keys or other sensitive information.

## 8. Observability

A production service should provide more than logs. Use three complementary signals:

- **Logs:** explain what happened and include safe diagnostic context.
- **Metrics:** show request rate, error rate, p95/p99 latency, JVM health and dependency saturation.
- **Traces:** show where one request spent time across application, database, HTTP and messaging boundaries.

Spring Boot uses Micrometer Observation for metrics and tracing. With Micrometer Tracing configured, trace and span IDs can also correlate log lines.

Implementation walkthrough: [Distributed Tracing and Observability for Spring Boot Microservices](https://github.com/RohiniNetekal/Microservices-Engineering-Lab/blob/master/docs/distributed-tracing-observability.md). It covers OTLP setup, W3C context propagation, custom observations, Kafka producer/consumer observations, asynchronous outbox context, sampling, cardinality and incident triage.

Important production checks:
- Do not assume JDBC/Oracle spans appear merely because Actuator is present; add and verify compatible datasource instrumentation or an agent.
- Use auto-configured HTTP client builders when trace propagation is required.
- Enable Kafka observations on both producer and consumer sides.
- Keep metric tags low-cardinality and never put secrets or unnecessary personal data in telemetry.
- Choose sampling deliberately; 100% sampling is for small local exercises, not a universal production setting.

A useful production question is:

> "The API became slow. Can you use metrics to find the affected operation, a trace to locate the slow span, and logs to explain the failure?"

## 9. Security baseline

Before exposing an API, review:

- Authentication
- Authorization
- Password hashing
- JWT validation if JWT is used
- Input validation
- CORS policy
- CSRF considerations
- TLS
- Rate limiting
- Secret management
- Dependency vulnerabilities
- Least privilege

Do not trust claims simply because they are present in a JWT; validate the token signature and relevant claims according to the security design.

## 10. Testing pyramid

```text
          E2E
        /-----\
       Integration
      /-----------\
         Unit
    /---------------\
```

A practical backend suite can combine:
- Fast unit tests for business logic
- Integration tests for Spring wiring/database behavior
- Testcontainers for realistic infrastructure integration
- A smaller number of end-to-end tests for critical flows

## 11. Container readiness

A production Docker image should have:
- Small base image where appropriate
- Non-root execution where practical
- Explicit port/configuration
- Environment-based configuration
- Health checks where appropriate
- Predictable startup behavior
- No secrets baked into the image

## 12. CI/CD gate

A basic pipeline can be:

```text
Pull Request
    |
    v
Compile
    |
    v
Unit Tests
    |
    v
Static/quality checks
    |
    v
Integration Tests
    |
    v
Build artifact/container image
    |
    v
Deploy to environment
```

The exact pipeline depends on the organization, but the principle is the same: catch defects before deployment and make releases repeatable.

## Production-readiness checklist

- [ ] Configuration externalized
- [ ] Secrets protected
- [ ] Input validation implemented
- [ ] Global error handling implemented
- [ ] Health/readiness checks available
- [ ] External calls have timeouts
- [ ] Retry policy is intentional
- [ ] Idempotency considered for retryable writes
- [ ] SQL/query performance reviewed
- [ ] Transaction boundaries reviewed
- [ ] Structured logging available
- [ ] Correlation/trace IDs considered
- [ ] Metrics and tracing considered
- [ ] Authentication/authorization reviewed
- [ ] Unit tests present
- [ ] Integration tests present for important infrastructure behavior
- [ ] Docker image reviewed
- [ ] CI pipeline runs before merge/deployment

## Interview questions

1. What is the difference between liveness and readiness?
2. Why can an API become slow even when CPU is low?
3. Why should every downstream HTTP call have a timeout?
4. When should you retry a failed request?
5. How can retries create a retry storm?
6. How would you investigate a sudden increase in API latency?
7. What should never be logged?
8. How would you test a Spring Boot service that depends on a real database?
9. Why should secrets not be stored in application.yml?
10. What makes a backend service production-ready beyond simply returning the correct HTTP response?
