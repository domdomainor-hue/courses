# Distributed Transactions Between Microservices and Kafka

Distributed transactions are needed when one business operation touches multiple independent systems, such as several microservices, a database, Kafka, an external payment provider, or another third-party API.

The hard part is that each system can succeed or fail independently.

In a monolith, `@Transactional` can usually protect one database transaction. In distributed systems, there is no single local transaction that can safely cover every service, database, message broker, and external API.

---

## What Is a Distributed Transaction?

A **distributed transaction** is a business operation that requires multiple resources to reach a consistent final state.

Examples:

- Create an order, reserve stock, and charge payment.
- Save data in a database and publish an event to Kafka.
- Update two microservices that each own their own database.
- Consume a Kafka event, write to a database, and publish another event.

The main challenge is partial failure:

| Step | Result |
| --- | --- |
| Database commit succeeds | Kafka publish fails |
| Service A succeeds | Service B is down |
| Payment succeeds | Order update fails |
| Kafka message is consumed | Consumer crashes before saving progress |

Distributed transaction design is mostly about handling these partial failures explicitly.

---

## Why `@Transactional` Is Not Enough

`@Transactional` protects only the resources managed by the same transaction manager.

It works well for:

- Multiple writes in the same database.
- Multiple repositories using the same datasource.
- A single local unit of work.

It does not automatically protect:

- A database write plus a Kafka publish.
- A write in Service A plus a write in Service B.
- A database commit plus an HTTP call.
- A Kafka offset commit plus a database write unless you design for it.

```java
@Transactional
public void createOrder(CreateOrderRequest request) {
    orderRepository.save(order);
    kafkaTemplate.send("orders", event); // Not part of the DB transaction
}
```

If the database commit succeeds but Kafka publishing fails, the order exists but no event is published.

If Kafka publishing succeeds but the database transaction rolls back, consumers may receive an event for data that does not exist.

---

## Core Patterns

| Pattern | Best For | Main Idea |
| --- | --- | --- |
| **Local transaction** | One service and one database | Keep all changes inside one database transaction. |
| **Saga** | Business workflows across microservices | Split the workflow into local transactions with compensating actions. |
| **Transactional Outbox** | Database write plus Kafka event | Save business data and event in the same database transaction. |
| **Transactional Inbox** | Idempotent Kafka consumers | Store processed message IDs with business changes. |
| **Kafka transactions** | Atomic Kafka consume-process-produce flows | Atomically write Kafka records and commit consumed offsets. |
| **CDC** | Reliable outbox publishing | Use tools like Debezium to stream DB changes to Kafka. |
| **2PC / XA** | Rare cases requiring strong atomicity | Coordinate multiple resources with a transaction coordinator. |

---

## Handling Transactions Between Microservices

Microservices should usually not share one distributed database transaction.

Each service owns its data and commits locally. The business workflow is coordinated with events, commands, retries, and compensating actions.

This is the **Saga pattern**.

### Example: Order Workflow

```text
Order Service
  1. Create order with status PENDING
  2. Publish OrderCreated event

Payment Service
  3. Charge payment
  4. Publish PaymentSucceeded or PaymentFailed

Inventory Service
  5. Reserve stock
  6. Publish StockReserved or StockRejected

Order Service
  7. Mark order CONFIRMED or CANCELLED
```

Each step is a local transaction. If one step fails, the system runs a compensation.

Examples of compensations:

- Cancel the order.
- Refund the payment.
- Release reserved inventory.
- Mark the workflow as failed and require manual review.

---

## Saga Styles

### Choreography

Each service reacts to events and publishes the next event.

```text
OrderCreated -> Payment Service -> PaymentSucceeded -> Inventory Service -> StockReserved
```

Benefits:

- Loose coupling.
- No central coordinator.
- Natural fit for Kafka.

Tradeoffs:

- Harder to understand the full workflow.
- Business logic is spread across services.
- Cycles and event storms are possible if events are not designed carefully.

### Orchestration

One orchestrator tells each service what to do.

```text
Order Saga Orchestrator
  -> charge payment
  -> reserve stock
  -> confirm order
```

Benefits:

