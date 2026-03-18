# Concurrency & Multithreading in Java

## 1. Thread Fundamentals

### What is a Thread?

A thread is the smallest unit of execution within a process. Java supports multithreading natively
through the `java.lang.Thread` class and `java.lang.Runnable` interface.

### Thread Lifecycle

A thread goes through the following states defined in `Thread.State`:

```
NEW → RUNNABLE → (BLOCKED | WAITING | TIMED_WAITING) → TERMINATED
```

| State             | Description                                                            |
|-------------------|------------------------------------------------------------------------|
| **NEW**           | Thread created but `start()` not yet called                            |
| **RUNNABLE**      | Executing or ready to execute (includes OS "running" and "ready")      |
| **BLOCKED**       | Waiting to acquire a monitor lock                                      |
| **WAITING**       | Waiting indefinitely for another thread (`wait()`, `join()`, `park()`) |
| **TIMED_WAITING** | Waiting with a timeout (`sleep()`, `wait(timeout)`, `join(timeout)`)   |
| **TERMINATED**    | Execution completed or exception thrown                                |

> **Reference
**: [Thread.State - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.State.html)

### Creating Threads

```java
// 1. Extending Thread
class MyThread extends Thread {

  public void run() { /* work */ }
}

// 2. Implementing Runnable (preferred — allows extending another class)
class MyTask implements Runnable {

  public void run() { /* work */ }
}

// 3. Using Lambda
Thread t = new Thread(() -> System.out.println("Running"));
t.

start();
```

### Daemon vs Non-Daemon Threads

- **Non-daemon**: JVM waits for all non-daemon threads to finish before shutting down.
- **Daemon**: Background threads (e.g., GC). JVM exits when only daemon threads remain.

```java
thread.setDaemon(true); // must be called before start()
```

---

## 2. Synchronization

### The Problem: Race Conditions

When multiple threads read/write shared mutable state without coordination, results become
unpredictable.

### `synchronized` Keyword

Provides **mutual exclusion** and **happens-before** guarantees.

```java
// Synchronized method — lock is on `this` (or Class object for static)
public synchronized void increment() {
  count++;
}

// Synchronized block — more granular
public void increment() {
  synchronized (this) {
    count++;
  }
}
```

**Key points:**

- Every object in Java has an **intrinsic lock (monitor)**.
- `synchronized` is **reentrant** — a thread can acquire the same lock it already holds.
- Always synchronize on the **same lock** when accessing shared state.

> **Reference
**: [Intrinsic Locks - Java Tutorials](https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html)

### `volatile` Keyword

Guarantees **visibility** across threads — reads/writes go directly to main memory, not thread-local
cache.

```java
private volatile boolean running = true;
```

**What `volatile` does:**

- Prevents instruction reordering around the volatile variable.
- Guarantees happens-before: a write to a volatile variable happens-before every subsequent read.

**What `volatile` does NOT do:**

- Does NOT provide atomicity. `count++` on a volatile variable is still **not thread-safe** (
  read-modify-write).

> **Reference
**: [Java Memory Model - JSR 133 FAQ](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)

---

## 3. Atomic Classes

The `java.util.concurrent.atomic` package provides lock-free, thread-safe operations using **CAS (
Compare-And-Swap)**.

| Class                | Description                                                             |
|----------------------|-------------------------------------------------------------------------|
| `AtomicInteger`      | Thread-safe int operations                                              |
| `AtomicLong`         | Thread-safe long operations                                             |
| `AtomicBoolean`      | Thread-safe boolean                                                     |
| `AtomicReference<V>` | Thread-safe object reference                                            |
| `LongAdder`          | High-throughput counter (better than AtomicLong under heavy contention) |

```java
AtomicInteger counter = new AtomicInteger(0);
counter.

incrementAndGet();    // atomic i++
counter.

compareAndSet(1,2);  // CAS: set to 2 only if current value is 1
```

**CAS (Compare-And-Swap):**

- Hardware-level instruction.
- Reads current value, compares with expected, sets new value only if they match.
- If another thread changed it, retry (spin).
- No lock needed — hence "lock-free".

> **Reference
**: [java.util.concurrent.atomic - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/package-summary.html)

---

## 4. Locks (java.util.concurrent.locks)

### ReentrantLock

