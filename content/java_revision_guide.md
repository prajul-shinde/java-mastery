# Java Interview Revision Guide
**Complete 8-Week Coverage - Essential Concepts Only**

---

## 📋 Quick Navigation
- [Week 1: Modern Syntax & IO](#week-1-modern-syntax--io)
- [Week 2: OOP I & II (Encapsulation, Inheritance)](#week-2-oop-i--ii)
- [Week 3: OOP III & IV (Polymorphism, Abstraction, Generics)](#week-3-oop-iii--iv)
- [Week 4: Essential APIs & Collections](#week-4-essential-apis--collections)
- [Week 5: Streams & Functional Programming](#week-5-streams--functional-programming)
- [Week 6: Concurrent Collections & Thread Safety](#week-6-concurrent-collections--thread-safety)
- [Week 7: Modules, IO & Architecture](#week-7-modules-io--architecture)
- [Week 8: Virtual Threads & Concurrency](#week-8-virtual-threads--concurrency)
- [Common Interview Questions by Topic](#common-interview-questions-by-topic)

---

## Week 1: Modern Syntax & IO

### Entry Point Evolution
```java
// Traditional
public static void main(String[] args) { }

// Modern (JEP 477)
void main() { }
```

### Implicit I/O
```java
println("Hello");        // Instead of System.out.println()
String name = readln();  // Instead of Scanner
```

### Module Imports
```java
import module java.base;  // Import entire module at once
```

### Variables
```java
var name = "Alice";      // Type inference (String)
var _ = list;            // Unnamed variable (JEP 456)

// Primitives (Stack): int, long, double, boolean, char, byte, short, float
// Wrappers (Heap): Integer, Long, Double, Boolean, Character, Byte, Short, Float
```

**Key Difference - Primitives vs Wrappers:**
- Primitives: Fast, no null, stack
- Wrappers: Slower, nullable, heap, autoboxing/unboxing

### Text Blocks
```java
String json = """
    {
        "name": "Alice",
        "age": 30
    }
    """;
```

### String Methods
```java
"hello".formatted("World")           // String formatting
"  text  ".strip()                   // Remove leading/trailing whitespace
"line1\nline2".lines()              // Stream<String>
```

---

## Week 2: OOP I & II

### Encapsulation - Records
```java
// Instead of traditional POJO
record User(String name, int age) {}  // Immutable, auto-generated: constructor, getters, equals, hashCode, toString

// Compact constructor (validation)
record User(String name, int age) {
    public User {  // No parameters!
        if (age < 0) throw new IllegalArgumentException();
    }
}

// Record patterns (deconstruction)
if (obj instanceof User(var name, var age)) {
    println(name);  // Direct access
}
```

### Inheritance - Sealed Classes
```java
sealed interface Shape permits Circle, Square {}  // Only these can implement

final class Circle implements Shape {}      // Must be final
non-sealed class Square implements Shape {} // Opens hierarchy again

// Why? Exhaustive pattern matching
switch (shape) {
    case Circle c -> ...
    case Square s -> ...
    // No default needed - compiler knows all types!
}
```

### Constructor Rules
```java
class Child extends Parent {
    Child() {
        // Can now add statements BEFORE super() (JEP 482)
        int x = 10;
        super(x);
    }
}
```

---

## Week 3: OOP III & IV

### Polymorphism - Pattern Matching

**For instanceof:**
```java
// Old way
if (obj instanceof String) {
    String s = (String) obj;
}

// New way
if (obj instanceof String s) {
    println(s.toUpperCase());  // s in scope
}
```

**For Switch:**
```java
switch (obj) {
    case Integer i when i > 0 -> println("Positive: " + i);
    case Integer i -> println("Non-positive: " + i);
    case String s -> println("String: " + s);
    case null -> println("Null");
    default -> println("Other");
}
```

### Abstraction

**Interfaces:**
```java
interface MyInterface {
    void abstractMethod();
    
    default void defaultMethod() {      // Since Java 8
        println("Default implementation");
    }
    
    private void helperMethod() {       // Since Java 9
        // Helper for default methods
    }
    
    static void staticMethod() {
        // Utility method
    }
}
```

**Abstract Classes vs Interfaces:**
- Abstract: Can have state (fields), constructors, single inheritance
- Interface: No state (except static final), multiple inheritance, contracts only

### Generics

**Basics:**
```java
class Box<T> {
    private T item;
    public void set(T item) { this.item = item; }
    public T get() { return item; }
}

Box<String> box = new Box<>();  // Diamond operator
```

**Wildcards:**
```java
List<?> list1;                    // Unknown type
List<? extends Number> list2;     // Number or subclass (Producer)
List<? super Integer> list3;      // Integer or superclass (Consumer)
```

**PECS Rule:** Producer Extends, Consumer Super

**Type Erasure:** Generics removed at runtime (backwards compatibility)
```java
List<String> → List (at runtime)
```

---

## Week 4: Essential APIs & Collections

### Time API
```java
LocalDate date = LocalDate.of(2026, 1, 30);
LocalTime time = LocalTime.of(14, 30);
LocalDateTime dateTime = LocalDateTime.now();
ZonedDateTime zoned = ZonedDateTime.now(ZoneId.of("America/New_York"));

Instant instant = Instant.now();        // Machine time (UTC)
Duration duration = Duration.ofHours(2); // Time-based
Period period = Period.ofDays(7);        // Date-based
```

### Optional
```java
Optional<String> opt = Optional.of("value");
opt.isPresent()                   // Check if value exists
opt.ifPresent(System.out::println) // Execute if present
opt.orElse("default")              // Get or default
opt.orElseThrow()                  // Get or throw
opt.map(String::toUpperCase)       // Transform
```

### Collections Framework

**Sequenced Collections (Java 21):**
```java
list.getFirst()     // Instead of list.get(0)
list.getLast()      // Instead of list.get(size-1)
list.reversed()     // Returns reversed view
```

**List:**
```java
ArrayList<String> list = new ArrayList<>();  // Fast random access, slow insert/delete
LinkedList<String> list = new LinkedList<>(); // Slow random access, fast insert/delete
```

**Set:**
```java
HashSet<String> set = new HashSet<>();      // No order, O(1) operations
LinkedHashSet<String> set = new LinkedHashSet<>(); // Insertion order
TreeSet<String> set = new TreeSet<>();      // Sorted order, O(log n)
```

**Map:**
```java
HashMap<K,V> map = new HashMap<>();         // No order, O(1) average
LinkedHashMap<K,V> map = new LinkedHashMap<>(); // Insertion order
TreeMap<K,V> map = new TreeMap<>();         // Sorted by keys

// HashMap internals
// Array of buckets → Each bucket is linked list/tree
// hash(key) % buckets.length = bucket index
// Collision handling: chaining (Java 8+: tree if >8 entries)
```

**Key Differences:**
- ArrayList vs LinkedList: Random access vs Insert/Delete
- HashSet vs TreeSet: Speed vs Sorting
- HashMap vs TreeMap: Speed vs Sorting

---

## Week 5: Streams & Functional Programming

### Functional Interfaces
```java
Predicate<T>      // T -> boolean        | test()
Function<T,R>     // T -> R              | apply()
Consumer<T>       // T -> void           | accept()
Supplier<T>       // () -> T             | get()
BiFunction<T,U,R> // (T,U) -> R          | apply()
```

### Stream Operations

**Creating:**
```java
Stream.of(1, 2, 3)
list.stream()
Arrays.stream(array)
Stream.generate(() -> Math.random())
Stream.iterate(0, n -> n + 1)
```

**Intermediate (lazy):**
```java
.map(x -> x * 2)          // Transform
.filter(x -> x > 0)       // Filter
.flatMap(list -> list.stream())  // Flatten
.distinct()               // Remove duplicates
.sorted()                 // Sort
.limit(10)                // First 10
.skip(5)                  // Skip first 5
```

**Terminal (eager):**
```java
.forEach(System.out::println)
.collect(Collectors.toList())
.reduce(0, (a,b) -> a+b)
.count()
.anyMatch(x -> x > 5)
.allMatch(x -> x > 0)
.findFirst()
```

### Collectors
```java
.collect(Collectors.toList())
.collect(Collectors.toSet())
.collect(Collectors.joining(", "))
.collect(Collectors.groupingBy(User::getAge))
.collect(Collectors.partitioningBy(x -> x > 10))
.collect(Collectors.counting())
.collect(Collectors.teeing(
    Collectors.counting(),
    Collectors.summingInt(x -> x),
    (count, sum) -> sum / count
))
```

### Gatherers (JEP 461)
```java
// Custom intermediate operation
stream.gather(Gatherers.windowFixed(3))  // Sliding window of 3
      .gather(Gatherers.fold(() -> 0, (acc, e) -> acc + e))
```

### Parallel Streams
```java
list.parallelStream()
    .map(...)
    .collect(Collectors.toList());

// Uses ForkJoinPool (work-stealing algorithm)
// Good for: CPU-intensive, large datasets
// Bad for: I/O operations, small datasets, order-dependent
```

---

## Week 6: Concurrent Collections & Thread Safety

### Thread Safety Mechanisms

**synchronized:**
```java
synchronized (lock) {
    // Critical section
}

synchronized void method() { }  // Locks on 'this'
```

**ReentrantLock:**
```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();  // Always in finally!
}
```

**Why ReentrantLock > synchronized:**
- Trylock with timeout
- Fair/unfair policies
- Interruptible locking
- Multiple condition variables
- Works with virtual threads (no pinning)

### Concurrent Collections

**ConcurrentHashMap:**
```java
ConcurrentHashMap<K,V> map = new ConcurrentHashMap<>();
map.putIfAbsent(key, value)
map.compute(key, (k,v) -> newValue)
map.merge(key, value, (old,new) -> merged)

// Internal: Segments with separate locks (fine-grained locking)
// No locking for reads, only for writes
```

**CopyOnWriteArrayList:**
```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
// Every write creates new copy
// Good for: Many reads, few writes
// Bad for: Many writes (expensive)
```

**BlockingQueue:**
```java
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(100);
queue.put(task);    // Blocks if full
Task t = queue.take(); // Blocks if empty

// Implementations:
// ArrayBlockingQueue: Bounded, array-based
// LinkedBlockingQueue: Optionally bounded, linked list
// PriorityBlockingQueue: Unbounded, priority heap
```

### Problems to Avoid

**Race Condition:**
```java
// BAD: Two threads increment
count++;  // Read-Modify-Write (not atomic!)

// GOOD: Use AtomicInteger
AtomicInteger count = new AtomicInteger();
count.incrementAndGet();
```

**Deadlock:**
```java
// Thread 1: lock A → lock B
// Thread 2: lock B → lock A
// Solution: Always acquire locks in same order
```

**Starvation:** Thread never gets CPU time (lower priority)
**Livelock:** Threads actively responding but making no progress

---

## Week 7: Modules, IO & Architecture

### Java Platform Module System (JPMS)

**module-info.java:**
```java
module my.module {
    requires java.sql;           // Dependency
    requires transitive java.xml; // Transitive (users get it too)
    
    exports com.example.api;     // Public API
    opens com.example.internal;  // Reflection access
    
    uses com.example.Service;    // Uses service
    provides com.example.Service 
        with com.example.ServiceImpl; // Provides implementation
}
```

**exports vs opens:**
- exports: Compile-time access (normal import)
- opens: Runtime reflection access (frameworks need this)

### ServiceLoader (Plugin System)
```java
// Define service interface
public interface MessagePlugin {
    void process(String message);
}

// In module-info.java
uses MessagePlugin;  // Consumer
provides MessagePlugin with MyPlugin;  // Provider

// Load plugins
ServiceLoader<MessagePlugin> loader = ServiceLoader.load(MessagePlugin.class);
for (MessagePlugin plugin : loader) {
    plugin.process(message);
}
```

### Modern IO (NIO.2)

**Path & Files:**
```java
Path path = Path.of("file.txt");
Files.readString(path)
Files.writeString(path, content)
Files.lines(path)  // Stream<String>
Files.walk(path)   // Walk directory tree
Files.copy(source, target)
Files.move(source, target)
Files.delete(path)
Files.exists(path)
```

**try-with-resources:**
```java
try (BufferedReader reader = Files.newBufferedReader(path)) {
    // Auto-closes even on exception
}
// Works with any AutoCloseable
```

### jlink (Custom Runtime)
```bash
jlink --module-path $JAVA_HOME/jmods:mods \
      --add-modules my.module \
      --output custom-runtime
```
Creates minimal JRE with only needed modules.

---

## Week 8: Virtual Threads & Concurrency

### Virtual Threads

**Key Concepts:**
```java
// Creating
Thread vt = Thread.startVirtualThread(() -> {
    // Task
});

Thread vt = Thread.ofVirtual()
    .name("worker")
    .start(() -> { });
```

**Characteristics:**
- Lightweight (~1KB vs ~2MB for platform threads)
- Can create millions
- JVM-managed (not OS)
- Cheap to block (unmount from carrier)

**Platform vs Virtual Threads:**
| Feature | Platform | Virtual |
|---------|----------|---------|
| Memory | ~2MB | ~1KB |
| Max Count | Thousands | Millions |
| Managed By | OS | JVM |
| Blocking | Blocks OS thread | Unmounts |
| Use Case | CPU-intensive | I/O-intensive |

**Mounting/Unmounting:**
```
Virtual Thread runs on Carrier (platform thread)
↓ Blocking operation (sleep, I/O)
Virtual Thread UNMOUNTS → Carrier free for others
↓ Blocking completes
Virtual Thread REMOUNTS (possibly different carrier)
```

**Pinning (BAD):**
```java
synchronized (lock) {
    Thread.sleep(1000);  // Virtual thread PINNED to carrier!
}

// Solution: Use ReentrantLock
lock.lock();
try {
    Thread.sleep(1000);  // Can unmount
} finally {
    lock.unlock();
}
```

### Structured Concurrency

**ShutdownOnFailure:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var task1 = scope.fork(() -> operation1());
    var task2 = scope.fork(() -> operation2());
    
    scope.join();           // Wait for all
    scope.throwIfFailed();  // Propagate exceptions
    
    // Both succeeded
    Result r1 = task1.get();
    Result r2 = task2.get();
}
```

**ShutdownOnSuccess:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(() -> fetchFromServerA());
    scope.fork(() -> fetchFromServerB());
    
    scope.join();
    String result = scope.result();  // First success
}
```

**Benefits:**
- Automatic cleanup (no resource leaks)
- Cancellation propagation
- Clear error handling
- Structured lifetime (tasks can't outlive scope)

### Scoped Values

**Instead of ThreadLocal:**
```java
// ThreadLocal (OLD)
static ThreadLocal<String> userId = new ThreadLocal<>();
userId.set("user-123");
userId.get();
userId.remove();  // Manual cleanup!

// ScopedValue (NEW)
static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

ScopedValue.where(USER_ID, "user-123").run(() -> {
    processRequest();  // Has access
}); // Auto-unbounded

void processRequest() {
    String id = USER_ID.get();  // Available anywhere in scope
}
```

**Advantages:**
- Immutable (can't change within scope)
- Automatic cleanup
- Efficient with virtual threads
- Safe inheritance to child threads

### HttpClient

**Synchronous:**
```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .build();

HttpResponse<String> response = client.send(
    request,
    HttpResponse.BodyHandlers.ofString()
);
```

**Async:**
```java
CompletableFuture<HttpResponse<String>> future = 
    client.sendAsync(request, HttpResponse.BodyHandlers.ofString());

future.thenAccept(response -> {
    println(response.body());
});
```

---

## Common Interview Questions by Topic

### OOP Fundamentals

**Q: Explain the 4 pillars of OOP.**
A: 
- **Encapsulation:** Bundling data and methods, hiding internals (Records, private fields)
- **Inheritance:** Code reuse through parent-child relationship (Sealed classes control it)
- **Polymorphism:** One interface, multiple implementations (Pattern matching for switch)
- **Abstraction:** Hide complexity, show only essentials (Interfaces, abstract classes)

**Q: Record vs Class?**
A: Records are immutable data carriers. Auto-generate constructor, getters, equals, hashCode, toString. Use for DTOs, value objects. Classes for mutable state, complex behavior.

**Q: Sealed classes benefit?**
A: Controlled inheritance → exhaustive pattern matching (no default needed). Compiler knows all subtypes.

**Q: Interface vs Abstract class?**
A:
- Interface: Contract only, multiple inheritance, no state (except static final), default/private methods
- Abstract: Can have state, constructors, single inheritance, partial implementation

### Collections

**Q: ArrayList vs LinkedList?**
A:
- ArrayList: Array-based, O(1) random access, O(n) insert/delete
- LinkedList: Linked nodes, O(n) random access, O(1) insert/delete at ends

**Q: HashMap internals?**
A: Array of buckets. hash(key) % array.length = index. Collisions handled by chaining (linked list, tree if >8). O(1) average, O(n) worst case (all collisions). Resizes at 75% load factor.

**Q: When to use ConcurrentHashMap?**
A: Multi-threaded access to map. Fine-grained locking (segments). No locking for reads. Use putIfAbsent, compute, merge for atomic operations.

### Streams

**Q: Stream vs Collection?**
A:
- Stream: Lazy, one-time use, functional pipeline, no storage
- Collection: Eager, reusable, storage structure

**Q: Intermediate vs Terminal operations?**
A:
- Intermediate: Lazy (map, filter), return Stream, chainable
- Terminal: Eager (collect, forEach), return result, ends pipeline

**Q: When to use parallel streams?**
A: Large datasets, CPU-intensive operations. Avoid for: I/O, small data, order-dependent operations.

### Concurrency

**Q: Synchronized vs ReentrantLock?**
A:
- synchronized: Simple, automatic release, JVM-level
- ReentrantLock: Trylock, timeout, fairness, interruptible, works with virtual threads

**Q: Race condition example?**
A: Two threads increment shared counter. Read-Modify-Write not atomic. Solution: AtomicInteger or synchronized.

**Q: Deadlock prevention?**
A: Always acquire locks in same order. Use tryLock with timeout. Avoid nested locks.

### Virtual Threads

**Q: Virtual vs Platform threads?**
A: Virtual are lightweight (1KB vs 2MB), JVM-managed, millions possible. Unmount when blocking. Use for I/O-bound tasks. Platform for CPU-intensive.

**Q: What is thread pinning?**
A: Virtual thread can't unmount from carrier. Caused by synchronized blocks or native calls. Solution: Use ReentrantLock.

**Q: Structured concurrency benefits?**
A: Tasks can't outlive scope. Automatic cleanup. Clear error propagation. Prevents resource leaks.

**Q: ScopedValue vs ThreadLocal?**
A: ScopedValue is immutable, automatic cleanup, efficient with virtual threads. ThreadLocal is mutable, manual cleanup, expensive with virtual threads.

### Design & Architecture

**Q: JPMS benefits?**
A: Strong encapsulation, reliable configuration, better performance (jlink). Explicit dependencies.

**Q: When to use ServiceLoader?**
A: Plugin architecture, decoupling, runtime discovery of implementations.

**Q: NIO.2 advantages over old IO?**
A: Non-blocking, better performance, Path API (modern), file watching, symbolic links.

---

## Quick Reference Cheat Sheet

### Common Patterns

**Null Safety:**
```java
Optional.ofNullable(value)
    .map(User::getName)
    .orElse("Unknown");
```

**Resource Management:**
```java
try (var resource = createResource()) {
    // Use resource
}
```

**Pattern Matching:**
```java
if (obj instanceof String s && s.length() > 5) {
    // Use s
}
```

**Streaming:**
```java
list.stream()
    .filter(x -> x > 0)
    .map(x -> x * 2)
    .collect(Collectors.toList());
```

**Virtual Threads:**
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var t1 = scope.fork(() -> task1());
    var t2 = scope.fork(() -> task2());
    scope.join();
    scope.throwIfFailed();
}
```

### Performance Tips

1. **Use primitive streams when possible:** `IntStream` instead of `Stream<Integer>`
2. **Avoid premature parallelization:** Measure first
3. **Use StringBuilder for string concatenation in loops**
4. **HashMap initial capacity:** `new HashMap<>(expectedSize / 0.75)`
5. **Virtual threads for I/O, not CPU work**
6. **ConcurrentHashMap over Collections.synchronizedMap**

### Common Mistakes

❌ **Don't:**
- Use `==` for String comparison
- Modify collection while iterating (use iterator.remove())
- Forget to close resources (use try-with-resources)
- Use synchronized with virtual threads
- Pool virtual threads
- Use parallel streams for small datasets

✅ **Do:**
- Use `.equals()` for object comparison
- Use Records for immutable data
- Use Optional to avoid null checks
- Use Sealed classes for finite type hierarchies
- Use virtual threads for I/O-bound tasks
- Use pattern matching to avoid casting

---

## Final Interview Tips

1. **Explain fundamentals first, then modern features**
   - Example: "Traditional inheritance has issues... Sealed classes solve this by..."

2. **Use concrete examples**
   - "HashMap uses separate chaining. For example, if keys 'abc' and 'xyz' hash to bucket 5..."

3. **Mention trade-offs**
   - "Virtual threads are great for I/O but not CPU-intensive work because..."

4. **Show evolution of Java**
   - "Before Java 8, we'd use anonymous classes. With lambdas, it's much cleaner..."

5. **Be honest about gaps**
   - "I haven't used Gatherers in production yet, but I understand they allow custom intermediate operations..."

**Good luck with your interviews! 🚀**
