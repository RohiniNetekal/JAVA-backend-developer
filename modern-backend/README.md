# Modern Backend Engineering — 2026 Topics

These are additional topics being added alongside the 52-week foundation because a modern Java backend portfolio should demonstrate awareness of production engineering, not only framework syntax.

## 1. Virtual Threads

Virtual threads are lightweight Java threads intended for high-throughput workloads with substantial blocking I/O. They are about scalability/throughput, not making CPU work execute faster.

Study:
- Platform vs virtual threads
- Thread-per-request model
- Blocking I/O
- Thread pools vs virtual-thread-per-task
- Pinning and debugging
- When virtual threads help and when they do not

## 2. Structured Concurrency

Learn how related concurrent tasks can be treated as one unit so cancellation, failures and lifecycle are easier to reason about.

Status matters: Structured Concurrency remains a preview feature in Java 25, so this repository treats it as a concept to learn rather than a production dependency to assume.

## 3. Observability

A production backend should make it possible to answer:
- What failed?
- Where did it fail?
- Why did it fail?
- Which downstream service/database was involved?

Study:
- Logs
- Metrics
- Distributed traces
- Correlation/context propagation
- OpenTelemetry
- Health checks
- Production debugging

Implementation walkthrough: [Distributed Tracing and Observability for Spring Boot Microservices](https://github.com/RohiniNetekal/Microservices-Engineering-Lab/blob/master/docs/distributed-tracing-observability.md). It covers Micrometer Observation, OpenTelemetry OTLP, trace/log correlation, HTTP and Kafka propagation, asynchronous outbox context, sampling, cardinality and incident triage.

## 4. Testcontainers

Move beyond only mocks and in-memory databases.

Study:
- Real database integration tests
- Kafka integration tests
- Container lifecycle
- Test isolation
- Reproducible integration environments

## 5. API Engineering

Add:
- OpenAPI/Swagger
- Contract-first thinking
- Pagination
- Idempotency keys
- Rate limiting
- Timeouts
- Retry/backoff
- API versioning

## 6. Performance

Study:
- JVM memory
- GC
- SQL execution plans
- Connection pools
- Thread pools
- Caching
- Profiling
- Load testing
- Bottleneck identification

## 7. Cloud & Platform

Foundation:
- Docker
- Kubernetes concepts
- AWS compute/storage/database/networking
- IAM
- Load balancing
- Secrets/configuration
- CI/CD

The goal is not to claim production expertise in every tool. The goal is to understand the architecture, trade-offs and terminology well enough to build and explain practical examples.
