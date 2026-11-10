# Rohini Netekal — Java Backend Engineering Portfolio

> **Java Backend Developer | 4+ Years Experience | Spring Boot | Hibernate/JPA | SQL/Oracle | REST APIs**

This repository is my **hands-on backend engineering learning and interview portfolio**. It documents my continuous progress from Core Java fundamentals to production-oriented backend concepts, with practical examples, SQL exercises, architecture notes, testing, security, messaging, containers, cloud and system design.

> **Important:** The code and learning material in this repository are personal/demo work. They do not contain proprietary code, data, credentials, or confidential implementation details from any employer or client.

---

## 👩‍💻 About Me

I am a **Java Backend Developer with 4+ years of experience** working on enterprise backend applications in the banking domain.

### Professional focus

- Java backend development
- Spring / Spring Boot
- RESTful API development
- Hibernate / JPA
- SQL and Oracle database development
- Business logic and backend integrations
- Query optimization and database troubleshooting
- Production issue analysis and RCA
- Git-based development workflows
- Microservices concepts and backend integration
- Kafka fundamentals
- Secure API concepts with Spring Security and JWT

### What I am building here

I use this repository to turn concepts into **small, understandable and interview-ready implementations** instead of keeping them only as theory.

---

## 🛠️ Technology Roadmap

| Area | Focus |
|---|---|
| Java | OOP, Collections, Generics, Streams, Exceptions, Concurrency, JVM |
| Modern Java | Java 8+ features, Records, modern language features |
| Spring | IoC, DI, Beans, Spring MVC, Spring Boot |
| APIs | REST, HTTP, DTOs, validation, error handling, pagination |
| Persistence | Hibernate, JPA, entity lifecycle, mappings, transactions |
| Database | SQL, Oracle, joins, aggregation, indexes, query tuning |
| Distributed systems | Microservices, reliability, service communication |
| Messaging | Kafka, partitions, offsets, consumer groups, idempotency |
| Security | Spring Security, authentication, authorization, JWT |
| Testing | JUnit, Mockito, unit testing, integration testing |
| DevOps | Git, Docker, CI/CD |
| Observability | Structured logs, Micrometer metrics, distributed tracing, OpenTelemetry |
| Cloud | AWS fundamentals |
| Architecture | Design patterns, SOLID, system design |

---

## 📚 52-Week Backend Engineering Roadmap

The repository contains a structured 52-week roadmap covering:

**Phase 1 — Core Java**
- Java fundamentals
- Methods and clean code
- OOP
- Inheritance and polymorphism
- Abstraction and interfaces
- Strings
- Arrays and Big-O
- Collections
- Generics
- Exception handling

**Phase 2 — Modern Java & JVM**
- Java 8 fundamentals
- Streams
- Optional
- Modern Java features
- Immutability
- equals/hashCode
- Concurrency
- CompletableFuture
- JVM internals
- Garbage collection

**Phase 3 — Spring Backend**
- Spring fundamentals
- Spring Boot
- REST APIs
- Spring MVC
- Global exception handling
- Hibernate
- JPA mappings
- Fetch strategies
- Transactions

**Phase 4 — SQL & Data**
- SQL foundations
- Joins
- Aggregations
- Indexes
- Query optimization
- Oracle fundamentals

**Phase 5 — Distributed Backend**
- Microservices
- API reliability
- Kafka
- Kafka delivery semantics
- Idempotent consumers

**Phase 6 — Security & Quality**
- Spring Security
- JWT
- Testing
- JUnit and Mockito
- Docker
- CI/CD
- Git
- Design patterns
- SOLID

**Phase 7 — Architecture**
- System design
- AWS fundamentals
- Backend integration
- Interview revision

---

## 🗂️ Repository Structure

```text
Computer-Science/
│
├── core-java/
│   ├── collections-streams/
│   ├── immutability/
│   └── concurrency/
│
├── spring-boot/
│   ├── rest/
│   └── error-handling/
│
├── hibernate-jpa/
│   └── entity/
│
├── sql/
│   └── query-optimization/
│
├── kafka/
│   └── idempotent-consumer/
│
├── spring-security/
│   └── jwt/
│
├── testing/
│   └── junit/
│
├── docker/
│
├── learning-journal/
│   ├── week-01/
│   ├── week-02/
│   ├── ...
│   └── week-52/
│
└── LEARNING-JOURNAL.md
```

---

## 🔎 Example Work Already Included

### Core Java — Collections & Streams

A backend-style order analytics example demonstrating:

