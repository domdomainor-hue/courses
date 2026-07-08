# Build-It (Machine Coding) Interview Cheat Sheet

> Goal: Build production-quality code, not just solve the problem.

---

# 1. Interview Mindset

Think like a backend engineer.

Priority:

1. Correctness
2. Clean design
3. Extensibility
4. Thread safety (if needed)
5. Performance
6. Tests

Never rush into coding.

---

# 2. First 5 Minutes

Always ask:

- Expected scale?
- Thread-safe?
- Duplicate requests?
- Input validation?
- Error handling?
- Time complexity expectations?
- Can requirements evolve?

Example:

> "I'll first implement a simple version, then make it thread-safe and optimize if needed."

Interviewers love this.

---

# 3. Suggested Project Structure

```text
src
 ├── model/
 ├── service/
 ├── repository/
 ├── exception/
 ├── util/
 └── tests/
```

Keep responsibilities separated.

---

# 4. Design Before Coding

Write down:

Entities

```text
Server
LoadBalancer
Request
```

Relationships

```text
LoadBalancer
    ↓
 List<Server>
```

Operations

```text
addServer()
removeServer()
findServer()
dispatch()
```

Then code.

---

# 5. SOLID (Keep It Simple)

Know these:

### Single Responsibility

One class = one job.

### Open/Closed

Easy to add features.

### Dependency Inversion

Program against interfaces.

---

# 6. Use Interfaces

Instead of

```java
ArrayList<Server>
```

Prefer

```java
List<Server>
```

Instead of

```java
HashMap
```

Prefer

```java
Map
```

---

# 7. Choose the Right Data Structure

Need uniqueness?

```text
HashSet
```

Lookup by id?

```text
HashMap
```

Ordering?

```text
TreeMap
LinkedHashMap
PriorityQueue
```

FIFO?

```text
Queue
Deque
```

Always explain WHY.

---

# 8. Complexity

Know complexity while coding.

Example

```text
HashMap lookup

O(1)
```

Removing from ArrayList middle

```text
O(n)
```

Mention improvements.

---

# 9. Validation

Never trust inputs.

Examples

```text
null

duplicate ids

negative capacity

invalid status
```

Throw meaningful exceptions.

---

# 10. Exceptions

Prefer

```java
ServerAlreadyExistsException
```

Instead of

```java
RuntimeException
```

---

# 11. Immutability

Make value objects immutable.

Example

```text
Request

Money

UserId
```

Benefits

- thread-safe
- predictable
- easier testing

---

# 12. Encapsulation

Bad

```java
server.status = DOWN;
```

Good

```java
server.markDown();
```

---

# 13. Thread Safety

Always ask if needed.

Common tools

```text
ConcurrentHashMap

AtomicInteger

synchronized

ReadWriteLock
```

Avoid

```text
HashMap + multiple threads
```

---

# 14. Common Patterns

Singleton

Factory

Strategy

Builder

Repository

Don't force patterns.

Use only when useful.

---

# 15. Logging

Don't spam.

Useful logs

```text
Server added

Request assigned

Retry

Failure
```

---

# 16. Testing

Tests are extremely important.

Typical order

### Happy path

### Edge cases

### Invalid input

### Duplicate input

### Concurrent execution

### Large dataset

---

Example

```text
addServer()

removeServer()

duplicate server

capacity exceeded

empty load balancer

100 concurrent requests
```

---

# 17. Write Readable Code

Good names

```java
assignRequest()

findLeastLoaded()

calculateHash()
```

Bad names

```java
doStuff()

x()

foo()
```

---

# 18. Refactor

Start simple.

Then improve.

Example

Version 1

```text
if else
```

Version 2

```text
Strategy Pattern
```

Interviewers like incremental improvement.

---

# 19. Requirement Changes

Very common at Revolut.

Examples

Initially

```text
Round Robin
```

Later

```text
Weighted Round Robin
```

Initially

```text
Single thread
```

Later

```text
Concurrent
```

Initially

```text
One datacenter
```

Later

```text
Multiple regions
```

Design for extension.

---

# 20. Common Machine Coding Questions

## Load Balancer ⭐⭐⭐⭐⭐

Need

- add server
- remove server
- dispatch
- unique ids
- capacity
- health
- thread safety

---

## LRU Cache ⭐⭐⭐⭐

Need

- O(1)
- HashMap
- Doubly Linked List

---

## Rate Limiter ⭐⭐⭐⭐⭐

Need

- Token Bucket
- Sliding Window
- Fixed Window

Know trade-offs.

---

## Parking Lot

Entities

```text
Vehicle

Spot

Ticket

ParkingLot
```

---

## Library

Need

- search
- borrow
- return

---

## Elevator

Need

- scheduling
- priority

---

## Payment Ledger ⭐⭐⭐⭐⭐

Need

- transactions
- balances
- idempotency
- locking

---

## Hit Counter

Need

Rolling window.

---

## Task Scheduler

Need

Priority Queue.

---

# 21. Things Interviewers Watch

Do you

✅ Ask questions?

✅ Think before coding?

✅ Separate responsibilities?

✅ Use correct collections?

✅ Handle edge cases?

✅ Explain complexity?

✅ Write tests?

✅ Refactor?

---

# 22. Build Order

Always follow this order

1. Clarify
2. Design
3. Entities
4. APIs
5. Happy path
6. Validation
7. Tests
8. Optimization
9. Thread safety
10. Requirement changes

---

# 23. Revolut Tips

Expect follow-up questions like

"What if two requests arrive simultaneously?"

"What if the server crashes?"

"How would you test this?"

"What if capacity changes?"

"How would this scale?"

"Can we make reads lock-free?"

Always have an answer.

---

# 24. Final Checklist

Before saying "I'm done"

☐ Naming is clear

☐ No duplicated code

☐ Correct data structures

☐ Complexity explained

☐ Exceptions handled

☐ Validation added

☐ Unit tests written

☐ Thread safety discussed

☐ Easy to extend

☐ Trade-offs explained

---

# 60-Second Interview Recipe

```
Clarify requirements
        ↓
Identify entities
        ↓
Choose data structures
        ↓
Define APIs
        ↓
Implement happy path
        ↓
Handle edge cases
        ↓
Write tests
        ↓
Discuss complexity
        ↓
Make thread-safe
        ↓
Handle new requirements
```

---

# Revolut Build-It Golden Rules

- Don't over-engineer initially.
- Build a working MVP first.
- Keep classes small and cohesive.
- Narrate every design decision.
- Expect changing requirements.
- Write tests as you go.
- Think about concurrency early.
- Optimize only after correctness.
- Always explain trade-offs.
- Code like it will go to production tomorrow.