More flexible alternative to `synchronized`.

```java
ReentrantLock lock = new ReentrantLock();

lock.

lock();
try{
    // critical section
    }finally{
    lock.

unlock(); // ALWAYS unlock in finally
}
```

**Advantages over synchronized:**

- `tryLock()` — non-blocking attempt to acquire lock.
- `tryLock(timeout, unit)` — timed attempt.
- `lockInterruptibly()` — can be interrupted while waiting.
- Fairness policy — `new ReentrantLock(true)` gives lock to longest-waiting thread.
- Can have multiple `Condition` objects (vs single wait/notify per monitor).

### ReadWriteLock

Allows **concurrent reads** but **exclusive writes**.

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.

readLock().

lock();   // multiple threads can hold this simultaneously
rwLock.

writeLock().

lock();  // exclusive — blocks all readers and writers
```

**Use when**: reads greatly outnumber writes (e.g., caching).

### StampedLock (Java 8+)

Optimistic read lock — even faster than ReadWriteLock for read-heavy workloads.

```java
StampedLock sl = new StampedLock();
long stamp = sl.tryOptimisticRead();
// read shared data
if(!sl.

validate(stamp)){
// fallback to pessimistic read
stamp =sl.

readLock();
    try{ /* re-read data */ }finally{sl.

unlockRead(stamp); }
    }
```

> **Reference
**: [java.util.concurrent.locks - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/package-summary.html)

---

## 5. ThreadLocal

Each thread gets its own **isolated copy** of a variable.

```java
ThreadLocal<SimpleDateFormat> formatter =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

// Each thread gets its own SimpleDateFormat instance
String date = formatter.get().format(new Date());
```

**Common use cases:**

- Per-thread database connections.
- Per-thread user context (e.g., SecurityContext in Spring Security).
- Avoiding synchronization for thread-confined objects.

**Pitfalls:**

- **Memory leaks** in thread pools — always call `remove()` when done.
- In thread pools, threads are reused, so stale ThreadLocal values can leak between tasks.

> **Reference
**: [ThreadLocal - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ThreadLocal.html)

---

## 6. Executor Framework

### Why Not `new Thread()` Directly?

- Thread creation is expensive (OS-level resource).
- No control over how many threads run simultaneously.
- No task queuing, scheduling, or result handling.

### ExecutorService

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

// Submit Runnable
executor.

submit(() ->System.out.

println("Task 1"));

// Submit Callable (returns a value)
Future<String> future = executor.submit(() -> "Result");
String result = future.get(); // blocks until done

executor.

shutdown(); // graceful shutdown
```

### Thread Pool Types

| Factory Method              | Pool Type                 | Use Case                    |
|-----------------------------|---------------------------|-----------------------------|
| `newFixedThreadPool(n)`     | Fixed size                | Known, bounded concurrency  |
| `newCachedThreadPool()`     | Unbounded, auto-shrinking | Many short-lived tasks      |
| `newSingleThreadExecutor()` | Single thread             | Sequential task execution   |
| `newScheduledThreadPool(n)` | Scheduled                 | Periodic/delayed tasks      |
| `newWorkStealingPool()`     | ForkJoinPool-based        | CPU-intensive parallel work |

> **Reference
**: [ExecutorService - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html)

### CompletableFuture (Java 8+)

Non-blocking, composable asynchronous programming.

```java
CompletableFuture.supplyAsync(() ->

fetchUser(id))
    .

thenApply(user ->user.

getName())
    .

thenAccept(name ->System.out.

println("Hello "+name))
    .

exceptionally(ex ->{log.

error(ex); return null;});

// Combine two futures
CompletableFuture<String> nameFuture = CompletableFuture.supplyAsync(() -> getName());
CompletableFuture<Integer> ageFuture = CompletableFuture.supplyAsync(() -> getAge());

CompletableFuture<String> combined = nameFuture.thenCombine(ageFuture,
    (name, age) -> name + " is " + age);
```

**Key methods:**

- `thenApply` — transform result (like `map`)
- `thenCompose` — chain another async operation (like `flatMap`)
- `thenCombine` — combine two independent futures
- `allOf` / `anyOf` — wait for all/any futures
- `exceptionally` / `handle` — error handling

> **Reference
**: [CompletableFuture - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html)

