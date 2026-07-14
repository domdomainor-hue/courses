# Java Concurrency Interview Guide 🔥

Here are the **most frequently tested** Java concurrency concepts in technical interviews, with detailed explanations:

---

## 🎯 1. **Thread Lifecycle & States**

### What You Need to Know

| State | Description | Method |
|-------|-------------|--------|
| NEW | Thread created but not started | `thread.start()` |
| RUNNABLE | Runnable (in JVM or waiting) | CPU bound or I/O wait |
| BLOCKED | Waiting for monitor lock | `synchronized` block |
| WAITING | Infinite wait | `wait()`, `park()` |
| TIMED_WAITING | Wait with timeout | `sleep()`, `join()`, `await()` |
| TERMINATED | Thread completed execution | N/A |

### ⭐ Common Interview Questions

**Q: How does a thread transition from RUNNABLE to BLOCKED?**  
**A:** When a thread enters a `synchronized` block but cannot acquire the monitor lock because another thread holds it.

```java
// Example
private final Object lock = new Object();

public void method() {
    synchronized(lock) {  // Blocked if another thread has lock
        // Critical section
    }
}
```

---

## 🎯 2. **synchronized Keyword**

### What You Need to Know

| Aspect | Details |
|--------|---------|
| **Type** | Implicit ReentrantLock (fairness = false) |
| **Visibility** | Guarantees visibility between threads |
| **Happens-Before** | Establishes memory ordering guarantees |
| **Reentrancy** | Same thread can acquire multiple times |

### ⭐ Common Interview Questions

**Q: What's the difference between synchronized method vs block?**  
**A:** 
```java
// 1. Synchronized Method - locks on 'this' instance
public synchronized void method() { }

class SyncMethod {
    public static synchronized void staticMethod() { }  // Locks on class object
    
    // 2. Synchronized Block - more flexible lock object
    private final Object lock = new Object();
    
    public void blockMethod() {
        synchronized(lock) { }  // Can lock any object
    }
}
```

**Q: What is the memory visibility problem?**  
**A:** Without synchronization, changes may not be visible to other threads due to CPU caching.

```java
class MemoryVisibility {
    static boolean flag = false;  // Shared state
    static int value = 0;         // Shared state
    
    public void increment() {
        flag = true;              // Write may be cached
    }
    
    public void read() {
        while (!flag) { }         // May see stale value=0 forever!
        System.out.println(value);
    }
}
```

**Solution:**
```java
volatile boolean flag = false;  // Ensures visibility
Object lock = new Object();      // Or use synchronized
```

---

## 🎯 3. **Atomic Classes**

### What You Need to Know

| Class | Primitive Type | Operations |
|-------|---------------|------------|
| `AtomicInteger` | int | CAS (Compare-And-Swap) |
| `AtomicLong` | long | CAS operations |
| `AtomicBoolean` | boolean | CAS operations |
| `AtomicReference<T>` | Object reference | Pointer swap |

### ⭐ Common Interview Questions

**Q: Why is AtomicInteger thread-safe?**  
**A:** It uses CPU-level atomic operations (CAS) that are guaranteed by hardware instructions.

```java
class AtomicOperations {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        // This is ATOMIC - no explicit synchronization needed!
        count.incrementAndGet();  // Uses CAS internally
        
        // Or manually:
        int expected;
        int updated;
        do {
            expected = count.get();
            updated = expected + 1;
        } while (!count.compareAndSet(expected, updated));
    }
}
```

**Q: What's the difference between volatile and Atomic?**  
**A:**
| Aspect | volatile | Atomic |
|--------|----------|--------|
| **Guarantees** | Visibility only | Visibility + Atomic updates |
| **Composite Ops** | `count++` is NOT atomic | `AtomicInteger.incrementAndGet()` IS atomic |
| **Lock-Free** | No | Yes (using CAS) |

---

## 🎯 4. **Thread Pool & ExecutorService**

### What You Need to Know

```java
// Factory methods
Executors.newSingleThreadExecutor();         // Single thread
Executors.newFixedThreadPool(n);             // Fixed pool size
Executors.newCachedThreadPool();             // No max, create as needed
Executors.newScheduledThreadPool(n);        // Scheduled tasks
Executors.newWorkStealingPool(n);           // Fork/Join pool (Java 8+)
```

### ⭐ Common Interview Questions

