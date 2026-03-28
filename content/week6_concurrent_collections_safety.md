# Week 6: Concurrent Collections & Thread Safety - Master Java

## 📋 Table of Contents
1. [Fundamentals of Thread Safety](#fundamentals-of-thread-safety)
2. [Synchronization Mechanisms](#synchronization-mechanisms)
3. [Race Conditions & Deadlocks](#race-conditions--deadlocks)
4. [Concurrent Collections](#concurrent-collections)
5. [Practice Exercises](#practice-exercises)
6. [Interview Questions & Answers](#interview-questions--answers)
7. [Real-World Scenarios](#real-world-scenarios)

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Understand what thread safety is and why it matters
- Master `synchronized` keyword and `ReentrantLock`
- Identify and prevent race conditions and deadlocks
- Use concurrent collections effectively
- Write production-ready concurrent code
- Confidently answer interview questions on concurrency

---

## Fundamentals of Thread Safety

### What is Thread Safety?

**Thread safety** means that a piece of code functions correctly when accessed from multiple threads simultaneously, regardless of the scheduling or interleaving of those threads by the runtime environment.

#### The Problem: Shared Mutable State

```java
// ❌ NOT THREAD-SAFE
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // This is actually 3 operations: read, modify, write
    }
    
    public int getCount() {
        return count;
    }
}
```

**Why is this unsafe?**

The `count++` operation is **not atomic**. It consists of:
1. **Read** the current value of count
2. **Add** 1 to that value
3. **Write** the new value back

When two threads execute simultaneously:

```
Thread 1: Read count (0)
Thread 2: Read count (0)
Thread 1: Add 1 (0 + 1 = 1)
Thread 2: Add 1 (0 + 1 = 1)
Thread 1: Write 1
Thread 2: Write 1
Final count: 1 (should be 2!)
```

#### Example: Demonstrating the Problem

```java
public class ThreadSafetyDemo {
    static class UnsafeCounter {
        private int count = 0;
        
        public void increment() {
            count++;
        }
        
        public int getCount() {
            return count;
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        UnsafeCounter counter = new UnsafeCounter();
        
        // Create 10 threads, each incrementing 1000 times
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        // Expected: 10,000, Actual: varies (often less)
        System.out.println("Final count: " + counter.getCount());
    }
}
```

**Output:** The count will likely be less than 10,000 and will vary on each run.

---

## Synchronization Mechanisms

### The `synchronized` Keyword

`synchronized` is Java's built-in mechanism for ensuring thread safety. It uses **intrinsic locks** (also called monitor locks).

#### Types of Synchronization

**1. Synchronized Instance Method**

```java
public class SynchronizedCounter {
    private int count = 0;
    
    // Locks on 'this' object
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

**2. Synchronized Static Method**

```java
public class GlobalCounter {
    private static int count = 0;
    
    // Locks on the Class object (GlobalCounter.class)
    public static synchronized void increment() {
        count++;
    }
    
    public static synchronized int getCount() {
        return count;
    }
}
```

**3. Synchronized Block**

```java
public class OptimizedCounter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        // Only synchronize the critical section
        synchronized(lock) {
            count++;
        }
    }
    
    public int getCount() {
        synchronized(lock) {
            return count;
        }
    }
}
```

#### When to Use Each Type

| Type | Use Case | Lock Object |
|------|----------|-------------|
| Synchronized Method | Simple, entire method is critical | `this` |
| Synchronized Static Method | Class-level shared state | `ClassName.class` |
| Synchronized Block | Fine-grained control, part of method | Any object |

#### Example: Bank Account Transfer

```java
public class BankAccount {
    private double balance;
    private final Object lock = new Object();
    
    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }
    
    public void deposit(double amount) {
        synchronized(lock) {
            System.out.println(Thread.currentThread().getName() + 
                             " depositing " + amount);
            balance += amount;
        }
    }
    
    public boolean withdraw(double amount) {
        synchronized(lock) {
            if (balance >= amount) {
                System.out.println(Thread.currentThread().getName() + 
                                 " withdrawing " + amount);
                balance -= amount;
                return true;
            }
            return false;
        }
    }
    
    public double getBalance() {
        synchronized(lock) {
            return balance;
        }
    }
    
    // Transfer between accounts - requires careful synchronization!
    public static void transfer(BankAccount from, BankAccount to, double amount) {
        // Problem: This can cause deadlock! (We'll fix this later)
        synchronized(from.lock) {
            synchronized(to.lock) {
                if (from.withdraw(amount)) {
                    to.deposit(amount);
                    System.out.println("Transferred " + amount);
                }
            }
        }
    }
}
```

### ReentrantLock

`ReentrantLock` is part of `java.util.concurrent.locks` and provides more flexibility than `synchronized`.

#### Basic Usage

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockCounter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();
    
    public void increment() {
        lock.lock();  // Acquire the lock
        try {
            count++;
        } finally {
            lock.unlock();  // ALWAYS unlock in finally block!
        }
    }
    
    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

#### Advanced Features of ReentrantLock

**1. Try-Lock (Non-blocking)**

```java
public class TryLockExample {
    private final Lock lock = new ReentrantLock();
    
    public void attemptOperation() {
        if (lock.tryLock()) {  // Try to acquire lock without blocking
            try {
                System.out.println("Got the lock, doing work...");
                // Critical section
            } finally {
                lock.unlock();
            }
        } else {
            System.out.println("Couldn't get lock, doing alternative action");
            // Do something else
        }
    }
}
```

**2. Try-Lock with Timeout**

```java
import java.util.concurrent.TimeUnit;

public class TimeoutLockExample {
    private final Lock lock = new ReentrantLock();
    
    public void timedOperation() throws InterruptedException {
        if (lock.tryLock(5, TimeUnit.SECONDS)) {  // Wait up to 5 seconds
            try {
                System.out.println("Acquired lock within timeout");
                // Critical section
            } finally {
                lock.unlock();
            }
        } else {
            System.out.println("Timeout: couldn't acquire lock");
        }
    }
}
```

**3. Interruptible Lock Acquisition**

```java
public class InterruptibleLockExample {
    private final Lock lock = new ReentrantLock();
    
    public void interruptibleOperation() throws InterruptedException {
        lock.lockInterruptibly();  // Can be interrupted while waiting
        try {
            // Critical section
            System.out.println("Working...");
        } finally {
            lock.unlock();
        }
    }
}
```

**4. Fair Locks**

```java
public class FairLockExample {
    // Fair lock: threads acquire lock in order they requested it
    private final Lock fairLock = new ReentrantLock(true);
    
    // Unfair lock (default): no order guarantee, better performance
    private final Lock unfairLock = new ReentrantLock(false);
    
    public void fairOperation() {
        fairLock.lock();
        try {
            // Threads get lock in FIFO order
            System.out.println(Thread.currentThread().getName() + " executing");
        } finally {
            fairLock.unlock();
        }
    }
}
```

**5. Condition Variables**

```java
import java.util.concurrent.locks.Condition;

public class BoundedBuffer<T> {
    private final Object[] items;
    private int count, putIndex, takeIndex;
    
    private final Lock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
    
    public BoundedBuffer(int capacity) {
        items = new Object[capacity];
    }
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) {
                notFull.await();  // Wait until not full
            }
            items[putIndex] = item;
            putIndex = (putIndex + 1) % items.length;
            count++;
            notEmpty.signal();  // Signal that buffer is not empty
        } finally {
            lock.unlock();
        }
    }
    
    @SuppressWarnings("unchecked")
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.await();  // Wait until not empty
            }
            T item = (T) items[takeIndex];
            items[takeIndex] = null;
            takeIndex = (takeIndex + 1) % items.length;
            count--;
            notFull.signal();  // Signal that buffer is not full
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

### `synchronized` vs `ReentrantLock`: Detailed Comparison

| Feature | synchronized | ReentrantLock |
|---------|-------------|---------------|
| **Syntax** | Simple, built-in keyword | Explicit lock/unlock calls |
| **Try-Lock** | ❌ Not available | ✅ `tryLock()` |
| **Timeout** | ❌ Not available | ✅ `tryLock(timeout)` |
| **Interruptible** | ❌ Cannot interrupt | ✅ `lockInterruptibly()` |
| **Fairness** | ❌ Not guaranteed | ✅ Optional fair mode |
| **Condition Variables** | ❌ Single wait/notify | ✅ Multiple `Condition` objects |
| **Performance** | Slightly faster (HotSpot optimized) | Comparable in modern JVMs |
| **Lock Release** | Automatic | Manual (must use try-finally) |
| **Readability** | More concise | More verbose |
| **Flexibility** | Limited | High |

#### When to Use Which?

**Use `synchronized` when:**
- You need simple mutual exclusion
- The entire method/block is the critical section
- You don't need advanced features
- Code simplicity is important

```java
public synchronized void simpleOperation() {
    // Simple critical section
    this.value++;
}
```

**Use `ReentrantLock` when:**
- You need try-lock or timed lock
- You need interruptible locks
- You need fairness guarantees
- You need multiple condition variables
- You need more complex synchronization logic

```java
public void complexOperation() {
    if (lock.tryLock(5, TimeUnit.SECONDS)) {
        try {
            // Complex critical section
        } finally {
            lock.unlock();
        }
    }
}
```

#### Real-World Example: Database Connection Pool

```java
import java.util.concurrent.locks.*;
import java.util.*;

public class ConnectionPool {
    private final Queue<Connection> available;
    private final Set<Connection> inUse;
    private final Lock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final int maxConnections;
    
    public ConnectionPool(int maxConnections) {
        this.maxConnections = maxConnections;
        this.available = new LinkedList<>();
        this.inUse = new HashSet<>();
        
        // Initialize pool
        for (int i = 0; i < maxConnections; i++) {
            available.offer(new Connection("conn-" + i));
        }
    }
    
    public Connection acquire(long timeout, TimeUnit unit) 
            throws InterruptedException {
        lock.lock();
        try {
            long nanos = unit.toNanos(timeout);
            while (available.isEmpty()) {
                if (nanos <= 0) {
                    return null;  // Timeout
                }
                nanos = notEmpty.awaitNanos(nanos);
            }
            Connection conn = available.poll();
            inUse.add(conn);
            return conn;
        } finally {
            lock.unlock();
        }
    }
    
    public void release(Connection conn) {
        lock.lock();
        try {
            if (inUse.remove(conn)) {
                available.offer(conn);
                notEmpty.signal();
            }
        } finally {
            lock.unlock();
        }
    }
    
    public int availableCount() {
        lock.lock();
        try {
            return available.size();
        } finally {
            lock.unlock();
        }
    }
    
    // Simple connection class for demo
    static class Connection {
        private final String id;
        Connection(String id) { this.id = id; }
        public String getId() { return id; }
    }
}
```

---

## Race Conditions & Deadlocks

### Race Conditions

A **race condition** occurs when the correctness of a program depends on the relative timing of multiple threads.

#### Example 1: Check-Then-Act

```java
// ❌ RACE CONDITION!
public class LazyInitialization {
    private Resource resource;
    
    public Resource getResource() {
        if (resource == null) {  // Check
            resource = new Resource();  // Act
        }
        return resource;
    }
}
```

**Problem:** Two threads can both see `resource == null` and both create a `Resource`.

**Solution 1: Synchronized**

```java
public class SafeLazyInitialization {
    private Resource resource;
    
    public synchronized Resource getResource() {
        if (resource == null) {
            resource = new Resource();
        }
        return resource;
    }
}
```

**Solution 2: Double-Checked Locking (with volatile)**

```java
public class DoubleCheckedLocking {
    private volatile Resource resource;  // volatile is crucial!
    
    public Resource getResource() {
        if (resource == null) {  // First check (no lock)
            synchronized(this) {
                if (resource == null) {  // Second check (with lock)
                    resource = new Resource();
                }
            }
        }
        return resource;
    }
}
```

**Solution 3: Initialization-on-Demand Holder (Recommended)**

```java
public class HolderPattern {
    private HolderPattern() {}
    
    private static class ResourceHolder {
        static final Resource INSTANCE = new Resource();
    }
    
    public static Resource getResource() {
        return ResourceHolder.INSTANCE;  // Thread-safe by JVM guarantee
    }
}
```

#### Example 2: Read-Modify-Write

```java
// ❌ RACE CONDITION!
public class NumberRange {
    private int lower = 0;
    private int upper = 10;
    
    public void setLower(int value) {
        if (value > upper) {  // Read
            throw new IllegalArgumentException("Lower > Upper");
        }
        lower = value;  // Write
    }
    
    public void setUpper(int value) {
        if (value < lower) {  // Read
            throw new IllegalArgumentException("Upper < Lower");
        }
        upper = value;  // Write
    }
}
```

**Problem:** 
```
Initial state: lower=0, upper=10
Thread 1: setLower(5)  - checks value <= 10, OK
Thread 2: setUpper(4)  - checks value >= 0, OK
Thread 1: sets lower = 5
Thread 2: sets upper = 4
Final state: lower=5, upper=4 (INVALID!)
```

**Solution:**

```java
public class SafeNumberRange {
    private int lower = 0;
    private int upper = 10;
    
    public synchronized void setLower(int value) {
        if (value > upper) {
            throw new IllegalArgumentException("Lower > Upper");
        }
        lower = value;
    }
    
    public synchronized void setUpper(int value) {
        if (value < lower) {
            throw new IllegalArgumentException("Upper < Lower");
        }
        upper = value;
    }
}
```

### Deadlocks

A **deadlock** occurs when two or more threads are blocked forever, each waiting for the other to release a resource.

#### Classic Deadlock Example

```java
public class DeadlockDemo {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized(lock1) {
            System.out.println(Thread.currentThread().getName() + 
                             ": Holding lock1...");
            
            try { Thread.sleep(10); } catch (InterruptedException e) {}
            
            System.out.println(Thread.currentThread().getName() + 
                             ": Waiting for lock2...");
            synchronized(lock2) {
                System.out.println(Thread.currentThread().getName() + 
                                 ": Acquired lock2!");
            }
        }
    }
    
    public void method2() {
        synchronized(lock2) {
            System.out.println(Thread.currentThread().getName() + 
                             ": Holding lock2...");
            
            try { Thread.sleep(10); } catch (InterruptedException e) {}
            
            System.out.println(Thread.currentThread().getName() + 
                             ": Waiting for lock1...");
            synchronized(lock1) {
                System.out.println(Thread.currentThread().getName() + 
                                 ": Acquired lock1!");
            }
        }
    }
    
    public static void main(String[] args) {
        DeadlockDemo demo = new DeadlockDemo();
        
        Thread t1 = new Thread(() -> demo.method1(), "Thread-1");
        Thread t2 = new Thread(() -> demo.method2(), "Thread-2");
        
        t1.start();
        t2.start();
        
        // Deadlock! Neither thread will complete.
    }
}
```

#### Four Conditions for Deadlock (Coffman Conditions)

1. **Mutual Exclusion**: Resources cannot be shared
2. **Hold and Wait**: Threads hold resources while waiting for others
3. **No Preemption**: Resources cannot be forcibly taken
4. **Circular Wait**: Circular chain of threads waiting for resources

#### Preventing Deadlocks

**Strategy 1: Lock Ordering**

Always acquire locks in a consistent global order.

```java
public class FixedDeadlock {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        // Always acquire lock1 first, then lock2
        synchronized(lock1) {
            synchronized(lock2) {
                // Do work
            }
        }
    }
    
    public void method2() {
        // Same order: lock1 first, then lock2
        synchronized(lock1) {
            synchronized(lock2) {
                // Do work
            }
        }
    }
}
```

**Strategy 2: Lock Timeout with ReentrantLock**

```java
import java.util.concurrent.locks.*;

public class TimeoutDeadlockPrevention {
    private final Lock lock1 = new ReentrantLock();
    private final Lock lock2 = new ReentrantLock();
    
    public boolean transferWithTimeout() throws InterruptedException {
        while (true) {
            if (lock1.tryLock(50, TimeUnit.MILLISECONDS)) {
                try {
                    if (lock2.tryLock(50, TimeUnit.MILLISECONDS)) {
                        try {
                            // Both locks acquired, do work
                            return true;
                        } finally {
                            lock2.unlock();
                        }
                    }
                } finally {
                    lock1.unlock();
                }
            }
            // Failed to acquire both locks, retry
            Thread.sleep(10);
        }
    }
}
```

**Strategy 3: Dynamic Lock Ordering**

```java
public class BankAccount {
    private final long id;
    private double balance;
    
    public BankAccount(long id, double balance) {
        this.id = id;
        this.balance = balance;
    }
    
    public long getId() { return id; }
    
    public static void transfer(BankAccount from, BankAccount to, double amount) {
        // Order locks by account ID to prevent deadlock
        BankAccount first = from.id < to.id ? from : to;
        BankAccount second = from.id < to.id ? to : from;
        
        synchronized(first) {
            synchronized(second) {
                if (from.balance >= amount) {
                    from.balance -= amount;
                    to.balance += amount;
                    System.out.println("Transferred " + amount);
                }
            }
        }
    }
}
```

#### Detecting Deadlocks

**Using Thread Dumps:**

```java
public class DeadlockDetector {
    public static void detectDeadlocks() {
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        long[] deadlockedThreads = threadMXBean.findDeadlockedThreads();
        
        if (deadlockedThreads != null) {
            ThreadInfo[] threadInfos = 
                threadMXBean.getThreadInfo(deadlockedThreads);
            
            System.out.println("Deadlock detected!");
            for (ThreadInfo threadInfo : threadInfos) {
                System.out.println(threadInfo.getThreadName() + 
                                 " is deadlocked");
            }
        }
    }
}
```

---

## Concurrent Collections

### Why Concurrent Collections?

Regular collections (ArrayList, HashMap, etc.) are not thread-safe. Simple synchronization (Collections.synchronizedList) has poor performance under high concurrency.

```java
// ❌ Poor concurrency - entire list locked for every operation
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

// ✅ Better - designed for concurrent access
CopyOnWriteArrayList<String> concurrentList = new CopyOnWriteArrayList<>();
```

### ConcurrentHashMap

`ConcurrentHashMap` allows concurrent reads and writes with fine-grained locking.

#### Basic Operations

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapBasics {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        
        // Basic operations (all thread-safe)
        map.put("apple", 1);
        map.put("banana", 2);
        map.putIfAbsent("apple", 10);  // Won't replace existing value
        
        System.out.println(map.get("apple"));  // 1
        
        // Atomic operations
        map.replace("apple", 1, 5);  // Replace only if current value is 1
        System.out.println(map.get("apple"));  // 5
        
        map.remove("banana", 2);  // Remove only if value is 2
    }
}
```

#### Advanced Atomic Operations

```java
public class ConcurrentHashMapAdvanced {
    private ConcurrentHashMap<String, Integer> wordCount = new ConcurrentHashMap<>();
    
