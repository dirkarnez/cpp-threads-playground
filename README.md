cpp-threads-playground
======================
### Poe
```
Sharing a variable between threads in Java depends on the variable’s type, the concurrency semantics you need (visibility, atomicity), and whether multiple threads will read and write. Below are correct approaches, when to use them, and pitfalls to avoid.

1) Use immutable objects for read-only sharing
- Make the object immutable (final fields, no setters). Immutable objects are thread-safe by design because state cannot change after construction.
- Example: share a final List created via List.of(...) or Collections.unmodifiableList(copy).

2) Use volatile for simple visibility (no compound operations)
- Declare the shared reference or primitive as volatile to ensure writes by one thread are visible to others immediately.
- Suitable for flags, single writes/reads, or replacing whole immutable objects:
- volatile boolean running;
- volatile MyConfig config;
- Not suitable for compound operations (x++; read-modify-write).

3) Use atomic classes for lock-free atomic operations
- java.util.concurrent.atomic provides AtomicInteger, AtomicLong, AtomicReference, AtomicBoolean, and classes for updaters.
- Use for atomic read-modify-write without locks:
- AtomicInteger counter = new AtomicInteger(0);
- counter.incrementAndGet();
- Use AtomicReferencewzxhzdk:0 to atomically update object references (compareAndSet, getAndSet).

4) Use synchronized for mutual exclusion and compound actions
- Use synchronized methods or synchronized blocks to protect access when multiple operations must be atomic together:
- synchronized(lock) { / read, modify, write / }
- Use private final Object lock = new Object(); instead of synchronizing on "this" or on mutable objects.
- synchronized provides both mutual exclusion and visibility guarantees (happens-before).

5) Use java.util.concurrent.locks for advanced locking
- ReentrantLock, ReadWriteLock, StampedLock provide features not available with synchronized (tryLock, timed waits, fair locking, separate read/write locks).
- ReadWriteLock is useful when many readers and few writers: readers acquire shared lock, writers exclusive lock.

6) Use concurrent collections for shared collections
- Prefer thread-safe collections from java.util.concurrent:
- ConcurrentHashMap, ConcurrentLinkedQueue, CopyOnWriteArrayList, BlockingQueue (e.g., ArrayBlockingQueue, LinkedBlockingQueue).
- Choose based on access pattern: CopyOnWriteArrayList for few writes/many reads; ConcurrentHashMap for concurrent map updates; BlockingQueue for producer-consumer.

7) Use higher-level concurrency utilities
- Executors (thread pools) for managing threads.
- CompletableFuture for async composition.
- CountDownLatch, CyclicBarrier, Phaser, Semaphore for coordination.
- TransferQueue and SynchronousQueue for handoff between threads.

8) Avoid common mistakes
- Do not rely on non-volatile/non-synchronized fields for cross-thread visibility — may see stale values.
- Do not perform compound operations on volatile variables expecting atomicity.
- Avoid synchronizing on public or mutable objects (deadlock/security risk).
- Prefer immutable snapshots or atomic replacements over fine-grained shared mutable state when possible.

9) Practical rules of thumb
- Single-writer-many-readers: use volatile for reference replacement or CopyOnWrite style.
- Frequent concurrent updates: use Atomic* types or Concurrent collections.
- Compound operations spanning multiple fields: use synchronization or a lock.
- Producer-consumer: use BlockingQueue.

10) Minimal examples
- Volatile flag:
- volatile boolean running = true;
- thread reads while (running) { … }
- other thread sets running = false;
- Atomic counter:
- AtomicInteger c = new AtomicInteger();
- c.incrementAndGet();
- Synchronized access:
- private final Object lock = new Object();
- synchronized(lock) { sharedList.add(x); }

References and behavior notes

Volatile guarantees visibility and ordering for single writes/reads; synchronized and locks provide mutual exclusion plus happens-before semantics.
Atomic classes use compare-and-swap under the hood and provide lock-free atomicity for individual variables.
Choose the simplest mechanism that provides the needed visibility and atomicity. For complex coordination, prefer java.util.concurrent utilities and immutable snapshots to reduce bugs.
```
### Tutorials
- [C++多线程示例-CSDN博客](https://chongbin.blog.csdn.net/article/details/150216681)

### TODOs
- [ ] CMake
  - ```CMake
    set (THREADS_PREFER_PTHREAD_FLAG ON)
    find_package(Threads)
    ```
