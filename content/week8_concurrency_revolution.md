# Week 8: The Concurrency Revolution (Project Loom)

## Table of Contents
1. [Introduction to Concurrency in Java](#introduction-to-concurrency-in-java)
2. [Traditional Threading: Platform Threads](#traditional-threading-platform-threads)
3. [Virtual Threads: The Game Changer](#virtual-threads-the-game-changer)
4. [Structured Concurrency (JEP 480)](#structured-concurrency-jep-480)
5. [Scoped Values (JEP 481)](#scoped-values-jep-481)
6. [Modern Networking with HttpClient](#modern-networking-with-httpclient)
7. [Practice Exercises](#practice-exercises)
8. [Interview Questions & Answers](#interview-questions--answers)

---

## Introduction to Concurrency in Java

### What is Concurrency?

**Concurrency** is the ability to execute multiple tasks **seemingly** at the same time. It's about **dealing with lots of things at once** (not necessarily doing them all simultaneously).

**Parallelism** is about **executing** multiple tasks **actually** at the same time, using multiple CPU cores.

```
Concurrency = Structure of your program (interleaving tasks)
Parallelism = Actual simultaneous execution (hardware-dependent)
```

### Why Do We Need Concurrency?

Modern applications need to:
- Handle multiple user requests simultaneously (web servers)
- Perform I/O operations without blocking (database queries, API calls)
- Improve responsiveness (UI applications)
- Utilize multi-core processors efficiently

### The Traditional Concurrency Model in Java

Before Project Loom, Java had **platform threads** (also called OS threads or kernel threads):

```
User Request → Platform Thread (1:1 mapping) → OS Thread → CPU Core
```

**Problem:** Platform threads are **expensive**:
- Each thread consumes **~2MB of memory** (stack space)
- Thread creation/destruction is **costly** (~1ms)
- Limited by OS resources (usually thousands, not millions)

---

## Traditional Threading: Platform Threads

### Creating Platform Threads (The Old Way)

#### Method 1: Extending Thread Class

```java
class MyTask extends Thread {
    private String taskName;
    
    public MyTask(String name) {
        this.taskName = name;
    }
    
    @Override
    public void run() {
        println("Task " + taskName + " started by: " + Thread.currentThread().getName());
        
        // Simulate work
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        println("Task " + taskName + " completed");
    }
}

// Usage
void main() {
    MyTask task1 = new MyTask("A");
    MyTask task2 = new MyTask("B");
    
    task1.start();  // Don't call run() directly!
    task2.start();
}
```

**Output (order may vary):**
```
Task A started by: Thread-0
Task B started by: Thread-1
Task A completed
Task B completed
```

#### Method 2: Implementing Runnable (Better)

```java
class MyTask implements Runnable {
    private String taskName;
    
    public MyTask(String name) {
        this.taskName = name;
    }
    
    @Override
    public void run() {
        println("Task " + taskName + " running");
    }
}

void main() {
    Thread t1 = new Thread(new MyTask("A"));
    Thread t2 = new Thread(new MyTask("B"));
    
    t1.start();
    t2.start();
}
```

**Why Runnable is better:**
- Separation of task logic from threading mechanism
- Can implement multiple interfaces (Java single inheritance limitation)
- Can be passed to thread pools

#### Method 3: Lambda Expressions (Modern)

```java
void main() {
    Thread t1 = new Thread(() -> {
        println("Task from lambda");
    });
    
    t1.start();
}
```

### Thread Lifecycle

```
NEW → RUNNABLE → RUNNING → TERMINATED
         ↓           ↓
      BLOCKED    WAITING/TIMED_WAITING
```

**States:**
1. **NEW**: Thread created but not started
2. **RUNNABLE**: Ready to run, waiting for CPU
3. **RUNNING**: Executing on CPU
4. **BLOCKED**: Waiting to acquire a lock
5. **WAITING**: Waiting indefinitely for another thread
6. **TIMED_WAITING**: Waiting for a specified time
7. **TERMINATED**: Execution completed

### Platform Thread Limitations

```java
void main() {
    // Try creating 100,000 platform threads (DON'T RUN THIS!)
    for (int i = 0; i < 100_000; i++) {
        Thread t = new Thread(() -> {
            try {
                Thread.sleep(60000); // Sleep for 1 minute
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        t.start();
    }
    // This will likely crash with OutOfMemoryError
}
```

**Why it fails:**
- Each thread needs ~2MB stack → 100K threads = 200GB memory!
- OS thread limit (typically 4K-32K threads)

---

## Virtual Threads: The Game Changer

### What are Virtual Threads?

**Virtual Threads** (introduced in Java 21 via Project Loom) are **lightweight threads** managed by the JVM, not the OS.

```
Many Virtual Threads → Few Platform Threads (Carrier Threads) → OS Threads → CPU Cores
```

**Key Characteristics:**
- Extremely lightweight (~1KB memory per thread)
- Cheap to create and destroy
- Can have **millions** of virtual threads
- Automatically managed by JVM

### Creating Virtual Threads

#### Method 1: Thread.startVirtualThread()

```java
void main() {
    Thread vThread = Thread.startVirtualThread(() -> {
        println("Hello from virtual thread: " + Thread.currentThread());
    });
    
    vThread.join(); // Wait for completion
}
```

**Output:**
```
Hello from virtual thread: VirtualThread[#21]/runnable@ForkJoinPool-1-worker-1
```

Notice: Virtual thread runs on a **carrier thread** (platform thread from ForkJoinPool)

#### Method 2: Thread.ofVirtual().start()

```java
void main() {
    Thread vThread = Thread.ofVirtual()
        .name("my-virtual-thread")
        .start(() -> {
            println("Named virtual thread: " + Thread.currentThread().getName());
        });
    
    vThread.join();
}
```

#### Method 3: Thread.Builder for Batch Creation

```java
void main() throws InterruptedException {
    Thread.Builder builder = Thread.ofVirtual().name("worker-", 0);
    
    List<Thread> threads = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        final int taskId = i;
        Thread t = builder.start(() -> {
            println("Task " + taskId + " on " + Thread.currentThread().getName());
        });
        threads.add(t);
    }
    
    // Wait for all to complete
    for (Thread t : threads) {
        t.join();
    }
}
```

### Virtual Threads at Scale: Million Threads Demo

```java
import java.time.Duration;
import java.time.Instant;

void main() throws InterruptedException {
    Instant start = Instant.now();
    
    List<Thread> threads = new ArrayList<>();
    for (int i = 0; i < 1_000_000; i++) {
        Thread vThread = Thread.startVirtualThread(() -> {
            try {
                Thread.sleep(1000); // Sleep for 1 second
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        threads.add(vThread);
    }
    
    // Wait for all threads to complete
    for (Thread t : threads) {
        t.join();
    }
    
    Instant end = Instant.now();
    println("1 million virtual threads completed in: " + 
            Duration.between(start, end).toSeconds() + " seconds");
}
```

**Output:**
```
1 million virtual threads completed in: 1 seconds
```

**Why this works:**
- Each virtual thread is ~1KB → 1M threads = ~1GB
- JVM schedules them on a small pool of carrier threads
- When a virtual thread sleeps, JVM unmounts it and reuses the carrier thread

### How Virtual Threads Work Internally

**Mounting and Unmounting:**

```java
void main() throws InterruptedException {
    Thread.startVirtualThread(() -> {
        println("Before blocking: " + Thread.currentThread());
        
        try {
            Thread.sleep(100); // Blocking operation
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        println("After blocking: " + Thread.currentThread());
    }).join();
}
```

**What happens:**
1. Virtual thread starts on carrier thread (e.g., ForkJoinPool-1-worker-1)
2. When `sleep()` is called, virtual thread is **unmounted** from carrier
3. Carrier thread is freed to run other virtual threads
4. After sleep, virtual thread is **remounted** (possibly on a different carrier)

**Key Insight:** Virtual threads are **cheap to block** because blocking doesn't block the carrier thread!

### Virtual Threads vs Platform Threads

| Feature | Platform Threads | Virtual Threads |
|---------|-----------------|-----------------|
| Memory | ~2MB per thread | ~1KB per thread |
| Creation Cost | ~1ms | ~1μs (1000x faster) |
| Max Count | Thousands | Millions |
| Managed By | Operating System | JVM |
| Blocking | Blocks OS thread | Unmounts from carrier |
| Use Case | CPU-intensive tasks | I/O-intensive tasks |

### Best Practices for Virtual Threads

#### ✅ DO: Use for I/O-bound tasks

```java
void main() {
    // Good: I/O operations (database, network, file)
    Thread.startVirtualThread(() -> {
        String data = fetchFromDatabase(); // I/O operation
        processData(data);
    });
}
```

#### ❌ DON'T: Use for CPU-intensive tasks

```java
void main() {
    // Bad: CPU-intensive computation
    Thread.startVirtualThread(() -> {
        for (int i = 0; i < 1_000_000_000; i++) {
            Math.sqrt(i); // CPU-bound work
        }
    });
    // Virtual threads don't make CPU work faster!
}
```

#### ✅ DO: Use thread-per-request model

```java
void handleHttpRequest(Request request) {
    // Good: One virtual thread per request
    Thread.startVirtualThread(() -> {
        String result = callExternalAPI(request);
        sendResponse(result);
    });
}
```

#### ❌ DON'T: Use thread pools with virtual threads

```java
// Bad: Don't pool virtual threads!
ExecutorService executor = Executors.newFixedThreadPool(100); // Wrong!

// Good: Create virtual threads directly
Thread.startVirtualThread(() -> {
    // Task
});
```

**Why?** Virtual threads are so cheap that pooling is counterproductive.

### Pinning: The Virtual Thread Gotcha

**Pinning** occurs when a virtual thread cannot be unmounted from its carrier thread.

#### Problem: synchronized blocks

```java
Object lock = new Object();

void main() {
    Thread.startVirtualThread(() -> {
        synchronized (lock) {
            // Virtual thread is PINNED to carrier during this block!
            try {
                Thread.sleep(1000); // Blocks carrier thread
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    });
}
```

**Why it's bad:** The carrier thread is blocked, reducing concurrency.

#### Solution: Use ReentrantLock

```java
import java.util.concurrent.locks.ReentrantLock;

ReentrantLock lock = new ReentrantLock();

void main() {
    Thread.startVirtualThread(() -> {
        lock.lock();
        try {
            // Virtual thread can be unmounted here!
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            lock.unlock();
        }
    });
}
```

**JVM Option to Detect Pinning:**
```bash
java -Djdk.tracePinnedThreads=full MyProgram.java
```

---

## Structured Concurrency (JEP 480)

### The Problem with Unstructured Concurrency

Traditional concurrent code is **fragile**:

```java
void main() throws Exception {
    ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
    
    Future<String> user = executor.submit(() -> fetchUser());
    Future<String> orders = executor.submit(() -> fetchOrders());
    
    // Problems:
    // 1. If fetchUser() fails, fetchOrders() keeps running
    // 2. Must manually handle cancellation
    // 3. Resource leaks if we forget to shutdown executor
    
    String userData = user.get();
    String orderData = orders.get();
    
    executor.shutdown();
}
```

### Structured Concurrency Solution

**Structured Concurrency** treats concurrent tasks as a **single unit of work**.

**Principle:** A task can't outlive its parent scope.

```java
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.StructuredTaskScope.Subtask;

void main() throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        Subtask<String> user = scope.fork(() -> fetchUser());
        Subtask<String> orders = scope.fork(() -> fetchOrders());
        
        scope.join();           // Wait for all subtasks
        scope.throwIfFailed();  // Propagate exceptions
        
        // Both tasks succeeded
        String userData = user.get();
        String orderData = orders.get();
        
        println("User: " + userData);
        println("Orders: " + orderData);
        
    } // Auto-cancels any still-running tasks
}

String fetchUser() throws InterruptedException {
    Thread.sleep(1000);
    return "User{id=123, name='Alice'}";
}

String fetchOrders() throws InterruptedException {
    Thread.sleep(500);
    return "Orders[Order{id=1}, Order{id=2}]";
}
```

**Key Benefits:**
1. **Automatic cleanup**: All tasks cancelled when scope closes
2. **Error propagation**: If one fails, all are cancelled
3. **Structured lifetime**: Tasks can't outlive the scope

### ShutdownOnFailure: Cancel All if One Fails

```java
import java.util.concurrent.StructuredTaskScope;

void main() {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        var task1 = scope.fork(() -> {
            Thread.sleep(100);
            return "Success";
        });
        
        var task2 = scope.fork(() -> {
            Thread.sleep(50);
            throw new RuntimeException("Task 2 failed!");
        });
        
        var task3 = scope.fork(() -> {
            Thread.sleep(200);
            return "This won't complete";
        });
        
        scope.join();
        scope.throwIfFailed();  // Throws RuntimeException from task2
        
    } catch (Exception e) {
        println("Caught: " + e.getMessage());
        // Output: Caught: Task 2 failed!
        // task3 was automatically cancelled
    }
}
```

### ShutdownOnSuccess: Cancel All After First Success

```java
import java.util.concurrent.StructuredTaskScope;

void main() throws Exception {
    // Race multiple servers, use first response
    try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
        
        scope.fork(() -> fetchFromServer("Server A", 200));
        scope.fork(() -> fetchFromServer("Server B", 100));
        scope.fork(() -> fetchFromServer("Server C", 300));
        
        scope.join();
        
        String result = scope.result();  // Gets first successful result
        println("Winner: " + result);
        // Output: Winner: Response from Server B
        // Other tasks were cancelled after B completed
    }
}

String fetchFromServer(String name, long delayMs) throws InterruptedException {
    Thread.sleep(delayMs);
    return "Response from " + name;
}
```

### Custom Scopes: Implementing Your Own Policy

```java
import java.util.concurrent.StructuredTaskScope;
import java.util.List;
import java.util.ArrayList;

class CollectAllScope<T> extends StructuredTaskScope<T> {
    private final List<T> results = new ArrayList<>();
    
    @Override
    protected void handleComplete(Subtask<? extends T> subtask) {
        if (subtask.state() == Subtask.State.SUCCESS) {
            results.add(subtask.get());
        }
    }
    
    public List<T> results() {
        return List.copyOf(results);
    }
}

void main() throws InterruptedException {
    try (var scope = new CollectAllScope<Integer>()) {
        
        scope.fork(() -> 10);
        scope.fork(() -> 20);
        scope.fork(() -> { throw new RuntimeException("Failed"); });
        scope.fork(() -> 30);
        
        scope.join();
        
        List<Integer> results = scope.results();
        println("Collected: " + results);
        // Output: Collected: [10, 20, 30]
        // Failed task is ignored, others succeeded
    }
}
```

### Real-World Example: Parallel Data Aggregation

```java
import java.util.concurrent.StructuredTaskScope;

record UserProfile(String name, List<String> orders, String preferences) {}

void main() throws Exception {
    int userId = 123;
    
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        var nameTask = scope.fork(() -> fetchUserName(userId));
        var ordersTask = scope.fork(() -> fetchOrders(userId));
        var prefsTask = scope.fork(() -> fetchPreferences(userId));
        
        scope.join();
        scope.throwIfFailed();
        
        UserProfile profile = new UserProfile(
            nameTask.get(),
            ordersTask.get(),
            prefsTask.get()
        );
        
        println(profile);
    }
}

String fetchUserName(int id) throws InterruptedException {
    Thread.sleep(100);
    return "Alice";
}

List<String> fetchOrders(int id) throws InterruptedException {
    Thread.sleep(150);
    return List.of("Order-1", "Order-2");
}

String fetchPreferences(int id) throws InterruptedException {
    Thread.sleep(80);
    return "Theme: Dark, Language: EN";
}
```

---

## Scoped Values (JEP 481)

### The Problem with ThreadLocal

**ThreadLocal** was used to pass context (user ID, transaction ID) through call stacks without explicit parameters.

```java
class UserContext {
    private static final ThreadLocal<String> userId = new ThreadLocal<>();
    
    public static void setUserId(String id) {
        userId.set(id);
    }
    
    public static String getUserId() {
        return userId.get();
    }
}

void main() {
    UserContext.setUserId("user-123");
    
    processRequest();  // Implicitly has access to user-123
    
    UserContext.setUserId(null); // Must manually clean up!
}

void processRequest() {
    String user = UserContext.getUserId();
    println("Processing for: " + user);
}
```

**Problems with ThreadLocal:**
1. **Mutable**: Value can be changed anywhere
2. **No automatic cleanup**: Memory leaks if not cleared
3. **Inheritance issues**: Child threads may inherit values unexpectedly
4. **Virtual thread overhead**: Each virtual thread gets its own copy

### Scoped Values: The Modern Solution

**Scoped Values** provide **immutable**, **bounded** context propagation.

```java
import java.util.concurrent.StructuredTaskScope;

// Define a scoped value
final static ScopedValue<String> USER_ID = ScopedValue.newInstance();

void main() throws Exception {
    // Bind value for a scope
    ScopedValue.where(USER_ID, "user-123").run(() -> {
        
        processRequest();  // Has access to user-123
        
        Thread.startVirtualThread(() -> {
            processRequest();  // Child thread inherits user-123
        }).join();
        
    }); // Value automatically unbounded here
    
    // USER_ID.get() would throw here - not in scope!
}

void processRequest() {
    String userId = USER_ID.get();
    println("Processing for: " + userId);
    
    logAction("viewed_profile");
}

void logAction(String action) {
    String userId = USER_ID.get();  // Can access in nested calls
    println("Log: User " + userId + " performed " + action);
}
```

**Output:**
```
Processing for: user-123
Log: User user-123 performed viewed_profile
Processing for: user-123
Log: User user-123 performed viewed_profile
```

### Multiple Scoped Values

```java
final static ScopedValue<String> USER_ID = ScopedValue.newInstance();
final static ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

void main() {
    ScopedValue.where(USER_ID, "user-123")
               .where(REQUEST_ID, "req-abc-456")
               .run(() -> {
                   
        handleRequest();
    });
}

void handleRequest() {
    String user = USER_ID.get();
    String request = REQUEST_ID.get();
    
    println("User: " + user + ", Request: " + request);
}
```

### Scoped Values with Structured Concurrency

```java
import java.util.concurrent.StructuredTaskScope;

final static ScopedValue<String> TRACE_ID = ScopedValue.newInstance();

void main() throws Exception {
    ScopedValue.where(TRACE_ID, "trace-xyz-789").run(() -> {
        
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            
            var task1 = scope.fork(() -> {
                String trace = TRACE_ID.get();  // Inherited!
                return "Task1 with trace: " + trace;
            });
            
            var task2 = scope.fork(() -> {
                String trace = TRACE_ID.get();  // Inherited!
                return "Task2 with trace: " + trace;
            });
            
            scope.join();
            scope.throwIfFailed();
            
            println(task1.get());
            println(task2.get());
        }
        
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

**Output:**
```
Task1 with trace: trace-xyz-789
Task2 with trace: trace-xyz-789
```

### Scoped Values vs ThreadLocal

| Feature | ThreadLocal | ScopedValue |
|---------|-------------|-------------|
| Mutability | Mutable | Immutable |
| Lifecycle | Manual cleanup required | Automatic (scope-bound) |
| Inheritance | Unpredictable with virtual threads | Predictable inheritance |
| Performance | Slower with many virtual threads | Optimized for virtual threads |
| Safety | Prone to leaks | Memory-safe |

### Real-World Example: Request Tracing

```java
import java.util.UUID;
import java.util.concurrent.StructuredTaskScope;

final static ScopedValue<String> TRACE_ID = ScopedValue.newInstance();
final static ScopedValue<String> USER_ID = ScopedValue.newInstance();

void main() throws Exception {
    // Simulate incoming HTTP request
    handleRequest("user-alice");
}

void handleRequest(String userId) throws Exception {
    String traceId = UUID.randomUUID().toString();
    
    ScopedValue.where(TRACE_ID, traceId)
               .where(USER_ID, userId)
               .run(() -> {
        try {
            processRequest();
        } catch (Exception e) {
            e.printStackTrace();
        }
    });
}

void processRequest() throws Exception {
    log("Request started");
    
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        var dbTask = scope.fork(() -> {
            log("Querying database");
            Thread.sleep(100);
            return "DB Result";
        });
        
        var cacheTask = scope.fork(() -> {
            log("Checking cache");
            Thread.sleep(50);
            return "Cache Result";
        });
        
        scope.join();
        scope.throwIfFailed();
        
        log("Request completed");
    }
}

void log(String message) {
    String trace = TRACE_ID.get();
    String user = USER_ID.get();
    println("[Trace: " + trace + ", User: " + user + "] " + message);
}
```

**Output:**
```
[Trace: 550e8400-e29b-41d4-a716-446655440000, User: user-alice] Request started
[Trace: 550e8400-e29b-41d4-a716-446655440000, User: user-alice] Querying database
[Trace: 550e8400-e29b-41d4-a716-446655440000, User: user-alice] Checking cache
[Trace: 550e8400-e29b-41d4-a716-446655440000, User: user-alice] Request completed
```

---

## Modern Networking with HttpClient

### The Old Way: HttpURLConnection (Don't Use!)

```java
// Old, verbose, and blocking
URL url = new URL("https://api.example.com/data");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
// ... many lines of code
```

### The Modern Way: HttpClient (Java 11+)

```java
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.URI;

void main() throws Exception {
    HttpClient client = HttpClient.newHttpClient();
    
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://jsonplaceholder.typicode.com/posts/1"))
        .GET()
        .build();
    
    HttpResponse<String> response = client.send(
        request,
        HttpResponse.BodyHandlers.ofString()
    );
    
    println("Status: " + response.statusCode());
    println("Body: " + response.body());
}
```

### Async Requests with Virtual Threads

```java
import java.net.http.*;
import java.net.URI;
import java.util.concurrent.CompletableFuture;

void main() throws Exception {
    HttpClient client = HttpClient.newHttpClient();
    
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://jsonplaceholder.typicode.com/posts/1"))
        .build();
    
    // Async request returns CompletableFuture
    CompletableFuture<HttpResponse<String>> future = 
        client.sendAsync(request, HttpResponse.BodyHandlers.ofString());
    
    // Process response when available
    future.thenAccept(response -> {
        println("Async Status: " + response.statusCode());
        println("Async Body: " + response.body());
    });
    
    // Wait for completion
    future.join();
}
```

### Multiple Concurrent Requests with Structured Concurrency

```java
import java.net.http.*;
import java.net.URI;
import java.util.concurrent.StructuredTaskScope;

void main() throws Exception {
    HttpClient client = HttpClient.newHttpClient();
    
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        var post1 = scope.fork(() -> fetchPost(client, 1));
        var post2 = scope.fork(() -> fetchPost(client, 2));
        var post3 = scope.fork(() -> fetchPost(client, 3));
        
        scope.join();
        scope.throwIfFailed();
        
        println("Post 1: " + post1.get());
        println("Post 2: " + post2.get());
        println("Post 3: " + post3.get());
    }
}

String fetchPost(HttpClient client, int id) throws Exception {
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://jsonplaceholder.typicode.com/posts/" + id))
        .build();
    
    HttpResponse<String> response = client.send(
        request,
        HttpResponse.BodyHandlers.ofString()
    );
    
    return response.body();
}
```

### POST Requests with JSON

```java
import java.net.http.*;
import java.net.URI;

void main() throws Exception {
    HttpClient client = HttpClient.newHttpClient();
    
    String jsonBody = """
        {
            "title": "New Post",
            "body": "This is the content",
            "userId": 1
        }
        """;
    
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://jsonplaceholder.typicode.com/posts"))
        .header("Content-Type", "application/json")
        .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
        .build();
    
    HttpResponse<String> response = client.send(
        request,
        HttpResponse.BodyHandlers.ofString()
    );
    
    println("Created: " + response.statusCode());
    println("Response: " + response.body());
}
```

### Custom Headers and Timeouts

```java
import java.net.http.*;
import java.net.URI;
import java.time.Duration;

void main() throws Exception {
    HttpClient client = HttpClient.newBuilder()
        .connectTimeout(Duration.ofSeconds(5))
        .build();
    
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.example.com/data"))
        .timeout(Duration.ofSeconds(10))
        .header("Authorization", "Bearer token123")
        .header("Accept", "application/json")
        .GET()
        .build();
    
    HttpResponse<String> response = client.send(
        request,
        HttpResponse.BodyHandlers.ofString()
    );
    
    println(response.body());
}
```

---

## Practice Exercises

### Exercise 1: Virtual Thread Basics
**Task:** Create 10,000 virtual threads that each sleep for 1 second and print their thread ID.

**Starter Code:**
```java
void main() throws InterruptedException {
    // Your code here
}
```

<details>
<summary>Solution</summary>

```java
void main() throws InterruptedException {
    List<Thread> threads = new ArrayList<>();
    
    for (int i = 0; i < 10_000; i++) {
        final int id = i;
        Thread vt = Thread.startVirtualThread(() -> {
            try {
                Thread.sleep(1000);
                if (id % 1000 == 0) { // Print every 1000th
                    println("Thread " + id + " completed");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        threads.add(vt);
    }
    
    for (Thread t : threads) {
        t.join();
    }
    
    println("All 10,000 threads completed!");
}
```
</details>

### Exercise 2: Structured Concurrency - Parallel File Processing
**Task:** Process 5 files concurrently. If any file processing fails, cancel all other operations.

**Starter Code:**
```java
import java.util.concurrent.StructuredTaskScope;

void main() {
    String[] files = {"file1.txt", "file2.txt", "file3.txt", "file4.txt", "file5.txt"};
    
    // Your code here
}

String processFile(String filename) throws InterruptedException {
    Thread.sleep(100);
    if (filename.equals("file3.txt")) {
        throw new RuntimeException("Failed to process " + filename);
    }
    return "Processed: " + filename;
}
```

<details>
<summary>Solution</summary>

```java
import java.util.concurrent.StructuredTaskScope;

void main() {
    String[] files = {"file1.txt", "file2.txt", "file3.txt", "file4.txt", "file5.txt"};
    
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        var tasks = new ArrayList<StructuredTaskScope.Subtask<String>>();
        for (String file : files) {
            tasks.add(scope.fork(() -> processFile(file)));
        }
        
        scope.join();
        scope.throwIfFailed();
        
        for (var task : tasks) {
            println(task.get());
        }
        
    } catch (Exception e) {
        println("Error: " + e.getMessage());
    }
}

String processFile(String filename) throws InterruptedException {
    Thread.sleep(100);
    if (filename.equals("file3.txt")) {
        throw new RuntimeException("Failed to process " + filename);
    }
    return "Processed: " + filename;
}
```
</details>

### Exercise 3: Scoped Values - Multi-Tenant Application
**Task:** Create a multi-tenant application where each request has a tenant ID that's accessible throughout the call stack.

**Requirements:**
- Define a scoped value for TENANT_ID
- Process 3 concurrent requests for different tenants
- Log the tenant ID at various depths of the call stack

<details>
<summary>Solution</summary>

```java
import java.util.concurrent.StructuredTaskScope;

final static ScopedValue<String> TENANT_ID = ScopedValue.newInstance();

void main() throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        scope.fork(() -> handleRequest("tenant-A"));
        scope.fork(() -> handleRequest("tenant-B"));
        scope.fork(() -> handleRequest("tenant-C"));
        
        scope.join();
        scope.throwIfFailed();
    }
}

Void handleRequest(String tenantId) {
    ScopedValue.where(TENANT_ID, tenantId).run(() -> {
        try {
            log("Request started");
            processData();
            log("Request completed");
        } catch (Exception e) {
            e.printStackTrace();
        }
    });
    return null;
}

void processData() throws InterruptedException {
    log("Processing data");
    Thread.sleep(100);
    saveToDatabase();
}

void saveToDatabase() {
    log("Saving to database");
}

void log(String message) {
    String tenant = TENANT_ID.get();
    println("[Tenant: " + tenant + "] " + message);
}
```
</details>

### Exercise 4: HttpClient - Concurrent API Calls
**Task:** Fetch posts 1-10 from JSONPlaceholder API concurrently and display their titles.

**API:** https://jsonplaceholder.typicode.com/posts/{id}

<details>
<summary>Solution</summary>

```java
import java.net.http.*;
import java.net.URI;
import java.util.concurrent.StructuredTaskScope;

void main() throws Exception {
    HttpClient client = HttpClient.newHttpClient();
    
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        var tasks = new ArrayList<StructuredTaskScope.Subtask<String>>();
        for (int i = 1; i <= 10; i++) {
            final int postId = i;
            tasks.add(scope.fork(() -> fetchPostTitle(client, postId)));
        }
        
        scope.join();
        scope.throwIfFailed();
        
        for (int i = 0; i < tasks.size(); i++) {
            println("Post " + (i + 1) + ": " + tasks.get(i).get());
        }
    }
}

String fetchPostTitle(HttpClient client, int id) throws Exception {
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://jsonplaceholder.typicode.com/posts/" + id))
        .build();
    
    HttpResponse<String> response = client.send(
        request,
        HttpResponse.BodyHandlers.ofString()
    );
    
    // Extract title from JSON (simple string parsing)
    String body = response.body();
    int titleStart = body.indexOf("\"title\": \"") + 10;
    int titleEnd = body.indexOf("\"", titleStart);
    return body.substring(titleStart, titleEnd);
}
```
</details>

### Exercise 5: Complete System - Order Processing
**Task:** Build an order processing system that:
1. Uses virtual threads for each order
2. Uses structured concurrency to fetch user data, inventory, and pricing in parallel
3. Uses scoped values to track order ID through the entire flow
4. Cancels all operations if any step fails

**Starter Code:**
```java
record Order(int orderId, int userId, String productId, int quantity) {}

void main() throws Exception {
    List<Order> orders = List.of(
        new Order(1, 101, "PROD-A", 2),
        new Order(2, 102, "PROD-B", 1),
        new Order(3, 103, "PROD-C", 3)
    );
    
    // Process all orders concurrently
}
```

<details>
<summary>Solution</summary>

```java
import java.util.concurrent.StructuredTaskScope;

record Order(int orderId, int userId, String productId, int quantity) {}
record UserData(String name, String address) {}
record Inventory(int available) {}
record Price(double amount) {}

final static ScopedValue<Integer> ORDER_ID = ScopedValue.newInstance();

void main() throws Exception {
    List<Order> orders = List.of(
        new Order(1, 101, "PROD-A", 2),
        new Order(2, 102, "PROD-B", 1),
        new Order(3, 103, "PROD-C", 3)
    );
    
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        for (Order order : orders) {
            scope.fork(() -> processOrder(order));
        }
        
        scope.join();
        scope.throwIfFailed();
        
        println("All orders processed successfully!");
    }
}

Void processOrder(Order order) {
    ScopedValue.where(ORDER_ID, order.orderId()).run(() -> {
        try {
            log("Processing order");
            
            try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
                
                var userTask = scope.fork(() -> fetchUserData(order.userId()));
                var inventoryTask = scope.fork(() -> checkInventory(order.productId()));
                var priceTask = scope.fork(() -> getPrice(order.productId()));
                
                scope.join();
                scope.throwIfFailed();
                
                UserData user = userTask.get();
                Inventory inventory = inventoryTask.get();
                Price price = priceTask.get();
                
                // Validate
                if (inventory.available() < order.quantity()) {
                    throw new RuntimeException("Insufficient inventory");
                }
                
                double total = price.amount() * order.quantity();
                log("Order total: $" + total + " for " + user.name());
                
            }
        } catch (Exception e) {
            log("Order failed: " + e.getMessage());
        }
    });
    return null;
}

UserData fetchUserData(int userId) throws InterruptedException {
    log("Fetching user data");
    Thread.sleep(100);
    return new UserData("User-" + userId, "123 Main St");
}

Inventory checkInventory(String productId) throws InterruptedException {
    log("Checking inventory");
    Thread.sleep(80);
    return new Inventory(10);
}

Price getPrice(String productId) throws InterruptedException {
    log("Getting price");
    Thread.sleep(60);
    return new Price(99.99);
}

void log(String message) {
    int orderId = ORDER_ID.get();
    println("[Order " + orderId + "] " + message);
}
```
</details>

---

## Interview Questions & Answers

### Q1: What are Virtual Threads and how do they differ from Platform Threads?

**Answer:**

Virtual threads are lightweight threads managed by the JVM (introduced in Java 21) that enable massive concurrency with minimal overhead.

**Key Differences:**

| Aspect | Platform Threads | Virtual Threads |
|--------|-----------------|-----------------|
| **Managed by** | Operating System | JVM |
| **Memory** | ~2MB per thread | ~1KB per thread |
| **Creation cost** | Expensive (~1ms) | Cheap (~1μs) |
| **Scalability** | Thousands | Millions |
| **Blocking** | Blocks OS thread | Unmounts from carrier |

**When blocking occurs:**
- Platform thread: Entire OS thread is blocked
- Virtual thread: Unmounts from carrier thread, which can run other virtual threads

**Best use case:** I/O-bound tasks (database calls, HTTP requests, file operations) where threads spend most time waiting.

### Q2: What is thread pinning and how do you avoid it?

**Answer:**

**Pinning** occurs when a virtual thread cannot be unmounted from its carrier thread, blocking the carrier.

**Causes:**
1. **synchronized blocks:** Virtual threads inside synchronized cannot unmount
2. **Native methods:** JNI calls pin the thread

**Example of pinning:**
```java
synchronized (lock) {
    Thread.sleep(1000); // Carrier thread is blocked!
}
```

**Solution:** Use ReentrantLock instead:
```java
lock.lock();
try {
    Thread.sleep(1000); // Can unmount
} finally {
    lock.unlock();
}
```

**Detection:** Run with JVM flag:
```bash
java -Djdk.tracePinnedThreads=full YourProgram.java
```

### Q3: Explain Structured Concurrency and its benefits.

**Answer:**

**Structured Concurrency** treats concurrent subtasks as a single unit of work with a defined lifetime.

**Core Principle:** Subtasks cannot outlive their parent scope.

**Key Classes:**
- `StructuredTaskScope.ShutdownOnFailure`: Cancels all if one fails
- `StructuredTaskScope.ShutdownOnSuccess`: Cancels all after first success

**Benefits:**
1. **Automatic cleanup:** No resource leaks
2. **Error propagation:** Failures handled uniformly
3. **Cancellation:** All subtasks cancelled when scope closes
4. **Observability:** Clear task hierarchy

**Example:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var task1 = scope.fork(() -> operation1());
    var task2 = scope.fork(() -> operation2());
    
    scope.join();
    scope.throwIfFailed();
    
    // Both succeeded
} // Auto-cancels any running tasks
```

### Q4: What are Scoped Values and how do they improve on ThreadLocal?

**Answer:**

**Scoped Values** provide immutable, bounded context propagation as a replacement for ThreadLocal.

**Problems with ThreadLocal:**
- Mutable (can be changed anywhere)
- Manual cleanup required (memory leaks)
- Expensive with virtual threads
- Unpredictable inheritance

**Scoped Values solve these:**
```java
final static ScopedValue<String> USER_ID = ScopedValue.newInstance();

ScopedValue.where(USER_ID, "user-123").run(() -> {
    processRequest(); // Has access
}); // Automatically unbounded
```

**Advantages:**
1. **Immutable:** Can't be changed within scope
2. **Automatic cleanup:** Scope-bounded lifecycle
3. **Efficient:** Optimized for virtual threads
4. **Safe inheritance:** Predictable with structured concurrency

### Q5: When should you NOT use Virtual Threads?

**Answer:**

**Don't use virtual threads for:**

1. **CPU-intensive tasks:**
   - Virtual threads don't speed up computation
   - Use platform threads or parallel streams for CPU work

2. **With thread pools:**
   - Virtual threads are cheap enough to create on-demand
   - Pooling adds unnecessary complexity

3. **When you need thread pinning:**
   - Heavy use of synchronized blocks
   - Extensive JNI/native calls

**Do use virtual threads for:**
- I/O-bound operations (DB, network, files)
- High concurrency scenarios (web servers)
- Blocking operations (sleep, wait)

### Q6: How does HttpClient integrate with virtual threads?

**Answer:**

HttpClient (Java 11+) works seamlessly with virtual threads for concurrent I/O.

**Synchronous (blocking):**
```java
HttpClient client = HttpClient.newHttpClient();
Thread.startVirtualThread(() -> {
    // Virtual thread blocks efficiently
    HttpResponse<String> response = client.send(request, ...);
});
```

**Asynchronous (non-blocking):**
```java
CompletableFuture<HttpResponse<String>> future = 
    client.sendAsync(request, ...);
```

**With Structured Concurrency:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var req1 = scope.fork(() -> client.send(request1, ...));
    var req2 = scope.fork(() -> client.send(request2, ...));
    
    scope.join();
    scope.throwIfFailed();
}
```

**Benefits:** Natural blocking style with virtual thread efficiency.

### Q7: Explain mounting and unmounting in virtual threads.

**Answer:**

**Mounting:** Attaching a virtual thread to a platform thread (carrier) for execution

**Unmounting:** Detaching a virtual thread from its carrier when it blocks

**Lifecycle:**
```
Virtual Thread (Parked)
    ↓ [mount]
Carrier Thread (Platform Thread)
    ↓ [execute until blocking]
Virtual Thread blocks (e.g., Thread.sleep())
    ↓ [unmount]
Virtual Thread (Parked) + Carrier Thread (free to run others)
    ↓ [after blocking completes]
    ↓ [mount - possibly different carrier]
Carrier Thread (continues execution)
```

**Example:**
```java
Thread.startVirtualThread(() -> {
    // [MOUNTED to carrier-1]
    println("Working");
    
    Thread.sleep(1000); // [UNMOUNTED, carrier-1 is free]
    
    // [REMOUNTED to carrier-2 (might be different)]
    println("Done");
});
```

**Key Point:** Unmounting enables massive concurrency because carriers aren't blocked.

### Q8: Compare StructuredTaskScope.ShutdownOnFailure vs ShutdownOnSuccess.

**Answer:**

| Aspect | ShutdownOnFailure | ShutdownOnSuccess |
|--------|-------------------|-------------------|
| **Goal** | All tasks must succeed | First success wins |
| **Cancellation** | On first failure | After first success |
| **Use Case** | All results needed | Race conditions |
| **Method** | `throwIfFailed()` | `result()` |

**ShutdownOnFailure Example:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var db = scope.fork(() -> queryDB());
    var cache = scope.fork(() -> queryCache());
    
    scope.join();
    scope.throwIfFailed(); // Throws if either failed
    
    // Both succeeded, use both results
}
```

**ShutdownOnSuccess Example:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(() -> fetchFromServerA());
    scope.fork(() -> fetchFromServerB());
    scope.fork(() -> fetchFromServerC());
    
    scope.join();
    String result = scope.result(); // First successful result
}
```

### Q9: What happens if you don't join() a StructuredTaskScope?

**Answer:**

**Failing to call `join()` results in compilation error or runtime exception.**

**Reason:** Subtask results are not accessible until `join()` is called.

**Example (won't compile):**
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var task = scope.fork(() -> "result");
    // task.get(); // IllegalStateException: not joined yet!
}
```

**Correct usage:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var task = scope.fork(() -> "result");
    
    scope.join(); // Must join before accessing results
    scope.throwIfFailed();
    
    String result = task.get(); // Now safe
}
```

**join() semantics:**
- Blocks until all subtasks complete (or scope policy triggers)
- Makes results available
- Required before accessing subtask state

### Q10: How do you handle timeouts with Structured Concurrency?

**Answer:**

Use `joinUntil(Instant deadline)` instead of `join()`:

```java
import java.time.Instant;
import java.time.Duration;

try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    
    var task1 = scope.fork(() -> slowOperation());
    var task2 = scope.fork(() -> fastOperation());
    
    // Wait max 2 seconds
    Instant deadline = Instant.now().plus(Duration.ofSeconds(2));
    scope.joinUntil(deadline);
    
    scope.throwIfFailed();
    
} catch (TimeoutException e) {
    println("Operations timed out");
    // Scope closes, cancelling all tasks
}
```

**Alternative:** Wrap individual tasks with timeout:
```java
var task = scope.fork(() -> {
    return performWithTimeout(Duration.ofSeconds(1));
});
```

---

## Summary

**Week 8 covered:**

1. **Virtual Threads:** Lightweight, JVM-managed threads for massive concurrency
   - Create millions of threads with minimal overhead
   - Automatic mounting/unmounting for efficient I/O

2. **Structured Concurrency:** Treat concurrent tasks as a unit
   - ShutdownOnFailure: All must succeed
   - ShutdownOnSuccess: First wins
   - Automatic cleanup and error handling

3. **Scoped Values:** Immutable context propagation
   - Replaces ThreadLocal
   - Safe, efficient, and automatic cleanup
   - Perfect with virtual threads

4. **HttpClient:** Modern HTTP communication
   - Synchronous and asynchronous APIs
   - Integrates seamlessly with virtual threads
   - Structured concurrency for parallel requests

**Next Steps:**
- Apply these concepts in the 5 capstone projects
- Focus on I/O-bound scenarios for virtual threads
- Use structured concurrency for reliable concurrent systems
- Leverage scoped values for clean context passing

**Remember:**
- Virtual threads are for I/O, not CPU work
- Always use structured concurrency for multiple tasks
- Avoid synchronized with virtual threads (use ReentrantLock)
- Scoped values are immutable and scope-bounded