---

## 7. Concurrent Collections

| Collection                  | Description                                                                |
|-----------------------------|----------------------------------------------------------------------------|
| `ConcurrentHashMap`         | Segmented locking (Java 8+: node-level CAS). No full-map lock.             |
| `CopyOnWriteArrayList`      | Creates a new copy on every write. Great for read-heavy, write-rare cases. |
| `ConcurrentLinkedQueue`     | Lock-free, non-blocking FIFO queue.                                        |
| `BlockingQueue` (interface) | Thread-safe queue with blocking `put()` and `take()`.                      |
| `LinkedBlockingQueue`       | Bounded or unbounded blocking queue.                                       |
| `ArrayBlockingQueue`        | Bounded, backed by array.                                                  |

### ConcurrentHashMap Deep Dive

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Atomic operations
map.

putIfAbsent("key",1);
map.

compute("key",(k, v) ->v ==null?1:v +1);  // atomic read-modify-write
    map.

merge("key",1,Integer::sum);                      // atomic merge
```

**Differences from `Collections.synchronizedMap()`:**

- `synchronizedMap` locks the **entire map** for every operation.
- `ConcurrentHashMap` uses fine-grained locking / CAS — much better throughput.
- `ConcurrentHashMap` does NOT allow null keys or values.

> **Reference
**: [ConcurrentHashMap - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)

---

## 8. Synchronization Utilities

### CountDownLatch

Wait for N events to complete.

```java
CountDownLatch latch = new CountDownLatch(3);

// In worker threads:
latch.

countDown(); // decrement count

// In main thread:
latch.

await(); // blocks until count reaches 0
```

**One-time use only** — cannot be reset.

### CyclicBarrier

N threads wait for each other at a barrier point. **Reusable**.

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("All arrived"));

// In each thread:
barrier.

await(); // blocks until all 3 threads call await()
```

### Semaphore

Controls access to a limited number of resources.

```java
Semaphore semaphore = new Semaphore(3); // 3 permits

semaphore.

acquire(); // blocks if no permits available
try{
    // access limited resource
    }finally{
    semaphore.

release();
}
```

### Phaser (Java 7+)

More flexible alternative to CountDownLatch + CyclicBarrier. Supports dynamic registration.

> **Reference
**: [java.util.concurrent - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/package-summary.html)

---

## 9. Common Concurrency Problems

### Deadlock

Two or more threads waiting for each other's locks forever.

```
Thread 1: holds Lock A, waiting for Lock B
Thread 2: holds Lock B, waiting for Lock A
```

**Prevention:**

- Always acquire locks in a **consistent global order**.
- Use `tryLock()` with timeout.
- Minimize lock scope.

### Livelock

Threads keep changing state in response to each other but make no progress (e.g., two people in a
hallway both stepping aside repeatedly).

### Starvation

A thread is perpetually denied access to resources (e.g., low-priority thread never gets CPU time).

### False Sharing

Different threads modify independent variables that happen to share the same **CPU cache line**,
causing unnecessary cache invalidation. Use `@Contended` annotation (JDK internal) to pad fields.

---

## 10. Virtual Threads (Project Loom — Java 21+)

### The Problem with Platform Threads

- Each platform thread maps 1:1 to an OS thread.
- OS threads are expensive: ~1MB stack memory each, limited by OS.
- Thread-per-request model breaks at high concurrency (e.g., 100k concurrent requests).

### What are Virtual Threads?

Virtual threads are **lightweight threads** managed by the JVM, not the OS.

- **Extremely cheap**: millions of virtual threads are feasible.
- Mounted/unmounted on **carrier threads** (a small pool of platform threads).
- When a virtual thread blocks (I/O, sleep, lock), it is **unmounted** from the carrier thread,
  freeing it for other virtual threads.

```java
// Creating virtual threads
Thread vt = Thread.ofVirtual().start(() -> {
      System.out.println("Running on: " + Thread.currentThread());
    });

// Using executor
try(
var executor = Executors.newVirtualThreadPerTaskExecutor()){
    for(
int i = 0;
i< 100_000;i++){
    executor.

submit(() ->{
    Thread.

sleep(Duration.ofSeconds(1));
    return"done";
    });
    }
    }
```

### Virtual Threads vs Platform Threads