**Q: How do you create a thread pool manually?**  
**A:**
```java
// Manual implementation with RejectedExecutionHandler
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    10,                              // Core pool size
    20,                              // Max pool size
    60L, TimeUnit.SECONDS,          // Keep-alive time
    new LinkedBlockingQueue<>(100), // Queue capacity
    new ThreadPoolExecutor.CallerRunsPolicy()  // Rejection policy
);

// Submit task
Future<String> future = executor.submit(() -> heavyTask());

// Shutdown gracefully
executor.shutdown();
if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
    executor.shutdownNow();
}
```

**Q: What are the rejection policies?**  
**A:**
| Policy | Behavior |
|--------|----------|
| `AbortPolicy` (Default) | Throws `RejectedExecutionException` |
| `CallerRunsPolicy` | Executes in calling thread |
| `DiscardOldestPolicy` | Discards oldest task, queues new one |

**Q: What's the difference between Future and CompletableFuture?**  
**A:**
```java
// Traditional Future - no async composition
Future<String> future1 = executor.submit(() -> fetchData());
future1.get();  // Blocks!

// CompletableFuture - rich async API
CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> fetchData());
CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> fetchConfig());

cf1.thenApply(data -> processData(data))
    .thenCombine(cf2, (data, config) -> process(data, config));
```

---

## 🎯 5. **Synchronization Tools**

### CountDownLatch - Wait for Multiple Threads

```java
CountDownLatch latch = new CountDownLatch(3);  // Wait for 3 threads

// Thread 1
new Thread(() -> {
    try {
        // Do work
        latch.countDown();  // Signal completion
    } catch (InterruptedException e) {}
}).start();

// Main thread - waits for all 3
try {
    latch.await();  // Blocks until all countDown() called
} catch (InterruptedException e) {}
```

**Interview Q:** When would you use CountDownLatch vs. Semaphore?  
**A:** Use CountDownLatch when threads complete tasks and signal completion; use Semaphore when controlling access to limited resources.

---

### CyclicBarrier - Synchronization Point

```java
CyclicBarrier barrier = new CyclicBarrier(3);  // Wait for 3 threads

// Thread 1
new Thread(() -> {
    try {
        // Do work
        barrier.await();  // Wait at barrier point
        // Next round if needed
    } catch (Exception e) {}
}).start();
```

---

### Semaphore - Resource Control

```java
Semaphore semaphore = new Semaphore(5);  // Allow max 5 concurrent tasks

// Task with limit
semaphore.acquire();
try {
    // Critical section
} finally {
    semaphore.release();
}
```

**Interview Q:** How to implement a rate limiter?  
**A:**
```java
class RateLimiter {
    private final Semaphore semaphore = new Semaphore(10);
    
    public void doWork() throws InterruptedException {
        semaphore.acquire();  // Acquire token
        try {
            work();
        } finally {
            semaphore.release();
        }
    }
}
```

---

## 🎯 6. **BlockingQueues**

### What You Need to Know

| Queue Type | Thread-Safe | Blocking | Use Case |
|------------|-------------|----------|----------|
| `ArrayBlockingQueue` | Yes | Yes | Fixed capacity |
| `LinkedBlockingQueue` | Yes | Yes | Bounded/unbounded |
| `PriorityBlockingQueue` | Yes | Yes | Prioritized tasks |

### ⭐ Common Interview Questions

**Q: Producer-Consumer Pattern Implementation?**  
**A:**
```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

// Producer Thread
new Thread(() -> {
    for (String item : items) {
        try {
            queue.put(item);      // Blocks if full
        } catch (InterruptedException e) {}
    }
}).start();

// Consumer Thread
new Thread(() -> {
    while (true) {
        try {
            String item = queue.take();  // Blocks until available
            process(item);
        } catch (InterruptedException e) {}
    }
}).start();
```

**Q: How to handle shutdown cleanly?**  
**A:**
```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();

// Close flag
AtomicBoolean closed = new AtomicBoolean(false);

new Thread(() -> {
    while (!closed.get()) {
        try {
            String item = queue.take();
            process(item);
        } catch (InterruptedException e) {}
    }
}, "Consumer").start();

// Shutdown
closed.set(true);  // Signal all consumers to stop
```

---

## 🎯 7. **ThreadLocal**

### What You Need to Know

| Concept | Details |
|---------|---------|
| **Storage** | Per-thread variable storage |
| **Use Case** | User context, DB connections, locale settings |
| **Memory Leak** | Must call `remove()` when done |

### ⭐ Common Interview Questions

