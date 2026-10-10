# Spring Transactions & Hibernate — Practical Guide

This guide explains how a business operation should define its database transaction boundary, what Spring's `@Transactional` actually controls, and how to reason about failures and concurrent updates.

> Independent learning/demo material. Examples use illustrative entities and repositories; they do not represent any employer's implementation.

## 1. Why transaction boundaries matter

Suppose a transfer moves money from one account to another. The debit and credit must not commit independently.

```text
Transfer request
      |
      v
BEGIN TRANSACTION
  1. Read source account
  2. Read destination account
  3. Validate amount and balance
  4. Debit source
  5. Credit destination
      |
      +---- all operations succeed ---> COMMIT
      |
      +---- failure ------------------> ROLLBACK
```

Without one transaction, a failure after the debit but before the credit could leave inconsistent state.

A transaction boundary should normally cover one coherent business operation—not an entire controller call chain by accident and not every individual DAO method separately.

## 2. Put the boundary in the service layer

Example using Spring and a repository abstraction:

```java
@Service
public class TransferService {

    private final AccountRepository accounts;

    public TransferService(AccountRepository accounts) {
        this.accounts = accounts;
    }

    @Transactional
    public void transfer(Long sourceId, Long destinationId,
                         BigDecimal amount) {
        if (amount == null || amount.signum() <= 0) {
            throw new IllegalArgumentException(
                    "Amount must be positive");
        }
        if (sourceId.equals(destinationId)) {
            throw new IllegalArgumentException(
                    "Accounts must be different");
        }

        Account source = accounts.findById(sourceId)
                .orElseThrow(() -> new AccountNotFoundException(sourceId));
        Account destination = accounts.findById(destinationId)
                .orElseThrow(() -> new AccountNotFoundException(destinationId));

        source.debit(amount);
        destination.credit(amount);
        // Managed entities are normally detected by dirty checking.
        // The transaction commits when this method completes successfully.
    }
}
```

This is an illustrative example: real financial operations also need concurrency control, currency/precision rules, auditability, idempotency and database constraints.

### What happens internally?

1. A call enters through Spring's transaction proxy/interceptor.
2. Spring starts or joins a transaction according to the configured manager and propagation.
3. Repository operations execute within that transaction.
4. Hibernate tracks managed entity changes in the persistence context.
5. At flush/commit, SQL is synchronized with the database.
6. Success commits; an eligible failure triggers rollback.

SQL may be flushed before the final commit, for example before a query or during explicit flush. A successful Java method return is not proof that the database commit succeeded until the transaction interceptor finishes.

## 3. Rollback rules

By default, Spring rolls back on unchecked exceptions (`RuntimeException`) and `Error`. Checked exceptions do not trigger rollback by default.

If a checked exception must roll back the operation, configure it explicitly:

```java
@Transactional(rollbackFor = Exception.class)
public void processBatch() throws Exception {
    // All-or-nothing work for this transaction
}
```

Do not catch an exception, log it, and silently return if the transaction should fail. Swallowing the exception can make Spring see a successful method completion. If you catch an exception, either handle it intentionally or rethrow/mark rollback according to the business requirement.

## 4. Propagation

Propagation defines what happens when a transactional method is called while a transaction may already exist.

| Mode | Typical meaning | Caution |
|---|---|---|
| `REQUIRED` | Join existing transaction or start one | Default; inner failure may mark shared transaction rollback-only |
| `REQUIRES_NEW` | Suspend current transaction and start a separate one | Inner work may commit even if outer work later rolls back |
| `SUPPORTS` | Join if present, otherwise run without one | Behavior differs depending on caller |
| `MANDATORY` | Require an existing transaction | Fails if none exists |
| `NOT_SUPPORTED` | Suspend existing transaction and run without one | Work is not part of the suspended transaction |
| `NEVER` | Fail if a transaction exists | Useful only for specific constraints |
| `NESTED` | Nested/savepoint behavior where supported | Depends on transaction manager and database support |

Do not choose `REQUIRES_NEW` simply to “fix” rollback behavior. It creates an independent commit boundary and can produce partial business outcomes. Use it only when independent persistence is a deliberate requirement, such as certain audit records.

## 5. Isolation and concurrency

Isolation controls what concurrent transactions can observe. Exact behavior depends on the database and its implementation.

| Level | General idea | Main concern |
|---|---|---|
| READ_UNCOMMITTED | May permit dirty reads | Often unsupported or treated differently by databases |
| READ_COMMITTED | Reads only committed data | Repeated reads may change |
| REPEATABLE_READ | Stronger consistency for rows read | Phantom behavior varies by database |
| SERIALIZABLE | Strongest standard isolation model | More contention, blocking or transaction retries may occur |