| Aspect        | Platform Thread               | Virtual Thread                |
|---------------|-------------------------------|-------------------------------|
| Managed by    | OS                            | JVM                           |
| Memory        | ~1MB stack                    | Few KB (grows as needed)      |
| Quantity      | Thousands                     | Millions                      |
| Scheduling    | OS scheduler                  | JVM scheduler (ForkJoinPool)  |
| Blocking cost | Expensive (ties up OS thread) | Cheap (unmounts from carrier) |
| Use case      | CPU-intensive work            | I/O-intensive work            |

### Key Behaviors and Gotchas

**Pinning**: A virtual thread gets **pinned** to its carrier thread (cannot unmount) when:

- Inside a `synchronized` block/method while blocking.
- Inside a native method (JNI).

**Solution**: Replace `synchronized` with `ReentrantLock` for blocking operations.

```java
// BAD — causes pinning
synchronized (lock){
    channel.

read(buffer); // blocks — virtual thread pinned to carrier
}

// GOOD — no pinning
    reentrantLock.

lock();
try{
    channel.

read(buffer); // blocks — virtual thread can unmount
}finally{
    reentrantLock.

unlock();
}
```

**ThreadLocal caution**: Since you can have millions of virtual threads, large ThreadLocal values
cause memory issues. Prefer **Scoped Values** (preview in Java 21).

```java
// Scoped Values (preview) — lightweight alternative to ThreadLocal
static final ScopedValue<String> USER = ScopedValue.newInstance();

ScopedValue.

where(USER, "alice").

run(() ->{
    System.out.

println(USER.get()); // "alice"
    });
```

### Structured Concurrency (Preview — Java 21+)

Treats groups of related tasks as a single unit of work.

```java
try(var scope = new StructuredTaskScope.ShutdownOnFailure()){
Subtask<String> user = scope.fork(() -> fetchUser());
Subtask<Integer> order = scope.fork(() -> fetchOrder());

    scope.

join();           // wait for all
    scope.

throwIfFailed();  // propagate errors

    return new

Response(user.get(),order.

get());
    }
// If one fails, the other is automatically cancelled
```

### When to Use Virtual Threads

- **Use**: I/O-bound workloads (HTTP servers, DB queries, file I/O, API calls).
- **Don't use**: CPU-bound workloads (compression, hashing, complex calculations) — use platform
  threads / ForkJoinPool instead.
- **Spring Boot 3.2+**: Enable with `spring.threads.virtual.enabled=true`.