**Q: When should you use ThreadLocal?**  
**A:** When you need to store data specific to each thread without synchronization.

```java
// Example: User Context in Web Apps
ThreadLocal<User> userContext = new ThreadLocal<>();

// In request processing
userContext.set(userFromSession());
try {
    service.processRequest();  // Has access to user via ThreadLocal
} finally {
    userContext.remove();  // CRITICAL: Prevent memory leak!
}
```

**Q: What's the memory leak risk?**  
**A:** If `remove()` isn't called, ThreadLocal data persists in thread memory even after thread ends. This can cause OOM in application pools.

---

## 🎯 8. **Volatile Keyword**

### What You Need to Know

| Feature | volatile guarantees |
|---------|---------------------|
| Visibility | Changes visible to all threads immediately |
| Order | Prevents certain reordering |
| Atomicity | Does NOT guarantee atomic updates (e.g., count++ is NOT atomic) |

### ⭐ Common Interview Questions

**Q: Difference between volatile and synchronized?**  
**A:**
```java
class VolatileVsSync {
    // 1. Volatile - visibility only, not atomic
    private volatile boolean running = true;
    
    public void stop() {
        running = false;  // Other thread sees this immediately
    }
    
    public void run() {
        while (running) {
            // May see running=false without synchronized block
        }
    }
    
    // 2. AtomicInteger - atomic operations
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Atomic!
    }
}
```

**Q: When would you use volatile?**  
**A:** For flags and state that don't require modification in multiple threads:
```java
private volatile boolean shutdown = false;

public void stop() {
    shutdown = true;  // Other thread immediately sees this
}
```

---

## 🎯 9. **Concurrent Collections**

### What You Need to Know

| Collection | Thread-Safe? | Concurrent Modifications Allowed? |
|------------|--------------|----------------------------------|
| `HashMap` | ❌ No | ❌ No (throws Exception) |
| `ConcurrentHashMap` | ✅ Yes | ✅ Yes (non-atomic values) |
| `CopyOnWriteArrayList` | ✅ Yes | ✅ Yes (on copy) |
| `Vector` | ✅ Yes | ❌ No |

### ⭐ Common Interview Questions

**Q: How is ConcurrentHashMap thread-safe?**  
**A:** Uses CAS operations and segmented locking (Java 8+ uses tree-based for large maps).

```java
// Thread-safe map with atomic operations
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

map.merge("key", 10, Integer::sum);   // Atomic add
int value = map.computeIfAbsent("key", k -> 0) + 1;  // Atomic computation
```

**Q: What's the difference between ConcurrentHashMap and synchronized HashMap?**  
**A:**
```java
// ConcurrentHashMap - finer-grained locking, higher throughput
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.merge("key", value, (oldVal, newVal) -> oldVal + newVal);

// Synchronized HashMap - coarse-grained locking, simpler but slower
Map<String, List<Integer>> syncMap = Collections.synchronizedMap(new HashMap<>());
List<Integer> list;
synchronized(syncMap) {  // Lock entire map!
    list = syncMap.get("key");
}
```

---

## 🎯 10. **CompletableFuture (Java 8+)**

### What You Need to Know

```java
// Async execution
CompletableFuture.supplyAsync(() -> fetchData(), executor)
    .thenApply(data -> processData(data))
    .exceptionally(ex -> handleException(ex))
    .whenComplete((result, ex) -> logCompletion(result));

// Combining futures
CompletableFuture<Void> all = CompletableFuture.allOf(
    fetchUser(), fetchPosts(), fetchComments()
);

all.thenAccept(v -> displayAll());  // All completed before this runs
```

**Interview Q:** How to handle async failures?  
**A:**
```java
CompletableFuture.supplyAsync(() -> fetchData())
    .thenCompose(data -> processData(data))
    .exceptionally(ex -> {
        log.error("Failed", ex);
        return defaultResponse();  // Handle gracefully
    });
```

---

## 🎯 11. **Deadlock Prevention**

### What You Need to Know