- Workflow is easier to understand.
- Central place for timeouts, retries, and compensation.
- Better for complex business processes.

Tradeoffs:

- More coupling to the orchestrator.
- The orchestrator becomes an important operational component.

---

## Handling Transactions Between a Database and Kafka

The common mistake is trying to update the database and publish to Kafka directly in the same method.

```java
@Transactional
public void createOrder(CreateOrderRequest request) {
    orderRepository.save(order);
    kafkaTemplate.send("order-created", event);
}
```

This looks simple, but it has failure windows.

| Failure | Result |
| --- | --- |
| DB commit succeeds, Kafka publish fails | Data exists but no event is emitted. |
| Kafka publish succeeds, DB rollback happens | Event exists but data is missing. |
| App crashes after DB commit before Kafka send | Event is lost. |
| Kafka retries after timeout | Duplicate events may be published. |

The usual solution is the **Transactional Outbox pattern**.

---

## Transactional Outbox Pattern

The outbox pattern saves the business change and the event in the same database transaction.

```text
Single DB transaction:
  1. Insert or update business row
  2. Insert event into outbox table
  3. Commit

Async publisher:
  4. Read unpublished outbox rows
  5. Publish events to Kafka
  6. Mark rows as published
```

Because the business row and outbox row are committed together, you do not lose the event.

### Outbox Table Example

```sql
CREATE TABLE outbox_event (
    id UUID PRIMARY KEY,
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id VARCHAR(100) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    payload JSONB NOT NULL,
    status VARCHAR(30) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    published_at TIMESTAMP
);
```

### Service Code Example

```java
@Transactional
public void createOrder(CreateOrderRequest request) {
    Order order = orderRepository.save(new Order(request));

    OutboxEvent event = OutboxEvent.orderCreated(order);
    outboxRepository.save(event);
}
```

The Kafka publish happens later, outside the request transaction.

### Publisher Example

```java
public void publishPendingEvents() {
    List<OutboxEvent> events = outboxRepository.findPendingEvents();

    for (OutboxEvent event : events) {
        kafkaTemplate.send(event.topic(), event.key(), event.payload());
        event.markPublished();
        outboxRepository.save(event);
    }
}
```

In production, this publisher should handle retries, duplicate sends, backoff, metrics, and locking.

---

## Outbox With CDC

Instead of polling the outbox table from application code, you can use **Change Data Capture (CDC)**.

Common setup:

```text
Application -> Database outbox table -> Debezium -> Kafka topic
```

Benefits:

- Very reliable event publishing.
- Less custom polling code.
- Good fit for high-throughput systems.

Tradeoffs:

- More infrastructure.
- Requires careful schema and topic design.
- Operational knowledge of CDC tooling is required.

---

## Transactional Inbox Pattern

Kafka consumers must assume messages can be delivered more than once.

The **Transactional Inbox pattern** stores the consumed message ID in the same database transaction as the business update.

```text
Single DB transaction:
  1. Check if message ID was already processed
  2. Apply business change
  3. Store message ID in inbox table
  4. Commit
```

### Inbox Table Example

```sql
CREATE TABLE inbox_message (
    message_id VARCHAR(200) PRIMARY KEY,
    topic VARCHAR(200) NOT NULL,
    consumed_at TIMESTAMP NOT NULL
);
```

### Consumer Example

```java
@Transactional
public void handle(OrderCreated event) {
    if (inboxRepository.existsById(event.messageId())) {
        return;
    }

    inventoryService.reserveStock(event.orderId(), event.items());
    inboxRepository.save(InboxMessage.from(event));
}
```

This makes the consumer idempotent.

---

## Kafka Transactions

Kafka transactions can atomically:

- Produce multiple records to Kafka topics.
- Commit consumed offsets together with produced records.

They are useful for Kafka-to-Kafka processing.

```text
Consume from topic A
Process message
Produce to topic B
Commit offset atomically with produced records
```

Kafka transactions do not automatically make a database write and a Kafka publish atomic.

For database plus Kafka consistency, prefer:

- Transactional Outbox.
- Transactional Inbox.
- Idempotent consumers.
- Retries and dead-letter topics.