> **References**:
> - [JEP 444: Virtual Threads](https://openjdk.org/jeps/444)
> - [JEP 453: Structured Concurrency (Preview)](https://openjdk.org/jeps/453)
> - [JEP 446: Scoped Values (Preview)](https://openjdk.org/jeps/446)
> - [Virtual Threads - Inside Java](https://inside.java/2023/10/16/virtual-threads/)
> - [Spring Boot Virtual Threads](https://docs.spring.io/spring-boot/reference/features/spring-application.html#features.spring-application.virtual-threads)

---

## Interview Questions Cheat Sheet

| Question                          | Key Points                                                                          |
|-----------------------------------|-------------------------------------------------------------------------------------|
| `synchronized` vs `ReentrantLock` | ReentrantLock: tryLock, fairness, multiple Conditions, interruptible                |
| `volatile` vs `AtomicInteger`     | volatile = visibility only; Atomic = visibility + atomicity (CAS)                   |
| `HashMap` vs `ConcurrentHashMap`  | ConcurrentHashMap: thread-safe, no null keys/values, fine-grained locking           |
| `wait()/notify()` vs `Condition`  | Condition is more flexible (multiple conditions per lock, timed wait)               |
| Thread pool sizing                | CPU-bound: N threads = N cores. I/O-bound: N threads = N cores × (1 + wait/compute) |
| How to detect deadlock            | `jstack`, `ThreadMXBean.findDeadlockedThreads()`, JConsole                          |
| Virtual vs Platform threads       | Virtual for I/O-bound (millions), Platform for CPU-bound (= cores)                  |
| What is pinning?                  | Virtual thread stuck on carrier in `synchronized` block — use ReentrantLock         |

---

## Deep Dive Resources (by Section)

### 1. Thread Fundamentals

- [Life Cycle of a Thread in Java — Baeldung](https://www.baeldung.com/java-thread-lifecycle)
- [Java Threads and Runnable Tutorial — DigitalOcean](https://www.digitalocean.com/community/tutorials/java-thread-example)

### 2. Synchronization

- [Guide to the synchronized Keyword — Baeldung](https://www.baeldung.com/java-synchronized)
- [Java Volatile Keyword Explained — Jenkov](https://jenkov.com/tutorials/java-concurrency/volatile.html)
- [Java Memory Model Explained (JSR-133 FAQ) — University of Maryland](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)

### 3. Atomic Classes

- [Introduction to Atomic Variables in Java — Baeldung](https://www.baeldung.com/java-atomic-variables)
- [LongAdder vs AtomicLong — Baeldung](https://www.baeldung.com/java-longadder-longaccumulator)
- [Understanding CAS (Compare-And-Swap) — Jenkov](https://jenkov.com/tutorials/java-concurrency/compare-and-swap.html)

### 4. Locks

- [Guide to java.util.concurrent.Locks — Baeldung](https://www.baeldung.com/java-concurrent-locks)
- [StampedLock in Java — Baeldung](https://www.baeldung.com/java-stamped-lock)
- [ReentrantLock vs synchronized — DZone](https://dzone.com/articles/what-are-reentrant-locks)

### 5. ThreadLocal

- [ThreadLocal in Java — Baeldung](https://www.baeldung.com/java-threadlocal)
- [The Dark Side of ThreadLocal — DZone](https://dzone.com/articles/an-in-depth-understanding-of-threadlocal-in-java)

### 6. Executor Framework & CompletableFuture

- [Java ExecutorService Guide — Baeldung](https://www.baeldung.com/java-executor-service-tutorial)
- [Guide to CompletableFuture — Baeldung](https://www.baeldung.com/java-completablefuture)
- [CompletableFuture — Callicoder (in-depth tutorial)](https://www.callicoder.com/java-8-completablefuture-tutorial/)
- [Thread Pool Sizing: Brian Goetz's formula explained — DZone](https://dzone.com/articles/how-to-calculate-thread-pool-size)

### 7. Concurrent Collections

- [ConcurrentHashMap Internals in Java 8 — DZone](https://dzone.com/articles/how-concurrenthashmap-works-internally-in-java)
- [Guide to ConcurrentHashMap — Baeldung](https://www.baeldung.com/java-concurrent-map)
- [BlockingQueue Guide — Baeldung](https://www.baeldung.com/java-blocking-queue)

### 8. Synchronization Utilities

- [CountDownLatch vs CyclicBarrier — Baeldung](https://www.baeldung.com/java-countdown-latch)
- [Semaphore in Java — Baeldung](https://www.baeldung.com/java-semaphore)
- [Phaser Guide — Baeldung](https://www.baeldung.com/java-phaser)

### 9. Common Concurrency Problems

- [Deadlock in Java with Examples — Baeldung](https://www.baeldung.com/java-deadlock-livelock)
- [False Sharing and CPU Cache Lines — Mechanical Sympathy (Martin Thompson)](https://mechanical-sympathy.blogspot.com/2011/07/false-sharing.html)
- [Java Concurrency in Practice (book) — Brian Goetz](https://jcip.net/)

### 10. Virtual Threads

- [JEP 444: Virtual Threads — OpenJDK](https://openjdk.org/jeps/444)
- [Virtual Threads: An Adoption Guide — Inside Java](https://inside.java/2024/02/15/virtual-threads-adoption-guide/)
- [The Ultimate Guide to Virtual Threads — Rock the JVM](https://blog.rockthejvm.com/ultimate-guide-to-java-virtual-threads/)
- [Virtual Thread Pinning: What It Is and How to Avoid It — Inside Java](https://inside.java/2024/07/30/virtual-thread-pinning/)
- [Structured Concurrency in Java — Inside Java](https://inside.java/2024/01/29/structured-concurrency/)
- [Scoped Values in Java (JEP 446) — Baeldung](https://www.baeldung.com/java-scoped-value)
- [Spring Boot Virtual Threads Guide — Spring Blog](https://spring.io/blog/2023/09/09/all-together-now-spring-boot-3-2-graalvm-native-images-java-21-and-virtual)