Isolation is not a substitute for modelling invariants. If two requests can update the same row, decide how conflicts should be detected and resolved.

### Optimistic locking

For entities where conflicts are uncommon, a version field can detect concurrent updates:

```java
@Version
private long version;
```

Hibernate includes the version in update checks. If another transaction has already changed the row, the stale update fails with an optimistic-locking exception instead of silently overwriting the newer value.

### Pessimistic locking

A database lock such as `SELECT ... FOR UPDATE` can be useful when a row must be locked before a critical read-modify-write operation. It can reduce concurrency and introduce blocking/deadlock risk, so keep the locked section short and use a consistent lock order.

## 6. Read-only transactions

```java
@Transactional(readOnly = true)
public AccountSummary getSummary(Long accountId) {
    // Read and map data
}
```

Read-only is a hint that can allow the transaction manager or persistence provider to optimize the operation. It is **not a universal guarantee** that writes are impossible. Use it for genuinely read-only service operations and do not rely on it as an authorization or data-integrity control.

## 7. Common proxy pitfalls

### Self-invocation

```java
public void outerOperation() {
    this.innerTransactionalOperation(); // may bypass the Spring proxy
}

@Transactional
public void innerTransactionalOperation() {
    // Do not assume a new transactional interceptor runs here.
}
```

In the usual proxy-based configuration, calls must pass through the Spring proxy for transaction advice to apply. Move the transactional operation to another Spring bean or redesign the boundary rather than relying on a self-call.

### Other pitfalls

- Calling a transactional method on an object created with `new` instead of Spring.
- Assuming a transaction automatically follows work onto another thread.
- Holding a database transaction open during slow HTTP calls.
- Performing long file/network operations while holding row locks.
- Catching and suppressing exceptions that should cause rollback.
- Expecting a checked exception to roll back without configuring rollback rules.
- Using `REQUIRES_NEW` without considering partial commits.
- Assuming `readOnly = true` enforces read-only access.
- Treating `flush()` as the same thing as commit.

## 8. Database transaction vs distributed operation

A Spring database transaction generally covers one transaction manager/resource. It does not make a remote REST call or Kafka publish atomic with a database commit.

For a database change plus event publication, consider the **transactional outbox** pattern:

```text
One DB transaction:
  update business row
  insert outbox event row
             |
           COMMIT
             |
             v
Separate publisher reads outbox and publishes event
             |
             v
Consumer processes event idempotently
```

This avoids the classic gap where the database commits but the event is never published, or the event is published while the database rolls back. The publisher and consumer still need retry, deduplication and monitoring strategies.

## 9. Testing transaction behavior

Test more than the happy path:

- A failure after the first database mutation leaves no partial update.
- Checked exceptions follow the configured rollback rule.
- Concurrent updates detect conflicts as designed.
- A read-only operation does not accidentally mutate data.
- Outbox rows commit or roll back with the business change.
- Tests use the actual database engine or a compatible Testcontainers setup when database-specific behavior matters.

Be careful with tests that run inside a test-managed transaction and automatically roll back: they may hide behavior that occurs at a real service commit boundary.

## 10. Troubleshooting checklist

When a write appears not to persist, check:

1. Is the service method actually called through a Spring-managed bean?
2. Is the expected transaction manager selected?
3. Is the method joining an existing transaction?
4. Was an exception swallowed or the transaction marked rollback-only?
5. Did flush/commit fail after the method body returned?
6. Is the entity managed, or was it detached?
7. Did another transaction overwrite the data or trigger optimistic locking?
8. Is the read querying the expected database/schema?
9. Is the test rolling back its enclosing transaction?

## 11. Interview questions

1. Why is a service-layer transaction boundary often preferred over DAO-level transactions?
2. What is the difference between `REQUIRED` and `REQUIRES_NEW`?
3. Which exceptions cause rollback by default in Spring?
4. Why might `@Transactional` not apply to a self-invoked method?
5. Does `readOnly = true` prevent every possible write?
6. What is the difference between flush and commit?
7. When would you choose optimistic vs pessimistic locking?
8. Why does a database transaction not make a Kafka publish atomic?
9. How can a transaction appear successful in Java but fail at commit?
10. What should you inspect when concurrent requests update the same record?

## Practice exercises

1. Write a transfer service and test rollback after a simulated second-step failure.
2. Compare `REQUIRED` and `REQUIRES_NEW` with two service beans.
3. Add `@Version` and simulate two updates based on the same version.
4. Compare the execution behavior of READ_COMMITTED and a stronger isolation level in your target database.
5. Design an outbox table and explain how the publisher handles retries and duplicate delivery.