    // ❌ NOT ATOMIC (race condition)
    public void incrementBad(String word) {
        Integer count = wordCount.get(word);
        if (count == null) {
            wordCount.put(word, 1);
        } else {
            wordCount.put(word, count + 1);
        }
    }
    
    // ✅ ATOMIC - using compute
    public void incrementGood(String word) {
        wordCount.compute(word, (key, oldValue) -> 
            oldValue == null ? 1 : oldValue + 1
        );
    }
    
    // ✅ ATOMIC - using merge (cleaner for this case)
    public void incrementBest(String word) {
        wordCount.merge(word, 1, Integer::sum);
    }
    
    // Example: Computing if absent
    public int getOrCompute(String key) {
        return wordCount.computeIfAbsent(key, k -> expensiveComputation(k));
    }
    
    private int expensiveComputation(String key) {
        // Only called if key is absent
        return key.length() * 10;
    }
}
```

#### Bulk Operations (Parallel Processing)

```java
public class ConcurrentHashMapBulk {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("A", 1);
        map.put("B", 2);
        map.put("C", 3);
        map.put("D", 4);
        
        // forEach (parallel)
        map.forEach(1, (key, value) -> 
            System.out.println(key + " = " + value)
        );
        
        // search (returns first match)
        String result = map.search(1, (key, value) -> 
            value > 2 ? key : null
        );
        System.out.println("First key with value > 2: " + result);
        