---

## 2PC and XA Transactions

**Two-Phase Commit (2PC)** coordinates multiple resources with a transaction manager.

```text
Phase 1: Prepare
  Ask every resource if it can commit.

Phase 2: Commit or rollback
  If all resources prepared successfully, commit all.
  Otherwise, roll back all.
```

Why it is usually avoided in microservices:

- It couples services and infrastructure tightly.
- It can block when the coordinator fails.
- It does not work well with Kafka, HTTP APIs, or external providers.
- It reduces availability and scalability.
- It is operationally complex.

Use it only when strong atomic consistency is mandatory and all resources properly support XA transactions.

---

## Reliability Building Blocks

Distributed transaction patterns depend on a few core engineering practices.

| Practice | Why It Matters |
| --- | --- |
| **Idempotency** | Retried commands and duplicate events must not corrupt data. |
| **Unique message IDs** | Consumers need a stable key to detect duplicates. |
| **Correlation IDs** | Logs and traces need to connect all steps of a workflow. |
| **Retries with backoff** | Temporary failures should recover automatically. |
| **Dead Letter Queue (DLQ)** | Poison messages should not block the whole consumer group. |
| **Compensation** | Failed business workflows need a deliberate undo or correction path. |
| **Monitoring** | Outbox lag, consumer lag, failed retries, and stuck sagas must be visible. |

---

## Decision Guide

| Situation | Recommended Approach |
| --- | --- |
| One service writes to one database | Local transaction with `@Transactional`. |
| One service writes to DB and publishes Kafka event | Transactional Outbox. |
| One service consumes Kafka and writes to DB | Transactional Inbox and idempotent consumer. |
| Kafka topic to Kafka topic processing | Kafka transactions if atomic offset-plus-produce matters. |
| Multi-service business workflow | Saga pattern. |
| Complex workflow with many steps | Orchestrated Saga. |
| Simple event reactions between services | Choreographed Saga. |
| Strong consistency across XA-capable resources | 2PC/XA, only if the tradeoff is acceptable. |
| External systems like payment providers | Saga with compensation and reconciliation. |

---

## Real-World Example: Order, Payment, Inventory

```text
1. Order Service creates order as PENDING.
2. Order Service writes OrderCreated event to outbox.
3. Outbox publisher sends OrderCreated to Kafka.
4. Payment Service consumes OrderCreated.
5. Payment Service checks inbox for duplicate message ID.
6. Payment Service charges customer.
7. Payment Service writes PaymentSucceeded or PaymentFailed to its outbox.
8. Inventory Service reserves stock after successful payment.
9. Order Service confirms or cancels the order based on final events.
```

If payment succeeds but inventory fails:

- Inventory publishes `StockRejected`.
- Order Service marks order as `CANCELLED`.
- Payment Service receives a compensation command or event.
- Payment Service refunds the customer.

The system becomes eventually consistent, not instantly consistent.

---

## Common Mistakes

- Assuming `@Transactional` covers Kafka publishes.
- Publishing events before the database transaction commits.
- Forgetting idempotency in consumers.
- Treating Kafka exactly-once semantics as database-plus-Kafka atomicity.
- Using distributed locks instead of explicit workflow design.
- Forgetting compensation paths.
- Not storing message IDs.
- Retrying forever without a DLQ.
- Building a saga with no observability.
- Using 2PC by default in microservices.

---

## Interview Questions

1. What is a distributed transaction?
2. Why is `@Transactional` not enough between a database and Kafka?
3. What problem does the Transactional Outbox pattern solve?
4. How do you make Kafka consumers idempotent?
5. What is the difference between Saga choreography and orchestration?
6. When would you use Kafka transactions?
7. Why is 2PC usually avoided in microservices?
8. How do you handle partial failure between payment and inventory services?
9. What is the role of a Dead Letter Queue?
10. What metrics would you monitor in an outbox-based system?

---

## Senior Developer Tip

In distributed systems, do not try to pretend that one global transaction exists everywhere. Design each service around local transactions, durable events, idempotent handlers, retries, and compensation. For database plus Kafka, the Transactional Outbox pattern is usually the safest default.
