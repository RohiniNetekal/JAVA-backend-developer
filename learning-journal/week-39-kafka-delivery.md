# One-Page Concept Reference

**Status:** ✅ Done — reference example created

## Kafka Delivery & Idempotency

### Short explanation
At-least-once processing can deliver the same event more than once. Consumers should make repeated processing safe.

### Handy example
```java
if (processedEventRepository.exists(eventId)) return;
process(event);
processedEventRepository.save(eventId);
```

### Key point
Remember: Design the business operation so duplicate messages do not create duplicate effects.

### Interview check
- What is it?
- Why would you use it?
- What is one common mistake or trade-off?


## What to cover
- [ ] At-most-once
- [ ] At-least-once
- [ ] Exactly-once concept
- [ ] Acknowledgements
- [ ] Offset commits
- [ ] Duplicate events
- [ ] Idempotent consumer
- [ ] Dead-letter concept

## Concept in slightly more detail
At-least-once processing can produce duplicates, so business processing should be safe to repeat.

## Practical example
```text
if(processedEventRepository.exists(eventId)) return; process(event); save(eventId);
```

## Transactional outbox — producer-side reliability

A consumer inbox protects against duplicate effects after a message arrives. The transactional outbox solves a different problem: reliably recording the fact that a committed database change must be published.

Without an outbox, an API can commit an order in Oracle and then fail before publishing `OrderCreated` to Kafka. Publishing first is also unsafe because the database transaction may roll back.

The outbox writes both records in one local transaction:

```text
@Transactional
  update ORDER
  insert OUTBOX_EVENT
COMMIT
  |
  v
publisher -> Kafka -> consumer inbox -> business effect
```

The publisher sends committed outbox rows and marks them published only after Kafka acknowledges. A crash after Kafka acknowledgement but before the status update can cause a duplicate, so consumers still need an inbox/unique event ID. The outbox provides durable at-least-once publication, not end-to-end exactly-once delivery.

**Deep implementation guide:** [Transactional Outbox — Spring Boot, Oracle and Kafka](https://github.com/RohiniNetekal/Microservices-Engineering-Lab/blob/master/docs/transactional-outbox-pattern.md).

### Interview questions
1. Why doesn't `@Transactional` make an Oracle commit and Kafka send atomic?
2. What is the dual-write problem?
3. Why can an outbox publisher send the same event twice?
4. How do a consumer inbox and unique event ID prevent duplicate business effects?
5. What does `FOR UPDATE SKIP LOCKED` help with when several publisher workers run?

## Done checklist
- [x] Reference created
- [ ] All checklist points understood
- [ ] Example explained without notes
- [ ] 3 interview questions practiced
