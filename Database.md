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

Instead of locking a row when someone is updating it, Postgres keeps multiple versions of that same row in the database at the same time (Snapshot).

🥞 How it Works (The Snapshot)
Every transaction gets a unique transaction ID and a "snapshot" of the database at the moment it starts.

Inserts ➕: Postgres creates a new row and marks it with the current transaction ID as its creation time (stored in a hidden column called xmin).

Updates 🔄: Instead of overwriting the old data, Postgres marks the old row version as expired (using a hidden column called xmax) and inserts a brand-new row version with the updated data.

Deletes ❌: Postgres doesn't immediately erase the row; it just marks it as deleted by setting its xmax value.

That **accumulation** of old, expired row versions is known as **bloat** 🎈 (or dead tuples), and the famous PostgreSQL process that cleans them up is called **VACUUM** 🧹.

Benefits:

- Readers don't block writers
- Writers don't block readers
- High concurrency
- Snapshot isolation

### 📚 MVCC (Multi-Version Concurrency Control) Quick Review

* **Core Purpose 🎯:** Allows multiple transactions to read and write simultaneously without blocking each other. **Readers never block writers, and writers never block readers.**
* **The Mechanism 🥞:** Instead of overwriting data on an `UPDATE` or deleting it instantly on a `DELETE`, Postgres creates and maintains **multiple versions** of the same row simultaneously.
* **Hidden Columns 🔍:** Every row tracking relies on hidden system metadata columns:
* `xmin`: The Transaction ID that *created* the row version.
* `xmax`: The Transaction ID that *expired/deleted* the row version (set to 0 if still active).


* **The Snapshot 📸:** When a transaction runs, it gets a "snapshot" of active transaction IDs. It can only see data committed *before* its snapshot was taken.

---

### 🧪 Isolation Levels & Snapshot Timing

Your isolation level dictates exactly *when* Postgres captures that MVCC snapshot:

| Isolation Level | Snapshot Timing ⏱️ | Anomalies Allowed 👾 |
| --- | --- | --- |
| **Read Committed** (Default) | A new snapshot is taken at the start of **each query**. | Non-Repeatable Reads, Phantom Reads. |
| **Repeatable Read** | One snapshot is taken at the start of the **entire transaction**. | Write Skew (but protects against Phantoms). |
| **Serializable** | Uses the single transaction snapshot + **SSI** tracking. | **None.** Full protection. |

---

### 🛡️ Complex Concurrency & Cleanup

* **Write Skew Anomaly ⚖️:** Occurs at *Repeatable Read* when two concurrent transactions read overlapping data, make separate changes that are individually valid, but together violate a business rule (e.g., the two doctors on call example).
* **SSI (Serializable Snapshot Isolation) 🕵️‍♂️:** Activated only in `SERIALIZABLE` mode. It tracks read-write footprints (SIREAD locks). If a conflict loop is detected at commit time, Postgres aborts the younger transaction with a serialization error.
* **The Bloat Problem 🎈:** Because MVCC leaves old row versions behind, tables accumulate "dead tuples" over time.
* **VACUUM 🧹:** The background process (**Autovacuum**) that scans tables, marks dead tuple space as reusable for future inserts, and updates visibility maps. *Note: Standard VACUUM does not return disk space to the OS.*

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

🔒 Explicit Locking Review
Implicit vs. Explicit: While Postgres handles most locking automatically, developers use explicit locking (SELECT ... FOR UPDATE) to protect critical business logic.

The Stand-off (Deadlocks) 🔄: Occur when two or more transactions are blocked, each waiting for a lock held by the other.

The Resolution ⚙️: Postgres uses a deadlock_timeout (default: 1 second). Once triggered, the deadlock detector scans for a circular loop, selects a "victim" transaction, aborts it, and rolls back its changes to free the locks.

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
- Hash : A specialized index that converts a value into a hash code. It only handles exact equality matches (=) and cannot do range scans, but it is highly optimized for that single job.
- GIN : Generalized Inverted Index (It is perfect for searching inside JSONB documents, arrays, or full-text search documents)
- GiST : (Generalized Search Tree) This is a highly flexible tree structure used when data can overlap or has multiple dimensions. It is the backbone for indexing geometric shapes, geographic coordinates (PostGIS), and range types (like a date range for a hotel booking).
- BRIN : (Block Range Index) It is incredibly tiny and works wonders for massive, naturally ordered data like a created_at timestamp on a logging table.

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

- Navigate B-TREE to get the TID ( Tuple ID ) -> disk space location.

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

| Component / Scenario | How It Works & How the Planner Reacts |
| --- | --- |
| **🧠 The Query Planner** | Acts like a GPS navigation system. It evaluates every possible path to fetch your data, calculates an estimated "cost" for each, and chooses the fastest route. |
| **📊 The `ANALYZE` Command** | Updates the database's internal statistics (such as data distribution and row counts). The planner relies entirely on these statistics to make accurate predictions. |
| **📉 High-Frequency Data (99% match)** | If a value appears in almost every row, the planner skips the B-Tree index and performs a **Sequential Scan**. Reading the table straight through is faster than constantly jumping back and forth between the index and the disk. |
| **🔀 Randomized Data + BRIN** | If data is scattered randomly, the minimum and maximum values for every physical block will overlap heavily. Because the planner can no longer skip any blocks, a BRIN index becomes useless, and the database falls back to a **Sequential Scan**. |

---

Always know:

```sql
EXPLAIN ANALYZE
```
`EXPLAIN ANALYZE` is the ultimate profiling tool in PostgreSQL used to inspect and debug query performance. 🚀

While a regular `EXPLAIN` command just shows the blueprint (the execution plan) that the query planner *thinks* it will use, adding `ANALYZE` forces Postgres to actually run the query and measure real-world performance.

Here is how they compare side-by-side:

| Command | Does it run the query? | What does it show? |
| --- | --- | --- |
| **`EXPLAIN`** 📋 | ❌ No | The planner's **estimated** cost and expected row counts. |
| **`EXPLAIN ANALYZE`** ⏱️ | Yes | The **actual** execution time, real row counts, and memory usage. |

When you read the output of an `EXPLAIN ANALYZE`, you get to compare the planner's guesses against reality. If the planner's estimates are wildly different from the actual execution results, it usually means the database's internal statistics are out of date, leading to poor index choices.

> ⚠️ **Safety Warning:** Because `EXPLAIN ANALYZE` actually runs the query, running `EXPLAIN ANALYZE DELETE FROM users;` will physically delete your data! To safely test a write query, you should always wrap it in a transaction and roll it back.

---
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