```java
// ✅ BAD - Potential deadlock (order of locks matters!)
ReentrantLock lockA = new ReentrantLock();
ReentrantLock lockB = new ReentrantLock();

void method1() {
    lockA.lock();
    try {
        lockB.lock();  // Can cause deadlock!
        // ...
    } finally {
        lockB.unlock();
        lockA.unlock();
    }
}

// ✅ GOOD - Consistent locking order
void safeMethod() {
    Lock lockA = new ReentrantLock();
    Lock lockB = new ReentrantLock();
    
    lock(lockA);
    try {
        lock(lockB);  // Same order as other methods
        // ...
    } finally {
        unlock(lockB);
        unlock(lockA);
    }
}

// ✅ BETTER - TryLock with timeout
void tryLockWithTimeout() {
    if (lock1.tryLock(5, TimeUnit.SECONDS)) {
        try {
            if (lock2.tryLock(5, TimeUnit.SECONDS)) {
                try {
                    // Safe to proceed
                } finally {
                    lock2.unlock();
                }
            }
        } finally {
            lock1.unlock();
        }
    } else {
        handleTimeout();  // Don't deadlock!
    }
}
```

---

## 🎯 12. **Common Interview Problems**

### Problem 1: Read-Write Lock

```java
class ReadWriteLock {
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    private ReadWriteLock.ReadHolder readerCount = null;
    
    class ReadHolder implements Runnable {
        public void run() {
            if (lock.tryLock()) {
                try {
                    while (!interrupted) {
                        // Read data safely
                    }
                } finally {
                    lock.unlock();
                }
            } else {
                retryLater();  // Backoff on contention
            }
        }
    }
}
```

---

### Problem 2: Single-Threaded Cache Loader

```java
class CacheLoader<T, R> implements Function<Cache<K, V>, Future<R>> {
    public Future<R> load(Future<V> future) {
        try {
            return future.get();
        } catch (Exception e) {
            // Handle async failure gracefully
        }
    }
}
```

---

### Problem 3: Thread Pool for Load Balancing

```java
class LoadBalancedPool {
    private final ExecutorService executor = Executors.newFixedThreadPool(8);
    
    public CompletableFuture<String> processRequest(String request) {
        CompletableFuture<String> future = 
            CompletableFuture.supplyAsync(() -> 
                new LoadBalancer().chooseExecutor()
            );
        
        return future;
    }
}
```
------

# Core synchronization utilities

This revision guide covers core synchronization utilities (`java.util.concurrent`), asynchronous orchestration (`CompletableFuture`), and modern JVM concurrency paradigms (Virtual Threads & Structured Concurrency).

---

## 1. High-Level Synchronization Utilities

| Utility | Reusable? | Dynamic Parties? | Core Mechanism | Primary Use Case |
| :--- | :---: | :---: | :--- | :--- |
| **`CountDownLatch`** | ❌ No | ❌ No | A one-way gate. Threads block via `await()` until the counter is decremented to zero via `countDown()`. | Main thread waiting for $N$ parallel initialization tasks to complete before starting. |
| **`CyclicBarrier`** |  Yes | ❌ No | A synchronization point where a fixed number of threads must wait for each other. Resets automatically. Supports an optional execution action when tripped. | Parallel algorithms (e.g., matrix processing, game loops) executing in repetitive lock-step phases. |
| **`Semaphore`** |  Yes | ❌ No | Controls access to a shared resource pool using a finite set of virtual "permits" (`acquire()` / `release()`). | Rate-limiting, throttling, or managing database connection pools (Bulkhead Pattern). |
| **`Phaser`** |  Yes |  Yes | A highly flexible, multi-phase synchronization barrier where the number of registered parties can change on the fly. | Multi-stage build pipelines or dynamic workflows where threads register/deregister mid-process. |

---

## 2. CompletableFuture (Asynchronous Orchestration)

`CompletableFuture` implements `Future` and `CompletionStage`, enabling non-blocking, event-driven pipelines instead of blocking thread-parking (`.get()`).

### 🚦 Thread Pool Routing (Crucial)
* **The Danger:** Calling async methods (e.g., `supplyAsync`, `thenApplyAsync`) without an explicit executor routes tasks to `ForkJoinPool.commonPool()`. If this pool is saturated by blocking I/O, it starves the entire JVM.
* **The Solution:** Always supply a dedicated, isolated custom `Executor` (Bulkhead Pattern) for I/O-bound tasks.

### ⛓️ API Orchestration Patterns
* **Sequential Chain (Dependent):** `thenCompose(Function)` (equivalent to monadic `flatMap`). Returns a flattened future when Step B depends on the output of Step A.
* **Parallel Combine (Independent):** `thenCombine(CompletionStage, BiFunction)`. Combines two independent futures concurrently and merges their results when both complete.
* **Fan-Out / Fan-In:** * `CompletableFuture.allOf(...)`: Blocks until all participating futures complete.
  * `CompletableFuture.anyOf(...)`: Returns immediately with the result of the fastest completing future.

