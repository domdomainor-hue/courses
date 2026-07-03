# Database Interview Cheat Sheet (Revolut)

> Goal: Quickly revise PostgreSQL and database concepts for a Mid/Senior Backend interview.

---

# 1. ACID

- **Atomicity** → All or nothing.
- **Consistency** → Database remains valid.
- **Isolation** → Concurrent transactions don't interfere.
- **Durability** → Committed data survives crashes.

---

# 2. Transaction Isolation Levels

| Level | Dirty Read | Non-repeatable | Phantom | Use case |
|--------|------------|----------------|----------|----------|
| Read Uncommitted | ✅ | ✅ | ✅ | Almost never |
| Read Committed (Postgres default) | ❌ | ✅ | ✅ | Most applications |
| Repeatable Read | ❌ | ❌ | ❌ (Postgres MVCC) | Financial reads |
| Serializable | ❌ | ❌ | ❌ | Highest consistency |

### Know the anomalies

- Dirty Read → Read uncommitted data
- Non-repeatable Read → Same row changes
- Phantom Read → New rows appear

- Read Uncommitted 🔓: Transactions can see changes made by other transactions even before they are committed.

- Read Committed 🔑: A transaction can only see data that has been fully committed. This is the default for many databases like PostgreSQL.

- Repeatable Read 🔄: Guarantees that if you read a row once, reading it again within the same transaction will yield the exact same data.

- Serializable 🚧: The ultimate isolation. It forces transactions to execute in a way that behaves as if they were running completely sequentially.
---

# 3. MVCC (PostgreSQL)

PostgreSQL uses **MVCC (Multi-Version Concurrency Control)**

Benefits:

- Readers don't block writers
- Writers don't block readers
- High concurrency
- Snapshot isolation

---

# 4. Locks

### Shared Lock
- Multiple readers

### Exclusive Lock
- Single writer

### Row-level lock

```sql
SELECT * FROM account
FOR UPDATE;
```

Prevents concurrent updates.

Useful in payments.

---

# 5. Optimistic vs Pessimistic Locking

## Optimistic

Uses version column.

```text
UPDATE ... WHERE id=? AND version=?
```

Pros

- High throughput
- Few conflicts

Cons

- Retry required

---

## Pessimistic

```sql
SELECT ... FOR UPDATE
```

Pros

- Safe

Cons

- Lower concurrency
- Deadlocks possible

---

# 6. Indexes

Purpose:

- Faster reads
- Slower writes

Common types

- B-Tree ⭐ (default)
- Hash
- GIN
- GiST
- BRIN

---

## Composite Index

```sql
(country, created_at)
```

Works for

```
country
country + created_at
```

Does NOT work efficiently for

```
created_at
```

(Leftmost prefix rule)

---

## Covering Index

Index contains all requested columns.

No table lookup needed.

---

## Partial Index

```sql
WHERE status='ACTIVE'
```

Smaller index.

Faster.

---

# 7. EXPLAIN

Always know:

```sql
EXPLAIN ANALYZE
```

Look for

- Seq Scan ❌
- Index Scan ✅
- Bitmap Scan
- Nested Loop
- Hash Join
- Merge Join

Red flags

- Large Sequential Scan
- High actual rows
- Large cost

---

# 8. Joins

- INNER
- LEFT
- RIGHT
- FULL
- CROSS

Most common interview joins

```sql
INNER JOIN
LEFT JOIN
```

---

# 9. Query Optimization

Avoid

```sql
SELECT *
```

Prefer

- Needed columns only
- Proper indexes
- LIMIT
- WHERE filtering
- Avoid functions on indexed columns

Bad

```sql
WHERE LOWER(email)=...
```

Good

Store normalized values or create functional index.

---

# 10. Pagination

## OFFSET

```sql
LIMIT 20 OFFSET 10000
```

Easy

❌ Slow on large tables

---

## Keyset (Cursor)

```sql
WHERE id > lastId
LIMIT 20
```

Preferred.

---

# 11. Normalization

1NF
- Atomic values

2NF
- Remove partial dependency

3NF
- Remove transitive dependency

Interview answer:

> Normalize for consistency.
> Denormalize for performance.

---

# 12. Transactions

Always keep transactions

- Small
- Fast
- Short

Never perform

- HTTP calls
- Kafka publishing
- External APIs

inside DB transaction.

---

# 13. Deadlocks

Example

Transaction A

```
locks row 1
waits row 2
```

Transaction B

```
locks row 2
waits row 1
```

Avoid

- Consistent lock order
- Small transactions
- Retry on deadlock

---

# 14. Idempotency (Payments)

Critical.

Use

- Idempotency Key
- Unique Constraint

Same request executed twice

➡️ Same result.

---

# 15. Constraints

- PRIMARY KEY
- UNIQUE ⭐
- FOREIGN KEY
- CHECK
- NOT NULL

Prefer DB constraints over application-only validation.

---

# 16. Foreign Keys

Pros

- Integrity

Cons

- More locking
- Harder sharding

Large distributed systems sometimes enforce integrity in application.

---

# 17. Sharding

Split data across databases.

Pros

- Scale writes
- Scale storage

Cons

- Cross-shard joins
- Transactions harder
- Rebalancing

---

# 18. Replication

Primary

↓

Replica(s)

Reads

➡️ Replica

Writes

➡️ Primary

Problems

- Replication lag
- Stale reads

---

# 19. Caching

Redis before PostgreSQL.

Problems

- Cache invalidation
- Stale data
- Cache stampede

Patterns

- Cache Aside ⭐
- Write Through
- Write Back

---

# 20. SQL Execution Order

```text
FROM
JOIN
WHERE
GROUP BY
HAVING
SELECT
ORDER BY
LIMIT
```

---

# 21. Common SQL Questions

Find duplicates

```sql
GROUP BY
HAVING COUNT(*) > 1
```

Second highest salary

- LIMIT/OFFSET
- DENSE_RANK()

Customers without orders

```sql
LEFT JOIN
WHERE order.id IS NULL
```

Top N

```sql
ROW_NUMBER()
RANK()
DENSE_RANK()
```

---

# 22. PostgreSQL Features

- MVCC
- JSONB
- Window Functions
- CTE
- Partial Indexes
- EXPLAIN ANALYZE
- VACUUM
- Materialized Views

---

# 23. Payments Best Practices

✅ Idempotency

✅ Row locking (`FOR UPDATE`)

✅ Read Committed or Repeatable Read

✅ Unique constraints

✅ Retry transient failures

✅ Outbox Pattern

✅ Optimistic locking when possible

---

# 24. Interview Red Flags

❌ Long transactions

❌ Missing indexes

❌ OFFSET on huge tables

❌ SELECT *

❌ N+1 queries

❌ Ignoring deadlocks

❌ No idempotency

❌ No unique constraints

---

# 25. Senior-Level Discussion Topics

Be ready to discuss:

- MVCC internals
- Isolation level trade-offs
- Index design
- EXPLAIN plans
- Query optimization
- Partitioning
- Sharding
- Replication lag
- Eventual consistency
- Outbox Pattern
- CQRS
- Read/Write splitting
- High-concurrency payment systems

---

# 30-Second Recap

- ACID
- MVCC
- Read Committed (default)
- Repeatable Read for money
- Serializable = safest, slowest
- B-Tree indexes
- Composite index = leftmost prefix
- EXPLAIN ANALYZE
- Avoid Sequential Scan
- Prefer Keyset Pagination
- Use `FOR UPDATE` for payments
- Keep transactions short
- Idempotency is mandatory
- Unique constraints prevent duplicates
- Know deadlocks, replication, sharding, caching
```