        // reduce (combine all values)
        Integer sum = map.reduce(1, 
            (key, value) -> value,  // Transform function
            Integer::sum            // Reduce function
        );
        System.out.println("Sum of all values: " + sum);
        
        // reduceValues
        Integer sumValues = map.reduceValues(1, Integer::sum);
        System.out.println("Sum using reduceValues: " + sumValues);
    }
}
```

#### Real-World Example: Cache Implementation

```java
import java.util.concurrent.*;
import java.util.function.Function;

public class ConcurrentCache<K, V> {
    private final ConcurrentHashMap<K, V> cache = new ConcurrentHashMap<>();
    private final ConcurrentHashMap<K, CompletableFuture<V>> futures = 
        new ConcurrentHashMap<>();
    
    /**
     * Thread-safe cache with automatic loading
     * Ensures expensive computation happens only once per key
     */
    public V get(K key, Function<K, V> loader) {
        // Fast path: value already cached
        V value = cache.get(key);
        if (value != null) {
            return value;
        }
        
        // Slow path: need to compute value
        CompletableFuture<V> future = futures.computeIfAbsent(key, k -> 
            CompletableFuture.supplyAsync(() -> loader.apply(k))
        );
        
        try {
            value = future.get();
            cache.put(key, value);
            return value;
        } catch (InterruptedException | ExecutionException e) {
            futures.remove(key);
            throw new RuntimeException(e);
        }
    }
    
