# Week 1: Syntax, IO & The New Workflow - Java Mastery Guide

## Table of Contents
1. [The Entry Point (JEP 477)](#the-entry-point-jep-477)
2. [Imports 2.0 (JEP 476)](#imports-20-jep-476)
3. [The Runtime (JEP 458)](#the-runtime-jep-458)
4. [Variables & Data](#variables--data)
5. [Modern Text](#modern-text)
6. [Practice Exercises](#practice-exercises)
7. [Interview Questions & Answers](#interview-questions--answers)

---

## The Entry Point (JEP 477)

### Understanding the Traditional Entry Point

Before we dive into modern Java, let's understand the fundamentals. In traditional Java, every application needs an entry point - the `main` method:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Why these keywords?**
- `public`: Accessible from anywhere (JVM needs to find and call it)
- `static`: Belongs to the class itself, not an instance (no object creation needed)
- `void`: Doesn't return any value
- `main`: The exact name the JVM looks for
- `String[] args`: Command-line arguments passed to the program

### The Modern Entry Point: `void main()`

**JEP 477** introduces simplified entry points for modern Java (Java 21+):

```java
// Modern Java - Instance Main Method
void main() {
    println("Hello, World!");
}
```

**Key Changes:**
1. **No `public`**: Implicitly public
2. **No `static`**: Can be instance method (more flexible)
3. **No class declaration needed**: For simple scripts
4. **No `String[] args`**: Added only when needed

### Understanding `println()` and `readln()`

**Traditional I/O:**
```java
System.out.println("Output");  // Print with newline
System.out.print("No newline");  // Print without newline
```

**Modern Implicit I/O (JEP 477):**
```java
void main() {
    println("Output");  // Simpler!
    print("No newline");
    
    // Reading input
    String name = readln("Enter your name: ");
    println("Hello, " + name);
}
```

**How it works:**
- `println()` is implicitly `System.out.println()`
- `readln()` reads a line from console (like `Scanner.nextLine()`)
- Only available in unnamed classes (simple scripts)

### When to Use Traditional vs Modern

**Use Traditional `main`:**
- Multi-file applications
- When you need explicit class structure
- Production code in large projects
- When using frameworks (Spring, etc.)

**Use Modern `main`:**
- Learning and teaching
- Quick scripts
- Prototyping
- Single-file utilities

### Complete Example: Traditional vs Modern

**Traditional Approach:**
```java
import java.util.Scanner;

public class Greeter {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter your name: ");
        String name = scanner.nextLine();
        System.out.println("Hello, " + name + "!");
        scanner.close();
    }
}
```

**Modern Approach:**
```java
void main() {
    String name = readln("Enter your name: ");
    println("Hello, " + name + "!");
}
```

### Deep Dive: Why Static?

Understanding `static` is crucial for Java mastery:

```java
class Counter {
    // Instance variable - each object has its own copy
    int instanceCount = 0;
    
    // Static variable - shared across all instances
    static int staticCount = 0;
    
    void increment() {
        instanceCount++;
        staticCount++;
    }
}

void main() {
    Counter c1 = new Counter();
    Counter c2 = new Counter();
    
    c1.increment();
    c2.increment();
    
    println(c1.instanceCount);  // 1
    println(c2.instanceCount);  // 1
    println(Counter.staticCount);  // 2 (shared!)
}
```

**Why traditional `main` is `static`:**
- JVM needs to call it without creating an object
- Application entry point must exist before any objects are created
- `static` members belong to the class, not instances

---

## Imports 2.0 (JEP 476)

### Understanding Traditional Imports

**Single Class Import:**
```java
import java.util.ArrayList;  // Import specific class
import java.util.HashMap;
```

**Wildcard Import:**
```java
import java.util.*;  // Import all classes from package
```

**Static Import:**
```java
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

void main() {
    println(PI);  // Use directly without Math.PI
    println(sqrt(16));
}
```

### Module Imports: The New Way

**Traditional way** - importing multiple classes from a module:
```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.HashSet;
// ... dozens more
```

**Modern Module Import (JEP 476):**
```java
import module java.base;  // Import entire module at once!

void main() {
    List<String> list = new ArrayList<>();
    Map<String, Integer> map = new HashMap<>();
    // All java.base classes available
}
```

### Understanding Java Modules

Java organizes code into modules (since Java 9). A module is a collection of related packages:

**Common Modules:**
- `java.base`: Core Java (String, Collections, IO, etc.) - imported by default
- `java.sql`: Database connectivity
- `java.xml`: XML processing
- `java.logging`: Logging APIs

**Traditional vs Module Import:**
```java
// Traditional: Import each class
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Stream;
import java.util.stream.Collectors;

// Modern: Import entire module
import module java.base;
```

### When to Use Module Imports

**Use Module Imports:**
- When using many classes from the same module
- Educational/learning purposes
- Rapid prototyping
- Simple scripts

**Use Specific Imports:**
- Production code (better IDE support)
- When you want to see dependencies clearly
- When name conflicts might occur
- For code review clarity

### Deep Dive: Module System

```java
// Viewing module information
void main() {
    Module module = String.class.getModule();
    println("Module: " + module.getName());  // java.base
    println("Packages: " + module.getPackages());
}
```

**Java Platform Modules:**
```
java.base (automatic)
├── java.lang
├── java.util
├── java.io
├── java.nio
└── java.math

java.sql
├── java.sql
└── javax.sql

java.xml
├── javax.xml
└── org.w3c.dom
```

### Complete Example: Import Strategies

```java
// Strategy 1: Specific imports (recommended for production)
import java.util.ArrayList;
import java.util.List;
import java.util.HashMap;
import java.util.Map;

// Strategy 2: Package wildcard
import java.util.*;  // Import all from package

// Strategy 3: Module import (modern)
import module java.base;  // Import all from module

// Strategy 4: Static import
import static java.lang.Math.*;

void main() {
    // All strategies work the same
    List<String> names = new ArrayList<>();
    Map<String, Integer> ages = new HashMap<>();
    println(sqrt(PI));
}
```

---

## The Runtime (JEP 458)

### Traditional Java Compilation

**The Old Way:**
```bash
# Step 1: Write code
# Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}

# Step 2: Compile
javac Main.java  # Creates Main.class

# Step 3: Run
java Main  # Runs the bytecode
```

### Source Code Launch: Direct Execution

**JEP 458** allows running Java source files directly:

```bash
# New way: Run directly!
java Main.java  # No compilation step needed!
```

**How it works:**
1. Java reads the source file
2. Compiles it in memory
3. Executes immediately
4. No `.class` files created on disk

### Multi-File Programs

**Traditional Approach:**
```bash
# Compile all files
javac *.java

# Run main class
java Main
```

**Modern Approach with Source Launch:**
```bash
# Runs Main.java and automatically compiles dependencies
java Main.java
```

**Example Multi-File Program:**

**Helper.java:**
```java
class Helper {
    static String greet(String name) {
        return "Hello, " + name + "!";
    }
}
```

**Main.java:**
```java
void main() {
    String message = Helper.greet("Java");
    println(message);
}
```

**Run:**
```bash
java Main.java  # Automatically finds and compiles Helper.java
```

### Understanding Compilation and Bytecode

**What happens during compilation:**

```java
// Source Code (Main.java)
public class Main {
    public static void main(String[] args) {
        int x = 5;
        int y = 10;
        System.out.println(x + y);
    }
}
```

**Bytecode (Main.class - simplified view):**
```
bipush 5      // Push 5 onto stack
istore_1      // Store in local variable 1 (x)
bipush 10     // Push 10 onto stack
istore_2      // Store in local variable 2 (y)
iload_1       // Load x
iload_2       // Load y
iadd          // Add
invokevirtual // Call println
```

### JVM Architecture Deep Dive

```
┌─────────────────────────────────────┐
│        Java Source (.java)          │
└──────────────┬──────────────────────┘
               │ javac (compiler)
               ▼
┌─────────────────────────────────────┐
│       Bytecode (.class)             │
└──────────────┬──────────────────────┘
               │ JVM
               ▼
┌─────────────────────────────────────┐
│  ┌────────────────────────────┐    │
│  │     Class Loader           │    │
│  └──────────┬─────────────────┘    │
│             ▼                       │
│  ┌────────────────────────────┐    │
│  │  Bytecode Verifier         │    │
│  └──────────┬─────────────────┘    │
│             ▼                       │
│  ┌────────────────────────────┐    │
│  │  Execution Engine          │    │
│  │  - Interpreter             │    │
│  │  - JIT Compiler            │    │
│  └────────────────────────────┘    │
│                                     │
│  Memory Areas:                      │
│  ┌──────────┐  ┌───────────┐      │
│  │  Heap    │  │   Stack   │      │
│  │ (Objects)│  │ (Methods) │      │
│  └──────────┘  └───────────┘      │
└─────────────────────────────────────┘
```

### When to Use Source Launch vs Compilation

**Use Source Launch (`java Main.java`):**
- Scripts and utilities
- Learning and experimentation
- Quick testing
- Small programs (< 10 files)

**Use Traditional Compilation (`javac` + `java`):**
- Production applications
- Large projects
- When you need to distribute `.class` or `.jar` files
- Performance-critical applications (avoids compilation overhead)

### Performance Implications

```java
// Test program
void main() {
    long start = System.nanoTime();
    
    int sum = 0;
    for (int i = 0; i < 1_000_000; i++) {
        sum += i;
    }
    
    long end = System.nanoTime();
    println("Time: " + (end - start) / 1_000_000 + " ms");
    println("Sum: " + sum);
}
```

**Performance:**
- **Source Launch**: First run slower (compilation time), subsequent runs cached
- **Pre-compiled**: Consistently fast, no compilation overhead

---

## Variables & Data

### Type Inference with `var`

**Traditional Explicit Types:**
```java
void main() {
    String name = "John";
    int age = 25;
    ArrayList<String> items = new ArrayList<String>();
}
```

**Modern Type Inference:**
```java
void main() {
    var name = "John";  // Inferred as String
    var age = 25;       // Inferred as int
    var items = new ArrayList<String>();  // Inferred as ArrayList<String>
}
```

### When to Use `var`

**Good use of `var`:**
```java
void main() {
    // Clear from right-hand side
    var list = new ArrayList<String>();
    var map = new HashMap<String, Integer>();
    var scanner = new Scanner(System.in);
    
    // Loop variables
    for (var item : list) {
        println(item);
    }
    
    // Try-with-resources
    try (var reader = new BufferedReader(new FileReader("file.txt"))) {
        // ...
    }
}
```

**Bad use of `var`:**
```java
void main() {
    var x = calculate();  // What type is this?
    var data = getData(); // Unclear type
    var result = process(x, data);  // Too vague
}
```

**Rule of thumb:** Use `var` when the type is obvious from the right-hand side.

### Unnamed Variables `_`

**The Problem:**
```java
void main() {
    for (int i = 0; i < 10; i++) {
        println("Hello");  // 'i' is never used!
    }
    
    // Processing items, don't care about index
    List<String> items = List.of("a", "b", "c");
    for (int i = 0; i < items.size(); i++) {
        println(items.get(i));  // 'i' only used for indexing
    }
}
```

**The Solution: Unnamed Variables (JEP 456):**
```java
void main() {
    // Don't care about loop counter
    for (var _ : List.of(1, 2, 3, 4, 5)) {
        println("Hello");  // Clear intent: loop 5 times
    }
    
    // Don't care about the item, just the count
    List<String> items = List.of("a", "b", "c");
    int count = 0;
    for (var _ : items) {
        count++;
    }
    println("Count: " + count);
}
```

**Common Use Cases:**
```java
void main() {
    // 1. Loop N times
    for (var _ : List.of(1, 2, 3)) {
        println("Executing task");
    }
    
    // 2. Try-catch when you don't care about exception
    try {
        riskyOperation();
    } catch (Exception _) {
        println("Something went wrong");  // Don't need exception details
    }
    
    // 3. Lambda parameters (coming in later weeks)
    items.forEach(_ -> println("Processing"));
}
```

### Primitives vs Wrappers: Deep Dive

This is crucial for interviews! Understanding the difference will help you write efficient code.

**Primitive Types:**
```java
void main() {
    // 8 primitive types in Java
    byte b = 127;           // 8-bit  (-128 to 127)
    short s = 32000;        // 16-bit (-32,768 to 32,767)
    int i = 100;            // 32-bit (-2^31 to 2^31-1)
    long l = 100L;          // 64-bit (-2^63 to 2^63-1)
    
    float f = 3.14f;        // 32-bit floating point
    double d = 3.14159;     // 64-bit floating point
    
    boolean flag = true;    // true or false
    char c = 'A';          // 16-bit Unicode character
}
```

**Wrapper Classes:**
```java
void main() {
    // Wrapper classes (reference types)
    Byte b = 127;
    Short s = 32000;
    Integer i = 100;
    Long l = 100L;
    
    Float f = 3.14f;
    Double d = 3.14159;
    
    Boolean flag = true;
    Character c = 'A';
}
```

### Stack vs Heap Memory

**The Stack:**
- Stores primitives and references
- Fast allocation/deallocation
- Limited size
- Thread-specific (each thread has its own stack)
- Automatic memory management (scope-based)

**The Heap:**
- Stores objects
- Slower allocation/deallocation
- Larger size
- Shared across threads
- Garbage collected

**Visual Example:**
```java
void main() {
    int x = 5;              // Stack
    Integer y = 10;         // Reference on stack, object on heap
    String s = "Hello";     // Reference on stack, object on heap
    
    process(x, y, s);
}

void process(int a, Integer b, String str) {
    int local = 100;        // Stack
    Integer obj = 200;      // Reference on stack, object on heap
}
```

**Memory Layout:**
```
Stack (main thread):          Heap:
┌──────────────┐             ┌──────────────────┐
│ x = 5        │             │ Integer(10)      │ ← y points here
│ y = 0x1A2B   │────────────→│ address: 0x1A2B  │
│ s = 0x3C4D   │────┐        ├──────────────────┤
└──────────────┘    │        │ String("Hello")  │
                    └───────→│ address: 0x3C4D  │
Stack (process):             └──────────────────┘
┌──────────────┐
│ a = 5        │
│ b = 0x1A2B   │────────────→ (same Integer object)
│ str = 0x3C4D │────────────→ (same String object)
│ local = 100  │
│ obj = 0x5E6F │────────────→ Integer(200) (new object)
└──────────────┘
```

### Autoboxing and Unboxing

**Autoboxing:** Automatic conversion from primitive to wrapper
**Unboxing:** Automatic conversion from wrapper to primitive

```java
void main() {
    // Autoboxing
    Integer x = 5;  // int 5 automatically converted to Integer(5)
    
    // Unboxing
    int y = x;  // Integer x automatically converted to int
    
    // In collections (must use wrappers)
    List<Integer> numbers = new ArrayList<>();
    numbers.add(10);  // Autoboxing: int 10 → Integer(10)
    
    int first = numbers.get(0);  // Unboxing: Integer → int
}
```

### Performance Implications

```java
void main() {
    // Primitives - fast!
    long start = System.nanoTime();
    int sum1 = 0;
    for (int i = 0; i < 1_000_000; i++) {
        sum1 += i;
    }
    long time1 = System.nanoTime() - start;
    
    // Wrappers - slower!
    start = System.nanoTime();
    Integer sum2 = 0;
    for (Integer i = 0; i < 1_000_000; i++) {  // Autoboxing on every iteration!
        sum2 += i;  // Unboxing + addition + autoboxing!
    }
    long time2 = System.nanoTime() - start;
    
    println("Primitive time: " + time1 / 1_000_000 + " ms");
    println("Wrapper time: " + time2 / 1_000_000 + " ms");
    // Wrapper version is typically 5-10x slower!
}
```

### Integer Cache

**Important interview topic!**

```java
void main() {
    // Integer caching for -128 to 127
    Integer a = 100;
    Integer b = 100;
    println(a == b);  // true (same cached object!)
    
    Integer x = 200;
    Integer y = 200;
    println(x == y);  // false (different objects!)
    
    // Always use .equals() for wrappers
    println(x.equals(y));  // true
}
```

**Why caching?**
- Small integers (-128 to 127) are commonly used
- Java caches them to save memory
- Created only once, reused for efficiency

**Memory View:**
```
Integer Cache (-128 to 127):
┌────────────────┐
│ Integer(-128)  │
│ Integer(-127)  │
│ ...            │
│ Integer(100)   │ ← Both a and b point here
│ ...            │
│ Integer(127)   │
└────────────────┘

Heap (outside cache):
┌────────────────┐
│ Integer(200)   │ ← x points here
└────────────────┘
┌────────────────┐
│ Integer(200)   │ ← y points here (different object!)
└────────────────┘
```

### `null` and Primitives

```java
void main() {
    // Primitives cannot be null
    int x = null;  // ❌ Compilation error!
    
    // Wrappers can be null
    Integer y = null;  // ✓ Valid
    
    // Danger: NullPointerException
    Integer z = null;
    int a = z;  // ❌ Runtime error: NullPointerException (unboxing null)
    
    // Safe handling
    Integer value = null;
    int result = (value != null) ? value : 0;  // Safe
}
```

---

## Modern Text

### Traditional String Concatenation

```java
void main() {
    String name = "John";
    int age = 25;
    
    // Traditional concatenation
    String message = "Name: " + name + ", Age: " + age;
    
    // Multi-line (ugly!)
    String html = "<html>\n" +
                  "  <body>\n" +
                  "    <h1>Hello</h1>\n" +
                  "  </body>\n" +
                  "</html>";
}
```

### Text Blocks: `"""`

**Modern multi-line strings (Java 15+):**

```java
void main() {
    // Text blocks - clean and readable!
    String html = """
        <html>
          <body>
            <h1>Hello</h1>
          </body>
        </html>
        """;
    
    println(html);
}
```

### Text Block Rules

**1. Indentation is preserved relative to closing `"""`:**

```java
void main() {
    // Closing """ at start - no indentation removed
    String text1 = """
        Line 1
        Line 2
        """;
    // Result: "    Line 1\n    Line 2\n"
    
    // Closing """ indented - indentation removed
    String text2 = """
        Line 1
        Line 2
    """;
    // Result: "    Line 1\n    Line 2\n" (4 spaces preserved)
    
    // Closing """ aligned with content - minimal indentation
    String text3 = """
            Line 1
            Line 2
            """;
    // Result: "Line 1\nLine 2\n" (12 spaces removed from both)
}
```

**2. Escape sequences work:**

```java
void main() {
    String text = """
        First line
        Second line with \t tab
        Third line with \"quotes\"
        """;
}
```

**3. Line terminators are normalized to `\n`:**

```java
void main() {
    // Works on Windows (CRLF), Unix (LF), Mac (CR)
    // Always normalized to LF (\n)
    String text = """
        Line 1
        Line 2
        """;
}
```

### Real-World Text Block Examples

**JSON:**
```java
void main() {
    String name = "John";
    int age = 25;
    
    // Without text blocks
    String json1 = "{" +
                   "\"name\": \"" + name + "\"," +
                   "\"age\": " + age +
                   "}";
    
    // With text blocks (but still need concatenation)
    String json2 = """
        {
          "name": "%s",
          "age": %d
        }
        """.formatted(name, age);
    
    println(json2);
}
```

**SQL:**
```java
void main() {
    String table = "users";
    
    String query = """
        SELECT u.id, u.name, u.email, o.total
        FROM %s u
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE u.created_at > NOW() - INTERVAL '30 days'
        ORDER BY u.created_at DESC
        LIMIT 100
        """.formatted(table);
    
    println(query);
}
```

**HTML:**
```java
void main() {
    String title = "My Page";
    String content = "Welcome to my website!";
    
    String html = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>%s</title>
        </head>
        <body>
            <h1>%s</h1>
            <p>%s</p>
        </body>
        </html>
        """.formatted(title, title, content);
    
    println(html);
}
```

### String.formatted(): The Modern Way

**Traditional String Formatting:**
```java
void main() {
    String name = "John";
    int age = 25;
    double salary = 50000.50;
    
    // Old way: String.format()
    String msg1 = String.format("Name: %s, Age: %d, Salary: $%.2f", 
                                name, age, salary);
    
    // Old way: printf
    System.out.printf("Name: %s, Age: %d, Salary: $%.2f%n", 
                      name, age, salary);
}
```

**Modern Way: `formatted()`**
```java
void main() {
    String name = "John";
    int age = 25;
    double salary = 50000.50;
    
    // New way: .formatted()
    String msg = "Name: %s, Age: %d, Salary: $%.2f".formatted(name, age, salary);
    println(msg);
    
    // With text blocks
    String report = """
        Employee Report
        ===============
        Name: %s
        Age: %d
        Salary: $%.2f
        """.formatted(name, age, salary);
    
    println(report);
}
```

### Format Specifiers

**Common format specifiers:**

```java
void main() {
    // %s - String
    String s = "Format: %s".formatted("text");
    
    // %d - Integer (decimal)
    String d = "Number: %d".formatted(42);
    
    // %f - Floating point
    String f = "Float: %f".formatted(3.14159);  // 3.141590
    String f2 = "Float: %.2f".formatted(3.14159);  // 3.14 (2 decimal places)
    
    // %n - Platform-specific line separator
    String n = "Line 1%nLine 2".formatted();
    
    // %% - Literal %
    String percent = "Progress: %d%%".formatted(75);  // Progress: 75%
    
    // Padding and alignment
    String pad1 = "%10s".formatted("right");    // "     right" (10 chars, right-aligned)
    String pad2 = "%-10s".formatted("left");    // "left      " (10 chars, left-aligned)
    String pad3 = "%05d".formatted(42);         // "00042" (5 digits, zero-padded)
}
```

### Complete Example: Building a Receipt

```java
void main() {
    String storeName = "Tech Store";
    String item1 = "Laptop";
    String item2 = "Mouse";
    String item3 = "Keyboard";
    
    double price1 = 999.99;
    double price2 = 25.50;
    double price3 = 75.00;
    
    int qty1 = 1;
    int qty2 = 2;
    int qty3 = 1;
    
    double subtotal = (price1 * qty1) + (price2 * qty2) + (price3 * qty3);
    double tax = subtotal * 0.08;
    double total = subtotal + tax;
    
    String receipt = """
        ╔════════════════════════════════════════╗
        ║          %s                    ║
        ╠════════════════════════════════════════╣
        ║ Item                Qty    Price       ║
        ║ %-18s  %2d   $%8.2f  ║
        ║ %-18s  %2d   $%8.2f  ║
        ║ %-18s  %2d   $%8.2f  ║
        ╠════════════════════════════════════════╣
        ║ Subtotal:                  $%8.2f  ║
        ║ Tax (8%%):                  $%8.2f  ║
        ║ ────────────────────────────────────  ║
        ║ TOTAL:                     $%8.2f  ║
        ╚════════════════════════════════════════╝
        """.formatted(
            storeName,
            item1, qty1, price1 * qty1,
            item2, qty2, price2 * qty2,
            item3, qty3, price3 * qty3,
            subtotal, tax, total
        );
    
    println(receipt);
}
```

### String Immutability

**Critical concept for interviews:**

```java
void main() {
    // Strings are immutable
    String s1 = "Hello";
    String s2 = s1.concat(" World");
    
    println(s1);  // "Hello" (unchanged!)
    println(s2);  // "Hello World" (new object)
    
    // This creates many String objects!
    String result = "";
    for (int i = 0; i < 1000; i++) {
        result += i;  // Creates new String on each iteration!
    }
    
    // Better: Use StringBuilder
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 1000; i++) {
        sb.append(i);
    }
    String betterResult = sb.toString();
}
```

**Memory view:**
```
String Pool (Heap):
┌──────────────┐
│ "Hello"      │ ← s1 points here
└──────────────┘

Heap:
┌──────────────┐
│ "Hello World"│ ← s2 points here (new object)
└──────────────┘
```

### StringBuilder vs String

```java
void main() {
    // String concatenation (slow for many operations)
    long start = System.nanoTime();
    String s = "";
    for (int i = 0; i < 10000; i++) {
        s += "a";  // Creates 10,000 String objects!
    }
    long stringTime = System.nanoTime() - start;
    
    // StringBuilder (fast)
    start = System.nanoTime();
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 10000; i++) {
        sb.append("a");  // Modifies same object
    }
    String result = sb.toString();
    long sbTime = System.nanoTime() - start;
    
    println("String: " + stringTime / 1_000_000 + " ms");
    println("StringBuilder: " + sbTime / 1_000_000 + " ms");
    // StringBuilder is typically 100-1000x faster!
}
```

---

## Practice Exercises

### Exercise 1: Simple Calculator (Entry Point)

Create a calculator that reads two numbers and an operator, then performs the calculation.

**Requirements:**
- Use modern `void main()`
- Use `println()` and `readln()`
- Handle +, -, *, / operators

**Solution:**
```java
void main() {
    println("=== Simple Calculator ===");
    
    double num1 = Double.parseDouble(readln("Enter first number: "));
    String operator = readln("Enter operator (+, -, *, /): ");
    double num2 = Double.parseDouble(readln("Enter second number: "));
    
    double result = 0;
    boolean validOp = true;
    
    switch (operator) {
        case "+" -> result = num1 + num2;
        case "-" -> result = num1 - num2;
        case "*" -> result = num1 * num2;
        case "/" -> {
            if (num2 == 0) {
                println("Error: Division by zero!");
                validOp = false;
            } else {
                result = num1 / num2;
            }
        }
        default -> {
            println("Error: Invalid operator!");
            validOp = false;
        }
    }
    
    if (validOp) {
        println("Result: %.2f %s %.2f = %.2f".formatted(num1, operator, num2, result));
    }
}
```

### Exercise 2: Variable Types and Memory

Write a program that demonstrates the difference between primitives and wrappers.

**Solution:**
```java
void main() {
    // Primitives
    int p1 = 100;
    int p2 = 100;
    println("Primitives: p1 == p2: " + (p1 == p2));  // true
    
    // Wrappers (cached)
    Integer w1 = 100;
    Integer w2 = 100;
    println("Wrappers (cached): w1 == w2: " + (w1 == w2));  // true (cached!)
    println("Wrappers (cached): w1.equals(w2): " + w1.equals(w2));  // true
    
    // Wrappers (not cached)
    Integer w3 = 200;
    Integer w4 = 200;
    println("Wrappers (not cached): w3 == w4: " + (w3 == w4));  // false
    println("Wrappers (not cached): w3.equals(w4): " + w3.equals(w4));  // true
    
    // Null handling
    Integer nullable = null;
    int defaultValue = (nullable != null) ? nullable : 0;
    println("Null handling: " + defaultValue);  // 0
}
```

### Exercise 3: Text Blocks for JSON Generation

Create a program that generates a JSON file for multiple users.

**Solution:**
```java
import java.util.List;

void main() {
    var users = List.of(
        new User("john_doe", "John Doe", 25),
        new User("jane_smith", "Jane Smith", 30),
        new User("bob_wilson", "Bob Wilson", 28)
    );
    
    println("{");
    println("  \"users\": [");
    
    for (int i = 0; i < users.size(); i++) {
        User u = users.get(i);
        String comma = (i < users.size() - 1) ? "," : "";
        
        String userJson = """
                {
                  "username": "%s",
                  "fullName": "%s",
                  "age": %d
                }%s""".formatted(u.username, u.fullName, u.age, comma);
        
        println(userJson);
    }
    
    println("  ]");
    println("}");
}

record User(String username, String fullName, int age) {}
```

### Exercise 4: String Performance Test

Compare String concatenation vs StringBuilder performance.

**Solution:**
```java
void main() {
    int iterations = 50000;
    
    // Test 1: String concatenation
    long start = System.nanoTime();
    String s = "";
    for (int i = 0; i < iterations; i++) {
        s += "a";
    }
    long stringTime = (System.nanoTime() - start) / 1_000_000;
    
    // Test 2: StringBuilder
    start = System.nanoTime();
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < iterations; i++) {
        sb.append("a");
    }
    String sbResult = sb.toString();
    long sbTime = (System.nanoTime() - start) / 1_000_000;
    
    // Results
    String report = """
        
        Performance Test Results
        ========================
        Iterations: %,d
        
        String concatenation: %,d ms
        StringBuilder:        %,d ms
        
        Speedup: %.1fx faster
        """.formatted(
            iterations,
            stringTime,
            sbTime,
            (double) stringTime / sbTime
        );
    
    println(report);
}
```

### Exercise 5: Advanced - Module Imports Demo

Create a program demonstrating module imports with various Java utilities.

**Solution:**
```java
import module java.base;

void main() {
    // Collections
    List<String> names = new ArrayList<>();
    names.add("Alice");
    names.add("Bob");
    names.add("Charlie");
    
    // Map
    Map<String, Integer> ages = new HashMap<>();
    ages.put("Alice", 25);
    ages.put("Bob", 30);
    ages.put("Charlie", 28);
    
    // Math
    double pi = Math.PI;
    double sqrt = Math.sqrt(16);
    
    // Date/Time (preview for Week 4)
    var now = java.time.LocalDateTime.now();
    
    // Formatting output
    String output = """
        
        Module Import Demo
        ==================
        Names: %s
        Ages: %s
        
        Math Constants:
        - Pi: %.5f
        - sqrt(16): %.1f
        
        Current Time: %s
        """.formatted(names, ages, pi, sqrt, now);
    
    println(output);
}
```

### Exercise 6: Build a Temperature Converter

Create a comprehensive temperature converter with formatted output.

**Solution:**
```java
void main() {
    println("=== Temperature Converter ===\n");
    
    double celsius = Double.parseDouble(readln("Enter temperature in Celsius: "));
    
    double fahrenheit = (celsius * 9/5) + 32;
    double kelvin = celsius + 273.15;
    
    String result = """
        
        Temperature Conversions
        =======================
        Celsius:    %8.2f°C
        Fahrenheit: %8.2f°F
        Kelvin:     %8.2fK
        
        Formula Details:
        ----------------
        °F = (°C × 9/5) + 32
        K  = °C + 273.15
        """.formatted(celsius, fahrenheit, kelvin);
    
    println(result);
}
```

---

## Interview Questions & Answers

### Fundamental Questions

**Q1: What is the difference between JDK, JRE, and JVM?**

**Answer:**
- **JVM (Java Virtual Machine)**: The runtime engine that executes Java bytecode. It's platform-dependent and provides features like garbage collection and memory management.
  
- **JRE (Java Runtime Environment)**: JVM + core libraries needed to run Java applications. If you only want to run Java programs, you need JRE.
  
- **JDK (Java Development Kit)**: JRE + development tools (compiler `javac`, debugger, etc.). If you want to develop Java programs, you need JDK.

```
JDK = JRE + Development Tools (javac, debugger, etc.)
JRE = JVM + Libraries (java.lang, java.util, etc.)
JVM = Execution Engine
```

---

**Q2: Explain the compilation and execution process in Java.**

**Answer:**
1. **Source Code (.java)**: You write human-readable code
2. **Compilation**: `javac` compiles to platform-independent bytecode (.class)
3. **Class Loading**: JVM's class loader loads .class files into memory
4. **Bytecode Verification**: JVM verifies bytecode for security
5. **Execution**: 
   - Interpreter executes bytecode line-by-line
   - JIT (Just-In-Time) compiler compiles hot code to native machine code for performance
6. **Garbage Collection**: JVM automatically manages memory

**Key advantage:** "Write once, run anywhere" - bytecode is platform-independent.

---

**Q3: What are the advantages of Java's platform independence?**

**Answer:**
- **Portability**: Same bytecode runs on Windows, Linux, Mac
- **Cost-effective**: No need to maintain multiple codebases
- **Easy deployment**: Distribute .class or .jar files
- **Backward compatibility**: Older bytecode runs on newer JVMs

The bytecode acts as an intermediate representation that the JVM translates to native code for the specific platform.

---

### Modern Java Features

**Q4: What is JEP 477 and how does it simplify Java programs?**

**Answer:**
JEP 477 (Implicitly Declared Classes and Instance Main Methods) introduces:

1. **Simplified main method**: No need for `public static void main(String[] args)`
   ```java
   // Old
   public class Main {
       public static void main(String[] args) {
           System.out.println("Hello");
       }
   }
   
   // New
   void main() {
       println("Hello");
   }
   ```

2. **Implicit I/O**: `println()` and `readln()` instead of `System.out.println()` and Scanner

3. **Unnamed classes**: No need for class declaration for simple scripts

**Benefits**: 
- Reduces boilerplate for learning
- Makes Java more beginner-friendly
- Suitable for scripting and prototyping

---

**Q5: Explain module imports (JEP 476) with examples.**

**Answer:**
Module imports allow importing entire modules instead of individual classes:

```java
// Traditional
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

// Module import
import module java.base;
```

**Advantages:**
- Reduces import statements
- Useful when using many classes from same module
- Better for learning and prototyping

**Disadvantages:**
- Less explicit dependencies
- Can cause name conflicts
- Not recommended for production code

Common modules:
- `java.base`: Core Java (automatically imported)
- `java.sql`: Database
- `java.xml`: XML processing

---

**Q6: What are text blocks and when should you use them?**

**Answer:**
Text blocks (introduced in Java 15) allow multi-line string literals:

```java
String json = """
    {
        "name": "John",
        "age": 25
    }
    """;
```

**Use cases:**
- JSON/XML/HTML content
- SQL queries
- Multi-line messages
- Any formatted text

**Rules:**
- Must start with `"""`
- Opening `"""` must be followed by line terminator
- Indentation is relative to closing `"""`
- Line endings normalized to `\n`

**Benefits:**
- Improved readability
- No need for escape sequences for quotes
- No manual line breaks (`\n`)

---

### Variables and Memory

**Q7: Explain the difference between primitives and wrapper classes.**

**Answer:**

| Aspect | Primitive | Wrapper |
|--------|-----------|---------|
| Type | Value type | Reference type |
| Memory | Stack | Heap (reference on stack) |
| Null | Cannot be null | Can be null |
| Performance | Faster | Slower (object overhead) |
| Collections | Not allowed | Required |
| Default | 0, false, etc. | null |

**8 Primitives and Wrappers:**
```java
byte    → Byte
short   → Short
int     → Integer
long    → Long
float   → Float
double  → Double
boolean → Boolean
char    → Character
```

**When to use:**
- **Primitives**: Performance-critical code, local variables, mathematical operations
- **Wrappers**: Collections, when null is needed, working with frameworks

---

**Q8: What is autoboxing and unboxing? What are the performance implications?**

**Answer:**

**Autoboxing**: Automatic conversion from primitive to wrapper
```java
Integer x = 5;  // int 5 → Integer(5)
```

**Unboxing**: Automatic conversion from wrapper to primitive
```java
int y = x;  // Integer x → int
```

**Performance impact:**
```java
// Slow - autoboxing on each iteration
Integer sum = 0;
for (int i = 0; i < 1_000_000; i++) {
    sum += i;  // Unbox sum, add, autobox result
}

// Fast - no boxing
int sum = 0;
for (int i = 0; i < 1_000_000; i++) {
    sum += i;
}
```

**Pitfalls:**
```java
Integer a = null;
int b = a;  // NullPointerException during unboxing!
```

**Best practice**: Use primitives for loops and calculations, wrappers only when necessary.

---

**Q9: Explain the Integer cache. Why does it exist?**

**Answer:**

Java caches Integer objects for values -128 to 127:

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);  // true (same cached object)

Integer x = 200;
Integer y = 200;
System.out.println(x == y);  // false (different objects)
```

**Why caching exists:**
- Small integers are used frequently
- Saves memory by reusing objects
- Improves performance (no object creation)

**Cache range:**
- **Guaranteed**: -128 to 127
- **Configurable**: Can be extended via JVM flag `-XX:AutoBoxCacheMax=<size>`

**Best practice:**
Always use `.equals()` for wrapper comparison:
```java
Integer x = 200;
Integer y = 200;
x.equals(y);  // true - correct!
x == y;       // false - compares references!
```

---

**Q10: What is the difference between `==` and `.equals()` for Strings?**

**Answer:**

**`==`**: Compares references (memory addresses)
**`.equals()`**: Compares values (content)

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

s1 == s2;        // true (same object in String pool)
s1 == s3;        // false (different objects)
s1.equals(s3);   // true (same content)
```

**String Pool:**
Java maintains a pool of string literals for memory efficiency:
```java
String a = "Test";  // Goes to string pool
String b = "Test";  // Reuses from pool
// a and b point to same object

String c = new String("Test");  // Creates new object on heap
// c points to different object
```

**Best practice**: Always use `.equals()` for String comparison.

---

### Advanced Concepts

**Q11: What is `var` and when should you use it?**

**Answer:**

`var` is local variable type inference (Java 10+):

```java
var name = "John";  // Inferred as String
var age = 25;       // Inferred as int
var list = new ArrayList<String>();  // Inferred as ArrayList<String>
```

**Rules:**
- Only for local variables
- Must be initialized
- Cannot be null (need explicit type)
- Cannot be used for fields, parameters, or return types

**Good usage:**
```java
var map = new HashMap<String, List<Integer>>();  // Clear from RHS
for (var entry : map.entrySet()) { }  // Obvious type
```

**Bad usage:**
```java
var x = process();  // What type?
var data = getData();  // Unclear
```

**Best practice**: Use when type is obvious from right-hand side.

---

**Q12: What are unnamed variables (JEP 456) and when are they useful?**

**Answer:**

Unnamed variables (`_`) indicate intentionally unused variables:

```java
// Loop N times, don't care about value
for (var _ : List.of(1, 2, 3)) {
    println("Execute");
}

// Catch exception, don't need details
try {
    riskyOp();
} catch (Exception _) {
    println("Error occurred");
}
```

**Benefits:**
- **Intent clarity**: Shows variable is intentionally unused
- **Code clarity**: Reduces noise from unused variables
- **Future-proof**: Compiler knows it's intentional

**Common use cases:**
- Looping N times
- Exception handling (when you don't need exception object)
- Lambda parameters (covered in Week 5)

---

**Q13: Explain String immutability and its implications.**

**Answer:**

**Immutability**: Once created, a String object cannot be modified.

```java
String s = "Hello";
s.concat(" World");  // Creates new String, s unchanged
System.out.println(s);  // Still "Hello"

String s2 = s.concat(" World");
System.out.println(s2);  // "Hello World" (new object)
```

**Why immutable?**
1. **Security**: Strings used for passwords, file paths, etc.
2. **Thread-safety**: Can be safely shared across threads
3. **Caching**: String pool optimization
4. **Hashing**: Hash code can be cached

**Performance implications:**
```java
// Bad - creates many objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // Creates new String each time
}

// Good - use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

**Best practice**: Use StringBuilder for many concatenations.

---

**Q14: What is the difference between `String`, `StringBuilder`, and `StringBuffer`?**

**Answer:**

| Feature | String | StringBuilder | StringBuffer |
|---------|--------|---------------|--------------|
| Mutability | Immutable | Mutable | Mutable |
| Thread-safe | Yes (immutable) | No | Yes (synchronized) |
| Performance | Slow for many operations | Fast | Slower than StringBuilder |
| Use case | Few operations | Single-threaded | Multi-threaded |

**Examples:**
```java
// String - immutable
String s = "Hello";
s = s + " World";  // Creates new object

// StringBuilder - mutable, not synchronized
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // Modifies same object

// StringBuffer - mutable, synchronized
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World");  // Thread-safe
```

**Best practice:**
- Use `String` for immutable data
- Use `StringBuilder` for single-threaded string building
- Use `StringBuffer` only when thread-safety is needed

---

**Q15: What is the advantage of `String.formatted()` over `String.format()`?**

**Answer:**

Both do the same thing, but `formatted()` is more readable:

```java
// Old way
String msg = String.format("Name: %s, Age: %d", name, age);

// New way (Java 15+)
String msg = "Name: %s, Age: %d".formatted(name, age);
```

**Advantages of `formatted()`:**
- More fluent/readable
- Follows method chaining pattern
- Works naturally with text blocks:
  ```java
  String report = """
      Name: %s
      Age: %d
      """.formatted(name, age);
  ```

Functionally identical, `formatted()` is just syntactic sugar.

---

### Practical Scenarios

**Q16: How would you read a file and process it line-by-line efficiently?**

**Answer:**
```java
import java.nio.file.Files;
import java.nio.file.Paths;
import java.io.IOException;

void main() {
    try {
        // Modern way - Files.lines() with try-with-resources
        Files.lines(Paths.get("data.txt"))
             .forEach(line -> println(line));
             
    } catch (IOException e) {
        println("Error reading file: " + e.getMessage());
    }
}
```

**Key points:**
- Use `Files.lines()` for large files (lazy loading)
- Use try-with-resources for automatic closing
- Handles exceptions properly

---

**Q17: How would you create a formatted report with proper alignment?**

**Answer:**
```java
void main() {
    var products = List.of(
        new Product("Laptop", 999.99, 5),
        new Product("Mouse", 25.50, 15),
        new Product("Keyboard", 75.00, 10)
    );
    
    String report = """
        ┌──────────────┬────────┬──────┬───────────┐
        │ Product      │ Price  │ Qty  │ Total     │
        ├──────────────┼────────┼──────┼───────────┤
        """;
    
    StringBuilder sb = new StringBuilder(report);
    
    for (var p : products) {
        sb.append("│ %-12s │ $%6.2f │ %4d │ $%8.2f │\n".formatted(
            p.name, p.price, p.qty, p.price * p.qty
        ));
    }
    
    sb.append("└──────────────┴────────┴──────┴───────────┘");
    
    println(sb.toString());
}

record Product(String name, double price, int qty) {}
```

**Format specifiers:**
- `%-12s`: Left-aligned string, 12 chars wide
- `%6.2f`: Float, 6 total chars, 2 decimal places
- `%4d`: Integer, 4 chars wide

---

**Q18: Write a program to demonstrate memory usage difference between primitives and wrappers.**

**Answer:**
```java
void main() {
    int iterations = 10_000_000;
    
    // Test 1: Primitives
    long start = System.currentTimeMillis();
    int sum1 = 0;
    for (int i = 0; i < iterations; i++) {
        sum1 += i;
    }
    long primTime = System.currentTimeMillis() - start;
    
    // Test 2: Wrappers
    start = System.currentTimeMillis();
    Integer sum2 = 0;
    for (Integer i = 0; i < iterations; i++) {
        sum2 += i;
    }
    long wrapperTime = System.currentTimeMillis() - start;
    
    String result = """
        
        Performance Comparison
        ======================
        Iterations: %,d
        
        Primitive (int):  %,d ms
        Wrapper (Integer): %,d ms
        
        Wrapper is %.1fx slower
        
        Why? Each iteration involves:
        - Autoboxing sum2
        - Unboxing for addition
        - Autoboxing result
        - Object creation overhead
        """.formatted(
            iterations,
            primTime,
            wrapperTime,
            (double) wrapperTime / primTime
        );
    
    println(result);
}
```

---

**Q19: How do you handle exceptions when using new I/O methods?**

**Answer:**
```java
import java.io.IOException;

void main() {
    // Multiple approaches:
    
    // 1. Try-catch
    try {
        String content = Files.readString(Path.of("file.txt"));
        println(content);
    } catch (IOException e) {
        println("Error: " + e.getMessage());
    }
    
    // 2. Try-with-resources (auto-close)
    try (var reader = Files.newBufferedReader(Path.of("file.txt"))) {
        String line;
        while ((line = reader.readLine()) != null) {
            println(line);
        }
    } catch (IOException e) {
        println("Error: " + e.getMessage());
    }
    
    // 3. Catch and ignore (when appropriate)
    try {
        process();
    } catch (Exception _) {
        // Intentionally ignored
    }
}
```

---

**Q20: Design a mini-project combining all Week 1 concepts.**

**Answer: Student Grade Manager**

```java
import java.util.*;

void main() {
    println("=== Student Grade Manager ===\n");
    
    // Using modern features
    var students = new ArrayList<Student>();
    
    while (true) {
        String menu = """
            
            1. Add Student
            2. View All Students
            3. Calculate Class Average
            4. Exit
            
            Choice: """;
        
        print(menu);
        int choice = Integer.parseInt(readln(""));
        
        switch (choice) {
            case 1 -> addStudent(students);
            case 2 -> viewStudents(students);
            case 3 -> calculateAverage(students);
            case 4 -> {
                println("Goodbye!");
                return;
            }
            default -> println("Invalid choice!");
        }
    }
}

void addStudent(List<Student> students) {
    String name = readln("Enter student name: ");
    int age = Integer.parseInt(readln("Enter age: "));
    double grade = Double.parseDouble(readln("Enter grade: "));
    
    students.add(new Student(name, age, grade));
    println("Student added successfully!");
}

void viewStudents(List<Student> students) {
    if (students.isEmpty()) {
        println("No students found.");
        return;
    }
    
    String header = """
        
        ┌──────────────────┬──────┬────────┬────────┐
        │ Name             │ Age  │ Grade  │ Status │
        ├──────────────────┼──────┼────────┼────────┤
        """;
    
    StringBuilder report = new StringBuilder(header);
    
    for (var student : students) {
        String status = student.grade >= 60 ? "Pass" : "Fail";
        report.append("│ %-16s │ %4d │ %6.2f │ %-6s │\n".formatted(
            student.name, student.age, student.grade, status
        ));
    }
    
    report.append("└──────────────────┴──────┴────────┴────────┘");
    println(report.toString());
}

void calculateAverage(List<Student> students) {
    if (students.isEmpty()) {
        println("No students to calculate average.");
        return;
    }
    
    double sum = 0;
    for (var student : students) {
        sum += student.grade;
    }
    
    double average = sum / students.size();
    
    String result = """
        
        Class Statistics
        ================
        Total Students: %d
        Average Grade:  %.2f
        Status:         %s
        """.formatted(
            students.size(),
            average,
            average >= 60 ? "Passing" : "Needs Improvement"
        );
    
    println(result);
}

record Student(String name, int age, double grade) {}
```

**This example demonstrates:**
- Modern `void main()`
- `println()` and `readln()`
- Text blocks for formatted output
- `var` type inference
- Records (preview for Week 2)
- String formatting
- Collections (ArrayList)
- Enhanced for loops
- Switch expressions

---

## Week 1 Summary

### Key Concepts Mastered

1. **Entry Point Evolution**
   - Traditional `public static void main(String[] args)`
   - Modern `void main()`
   - Implicit I/O with `println()` and `readln()`

2. **Import System**
   - Traditional imports
   - Wildcard imports
   - Static imports
   - Module imports (`import module java.base`)

3. **Runtime Execution**
   - Traditional compile + run
   - Source launch (`java Main.java`)
   - Multi-file source launch

4. **Variables**
   - Primitives vs wrappers
   - Stack vs heap memory
   - Autoboxing and unboxing
   - Integer cache
   - Type inference with `var`
   - Unnamed variables `_`

5. **Modern Text**
   - Text blocks `"""`
   - String formatting with `formatted()`
   - String immutability
   - StringBuilder vs String performance

### Next Steps

In **Week 2**, we'll cover:
- Records and Record Patterns
- Sealed Classes
- Inheritance fundamentals
- Encapsulation deep dive
- The Module System

**Recommended Practice Before Week 2:**
1. Complete all exercises in this guide
2. Experiment with text blocks for different formats (JSON, SQL, HTML)
3. Benchmark primitive vs wrapper performance
4. Build a small CLI application using all Week 1 concepts
5. Review and memorize interview questions

**Additional Resources:**
- Java Documentation: https://docs.oracle.com/javase/
- JEP 477: https://openjdk.org/jeps/477
- JEP 476: https://openjdk.org/jeps/476
- JEP 458: https://openjdk.org/jeps/458

---

## Final Challenge: Build Your Own Project

Create a **Personal Finance Tracker** that demonstrates all Week 1 concepts:

**Requirements:**
1. Use modern `void main()` and implicit I/O
2. Store transactions with amount, category, description
3. Display formatted reports using text blocks
4. Calculate totals and statistics
5. Demonstrate primitive vs wrapper usage appropriately
6. Use `var` where appropriate
7. Handle input validation and errors
8. Create professional formatted output

This project will solidify your understanding and prepare you for Week 2!

---

**Congratulations on completing Week 1! You now have a solid foundation in modern Java syntax and fundamentals. Keep practicing, and get ready for OOP concepts in Week 2!**