- Java records
- Collections
- Stream filtering and mapping
- `groupingBy`
- `summingDouble`
- sorting
- finding maximum values
- focused methods
- complexity analysis

➡️ [View OrderAnalytics.java](core-java/collections-streams/OrderAnalytics.java)

### Spring Boot — REST API Design

Covers practical REST design decisions including:

- HTTP methods
- status codes
- DTOs
- validation
- pagination
- idempotency
- resource-oriented API design

➡️ [View REST API notes](spring-boot/rest/API_DESIGN.md)

### Hibernate / JPA

Covers:

- entity lifecycle
- persistence context
- dirty checking
- entity mappings
- lazy/eager loading
- transaction concepts

➡️ [View Hibernate/JPA example](hibernate-jpa/entity/Merchant.java)

### SQL & Query Optimization

Focuses on backend interview and production-oriented database concepts:

- indexes
- execution plans
- filtering
- sorting
- pagination
- query tuning
- avoiding unnecessary data retrieval

➡️ [View SQL notes](sql/query-optimization/README.md)

### Kafka

Includes concepts around:

- producers
- consumers
- topics
- partitions
- offsets
- consumer groups
- duplicate processing
- idempotent consumers

➡️ [View Kafka notes](kafka/idempotent-consumer/README.md)

For producer-side delivery reliability, see the [Transactional Outbox guide](https://github.com/RohiniNetekal/Microservices-Engineering-Lab/blob/master/docs/transactional-outbox-pattern.md), which complements consumer idempotency.

### Observability — logs, metrics and distributed traces

The [Distributed Tracing and Observability guide](https://github.com/RohiniNetekal/Microservices-Engineering-Lab/blob/master/docs/distributed-tracing-observability.md) connects Spring Boot/Micrometer instrumentation, HTTP and Kafka context propagation, Oracle/JDBC visibility, outbox tracing, sampling, cardinality and incident investigation.

### Spring Security — OAuth2 JWT Resource Server

The [OAuth2 Resource Server implementation guide](spring-security/jwt/OAUTH2_RESOURCE_SERVER_IMPLEMENTATION.md) demonstrates a modern Spring MVC API that validates externally issued JWT access tokens using issuer, JWKS, audience and algorithm configuration. It covers the security filter chain, scope-to-authority mapping, object-level authorization, 401 vs 403 behavior and the difference between MockMvc authorization tests and real signed-token validation.

---

## 🎯 Learning Approach

1. **Concept** — what it means
2. **Why it matters** — where it is useful
3. **Practical example** — small implementation
4. **Backend use case** — how it appears in real applications
5. **Complexity / trade-offs** — where relevant
6. **Interview questions** — common ways the concept is tested
7. **Practice checklist** — what still needs deeper work

---

## 📈 Continuous Development

- HashMap internals and concurrency
- Advanced Java concurrency
- JVM troubleshooting
- Spring Boot production patterns
- Hibernate performance and N+1 solutions
- Advanced SQL problems and query tuning
- Microservice communication patterns
- Kafka reliability patterns
- Security implementation examples
- JUnit + Mockito test suites
- Dockerized backend applications
- CI/CD examples
- Redis and caching concepts
- AWS deployment concepts
- System design case studies
- End-to-end backend projects

---

## 💼 Professional Experience Context

My professional experience is in **enterprise Java backend development in the banking domain**.

This repository is intentionally separate from employer code. It demonstrates transferable engineering concepts through independent examples and learning projects without exposing proprietary information.

---

## 📌 Current Goal

Build a strong, transparent and continuously improving:

**Core Java → Spring Boot → REST → Hibernate/JPA → SQL → Microservices → Kafka → Security → Testing → Docker/CI/CD → AWS → System Design**

with enough practical depth to support **Java Backend Developer / Senior Java Backend interview preparation**.


> **I learn backend concepts by implementing them, documenting the reasoning, testing the ideas, and continuously improving the depth of the work.**


## CI/CD and GitHub Actions

The [Week 45 CI/CD reference](learning-journal/week-45-cicd.md) now connects Maven's verify lifecycle, Surefire/Failsafe, Testcontainers, Java version matrices and secure artifact promotion. The detailed [Java backend CI/CD guide](https://github.com/RohiniNetekal/Git-Engineering-Workflow-Lab/blob/main/docs/java-backend-ci-cd.md) includes a reusable workflow template and deployment-security guidance.

The Git workflow repository has an active Markdown quality workflow. The Maven YAML is a template for a real Spring Boot project, not a claim that this documentation-focused repository currently runs a Java build.