    public void invalidate(K key) {
        cache.remove(key);
        futures.remove(key);
    }
    
    public int size() {
        return cache.size();
    }
}

// Usage
public class CacheDemo {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentCache<String, String> cache = new ConcurrentCache<>();
        
        // Simulate multiple threads requesting same data
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        for (int i = 0; i < 100; i++) {
            executor.submit(() -> {
                String result = cache.get("expensive-key", key -> {
                    System.out.println("Computing expensive value...");
                    try {
                        Thread.sleep(2000);  // Simulate expensive operation
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                    return "Expensive Result";
                });
                System.out.println("Got: " + result);
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        
        // "Computing expensive value..." printed only once!
    }
}
```

### CopyOnWriteArrayList

Best for scenarios with many reads and few writes. Every modification creates a new copy of the underlying array.

#### Characteristics

- **Thread-safe** without external synchronization
- **No ConcurrentModificationException** during iteration
- **High memory cost** for modifications
- **Weakly consistent iterators** (see snapshot at time of creation)

#### Basic Usage

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteDemo {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        
        list.add("A");
        list.add("B");
        list.add("C");
        
        // Iterator sees snapshot - modifications don't affect it
        for (String item : list) {
            System.out.println(item);
            list.add("D");  // No ConcurrentModificationException!
        }
        
        System.out.println("Final list: " + list);
        // Output: [A, B, C, D, D, D] - D added 3 times during iteration
    }
}
```

#### When to Use

```java
public class EventListenerRegistry {
    // Perfect use case: many reads (firing events), few writes (adding listeners)
    private final CopyOnWriteArrayList<EventListener> listeners = 
        new CopyOnWriteArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(listener);  // Infrequent write
    }
    
    public void removeListener(EventListener listener) {
        listeners.remove(listener);  // Infrequent write
    }
    
    public void fireEvent(Event event) {
        // Frequent reads - no locking needed!
        for (EventListener listener : listeners) {
            listener.onEvent(event);
        }
    }
    
    interface EventListener {
        void onEvent(Event event);
    }
    
    static class Event {
        private final String message;
        Event(String message) { this.message = message; }
        public String getMessage() { return message; }
    }
}
```

#### Performance Comparison

```java
public class ListPerformanceTest {
    private static final int THREADS = 10;
    private static final int OPERATIONS = 10000;
    
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Testing synchronized ArrayList:");
        testList(Collections.synchronizedList(new ArrayList<>()));
        
        System.out.println("\nTesting CopyOnWriteArrayList:");
        testList(new CopyOnWriteArrayList<>());
    }
    
    private static void testList(List<String> list) throws InterruptedException {
        long start = System.currentTimeMillis();
        
        Thread[] threads = new Thread[THREADS];
        for (int i = 0; i < THREADS; i++) {
            final int threadId = i;
            threads[i] = new Thread(() -> {
                // 90% reads, 10% writes (typical read-heavy scenario)
                for (int j = 0; j < OPERATIONS; j++) {
                    if (j % 10 == 0) {
                        list.add("Item-" + threadId + "-" + j);
                    } else {
                        if (!list.isEmpty()) {
                            list.get(0);
                        }
                    }
                }
            });
            threads[i].start();
        }
        
        for (Thread thread : threads) {
            thread.join();
        }
        
        long end = System.currentTimeMillis();
        System.out.println("Time: " + (end - start) + "ms");
        System.out.println("Final size: " + list.size());
    }
}
```

### BlockingQueue

A queue that supports operations that wait for the queue to become non-empty when retrieving, and wait for space to become available when storing.

#### Common Implementations

| Implementation | Characteristics |
|---------------|-----------------|
| `ArrayBlockingQueue` | Bounded, array-backed, FIFO |
| `LinkedBlockingQueue` | Optionally bounded, linked nodes, FIFO |
| `PriorityBlockingQueue` | Unbounded, priority heap |
| `SynchronousQueue` | No capacity, direct handoff |
| `DelayQueue` | Unbounded, elements available after delay |

#### Basic Operations

```java
import java.util.concurrent.*;

public class BlockingQueueBasics {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);
        
        // Blocking operations
        queue.put("item1");     // Blocks if queue is full
        String item = queue.take();  // Blocks if queue is empty
        
        // Timed blocking operations
        boolean added = queue.offer("item2", 5, TimeUnit.SECONDS);
        String polled = queue.poll(5, TimeUnit.SECONDS);
        
        // Non-blocking operations (return immediately)
        queue.offer("item3");   // Returns false if full
        queue.poll();           // Returns null if empty
    }
}
```

#### Producer-Consumer Pattern

```java
public class ProducerConsumerExample {
    private static final int QUEUE_CAPACITY = 5;
    
    static class Producer implements Runnable {
        private final BlockingQueue<Integer> queue;
        private final int id;
        
        Producer(BlockingQueue<Integer> queue, int id) {
            this.queue = queue;
            this.id = id;
        }
        
        @Override
        public void run() {
            try {
                for (int i = 0; i < 10; i++) {
                    int item = id * 100 + i;
                    System.out.println("Producer " + id + " producing: " + item);
                    queue.put(item);  // Blocks if queue is full
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    static class Consumer implements Runnable {
        private final BlockingQueue<Integer> queue;
        private final int id;
        
        Consumer(BlockingQueue<Integer> queue, int id) {
            this.queue = queue;
            this.id = id;
        }
        
        @Override
        public void run() {
            try {
                while (true) {
                    Integer item = queue.take();  // Blocks if queue is empty
                    System.out.println("Consumer " + id + " consumed: " + item);
                    Thread.sleep(200);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(QUEUE_CAPACITY);
        
        // Start 2 producers
        Thread producer1 = new Thread(new Producer(queue, 1));
        Thread producer2 = new Thread(new Producer(queue, 2));
        
        // Start 3 consumers
        Thread consumer1 = new Thread(new Consumer(queue, 1));
        Thread consumer2 = new Thread(new Consumer(queue, 2));
        Thread consumer3 = new Thread(new Consumer(queue, 3));
        
        producer1.start();
        producer2.start();
        consumer1.start();
        consumer2.start();
        consumer3.start();
        
        Thread.sleep(5000);
        System.out.println("Shutting down...");
        System.exit(0);
    }
}
```

#### Real-World Example: Task Processing System

```java
public class TaskProcessor {
    private final BlockingQueue<Task> taskQueue;
    private final ExecutorService executorService;
    private final int numWorkers;
    
    public TaskProcessor(int queueCapacity, int numWorkers) {
        this.taskQueue = new LinkedBlockingQueue<>(queueCapacity);
        this.numWorkers = numWorkers;
        this.executorService = Executors.newFixedThreadPool(numWorkers);
    }
    
    public void start() {
        for (int i = 0; i < numWorkers; i++) {
            executorService.submit(new Worker(i));
        }
    }
    
    public boolean submitTask(Task task, long timeout, TimeUnit unit) 
            throws InterruptedException {
        return taskQueue.offer(task, timeout, unit);
    }
    
    public void shutdown() {
        executorService.shutdown();
    }
    
    private class Worker implements Runnable {
        private final int id;
        
        Worker(int id) {
            this.id = id;
        }
        
        @Override
        public void run() {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    Task task = taskQueue.take();
                    System.out.println("Worker " + id + " processing: " + 
                                     task.getName());
                    task.execute();
                    System.out.println("Worker " + id + " completed: " + 
                                     task.getName());
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    static class Task {
        private final String name;
        private final Runnable action;
        
        Task(String name, Runnable action) {
            this.name = name;
            this.action = action;
        }
        
        public String getName() { return name; }
        
        public void execute() {
            action.run();
        }
    }
    
    // Usage
    public static void main(String[] args) throws InterruptedException {
        TaskProcessor processor = new TaskProcessor(100, 5);
        processor.start();
        
        // Submit tasks
        for (int i = 0; i < 20; i++) {
            final int taskId = i;
            Task task = new Task("Task-" + taskId, () -> {
                try {
                    Thread.sleep(1000);  // Simulate work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
            
            processor.submitTask(task, 5, TimeUnit.SECONDS);
        }
        
        Thread.sleep(25000);
        processor.shutdown();
    }
}
```

#### PriorityBlockingQueue Example

```java
public class PriorityTaskQueue {
    static class PriorityTask implements Comparable<PriorityTask> {
        private final String name;
        private final int priority;  // Higher number = higher priority
        
        PriorityTask(String name, int priority) {
            this.name = name;
            this.priority = priority;
        }
        
        @Override
        public int compareTo(PriorityTask other) {
            return Integer.compare(other.priority, this.priority);  // Reverse for max-heap
        }
        
        @Override
        public String toString() {
            return name + " (priority: " + priority + ")";
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<PriorityTask> queue = new PriorityBlockingQueue<>();
        
        // Add tasks with different priorities
        queue.put(new PriorityTask("Low priority task", 1));
        queue.put(new PriorityTask("High priority task", 10));
        queue.put(new PriorityTask("Medium priority task", 5));
        queue.put(new PriorityTask("Critical task", 20));
        
        // Tasks are retrieved by priority
        while (!queue.isEmpty()) {
            System.out.println("Processing: " + queue.take());
        }
        // Output:
        // Processing: Critical task (priority: 20)
        // Processing: High priority task (priority: 10)
        // Processing: Medium priority task (priority: 5)
        // Processing: Low priority task (priority: 1)
    }
}
```

### Summary Table: Concurrent Collections

| Collection | Thread-Safe? | Blocking? | Best For |
|-----------|--------------|-----------|----------|
| `ConcurrentHashMap` | ✅ | ❌ | High-concurrency maps |
| `CopyOnWriteArrayList` | ✅ | ❌ | Read-heavy lists |
| `CopyOnWriteArraySet` | ✅ | ❌ | Read-heavy sets |
| `ConcurrentLinkedQueue` | ✅ | ❌ | Non-blocking queue |
| `ArrayBlockingQueue` | ✅ | ✅ | Bounded producer-consumer |
| `LinkedBlockingQueue` | ✅ | ✅ | Unbounded producer-consumer |
| `PriorityBlockingQueue` | ✅ | ✅ | Priority-based processing |
| `SynchronousQueue` | ✅ | ✅ | Direct thread handoff |
| `Collections.synchronized*` | ✅ | ❌ | Legacy (poor performance) |

---

## Practice Exercises

### Exercise 1: Thread-Safe Counter (Easy)

Implement a thread-safe counter with the following methods:
- `increment()`: Increase count by 1
- `decrement()`: Decrease count by 1
- `getCount()`: Get current count

**Requirements:**
1. Implement using `synchronized`
2. Implement using `ReentrantLock`
3. Implement using `AtomicInteger`
4. Write a test that uses 10 threads, each incrementing 1000 times

<details>
<summary>Solution</summary>

```java
// Solution 1: synchronized
class SynchronizedCounter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized void decrement() {
        count--;
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// Solution 2: ReentrantLock
class LockCounter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
    
    public void decrement() {
        lock.lock();
        try {
            count--;
        } finally {
            lock.unlock();
        }
    }
    
    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}

// Solution 3: AtomicInteger
class AtomicCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public void decrement() {
        count.decrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}

// Test
class CounterTest {
    public static void main(String[] args) throws InterruptedException {
        testCounter(new SynchronizedCounter());
        testCounter(new LockCounter());
        testCounter(new AtomicCounter());
    }
    
    private static void testCounter(Object counter) throws InterruptedException {
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    if (counter instanceof SynchronizedCounter c) c.increment();
                    else if (counter instanceof LockCounter c) c.increment();
                    else if (counter instanceof AtomicCounter c) c.increment();
                }
            });
            threads[i].start();
        }
        
        for (Thread t : threads) t.join();
        
        int finalCount = 0;
        if (counter instanceof SynchronizedCounter c) finalCount = c.getCount();
        else if (counter instanceof LockCounter c) finalCount = c.getCount();
        else if (counter instanceof AtomicCounter c) finalCount = c.getCount();
        
        System.out.println(counter.getClass().getSimpleName() + 
                         " final count: " + finalCount);
    }
}
```
</details>

### Exercise 2: Thread-Safe Singleton (Medium)

Implement a thread-safe singleton pattern with lazy initialization.

**Requirements:**
1. Implement using double-checked locking
2. Implement using initialization-on-demand holder
3. Test with multiple threads trying to get the instance simultaneously

<details>
<summary>Solution</summary>

```java
// Solution 1: Double-Checked Locking
class DoubleCheckedSingleton {
    private static volatile DoubleCheckedSingleton instance;
    private final long creationTime;
    
    private DoubleCheckedSingleton() {
        creationTime = System.currentTimeMillis();
        System.out.println("Creating instance at " + creationTime);
    }
    
    public static DoubleCheckedSingleton getInstance() {
        if (instance == null) {
            synchronized (DoubleCheckedSingleton.class) {
                if (instance == null) {
                    instance = new DoubleCheckedSingleton();
                }
            }
        }
        return instance;
    }
    
    public long getCreationTime() { return creationTime; }
}

// Solution 2: Initialization-on-Demand Holder
class HolderSingleton {
    private final long creationTime;
    
    private HolderSingleton() {
        creationTime = System.currentTimeMillis();
        System.out.println("Creating instance at " + creationTime);
    }
    
    private static class Holder {
        private static final HolderSingleton INSTANCE = new HolderSingleton();
    }
    
    public static HolderSingleton getInstance() {
        return Holder.INSTANCE;
    }
    
    public long getCreationTime() { return creationTime; }
}

// Test
class SingletonTest {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Testing DoubleCheckedSingleton:");
        testSingleton(() -> DoubleCheckedSingleton.getInstance().getCreationTime());
        
        System.out.println("\nTesting HolderSingleton:");
        testSingleton(() -> HolderSingleton.getInstance().getCreationTime());
    }
    
    private static void testSingleton(Supplier<Long> getter) 
            throws InterruptedException {
        Set<Long> creationTimes = ConcurrentHashMap.newKeySet();
        
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                creationTimes.add(getter.get());
            });
            threads[i].start();
        }
        
        for (Thread t : threads) t.join();
        
        System.out.println("Number of instances created: " + creationTimes.size());
        // Should always be 1
    }
}
```
</details>

### Exercise 3: Bank Account Transfer (Hard)

Implement a thread-safe bank account transfer system that prevents deadlocks.

**Requirements:**
1. Multiple accounts can transfer money concurrently
2. Prevent deadlocks using lock ordering
3. Ensure account balances never go negative
4. Simulate 100 random transfers between 10 accounts

<details>
<summary>Solution</summary>

```java
import java.util.*;
import java.util.concurrent.*;

class BankAccount {
    private final long id;
    private double balance;
    private final Lock lock = new ReentrantLock();
    
    public BankAccount(long id, double initialBalance) {
        this.id = id;
        this.balance = initialBalance;
    }
    
    public long getId() { return id; }
    
    public double getBalance() {
        lock.lock();
        try {
            return balance;
        } finally {
            lock.unlock();
        }
    }
    
    // Thread-safe transfer with deadlock prevention
    public static boolean transfer(BankAccount from, BankAccount to, double amount) {
        if (from.id == to.id) return false;
        
        // Lock ordering prevents deadlock
        BankAccount first = from.id < to.id ? from : to;
        BankAccount second = from.id < to.id ? to : from;
        
        first.lock.lock();
        try {
            second.lock.lock();
            try {
                if (from.balance >= amount) {
                    from.balance -= amount;
                    to.balance += amount;
                    System.out.printf("Transferred %.2f from Account %d to Account %d%n",
                        amount, from.id, to.id);
                    return true;
                }
                return false;
            } finally {
                second.lock.unlock();
            }
        } finally {
            first.lock.unlock();
        }
    }
}

class BankTransferTest {
    public static void main(String[] args) throws InterruptedException {
        // Create 10 accounts with $1000 each
        List<BankAccount> accounts = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            accounts.add(new BankAccount(i, 1000));
        }
        
        double initialTotal = accounts.stream()
            .mapToDouble(BankAccount::getBalance)
            .sum();
        
        System.out.println("Initial total: $" + initialTotal);
        
        // Perform 100 random transfers
        ExecutorService executor = Executors.newFixedThreadPool(5);
        Random random = new Random();
        
        for (int i = 0; i < 100; i++) {
            executor.submit(() -> {
                int fromIdx = random.nextInt(10);
                int toIdx = random.nextInt(10);
                double amount = random.nextDouble() * 100;
                
                BankAccount.transfer(
                    accounts.get(fromIdx),
                    accounts.get(toIdx),
                    amount
                );
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.MINUTES);
        
        // Verify total is unchanged
        double finalTotal = accounts.stream()
            .mapToDouble(BankAccount::getBalance)
            .sum();
        
        System.out.println("\nFinal total: $" + finalTotal);
        System.out.println("Money preserved: " + 
            (Math.abs(initialTotal - finalTotal) < 0.01));
        
        // Print final balances
        for (BankAccount account : accounts) {
            System.out.printf("Account %d: $%.2f%n",
                account.getId(), account.getBalance());
        }
    }
}
```
</details>

### Exercise 4: Thread-Safe LRU Cache (Hard)

Implement a thread-safe Least Recently Used (LRU) cache.

**Requirements:**
1. Fixed capacity
2. Thread-safe get and put operations
3. O(1) time complexity for both operations
4. Evict least recently used item when capacity is exceeded

<details>
<summary>Solution</summary>

```java
import java.util.concurrent.locks.*;

class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, Node<K, V>> map;
    private final Node<K, V> head;
    private final Node<K, V> tail;
    private final Lock lock = new ReentrantLock();
    
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev;
        Node<K, V> next;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.head = new Node<>(null, null);
        this.tail = new Node<>(null, null);
        head.next = tail;
        tail.prev = head;
    }
    
    public V get(K key) {
        lock.lock();
        try {
            Node<K, V> node = map.get(key);
            if (node == null) {
                return null;
            }
            moveToHead(node);
            return node.value;
        } finally {
            lock.unlock();
        }
    }
    
    public void put(K key, V value) {
        lock.lock();
        try {
            Node<K, V> node = map.get(key);
            
            if (node != null) {
                node.value = value;
                moveToHead(node);
            } else {
                Node<K, V> newNode = new Node<>(key, value);
                map.put(key, newNode);
                addToHead(newNode);
                
                if (map.size() > capacity) {
                    Node<K, V> removed = removeTail();
                    map.remove(removed.key);
                    System.out.println("Evicted: " + removed.key);
                }
            }
        } finally {
            lock.unlock();
        }
    }
    
    private void addToHead(Node<K, V> node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
    
    private void removeNode(Node<K, V> node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private void moveToHead(Node<K, V> node) {
        removeNode(node);
        addToHead(node);
    }
    
    private Node<K, V> removeTail() {
        Node<K, V> node = tail.prev;
        removeNode(node);
        return node;
    }
    
    public int size() {
        lock.lock();
        try {
            return map.size();
        } finally {
            lock.unlock();
        }
    }
}

// Test
class LRUCacheTest {
    public static void main(String[] args) throws InterruptedException {
        LRUCache<Integer, String> cache = new LRUCache<>(3);
        
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Thread 1: Put items
        executor.submit(() -> {
            cache.put(1, "One");
            cache.put(2, "Two");
            cache.put(3, "Three");
        });
        
        // Thread 2: Access item 1
        executor.submit(() -> {
            try {
                Thread.sleep(50);
                System.out.println("Get 1: " + cache.get(1));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Thread 3: Put item 4 (should evict 2)
        executor.submit(() -> {
            try {
                Thread.sleep(100);
                cache.put(4, "Four");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.SECONDS);
        
        System.out.println("Final cache size: " + cache.size());
    }
}
```
</details>

### Exercise 5: Rate Limiter (Hard)

Implement a thread-safe token bucket rate limiter.

**Requirements:**
1. Allow N requests per second
2. Thread-safe
3. Blocking `acquire()` method
4. Non-blocking `tryAcquire()` method

<details>
<summary>Solution</summary>

```java
import java.util.concurrent.locks.*;

class RateLimiter {
    private final int capacity;
    private final int refillRate;  // tokens per second
    private int availableTokens;
    private long lastRefillTime;
    private final Lock lock = new ReentrantLock();
    private final Condition tokensAvailable = lock.newCondition();
    
    public RateLimiter(int capacity, int refillRate) {
        this.capacity = capacity;
        this.refillRate = refillRate;
        this.availableTokens = capacity;
        this.lastRefillTime = System.nanoTime();
    }
    
    public void acquire() throws InterruptedException {
        lock.lock();
        try {
            while (true) {
                refill();
                if (availableTokens > 0) {
                    availableTokens--;
                    return;
                }
                tokensAvailable.await();
            }
        } finally {
            lock.unlock();
        }
    }
    
    public boolean tryAcquire() {
        lock.lock();
        try {
            refill();
            if (availableTokens > 0) {
                availableTokens--;
                return true;
            }
            return false;
        } finally {
            lock.unlock();
        }
    }
    
    private void refill() {
        long now = System.nanoTime();
        long elapsedNanos = now - lastRefillTime;
        int tokensToAdd = (int) (elapsedNanos * refillRate / 1_000_000_000L);
        
        if (tokensToAdd > 0) {
            availableTokens = Math.min(capacity, availableTokens + tokensToAdd);
            lastRefillTime = now;
            tokensAvailable.signalAll();
        }
    }
}

// Test
class RateLimiterTest {
    public static void main(String[] args) throws InterruptedException {
        RateLimiter limiter = new RateLimiter(5, 2);  // 5 capacity, 2 per second
        
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        for (int i = 0; i < 20; i++) {
            final int requestId = i;
            executor.submit(() -> {
                try {
                    limiter.acquire();
                    System.out.println("Request " + requestId + " at " +
                        System.currentTimeMillis());
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(15, TimeUnit.SECONDS);
    }
}
```
</details>

---

## Interview Questions & Answers

### Theoretical Questions

**Q1: What is thread safety? Why is it important?**

**Answer:** Thread safety means that a piece of code functions correctly when accessed from multiple threads simultaneously. It's important because:

1. **Data Integrity:** Prevents data corruption from concurrent modifications
2. **Predictable Behavior:** Ensures consistent results regardless of thread scheduling
3. **No Race Conditions:** Eliminates timing-dependent bugs

A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of those threads, with no additional synchronization in the calling code.

**Q2: Explain the difference between `synchronized` and `ReentrantLock`.**

**Answer:**

| Aspect | synchronized | ReentrantLock |
|--------|-------------|---------------|
| **Type** | Language keyword | Library class |
| **Lock acquisition** | Implicit, automatic | Explicit lock()/unlock() |
| **Try-lock** | No | Yes, tryLock() |
| **Timeout** | No | Yes, tryLock(timeout) |
| **Interruptible** | No | Yes, lockInterruptibly() |
| **Fairness** | No | Optional fair mode |
| **Condition variables** | One (wait/notify) | Multiple (Condition) |
| **Lock release** | Automatic | Manual (must use finally) |

Use `synchronized` for simple cases. Use `ReentrantLock` when you need advanced features like try-lock, timeouts, or multiple conditions.

**Q3: What is a deadlock? How can you prevent it?**

**Answer:** A deadlock is a situation where two or more threads are blocked forever, each waiting for the other to release a resource.

**Four Coffman Conditions for Deadlock:**
1. Mutual Exclusion
2. Hold and Wait
3. No Preemption
4. Circular Wait

**Prevention Strategies:**
1. **Lock Ordering:** Always acquire locks in a consistent global order
2. **Lock Timeout:** Use `tryLock()` with timeout
3. **Single Lock:** Use one lock instead of multiple
4. **Lock-Free Algorithms:** Use atomic variables where possible

Example of prevention:
```java
// Always lock accounts in order of ID
BankAccount first = account1.id < account2.id ? account1 : account2;
BankAccount second = account1.id < account2.id ? account2 : account1;

synchronized(first) {
    synchronized(second) {
        // Transfer logic
    }
}
```

**Q4: What is a race condition? Give an example.**

**Answer:** A race condition occurs when the correctness of a program depends on the relative timing or interleaving of multiple threads.

**Example: Check-Then-Act**
```java
// NOT thread-safe
if (map.containsKey(key)) {  // Check
    value = map.get(key);     // Act
}
```

Problem: Between the check and the act, another thread might remove the key.

**Solution:**
```java
// Thread-safe
value = map.get(key);
if (value != null) {
    // Use value
}

// Or use compute methods
value = map.computeIfAbsent(key, k -> createValue());
```

**Q5: Explain volatile keyword.**

**Answer:** `volatile` is a keyword that ensures:

1. **Visibility:** Changes made by one thread are immediately visible to other threads
2. **No Reordering:** Prevents compiler/CPU from reordering operations around volatile reads/writes
3. **Atomicity:** Only for reads and writes (not for compound operations like `++`)

**When to use:**
```java
class TaskRunner {
    private volatile boolean running = true;  // ✅ Correct
    
    public void stop() {
        running = false;  // Immediately visible to other threads
    }
    
    public void run() {
        while (running) {
            // Do work
        }
    }
}
```

**When NOT to use:**
```java
private volatile int count = 0;

public void increment() {
    count++;  // ❌ NOT atomic! Still needs synchronization
}
```

**Q6: What is `ConcurrentHashMap`? How does it differ from `Hashtable`?**

**Answer:**

| Feature | Hashtable | ConcurrentHashMap |
|---------|-----------|-------------------|
| **Locking** | Entire table | Fine-grained (segments/buckets) |
| **Performance** | Poor under concurrency | Excellent under concurrency |
| **Null keys/values** | Not allowed | Not allowed |
| **Iterators** | Fail-fast | Weakly consistent |
| **Atomic operations** | No | Yes (compute, merge, etc.) |

**ConcurrentHashMap** allows concurrent reads and writes by:
- Dividing the map into segments/buckets
- Locking only the relevant segment during writes
- Allowing lock-free reads

**Example:**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Atomic operations
map.computeIfAbsent("key", k -> expensiveComputation());
map.merge("count", 1, Integer::sum);

// Multiple threads can read/write simultaneously without blocking
```

**Q7: When would you use `CopyOnWriteArrayList`?**

**Answer:** Use `CopyOnWriteArrayList` when:

1. **Many reads, few writes** (read-to-write ratio > 100:1)
2. **No ConcurrentModificationException** needed during iteration
3. **Memory overhead is acceptable**

**Perfect use cases:**
- Event listener lists
- Observer pattern implementations
- Configuration data that rarely changes
- Blacklists/whitelists

**Example:**
```java
CopyOnWriteArrayList<EventListener> listeners = new CopyOnWriteArrayList<>();

// Write (infrequent) - creates new copy
listeners.add(newListener);

// Read (frequent) - no locking!
for (EventListener listener : listeners) {
    listener.onEvent(event);  // Fast, no locks
}
```

**NOT suitable when:**
- Many writes
- Large lists
- Memory-constrained environments

### Coding Questions

**Q8: Implement a thread-safe bounded buffer.**

```java
import java.util.concurrent.locks.*;

class BoundedBuffer<T> {
    private final Object[] buffer;
    private int count, putIndex, takeIndex;
    private final Lock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
    
    public BoundedBuffer(int capacity) {
        buffer = new Object[capacity];
    }
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (count == buffer.length) {
                notFull.await();
            }
            buffer[putIndex] = item;
            putIndex = (putIndex + 1) % buffer.length;
            count++;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    
    @SuppressWarnings("unchecked")
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.await();
            }
            T item = (T) buffer[takeIndex];
            buffer[takeIndex] = null;
            takeIndex = (takeIndex + 1) % buffer.length;
            count--;
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

**Q9: How would you implement a thread-safe singleton?**

```java
// Best approach: Initialization-on-Demand Holder
public class Singleton {
    private Singleton() {}
    
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

// Alternative: Double-Checked Locking
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Q10: Implement a thread-safe object pool.**

```java
import java.util.concurrent.*;

class ObjectPool<T> {
    private final BlockingQueue<T> pool;
    private final Factory<T> factory;
    private final int maxSize;
    
    interface Factory<T> {
        T create();
    }
    
    public ObjectPool(int size, Factory<T> factory) {
        this.maxSize = size;
        this.factory = factory;
        this.pool = new LinkedBlockingQueue<>(size);
        
        // Pre-populate pool
        for (int i = 0; i < size; i++) {
            pool.offer(factory.create());
        }
    }
    
    public T acquire() throws InterruptedException {
        return pool.take();
    }
    
    public boolean acquire(long timeout, TimeUnit unit) throws InterruptedException {
        T obj = pool.poll(timeout, unit);
        return obj != null;
    }
    
    public void release(T object) {
        if (object != null) {
            pool.offer(object);
        }
    }
}
```

---

## Real-World Scenarios

### Scenario 1: Web Server Request Processing

**Problem:** Design a thread-safe request processor for a web server that:
1. Limits concurrent requests to 100
2. Queues additional requests (up to 1000)
3. Tracks active request count
4. Provides statistics (total processed, rejected, etc.)

**Solution:**

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;

public class WebServerRequestProcessor {
    private final Semaphore activeSemaphore;
    private final BlockingQueue<Request> requestQueue;
    private final ExecutorService executorService;
    
    private final AtomicLong totalProcessed = new AtomicLong(0);
    private final AtomicLong totalRejected = new AtomicLong(0);
    private final AtomicInteger activeRequests = new AtomicInteger(0);
    
    public WebServerRequestProcessor(int maxConcurrent, int queueSize, int threads) {
        this.activeSemaphore = new Semaphore(maxConcurrent);
        this.requestQueue = new LinkedBlockingQueue<>(queueSize);
        this.executorService = Executors.newFixedThreadPool(threads);
        
        // Start worker threads
        for (int i = 0; i < threads; i++) {
            executorService.submit(new Worker());
        }
    }
    
    public boolean submitRequest(Request request) {
        if (requestQueue.offer(request)) {
            return true;
        }
        totalRejected.incrementAndGet();
        return false;
    }
    
    private class Worker implements Runnable {
        @Override
        public void run() {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    Request request = requestQueue.take();
                    processRequest(request);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        private void processRequest(Request request) {
            try {
                activeSemaphore.acquire();
                activeRequests.incrementAndGet();
                
                System.out.println("Processing: " + request.getId());
                request.process();
                totalProcessed.incrementAndGet();
                
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                activeRequests.decrementAndGet();
                activeSemaphore.release();
            }
        }
    }
    
    public Statistics getStatistics() {
        return new Statistics(
            totalProcessed.get(),
            totalRejected.get(),
            activeRequests.get(),
            requestQueue.size()
        );
    }
    
    public void shutdown() {
        executorService.shutdown();
    }
    
    record Statistics(long processed, long rejected, int active, int queued) {}
    
    static class Request {
        private final String id;
        
        Request(String id) { this.id = id; }
        
        public String getId() { return id; }
        
        public void process() {
            try {
                Thread.sleep(100);  // Simulate work
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

### Scenario 2: Distributed Cache with Write-Through

**Problem:** Implement a distributed cache that:
1. Supports concurrent reads/writes
2. Implements write-through to database
3. Handles cache invalidation
4. Provides statistics

**Solution:**

```java
import java.util.concurrent.*;

public class DistributedCache<K, V> {
    private final ConcurrentHashMap<K, CacheEntry<V>> cache = new ConcurrentHashMap<>();
    private final Database<K, V> database;
    private final long ttlMillis;
    private final ExecutorService writeExecutor = Executors.newFixedThreadPool(5);
    
    private final AtomicLong hits = new AtomicLong(0);
    private final AtomicLong misses = new AtomicLong(0);
    
    public DistributedCache(Database<K, V> database, long ttlMillis) {
        this.database = database;
        this.ttlMillis = ttlMillis;
        
        // Start background eviction thread
        ScheduledExecutorService evictor = Executors.newSingleThreadScheduledExecutor();
        evictor.scheduleAtFixedRate(this::evictExpired, 60, 60, TimeUnit.SECONDS);
    }
    
    public V get(K key) {
        CacheEntry<V> entry = cache.get(key);
        
        if (entry != null && !entry.isExpired()) {
            hits.incrementAndGet();
            return entry.value;
        }
        
        misses.incrementAndGet();
        
        // Cache miss - load from database
        V value = database.load(key);
        if (value != null) {
            cache.put(key, new CacheEntry<>(value, System.currentTimeMillis() + ttlMillis));
        }
        return value;
    }
    
    public void put(K key, V value) {
        // Write-through: update cache immediately
        cache.put(key, new CacheEntry<>(value, System.currentTimeMillis() + ttlMillis));
        
        // Asynchronously write to database
        writeExecutor.submit(() -> database.save(key, value));
    }
    
    public void invalidate(K key) {
        cache.remove(key);
    }
    
    private void evictExpired() {
        long now = System.currentTimeMillis();
        cache.entrySet().removeIf(entry -> entry.getValue().isExpired(now));
    }
    
    public CacheStats getStats() {
        long totalRequests = hits.get() + misses.get();
        double hitRate = totalRequests > 0 ? 
            (double) hits.get() / totalRequests : 0.0;
        
        return new CacheStats(
            cache.size(),
            hits.get(),
            misses.get(),
            hitRate
        );
    }
    
    private static class CacheEntry<V> {
        final V value;
        final long expirationTime;
        
        CacheEntry(V value, long expirationTime) {
            this.value = value;
            this.expirationTime = expirationTime;
        }
        
        boolean isExpired() {
            return isExpired(System.currentTimeMillis());
        }
        
        boolean isExpired(long currentTime) {
            return currentTime > expirationTime;
        }
    }
    
    record CacheStats(int size, long hits, long misses, double hitRate) {}
    
    interface Database<K, V> {
        V load(K key);
        void save(K key, V value);
    }
}
```

---

## Summary & Best Practices

### Key Takeaways

1. **Thread Safety First:**
   - Always consider concurrency when designing shared mutable state
   - Use immutable objects when possible
   - Minimize shared mutable state

2. **Choose the Right Tool:**
   - `synchronized`: Simple, built-in protection
   - `ReentrantLock`: Advanced features (try-lock, timeouts)
   - Atomic classes: Lock-free single variables
   - Concurrent collections: Designed for high concurrency

3. **Prevent Deadlocks:**
   - Use consistent lock ordering
   - Use timeouts with `tryLock()`
   - Keep critical sections small
   - Avoid nested locks when possible

4. **Use Concurrent Collections:**
   - `ConcurrentHashMap` for maps
   - `CopyOnWriteArrayList` for read-heavy lists
   - `BlockingQueue` for producer-consumer

5. **Performance:**
   - Minimize lock contention
   - Use fine-grained locking
   - Prefer concurrent collections over synchronized collections
   - Use lock-free algorithms (atomics) when possible

### Common Pitfalls

❌ **Don't:**
- Forget to unlock in `finally` block with `ReentrantLock`
- Use `synchronized` on non-final objects
- Hold locks while doing I/O or expensive operations
- Assume `volatile` makes compound operations atomic
- Use `wait()`/`notify()` without synchronized

✅ **Do:**
- Always unlock in `finally` block
- Keep critical sections short
- Use higher-level concurrency utilities
- Test concurrent code thoroughly
- Use immutable objects when possible

---

**Congratulations!** You've completed Week 6. You now understand:
- Thread safety fundamentals
- Synchronization mechanisms (`synchronized` vs `ReentrantLock`)
- Race conditions and deadlocks (detection and prevention)
- Concurrent collections (`ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue`)
- Real-world concurrent programming patterns

**Next:** Week 7 - Advanced Architecture, IO & Modules