### 🛡️ Exception Handling & Resilience
* `exceptionally(Function)`: Acts as a standard `catch` block. Catches upstream exceptions and recovers with a default/fallback value.
* `handle(BiFunction)`: Always executes. Receives both the result and exception, allowing transformation of either.
* `completeOnTimeout(value, duration, unit)`: Proactively recovers with a default value if the task exceeds the specified timeout threshold.
* `orTimeout(duration, unit)`: Completes the future exceptionally with a `TimeoutException` if the execution boundary is breached.

---

## 3. Virtual Threads (Project Loom - Java 21+)

Virtual Threads are lightweight, user-mode threads managed entirely by the JVM, rather than the Operating System.

### ⚙️ Mechanics
* **Platform Threads:** 1:1 mapping to OS kernel threads. High memory footprint (~1MB stack), expensive context switching, limited to thousands.
* **Virtual Threads:** $M:N$ mapping. Millions of virtual threads are multiplexed over a small pool of underlying platform carrier threads. 
* **Non-Blocking Yielding:** When a virtual thread performs blocking operations (I/O, locks, sleep), the JVM automatically unmounts it from the carrier thread, parking it until the resource is ready. The carrier thread is immediately freed up to execute other virtual threads.

### ⚠️ Senior Production Pitfalls (Interview Traps)
1. **Carrier Thread Pinning:** Occurs when a virtual thread blocks inside a `synchronized` block or executes a native JNI call. The carrier thread is held hostage. 
   * *The Fix:* Replace `synchronized` blocks with `ReentrantLock`.
2. **Pooling Virtual Threads is an Antipattern:** Virtual threads are incredibly cheap and designed to be short-lived and disposable. **Never pool them.** To throttle database queries or rate-limit resources, run virtual threads unpooled and limit concurrent access inside the threads using a `Semaphore`.

---

## 4. Structured Concurrency

Structured Concurrency binds the lifecycles of concurrent subtasks within a strict lexical scope block.

### 📏 Rules of Engagement
* Uses `StructuredTaskScope` inside a **try-with-resources** block.
* Subtasks are spawned using `scope.fork()`, running on virtual threads.
* All spawned subtasks are guaranteed to complete or terminate before control exits the try-with-resources block. No "orphan threads" can leak.

### 🧩 Core Concepts & Joining Policies
Configure the scope's completion and error handling via specific `Joiner` policies passed to `StructuredTaskScope.open()`:

* **`Joiner.allSuccessfulOrThrow()`**: 
  * *Behavior:* Short-circuits immediately.
  * *Action:* If any subtask fails, all other active sibling subtasks are automatically interrupted/cancelled, and the scope throws a `FailedException`.
* **`Joiner.anySuccessfulOrThrow()`**: 
  * *Behavior:* First to finish wins.
  * *Action:* Returns the result of the fastest successful subtask and immediately cancels all slower, remaining threads (ideal for redundant racing calls).
* **`Joiner.awaitAll()`**: 
  * *Behavior:* No short-circuit.
  * *Action:* Waits for all forks to finish regardless of success/failure, allowing manual state inspection of each task via `.state()`.

---

## 📋 **Summary: Interview Priority Matrix**

| Concept | Frequency | Difficulty | Prep Time |
|---------|-----------|------------|-----------|
| **Thread Pool & ExecutorService** | 🔴 High | Medium | 2-3 hrs |
| **Atomic Classes** | 🔴 High | Medium | 1-2 hrs |
| **Synchronized vs volatile** | 🟠 Medium | Easy-Medium | 1-2 hrs |
| **BlockingQueue & Producer-Consumer** | 🟠 Medium | Medium-Hard | 2-3 hrs |
| **CompletableFuture** | 🔴 High | Medium | 2-3 hrs |
| **Concurrent Collections** | 🟠 Medium | Easy-Medium | 1-2 hrs |
| **Synchronization Tools (CountDownLatch, etc.)** | 🟡 Low-Medium | Easy | 1-2 hrs |

---

## 📚 **Resources to Master These Concepts**

1. **Book:** [Java Concurrency in Practice](https://jcip.net/) (Definitive reference)
2. **Article:** [Baeldung Java Concurrent Tutorial](https://baeldung.com/java-concurrency/)
3. **Practice:** LeetCode Medium/Hard concurrency problems

---
