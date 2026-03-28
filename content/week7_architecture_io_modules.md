# Week 7: Architecture, IO & Advanced Modules - Master Java

## 📋 Table of Contents
1. [Java Platform Module System (JPMS)](#java-platform-module-system-jpms)
2. [Advanced Module Features](#advanced-module-features)
3. [Modern IO with NIO.2](#modern-io-with-nio2)
4. [Resource Management](#resource-management)
5. [Practice Exercises](#practice-exercises)
6. [Interview Questions & Answers](#interview-questions--answers)
7. [Real-World Scenarios](#real-world-scenarios)

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Master the Java Platform Module System (JPMS)
- Understand `exports` vs `opens` and their use cases
- Implement services using `uses` and `provides` with `ServiceLoader`
- Create custom runtime images with `jlink`
- Master modern IO operations with NIO.2 (`Path`, `Files`)
- Write robust code with `try-with-resources` and `AutoCloseable`
- Design modular, maintainable Java applications
- Confidently answer interview questions on modules and IO

---

## Java Platform Module System (JPMS)

### What is JPMS?

Introduced in Java 9, the **Java Platform Module System** (also called **Project Jigsaw**) provides:

1. **Strong Encapsulation**: Hide implementation details
2. **Reliable Configuration**: Explicit dependencies at compile time
3. **Scalable Platform**: Build smaller, optimized runtime images
4. **Better Security**: Reduce attack surface by limiting accessibility

### Before JPMS: The Classpath Problem

**Problems with the classpath:**

```java
// Before Java 9 - everything on classpath is accessible
// Problems:
// 1. JAR Hell: version conflicts
// 2. No encapsulation: all public classes accessible
// 3. Runtime errors: missing dependencies discovered at runtime
// 4. Huge runtime: JRE includes everything, even unused parts
```

### Module Basics

A **module** is a group of related packages with explicit dependencies.

#### Creating Your First Module

**Project Structure:**
```
myapp/
├── src/
│   └── com.example.app/
│       ├── module-info.java
│       └── com/
│           └── example/
│               └── app/
│                   └── Main.java
```

**module-info.java:**
```java
module com.example.app {
    // This is a module descriptor
}
```

**Main.java:**
```java
package com.example.app;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello from module!");
    }
}
```

**Compiling and Running:**
```bash
# Compile
javac -d out/com.example.app \
    src/com.example.app/module-info.java \
    src/com.example.app/com/example/app/Main.java

# Run
java --module-path out \
     --module com.example.app/com.example.app.Main
```

### Module Directives Overview

| Directive | Purpose | Example |
|-----------|---------|---------|
| `requires` | Declare dependency on another module | `requires java.sql;` |
| `exports` | Make package available to other modules | `exports com.example.api;` |
| `opens` | Allow reflective access to package | `opens com.example.internal;` |
| `uses` | Declare service consumption | `uses com.example.Service;` |
| `provides` | Declare service implementation | `provides Service with Impl;` |

### The `requires` Directive

Declares that your module depends on another module.

**Example: Using java.sql module**

```java
module com.example.database {
    requires java.sql;  // Dependency on java.sql module
}
```

**Types of `requires`:**

```java
module com.example.app {
    // Regular dependency
    requires com.example.util;
    
    // Transitive dependency (exports to consumers)
    requires transitive com.example.api;
    
    // Static dependency (optional at runtime)
    requires static com.example.optional;
}
```

**Example: Transitive Dependencies**

```java
// Module A
module com.example.api {
    exports com.example.api;
}

// Module B
module com.example.impl {
    requires transitive com.example.api;  // Transitive!
    exports com.example.impl;
}

// Module C
module com.example.client {
    requires com.example.impl;  // Gets com.example.api automatically!
    // Can use classes from both com.example.impl AND com.example.api
}
```

### The `exports` Directive

Makes packages available to other modules. **Only exported packages are accessible.**

**Basic Export:**

```java
module com.example.library {
    exports com.example.library.api;
    // com.example.library.internal is NOT exported (encapsulated)
}
```

**Project Structure:**
```
com.example.library/
├── module-info.java
└── com/
    └── example/
        └── library/
            ├── api/           ← Exported (public API)
            │   └── Service.java
            └── internal/      ← Not exported (implementation)
                └── ServiceImpl.java
```

**Example: Public API vs Internal Implementation**

```java
// module-info.java
module com.example.library {
    exports com.example.library.api;
    // internal package is hidden
}

// com/example/library/api/Service.java (EXPORTED)
package com.example.library.api;

public interface Service {
    String execute();
}

// com/example/library/api/ServiceFactory.java (EXPORTED)
package com.example.library.api;

import com.example.library.internal.ServiceImpl;

public class ServiceFactory {
    public static Service createService() {
        return new ServiceImpl();  // Can access internal from same module
    }
}

// com/example/library/internal/ServiceImpl.java (HIDDEN)
package com.example.library.internal;

import com.example.library.api.Service;

public class ServiceImpl implements Service {
    @Override
    public String execute() {
        return "Implementation details hidden!";
    }
}
```

**Client Module:**

```java
module com.example.client {
    requires com.example.library;
}

// Client code
package com.example.client;

import com.example.library.api.Service;
import com.example.library.api.ServiceFactory;
// import com.example.library.internal.ServiceImpl;  // ❌ COMPILATION ERROR!

public class Client {
    public static void main(String[] args) {
        Service service = ServiceFactory.createService();  // ✅ Works
        System.out.println(service.execute());
        
        // new ServiceImpl();  // ❌ Cannot access - not exported
    }
}
```

**Qualified Export (Export to Specific Modules):**

```java
module com.example.library {
    exports com.example.library.api;
    exports com.example.library.internal to com.example.test;  // Only to test module
}
```

---

## Advanced Module Features

### `exports` vs `opens`

| Directive | Access | Reflection | Use Case |
|-----------|--------|------------|----------|
| `exports` | ✅ Compile-time | ❌ Reflection blocked | Public APIs |
| `opens` | ❌ No compile access | ✅ Full reflection | Frameworks (DI, serialization) |

**Understanding the Difference:**

```java
module com.example.app {
    // exports: Others can import and use at compile time
    exports com.example.app.api;
    
    // opens: Others CANNOT import, but reflection works at runtime
    opens com.example.app.entities;
    
    // opens to: Qualified opens (only specific modules)
    opens com.example.app.internal to com.google.gson;
}
```

### Why `opens` Exists: The Reflection Problem

Before modules, all public classes were accessible via reflection. With modules, reflection is blocked by default for strong encapsulation.

**Example: Frameworks Need Reflection**

```java
// Entity class (should not be exported as API)
package com.example.app.entities;

public class User {
    private Long id;
    private String name;
    
    // Framework needs to access private fields via reflection
    // for serialization/deserialization
}

// module-info.java
module com.example.app {
    // DON'T export entities (not part of API)
    // exports com.example.app.entities;  ❌ Bad practice
    
    // Instead, open for reflection
    opens com.example.app.entities;  // ✅ Correct
    
    requires com.google.gson;
}

// Usage
public class Example {
    public static void main(String[] args) {
        Gson gson = new Gson();
        User user = new User();
        user.setName("Alice");
        
        // Gson can access private fields via reflection
        String json = gson.toJson(user);  // ✅ Works because of 'opens'
    }
}
```

### Real-World Example: Library with Public API and Hidden Implementation

**Project Structure:**
```
mathlib/
└── src/
    └── com.example.mathlib/
        ├── module-info.java
        └── com/
            └── example/
                └── mathlib/
                    ├── api/
                    │   ├── Calculator.java
                    │   └── CalculatorFactory.java
                    └── internal/
                        └── AdvancedCalculator.java
```

**module-info.java:**
```java
module com.example.mathlib {
    exports com.example.mathlib.api;  // Public API only
    // internal package is completely hidden
}
```

**com/example/mathlib/api/Calculator.java:**
```java
package com.example.mathlib.api;

public interface Calculator {
    double add(double a, double b);
    double multiply(double a, double b);
}
```

**com/example/mathlib/api/CalculatorFactory.java:**
```java
package com.example.mathlib.api;

import com.example.mathlib.internal.AdvancedCalculator;

public class CalculatorFactory {
    public static Calculator createCalculator() {
        return new AdvancedCalculator();
    }
}
```

**com/example/mathlib/internal/AdvancedCalculator.java:**
```java
package com.example.mathlib.internal;

import com.example.mathlib.api.Calculator;

public class AdvancedCalculator implements Calculator {
    @Override
    public double add(double a, double b) {
        // Complex implementation
        return a + b;
    }
    
    @Override
    public double multiply(double a, double b) {
        return a * b;
    }
}
```

### Services: `uses` and `provides` with `ServiceLoader`

**Service Provider Interface (SPI)** pattern allows loose coupling between API and implementation.

#### Understanding the Service Pattern

**Without Services (Tight Coupling):**
```java
// Client directly depends on implementation
Calculator calc = new AdvancedCalculator();  // ❌ Tight coupling
```

**With Services (Loose Coupling):**
```java
// Client depends only on interface, implementation discovered at runtime
ServiceLoader<Calculator> loader = ServiceLoader.load(Calculator.class);
Calculator calc = loader.findFirst().orElseThrow();  // ✅ Loose coupling
```

#### Complete Service Example

**1. API Module (Interface Definition)**

```java
// module-info.java
module com.example.calculator.api {
    exports com.example.calculator.api;
}

// com/example/calculator/api/Calculator.java
package com.example.calculator.api;

public interface Calculator {
    double calculate(double a, double b);
    String getOperationName();
}
```

**2. Implementation Module (Service Provider)**

```java
// module-info.java
module com.example.calculator.basic {
    requires com.example.calculator.api;
    provides com.example.calculator.api.Calculator 
        with com.example.calculator.basic.AddCalculator;
}

// com/example/calculator/basic/AddCalculator.java
package com.example.calculator.basic;

import com.example.calculator.api.Calculator;

public class AddCalculator implements Calculator {
    @Override
    public double calculate(double a, double b) {
        return a + b;
    }
    
    @Override
    public String getOperationName() {
        return "Addition";
    }
}
```

**3. Consumer Module (Uses Service)**

```java
// module-info.java
module com.example.calculator.client {
    requires com.example.calculator.api;
    uses com.example.calculator.api.Calculator;
}

// com/example/calculator/client/Main.java
package com.example.calculator.client;

import com.example.calculator.api.Calculator;
import java.util.ServiceLoader;

public class Main {
    public static void main(String[] args) {
        ServiceLoader<Calculator> loader = ServiceLoader.load(Calculator.class);
        
        // Discover all implementations
        for (Calculator calc : loader) {
            System.out.println(calc.getOperationName() + ": " + 
                             calc.calculate(10, 5));
        }
    }
}
```

#### Multiple Service Providers

```java
// Module: com.example.calculator.basic
module com.example.calculator.basic {
    requires com.example.calculator.api;
    provides com.example.calculator.api.Calculator 
        with com.example.calculator.basic.AddCalculator,
             com.example.calculator.basic.SubtractCalculator;
}

// AddCalculator.java
public class AddCalculator implements Calculator {
    public double calculate(double a, double b) { return a + b; }
    public String getOperationName() { return "Addition"; }
}

// SubtractCalculator.java
public class SubtractCalculator implements Calculator {
    public double calculate(double a, double b) { return a - b; }
    public String getOperationName() { return "Subtraction"; }
}
```

#### Advanced ServiceLoader Usage

```java
public class CalculatorClient {
    public static void main(String[] args) {
        ServiceLoader<Calculator> loader = ServiceLoader.load(Calculator.class);
        
        // 1. Find first available
        Calculator calc = loader.findFirst()
            .orElseThrow(() -> new RuntimeException("No calculator found"));
        
        // 2. Stream API (Java 9+)
        loader.stream()
            .map(ServiceLoader.Provider::get)
            .forEach(c -> System.out.println(c.getOperationName()));
        
        // 3. Filter and select
        Calculator addCalc = loader.stream()
            .map(ServiceLoader.Provider::get)
            .filter(c -> c.getOperationName().equals("Addition"))
            .findFirst()
            .orElse(null);
        
        // 4. Reload (discover new implementations)
        loader.reload();
    }
}
```

#### Real-World Use Case: Plugin Architecture

**Database Driver Plugin System:**

```java
// ========== API Module ==========
module com.example.db.api {
    exports com.example.db.api;
}

package com.example.db.api;

public interface DatabaseDriver {
    String getDriverName();
    Connection connect(String url, String user, String password);
    boolean supports(String url);
}

// ========== MySQL Plugin ==========
module com.example.db.mysql {
    requires com.example.db.api;
    provides com.example.db.api.DatabaseDriver 
        with com.example.db.mysql.MySQLDriver;
}

public class MySQLDriver implements DatabaseDriver {
    @Override
    public String getDriverName() {
        return "MySQL Driver";
    }
    
    @Override
    public boolean supports(String url) {
        return url.startsWith("jdbc:mysql:");
    }
    
    @Override
    public Connection connect(String url, String user, String password) {
        // MySQL-specific connection logic
        return new MySQLConnection(url, user, password);
    }
}

// ========== PostgreSQL Plugin ==========
module com.example.db.postgresql {
    requires com.example.db.api;
    provides com.example.db.api.DatabaseDriver 
        with com.example.db.postgresql.PostgreSQLDriver;
}

public class PostgreSQLDriver implements DatabaseDriver {
    @Override
    public String getDriverName() {
        return "PostgreSQL Driver";
    }
    
    @Override
    public boolean supports(String url) {
        return url.startsWith("jdbc:postgresql:");
    }
    
    @Override
    public Connection connect(String url, String user, String password) {
        return new PostgreSQLConnection(url, user, password);
    }
}

// ========== Application ==========
module com.example.db.app {
    requires com.example.db.api;
    uses com.example.db.api.DatabaseDriver;
}

public class DatabaseManager {
    private final Map<String, DatabaseDriver> drivers = new HashMap<>();
    
    public DatabaseManager() {
        // Load all available drivers
        ServiceLoader.load(DatabaseDriver.class)
            .forEach(driver -> {
                drivers.put(driver.getDriverName(), driver);
                System.out.println("Loaded: " + driver.getDriverName());
            });
    }
    
    public Connection connect(String url, String user, String password) {
        return drivers.values().stream()
            .filter(driver -> driver.supports(url))
            .findFirst()
            .map(driver -> driver.connect(url, user, password))
            .orElseThrow(() -> new IllegalArgumentException("No driver for: " + url));
    }
}
```

### Custom Runtime Images with `jlink`

`jlink` creates optimized, standalone Java runtime images containing only the modules your application needs.

#### Why jlink?

**Traditional Deployment:**
- Ship entire JRE (~200 MB)
- Includes unused modules
- Larger download size

**With jlink:**
- Custom runtime with only required modules
- Much smaller (can be ~30-50 MB)
- Faster startup
- Better security (fewer modules = smaller attack surface)

#### Basic jlink Usage

**Step 1: Compile your modular application**
```bash
javac -d out/com.example.app \
    src/com.example.app/module-info.java \
    src/com.example.app/**/*.java
```

**Step 2: Create custom runtime image**
```bash
jlink --module-path $JAVA_HOME/jmods:out \
      --add-modules com.example.app \
      --output myapp-runtime \
      --launcher myapp=com.example.app/com.example.app.Main
```

**Step 3: Run your application**
```bash
./myapp-runtime/bin/myapp
```

#### jlink Options

```bash
jlink \
  --module-path $JAVA_HOME/jmods:out \  # Where to find modules
  --add-modules com.example.app \         # Modules to include
  --output myapp-runtime \                # Output directory
  --launcher myapp=com.example.app/com.example.app.Main \  # Create launcher script
  --strip-debug \                         # Remove debug info (smaller size)
  --compress=2 \                          # Compress resources (0=none, 1=strings, 2=all)
  --no-header-files \                     # Exclude header files
  --no-man-pages                          # Exclude man pages
```

#### Complete jlink Example

**Project Structure:**
```
myapp/
├── src/
│   └── com.example.app/
│       ├── module-info.java
│       └── com/
│           └── example/
│               └── app/
│                   └── Main.java
└── build.sh
```

**module-info.java:**
```java
module com.example.app {
    requires java.base;  // Implicit, but shown for clarity
    requires java.sql;
    requires java.logging;
}
```

**Main.java:**
```java
package com.example.app;

import java.sql.*;
import java.util.logging.*;

public class Main {
    private static final Logger LOGGER = Logger.getLogger(Main.class.getName());
    
    public static void main(String[] args) {
        LOGGER.info("Application started");
        System.out.println("Custom Runtime Image!");
        LOGGER.info("Application finished");
    }
}
```

**build.sh:**
```bash
#!/bin/bash

# Clean
rm -rf out dist

# Compile
mkdir -p out/com.example.app
javac -d out/com.example.app \
    src/com.example.app/module-info.java \
    src/com.example.app/com/example/app/Main.java

# Create runtime image
jlink --module-path $JAVA_HOME/jmods:out \
      --add-modules com.example.app \
      --output dist/myapp \
      --launcher myapp=com.example.app/com.example.app.Main \
      --strip-debug \
      --compress=2 \
      --no-header-files \
      --no-man-pages

# Show size comparison
echo "Full JDK size: $(du -sh $JAVA_HOME | cut -f1)"
echo "Custom runtime size: $(du -sh dist/myapp | cut -f1)"

# Run
echo "Running application..."
./dist/myapp/bin/myapp
```

#### Analyzing Module Dependencies

Before creating a runtime image, analyze dependencies:

```bash
# List all dependencies of your module
jdeps --module-path out -m com.example.app

# Generate graphical dependency graph
jdeps --module-path out -m com.example.app --dot-output .
dot -Tpng summary.dot -o dependencies.png

# Check for missing dependencies
jdeps --check com.example.app --module-path out
```

---

## Modern IO with NIO.2

### From Old IO to NIO.2

**Old IO (java.io):**
```java
File file = new File("data.txt");
FileInputStream fis = new FileInputStream(file);
// Limited functionality, verbose
```

**NIO.2 (java.nio.file):**
```java
Path path = Path.of("data.txt");
String content = Files.readString(path);
// Modern, concise, powerful
```

### The Path Interface

`Path` represents a file system path (file or directory).

#### Creating Paths

```java
import java.nio.file.Path;
import java.nio.file.Paths;

public class PathExamples {
    public static void main(String[] args) {
        // Method 1: Path.of() (Java 11+)
        Path path1 = Path.of("data.txt");
        Path path2 = Path.of("src", "main", "java", "Main.java");
        Path path3 = Path.of("/usr/local/bin");
        
        // Method 2: Paths.get() (Java 7+)
        Path path4 = Paths.get("data.txt");
        Path path5 = Paths.get("src", "main", "java");
        
        // From URI
        Path path6 = Path.of(URI.create("file:///home/user/data.txt"));
        
        // Current directory
        Path currentDir = Path.of(".");
        Path absoluteCurrent = currentDir.toAbsolutePath();
        
        System.out.println("Current: " + absoluteCurrent);
    }
}
```

#### Path Operations

```java
import java.nio.file.*;

public class PathOperations {
    public static void main(String[] args) {
        Path path = Path.of("src/main/java/com/example/Main.java");
        
        // Get components
        System.out.println("File name: " + path.getFileName());  // Main.java
        System.out.println("Parent: " + path.getParent());       // src/main/java/com/example
        System.out.println("Root: " + path.getRoot());           // null (relative path)
        System.out.println("Name count: " + path.getNameCount()); // 5
        System.out.println("Name at 2: " + path.getName(2));     // java
        
        // Absolute vs Relative
        System.out.println("Is absolute: " + path.isAbsolute()); // false
        Path absolute = path.toAbsolutePath();
        System.out.println("Absolute: " + absolute);
        
        // Normalize (resolve . and ..)
        Path messy = Path.of("src/../src/./main/java");
        System.out.println("Normalized: " + messy.normalize());  // src/main/java
        
        // Resolve (combine paths)
        Path base = Path.of("/home/user");
        Path resolved = base.resolve("documents/file.txt");
        System.out.println("Resolved: " + resolved);  // /home/user/documents/file.txt
        
        // Relativize (create relative path)
        Path from = Path.of("/home/user/documents");
        Path to = Path.of("/home/user/pictures/photo.jpg");
        Path relative = from.relativize(to);
        System.out.println("Relative: " + relative);  // ../pictures/photo.jpg
    }
}
```

### The Files Utility Class

`Files` provides static methods for common file operations.

#### Reading Files

```java
import java.nio.file.*;
import java.io.IOException;
import java.util.List;
import java.util.stream.Stream;

public class ReadingFiles {
    public static void main(String[] args) throws IOException {
        Path path = Path.of("data.txt");
        
        // 1. Read entire file as String (Java 11+)
        String content = Files.readString(path);
        System.out.println(content);
        
        // 2. Read all lines into List
        List<String> lines = Files.readAllLines(path);
        lines.forEach(System.out::println);
        
        // 3. Read as Stream (memory-efficient for large files)
        try (Stream<String> stream = Files.lines(path)) {
            stream
                .filter(line -> line.contains("important"))
                .forEach(System.out::println);
        }
        
        // 4. Read as bytes
        byte[] bytes = Files.readAllBytes(path);
        
        // 5. Buffered reading (old-style, but still useful)
        try (var reader = Files.newBufferedReader(path)) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }
    }
}
```

#### Writing Files

```java
import java.nio.file.*;
import java.io.IOException;
import java.util.List;

public class WritingFiles {
    public static void main(String[] args) throws IOException {
        Path path = Path.of("output.txt");
        
        // 1. Write String to file (Java 11+)
        Files.writeString(path, "Hello, World!");
        
        // 2. Write with StandardOpenOption
        Files.writeString(path, "\nNew line", 
            StandardOpenOption.APPEND);  // Append instead of overwrite
        
        // 3. Write lines
        List<String> lines = List.of("Line 1", "Line 2", "Line 3");
        Files.write(path, lines);
        
        // 4. Write bytes
        byte[] bytes = "Binary data".getBytes();
        Files.write(path, bytes);
        
        // 5. Buffered writing (for large data)
        try (var writer = Files.newBufferedWriter(path)) {
            writer.write("Line 1\n");
            writer.write("Line 2\n");
        }
        
        // Common options
        Files.writeString(path, "Content",
            StandardOpenOption.CREATE,        // Create if not exists
            StandardOpenOption.TRUNCATE_EXISTING,  // Clear existing content
            StandardOpenOption.WRITE);
    }
}
```

#### File Operations

```java
import java.nio.file.*;
import java.io.IOException;

public class FileOperations {
    public static void main(String[] args) throws IOException {
        Path source = Path.of("source.txt");
        Path target = Path.of("target.txt");
        Path dir = Path.of("mydir");
        
        // Create file
        if (!Files.exists(source)) {
            Files.createFile(source);
        }
        
        // Create directory
        Files.createDirectory(dir);
        
        // Create directories (including parents)
        Files.createDirectories(Path.of("a/b/c/d"));
        
        // Copy file
        Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
        
        // Move/Rename file
        Files.move(source, target, StandardCopyOption.REPLACE_EXISTING);
        
        // Delete file
        Files.delete(target);  // Throws exception if not exists
        Files.deleteIfExists(target);  // No exception
        
        // Check existence
        boolean exists = Files.exists(source);
        boolean notExists = Files.notExists(source);
        
        // Check type
        boolean isFile = Files.isRegularFile(source);
        boolean isDir = Files.isDirectory(dir);
        boolean isSymLink = Files.isSymbolicLink(source);
        
        // Check permissions
        boolean readable = Files.isReadable(source);
        boolean writable = Files.isWritable(source);
        boolean executable = Files.isExecutable(source);
        
        // Get size
        long size = Files.size(source);
        System.out.println("Size: " + size + " bytes");
        
        // Get modification time
        var modTime = Files.getLastModifiedTime(source);
        System.out.println("Modified: " + modTime);
    }
}
```

#### Walking Directory Trees

```java
import java.nio.file.*;
import java.io.IOException;
import java.util.stream.Stream;

public class DirectoryWalking {
    public static void main(String[] args) throws IOException {
        Path start = Path.of("src");
        
        // 1. List directory contents (one level)
        try (Stream<Path> stream = Files.list(start)) {
            stream.forEach(System.out::println);
        }
        
        // 2. Walk directory tree (all levels)
        try (Stream<Path> stream = Files.walk(start)) {
            stream
                .filter(Files::isRegularFile)
                .filter(p -> p.toString().endsWith(".java"))
                .forEach(System.out::println);
        }
        
        // 3. Walk with depth limit
        try (Stream<Path> stream = Files.walk(start, 2)) {  // Max 2 levels
            stream.forEach(System.out::println);
        }
        
        // 4. Find files (with matcher)
        try (Stream<Path> stream = Files.find(start, Integer.MAX_VALUE,
                (path, attrs) -> path.toString().endsWith(".java") 
                              && attrs.size() > 1000)) {
            stream.forEach(System.out::println);
        }
    }
    
    // Example: Count files by extension
    public static void countFilesByExtension(Path directory) throws IOException {
        try (Stream<Path> stream = Files.walk(directory)) {
            stream
                .filter(Files::isRegularFile)
                .collect(Collectors.groupingBy(
                    path -> {
                        String name = path.getFileName().toString();
                        int lastDot = name.lastIndexOf('.');
                        return lastDot > 0 ? name.substring(lastDot) : "no-extension";
                    },
                    Collectors.counting()
                ))
                .forEach((ext, count) -> 
                    System.out.println(ext + ": " + count + " files"));
        }
    }
    
    // Example: Copy directory tree
    public static void copyDirectory(Path source, Path target) throws IOException {
        Files.walk(source).forEach(src -> {
            try {
                Path dest = target.resolve(source.relativize(src));
                if (Files.isDirectory(src)) {
                    Files.createDirectories(dest);
                } else {
                    Files.copy(src, dest, StandardCopyOption.REPLACE_EXISTING);
                }
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        });
    }
    
    // Example: Delete directory tree
    public static void deleteDirectory(Path directory) throws IOException {
        if (Files.exists(directory)) {
            Files.walk(directory)
                .sorted(Comparator.reverseOrder())  // Delete files before directories
                .forEach(path -> {
                    try {
                        Files.delete(path);
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                });
        }
    }
}
```

#### Watching for File Changes

```java
import java.nio.file.*;
import java.io.IOException;

public class FileWatcher {
    public static void main(String[] args) throws IOException, InterruptedException {
        Path directory = Path.of("watched-dir");
        Files.createDirectories(directory);
        
        // Create watch service
        try (WatchService watchService = FileSystems.getDefault().newWatchService()) {
            
            // Register directory for events
            directory.register(watchService,
                StandardWatchEventKinds.ENTRY_CREATE,
                StandardWatchEventKinds.ENTRY_MODIFY,
                StandardWatchEventKinds.ENTRY_DELETE);
            
            System.out.println("Watching: " + directory);
            
            // Watch loop
            while (true) {
                WatchKey key = watchService.take();  // Blocks until event occurs
                
                for (WatchEvent<?> event : key.pollEvents()) {
                    WatchEvent.Kind<?> kind = event.kind();
                    Path filename = (Path) event.context();
                    
                    if (kind == StandardWatchEventKinds.ENTRY_CREATE) {
                        System.out.println("Created: " + filename);
                    } else if (kind == StandardWatchEventKinds.ENTRY_MODIFY) {
                        System.out.println("Modified: " + filename);
                    } else if (kind == StandardWatchEventKinds.ENTRY_DELETE) {
                        System.out.println("Deleted: " + filename);
                    }
                }
                
                // Reset key to receive further events
                if (!key.reset()) {
                    break;
                }
            }
        }
    }
}
```

#### File Attributes

```java
import java.nio.file.*;
import java.nio.file.attribute.*;
import java.io.IOException;

public class FileAttributes {
    public static void main(String[] args) throws IOException {
        Path path = Path.of("file.txt");
        
        // Basic attributes
        BasicFileAttributes attrs = Files.readAttributes(path, BasicFileAttributes.class);
        
        System.out.println("Creation time: " + attrs.creationTime());
        System.out.println("Last modified: " + attrs.lastModifiedTime());
        System.out.println("Last accessed: " + attrs.lastAccessTime());
        System.out.println("Size: " + attrs.size());
        System.out.println("Is directory: " + attrs.isDirectory());
        System.out.println("Is regular file: " + attrs.isRegularFile());
        System.out.println("Is symbolic link: " + attrs.isSymbolicLink());
        
        // Set modification time
        FileTime newTime = FileTime.fromMillis(System.currentTimeMillis());
        Files.setLastModifiedTime(path, newTime);
        
        // POSIX attributes (Unix/Linux)
        if (FileSystems.getDefault().supportedFileAttributeViews().contains("posix")) {
            PosixFileAttributes posixAttrs = 
                Files.readAttributes(path, PosixFileAttributes.class);
            
            System.out.println("Owner: " + posixAttrs.owner());
            System.out.println("Group: " + posixAttrs.group());
            System.out.println("Permissions: " + 
                PosixFilePermissions.toString(posixAttrs.permissions()));
            
            // Set permissions (rwxr-xr-x)
            Set<PosixFilePermission> perms = PosixFilePermissions.fromString("rwxr-xr-x");
            Files.setPosixFilePermissions(path, perms);
        }
        
        // DOS attributes (Windows)
        if (FileSystems.getDefault().supportedFileAttributeViews().contains("dos")) {
            DosFileAttributes dosAttrs = 
                Files.readAttributes(path, DosFileAttributes.class);
            
            System.out.println("Hidden: " + dosAttrs.isHidden());
            System.out.println("Read-only: " + dosAttrs.isReadOnly());
            System.out.println("System: " + dosAttrs.isSystem());
            System.out.println("Archive: " + dosAttrs.isArchive());
            
            // Set hidden
            Files.setAttribute(path, "dos:hidden", true);
        }
    }
}
```

---

## Resource Management

### The Problem: Resource Leaks

**Before Java 7:**
```java
// ❌ Verbose and error-prone
FileInputStream fis = null;
try {
    fis = new FileInputStream("file.txt");
    // Use fis
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fis != null) {
        try {
            fis.close();  // Must close manually
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### try-with-resources (Java 7+)

**Automatic resource management:**

```java
// ✅ Clean and safe
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // Use fis
}  // Automatically closed, even if exception occurs
catch (IOException e) {
    e.printStackTrace();
}
```

### The AutoCloseable Interface

Resources used in try-with-resources must implement `AutoCloseable`:

```java
public interface AutoCloseable {
    void close() throws Exception;
}

// Specialized version (narrower exception)
public interface Closeable extends AutoCloseable {
    void close() throws IOException;
}
```

### Examples with Multiple Resources

```java
import java.io.*;
import java.nio.file.*;

public class TryWithResourcesExamples {
    // Multiple resources (closed in reverse order)
    public static void copyFile(String source, String target) throws IOException {
        try (InputStream in = Files.newInputStream(Path.of(source));
             OutputStream out = Files.newOutputStream(Path.of(target))) {
            
            byte[] buffer = new byte[8192];
            int n;
            while ((n = in.read(buffer)) > 0) {
                out.write(buffer, 0, n);
            }
        }  // out.close() called first, then in.close()
    }
    
    // Modern NIO.2 version
    public static void copyFileModern(String source, String target) throws IOException {
        Files.copy(Path.of(source), Path.of(target), 
                  StandardCopyOption.REPLACE_EXISTING);
    }
    
    // BufferedReader example
    public static void readFile(String filename) throws IOException {
        try (BufferedReader reader = Files.newBufferedReader(Path.of(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }
    }
    
    // BufferedWriter example
    public static void writeFile(String filename, List<String> lines) throws IOException {
        try (BufferedWriter writer = Files.newBufferedWriter(Path.of(filename))) {
            for (String line : lines) {
                writer.write(line);
                writer.newLine();
            }
        }
    }
}
```

### Creating Custom AutoCloseable Resources

```java
public class DatabaseConnection implements AutoCloseable {
    private final String connectionString;
    private boolean closed = false;
    
    public DatabaseConnection(String connectionString) {
        this.connectionString = connectionString;
        System.out.println("Opening connection to: " + connectionString);
    }
    
    public void executeQuery(String query) {
        if (closed) {
            throw new IllegalStateException("Connection is closed");
        }
        System.out.println("Executing: " + query);
    }
    
    @Override
    public void close() {
        if (!closed) {
            System.out.println("Closing connection to: " + connectionString);
            closed = true;
        }
    }
    
    // Usage
    public static void main(String[] args) {
        try (DatabaseConnection conn = new DatabaseConnection("jdbc:mysql://localhost")) {
            conn.executeQuery("SELECT * FROM users");
        }  // Automatically closed
        
        // With multiple resources
        try (DatabaseConnection conn1 = new DatabaseConnection("db1");
             DatabaseConnection conn2 = new DatabaseConnection("db2")) {
            conn1.executeQuery("SELECT * FROM table1");
            conn2.executeQuery("SELECT * FROM table2");
        }  // conn2 closed first, then conn1
    }
}
```

### Enhanced try-with-resources (Java 9+)

**Java 7-8:** Variables must be declared in try header

```java
BufferedReader reader = Files.newBufferedReader(path);
try (BufferedReader r = reader) {  // Must create new variable
    // Use r
}
```

**Java 9+:** Can use effectively final variables directly

```java
BufferedReader reader = Files.newBufferedReader(path);
try (reader) {  // ✅ Can use existing variable
    // Use reader
}
```

### Exception Handling with try-with-resources

```java
public class ResourceExceptionHandling {
    static class Resource implements AutoCloseable {
        private final String name;
        private final boolean throwOnClose;
        
        Resource(String name, boolean throwOnClose) {
            this.name = name;
            this.throwOnClose = throwOnClose;
            System.out.println("Opening: " + name);
        }
        
        public void use() {
            System.out.println("Using: " + name);
            throw new RuntimeException("Error in use()");
        }
        
        @Override
        public void close() {
            System.out.println("Closing: " + name);
            if (throwOnClose) {
                throw new RuntimeException("Error in close()");
            }
        }
    }
    
    public static void main(String[] args) {
        // When both use() and close() throw exceptions
        try (Resource res = new Resource("MyResource", true)) {
            res.use();  // Throws exception
        } catch (Exception e) {
            System.out.println("Caught: " + e.getMessage());
            // Exception from use() is primary
            // Exception from close() is suppressed
            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println("Suppressed: " + suppressed.getMessage());
            }
        }
    }
}
```

### Real-World Example: Transaction Manager

```java
public class TransactionManager implements AutoCloseable {
    private final Connection connection;
    private boolean committed = false;
    
    public TransactionManager(Connection connection) throws SQLException {
        this.connection = connection;
        this.connection.setAutoCommit(false);
        System.out.println("Transaction started");
    }
    
    public void commit() throws SQLException {
        connection.commit();
        committed = true;
        System.out.println("Transaction committed");
    }
    
    @Override
    public void close() throws SQLException {
        if (!committed) {
            connection.rollback();
            System.out.println("Transaction rolled back");
        }
        connection.setAutoCommit(true);
    }
    
    // Usage
    public static void transferMoney(Connection conn, int fromAccount, 
                                    int toAccount, double amount) throws SQLException {
        try (TransactionManager tx = new TransactionManager(conn)) {
            // Deduct from source
            try (PreparedStatement ps = conn.prepareStatement(
                    "UPDATE accounts SET balance = balance - ? WHERE id = ?")) {
                ps.setDouble(1, amount);
                ps.setInt(2, fromAccount);
                ps.executeUpdate();
            }
            
            // Add to destination
            try (PreparedStatement ps = conn.prepareStatement(
                    "UPDATE accounts SET balance = balance + ? WHERE id = ?")) {
                ps.setDouble(1, amount);
                ps.setInt(2, toAccount);
                ps.executeUpdate();
            }
            
            tx.commit();  // Success
        }  // Auto-rollback if exception occurs
    }
}
```

---

## Practice Exercises

### Exercise 1: Create a Modular Calculator (Easy)

Create a modular calculator application with separate modules for API and implementation.

**Requirements:**
1. API module with Calculator interface
2. Basic implementation module with Add, Subtract operations
3. Advanced implementation module with Multiply, Divide
4. Client module that discovers and uses all calculators

<details>
<summary>Solution</summary>

```
calculator/
├── api/
│   └── src/
│       └── com.calculator.api/
│           ├── module-info.java
│           └── com/calculator/api/Calculator.java
├── basic/
│   └── src/
│       └── com.calculator.basic/
│           ├── module-info.java
│           └── com/calculator/basic/
│               ├── AddCalculator.java
│               └── SubtractCalculator.java
├── advanced/
│   └── src/
│       └── com.calculator.advanced/
│           ├── module-info.java
│           └── com/calculator/advanced/
│               ├── MultiplyCalculator.java
│               └── DivideCalculator.java
└── client/
    └── src/
        └── com.calculator.client/
            ├── module-info.java
            └── com/calculator/client/Main.java
```

**api/module-info.java:**
```java
module com.calculator.api {
    exports com.calculator.api;
}
```

**api/Calculator.java:**
```java
package com.calculator.api;

public interface Calculator {
    String getName();
    double calculate(double a, double b);
}
```

**basic/module-info.java:**
```java
module com.calculator.basic {
    requires com.calculator.api;
    provides com.calculator.api.Calculator with
        com.calculator.basic.AddCalculator,
        com.calculator.basic.SubtractCalculator;
}
```

**basic/AddCalculator.java:**
```java
package com.calculator.basic;

import com.calculator.api.Calculator;

public class AddCalculator implements Calculator {
    @Override
    public String getName() { return "Add"; }
    
    @Override
    public double calculate(double a, double b) {
        return a + b;
    }
}
```

**basic/SubtractCalculator.java:**
```java
package com.calculator.basic;

import com.calculator.api.Calculator;

public class SubtractCalculator implements Calculator {
    @Override
    public String getName() { return "Subtract"; }
    
    @Override
    public double calculate(double a, double b) {
        return a - b;
    }
}
```

**advanced/module-info.java:**
```java
module com.calculator.advanced {
    requires com.calculator.api;
    provides com.calculator.api.Calculator with
        com.calculator.advanced.MultiplyCalculator,
        com.calculator.advanced.DivideCalculator;
}
```

**client/module-info.java:**
```java
module com.calculator.client {
    requires com.calculator.api;
    uses com.calculator.api.Calculator;
}
```

**client/Main.java:**
```java
package com.calculator.client;

import com.calculator.api.Calculator;
import java.util.ServiceLoader;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        // Load all calculators
        ServiceLoader<Calculator> loader = ServiceLoader.load(Calculator.class);
        
        System.out.println("Available operations:");
        loader.forEach(calc -> System.out.println("- " + calc.getName()));
        
        System.out.print("\nEnter operation: ");
        String operation = scanner.nextLine();
        
        System.out.print("Enter first number: ");
        double a = scanner.nextDouble();
        
        System.out.print("Enter second number: ");
        double b = scanner.nextDouble();
        
        Calculator calculator = loader.stream()
            .map(ServiceLoader.Provider::get)
            .filter(calc -> calc.getName().equalsIgnoreCase(operation))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("Unknown operation"));
        
        double result = calculator.calculate(a, b);
        System.out.printf("%.2f %s %.2f = %.2f%n", a, operation, b, result);
    }
}
```
</details>

### Exercise 2: File Search Utility (Medium)

Create a utility that searches for files matching a pattern in a directory tree.

**Requirements:**
1. Search by file name pattern
2. Search by file size range
3. Search by modification date range
4. Display results with size and modification time

<details>
<summary>Solution</summary>

```java
import java.nio.file.*;
import java.io.IOException;
import java.time.*;
import java.util.*;
import java.util.stream.*;

public class FileSearchUtility {
    
    static class SearchCriteria {
        String namePattern;
        Long minSize;
        Long maxSize;
        LocalDateTime modifiedAfter;
        LocalDateTime modifiedBefore;
    }
    
    public static List<Path> search(Path root, SearchCriteria criteria) throws IOException {
        try (Stream<Path> stream = Files.walk(root)) {
            return stream
                .filter(Files::isRegularFile)
                .filter(path -> matchesPattern(path, criteria.namePattern))
                .filter(path -> matchesSize(path, criteria.minSize, criteria.maxSize))
                .filter(path -> matchesModificationDate(path, 
                    criteria.modifiedAfter, criteria.modifiedBefore))
                .collect(Collectors.toList());
        }
    }
    
    private static boolean matchesPattern(Path path, String pattern) {
        if (pattern == null) return true;
        String fileName = path.getFileName().toString();
        return fileName.matches(pattern.replace("*", ".*"));
    }
    
    private static boolean matchesSize(Path path, Long minSize, Long maxSize) {
        try {
            long size = Files.size(path);
            if (minSize != null && size < minSize) return false;
            if (maxSize != null && size > maxSize) return false;
            return true;
        } catch (IOException e) {
            return false;
        }
    }
    
    private static boolean matchesModificationDate(Path path, 
            LocalDateTime after, LocalDateTime before) {
        try {
            var modTime = Files.getLastModifiedTime(path);
            var modDate = LocalDateTime.ofInstant(
                modTime.toInstant(), ZoneId.systemDefault());
            
            if (after != null && modDate.isBefore(after)) return false;
            if (before != null && modDate.isAfter(before)) return false;
            return true;
        } catch (IOException e) {
            return false;
        }
    }
    
    public static void printResults(List<Path> results) throws IOException {
        System.out.printf("Found %d files:%n%n", results.size());
        
        for (Path path : results) {
            long size = Files.size(path);
            var modTime = Files.getLastModifiedTime(path);
            
            System.out.printf("%-50s  %10d bytes  %s%n",
                path.getFileName(),
                size,
                modTime);
        }
    }
    
    public static void main(String[] args) throws IOException {
        SearchCriteria criteria = new SearchCriteria();
        criteria.namePattern = "*.java";  // All Java files
        criteria.minSize = 1000L;         // At least 1KB
        criteria.modifiedAfter = LocalDateTime.now().minusDays(7); // Last 7 days
        
        List<Path> results = search(Path.of("src"), criteria);
        printResults(results);
    }
}
```
</details>

### Exercise 3: Configuration File Manager (Medium)

Create a configuration manager that watches for changes and reloads automatically.

**Requirements:**
1. Load configuration from properties file
2. Watch for file changes
3. Auto-reload when file is modified
4. Notify listeners of configuration changes

<details>
<summary>Solution</summary>

```java
import java.nio.file.*;
import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class ConfigurationManager {
    private final Path configFile;
    private final Properties properties = new Properties();
    private final List<ConfigChangeListener> listeners = new CopyOnWriteArrayList<>();
    private final ExecutorService watcherThread = Executors.newSingleThreadExecutor();
    
    public interface ConfigChangeListener {
        void onConfigChanged(Properties newConfig);
    }
    
    public ConfigurationManager(Path configFile) throws IOException {
        this.configFile = configFile;
        loadConfiguration();
        startWatching();
    }
    
    private void loadConfiguration() throws IOException {
        synchronized (properties) {
            properties.clear();
            try (InputStream in = Files.newInputStream(configFile)) {
                properties.load(in);
            }
            System.out.println("Configuration loaded: " + properties.size() + " properties");
        }
    }
    
    private void startWatching() {
        watcherThread.submit(() -> {
            try (WatchService watchService = FileSystems.getDefault().newWatchService()) {
                Path directory = configFile.getParent();
                directory.register(watchService, StandardWatchEventKinds.ENTRY_MODIFY);
                
                System.out.println("Watching for changes: " + configFile);
                
                while (true) {
                    WatchKey key = watchService.take();
                    
                    for (WatchEvent<?> event : key.pollEvents()) {
                        Path changed = (Path) event.context();
                        if (changed.equals(configFile.getFileName())) {
                            System.out.println("Configuration file changed, reloading...");
                            Thread.sleep(100);  // Wait for write to complete
                            reloadConfiguration();
                        }
                    }
                    
                    if (!key.reset()) {
                        break;
                    }
                }
            } catch (IOException | InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
    
    private void reloadConfiguration() {
        try {
            Properties oldProperties = new Properties();
            synchronized (properties) {
                oldProperties.putAll(properties);
            }
            
            loadConfiguration();
            notifyListeners();
            
        } catch (IOException e) {
            System.err.println("Failed to reload configuration: " + e.getMessage());
        }
    }
    
    public String getProperty(String key) {
        synchronized (properties) {
            return properties.getProperty(key);
        }
    }
    
    public String getProperty(String key, String defaultValue) {
        synchronized (properties) {
            return properties.getProperty(key, defaultValue);
        }
    }
    
    public void addListener(ConfigChangeListener listener) {
        listeners.add(listener);
    }
    
    private void notifyListeners() {
        Properties snapshot;
        synchronized (properties) {
            snapshot = new Properties();
            snapshot.putAll(properties);
        }
        
        for (ConfigChangeListener listener : listeners) {
            try {
                listener.onConfigChanged(snapshot);
            } catch (Exception e) {
                System.err.println("Error in listener: " + e.getMessage());
            }
        }
    }
    
    public void close() {
        watcherThread.shutdownNow();
    }
    
    // Demo
    public static void main(String[] args) throws Exception {
        Path configFile = Path.of("app.properties");
        
        // Create initial config file
        Properties initial = new Properties();
        initial.setProperty("app.name", "MyApp");
        initial.setProperty("app.version", "1.0");
        try (OutputStream out = Files.newOutputStream(configFile)) {
            initial.store(out, "Application Configuration");
        }
        
        // Create manager
        ConfigurationManager manager = new ConfigurationManager(configFile);
        
        // Add listener
        manager.addListener(newConfig -> {
            System.out.println("Configuration changed!");
            newConfig.forEach((key, value) -> 
                System.out.println("  " + key + " = " + value));
        });
        
        System.out.println("Initial config:");
        System.out.println("  app.name = " + manager.getProperty("app.name"));
        System.out.println("  app.version = " + manager.getProperty("app.version"));
        
        // Simulate config change after 5 seconds
        Thread.sleep(5000);
        System.out.println("\nModifying configuration file...");
        initial.setProperty("app.version", "2.0");
        try (OutputStream out = Files.newOutputStream(configFile)) {
            initial.store(out, "Updated Configuration");
        }
        
        // Wait to see the reload
        Thread.sleep(2000);
        
        System.out.println("\nCurrent config:");
        System.out.println("  app.version = " + manager.getProperty("app.version"));
        
        manager.close();
    }
}
```
</details>

### Exercise 4: Custom Runtime Image Builder (Hard)

Create a build script that creates an optimized runtime image for a modular application.

**Requirements:**
1. Analyze module dependencies
2. Create custom runtime with jlink
3. Compare sizes (full JDK vs custom runtime)
4. Create launcher script

<details>
<summary>Solution</summary>

```bash
#!/bin/bash
# build-runtime.sh

set -e  # Exit on error

PROJECT_NAME="myapp"
MAIN_MODULE="com.example.app"
MAIN_CLASS="com.example.app.Main"
OUTPUT_DIR="dist"

echo "=== Building Custom Runtime for $PROJECT_NAME ==="

# Clean previous builds
echo "Cleaning previous builds..."
rm -rf out $OUTPUT_DIR

# Create output directory
mkdir -p out/$MAIN_MODULE

# Compile
echo "Compiling modules..."
javac -d out/$MAIN_MODULE \
    $(find src/$MAIN_MODULE -name "*.java")

# Analyze dependencies
echo "Analyzing dependencies..."
jdeps --module-path out -m $MAIN_MODULE

# Count dependencies
MODULE_COUNT=$(jdeps --module-path out -m $MAIN_MODULE | grep "requires" | wc -l)
echo "Found $MODULE_COUNT required modules"

# Create runtime image
echo "Creating custom runtime image..."
jlink \
    --module-path "$JAVA_HOME/jmods:out" \
    --add-modules $MAIN_MODULE \
    --output $OUTPUT_DIR/$PROJECT_NAME \
    --launcher $PROJECT_NAME=$MAIN_MODULE/$MAIN_CLASS \
    --strip-debug \
    --compress=2 \
    --no-header-files \
    --no-man-pages \
    --verbose

# Size comparison
echo ""
echo "=== Size Comparison ==="
JDK_SIZE=$(du -sh $JAVA_HOME | cut -f1)
RUNTIME_SIZE=$(du -sh $OUTPUT_DIR/$PROJECT_NAME | cut -f1)
echo "Full JDK size: $JDK_SIZE"
echo "Custom runtime size: $RUNTIME_SIZE"

# List included modules
echo ""
echo "=== Included Modules ==="
$OUTPUT_DIR/$PROJECT_NAME/bin/java --list-modules

# Create distribution package
echo ""
echo "Creating distribution package..."
cd $OUTPUT_DIR
tar czf $PROJECT_NAME.tar.gz $PROJECT_NAME
TAR_SIZE=$(du -sh $PROJECT_NAME.tar.gz | cut -f1)
echo "Distribution package size: $TAR_SIZE"
cd ..

# Test the runtime
echo ""
echo "=== Testing Runtime ==="
$OUTPUT_DIR/$PROJECT_NAME/bin/$PROJECT_NAME

echo ""
echo "Build complete! Runtime image: $OUTPUT_DIR/$PROJECT_NAME"
echo "Run with: ./$OUTPUT_DIR/$PROJECT_NAME/bin/$PROJECT_NAME"
```

**Java Application Example:**

```java
// module-info.java
module com.example.app {
    requires java.base;
    requires java.logging;
}

// Main.java
package com.example.app;

import java.util.logging.*;

public class Main {
    private static final Logger LOGGER = Logger.getLogger(Main.class.getName());
    
    public static void main(String[] args) {
        LOGGER.info("Application started");
        System.out.println("Hello from custom runtime!");
        System.out.println("Java Version: " + System.getProperty("java.version"));
        
        // Show available modules
        System.out.println("\nAvailable modules:");
        ModuleLayer.boot().modules().stream()
            .map(Module::getName)
            .sorted()
            .forEach(name -> System.out.println("  - " + name));
        
        LOGGER.info("Application finished");
    }
}
```
</details>

### Exercise 5: Log File Rotator (Hard)

Create a log file rotator that automatically rotates log files based on size or time.

**Requirements:**
1. Rotate logs when size exceeds threshold
2. Keep N most recent log files
3. Compress old log files
4. Thread-safe writing

<details>
<summary>Solution</summary>

```java
import java.nio.file.*;
import java.io.*;
import java.time.*;
import java.time.format.*;
import java.util.*;
import java.util.zip.*;
import java.util.concurrent.locks.*;

public class LogRotator implements AutoCloseable {
    private final Path logDirectory;
    private final String baseFileName;
    private final long maxFileSize;
    private final int maxFiles;
    private final Lock lock = new ReentrantLock();
    private BufferedWriter currentWriter;
    private Path currentFile;
    private long currentSize;
    
    public LogRotator(Path logDirectory, String baseFileName, 
                     long maxFileSizeMB, int maxFiles) throws IOException {
        this.logDirectory = logDirectory;
        this.baseFileName = baseFileName;
        this.maxFileSize = maxFileSizeMB * 1024 * 1024;
        this.maxFiles = maxFiles;
        
        Files.createDirectories(logDirectory);
        openNewLogFile();
    }
    
    private void openNewLogFile() throws IOException {
        if (currentWriter != null) {
            currentWriter.close();
        }
        
        String timestamp = LocalDateTime.now()
            .format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss"));
        currentFile = logDirectory.resolve(baseFileName + "_" + timestamp + ".log");
        currentWriter = Files.newBufferedWriter(currentFile);
        currentSize = 0;
        
        System.out.println("Opened new log file: " + currentFile);
    }
    
    public void log(String message) throws IOException {
        lock.lock();
        try {
            String logEntry = LocalDateTime.now()
                .format(DateTimeFormatter.ISO_LOCAL_DATE_TIME) 
                + " - " + message + "\n";
            
            byte[] bytes = logEntry.getBytes();
            
            // Check if rotation needed
            if (currentSize + bytes.length > maxFileSize) {
                rotate();
            }
            
            currentWriter.write(logEntry);
            currentWriter.flush();
            currentSize += bytes.length;
            
        } finally {
            lock.unlock();
        }
    }
    
    private void rotate() throws IOException {
        System.out.println("Rotating log file...");
        
        // Close current file
        currentWriter.close();
        
        // Compress old file
        compressFile(currentFile);
        
        // Delete old compressed files
        cleanupOldFiles();
        
        // Open new file
        openNewLogFile();
    }
    
    private void compressFile(Path file) throws IOException {
        Path gzFile = Path.of(file.toString() + ".gz");
        
        try (InputStream in = Files.newInputStream(file);
             OutputStream out = Files.newOutputStream(gzFile);
             GZIPOutputStream gzOut = new GZIPOutputStream(out)) {
            
            byte[] buffer = new byte[8192];
            int n;
            while ((n = in.read(buffer)) > 0) {
                gzOut.write(buffer, 0, n);
            }
        }
        
        // Delete original file
        Files.delete(file);
        System.out.println("Compressed: " + gzFile);
    }
    
    private void cleanupOldFiles() throws IOException {
        // Find all compressed log files
        List<Path> logFiles = new ArrayList<>();
        try (var stream = Files.list(logDirectory)) {
            stream
                .filter(p -> p.getFileName().toString().startsWith(baseFileName))
                .filter(p -> p.getFileName().toString().endsWith(".gz"))
                .sorted(Comparator.comparing(p -> {
                    try {
                        return Files.getLastModifiedTime(p);
                    } catch (IOException e) {
                        return FileTime.fromMillis(0);
                    }
                }))
                .forEach(logFiles::add);
        }
        
        // Delete oldest files if we have too many
        while (logFiles.size() > maxFiles) {
            Path oldest = logFiles.remove(0);
            Files.delete(oldest);
            System.out.println("Deleted old log: " + oldest);
        }
    }
    
    @Override
    public void close() throws IOException {
        lock.lock();
        try {
            if (currentWriter != null) {
                currentWriter.close();
                compressFile(currentFile);
            }
        } finally {
            lock.unlock();
        }
    }
    
    // Demo
    public static void main(String[] args) throws Exception {
        Path logDir = Path.of("logs");
        
        try (LogRotator rotator = new LogRotator(logDir, "app", 1, 3)) {
            // Generate lots of log messages
            for (int i = 0; i < 100000; i++) {
                rotator.log("Log message #" + i + " - " + 
                    "Some important information that needs to be logged. ".repeat(10));
                
                if (i % 1000 == 0) {
                    System.out.println("Generated " + i + " log messages");
                }
            }
        }
        
        System.out.println("\nLog files created:");
        try (var stream = Files.list(logDir)) {
            stream.forEach(file -> {
                try {
                    long size = Files.size(file);
                    System.out.printf("  %s (%.2f MB)%n", 
                        file.getFileName(), size / 1024.0 / 1024.0);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            });
        }
    }
}
```
</details>

---

## Interview Questions & Answers

### Theoretical Questions

**Q1: What is the Java Platform Module System (JPMS)? What problems does it solve?**

**Answer:** JPMS (Project Jigsaw) is a module system introduced in Java 9 that provides:

**Problems it solves:**
1. **JAR Hell**: Eliminates version conflicts and classpath issues
2. **Weak Encapsulation**: Before JPMS, all public classes were accessible
3. **Unreliable Configuration**: Dependencies discovered at runtime, causing NoClassDefFoundError
4. **Monolithic JRE**: JRE was huge (~200MB) even for small applications

**Benefits:**
- **Strong Encapsulation**: Only exported packages are accessible
- **Reliable Configuration**: Compile-time dependency checking
- **Custom Runtimes**: Create optimized JRE with only needed modules (jlink)
- **Better Security**: Reduced attack surface

**Example:**
```java
module com.example.app {
    requires java.sql;           // Explicit dependency
    exports com.example.api;     // Public API
    // com.example.internal hidden
}
```

**Q2: Explain the difference between `exports` and `opens`.**

**Answer:**

| Aspect | exports | opens |
|--------|---------|-------|
| **Compile-time access** | ✅ Yes | ❌ No |
| **Reflection access** | ❌ Blocked | ✅ Allowed |
| **Use case** | Public APIs | Framework internals |

**exports**: Makes packages available for compile-time use by other modules
```java
module com.example.library {
    exports com.example.api;  // Others can import and use
}
```

**opens**: Allows reflective access without compile-time visibility
```java
module com.example.app {
    opens com.example.entities;  // JPA/Hibernate can reflect, but not import
}
```

**Why `opens` exists:**
Frameworks (Spring, Hibernate, Jackson) need reflection to access private fields for DI, ORM, serialization. Before modules, this was allowed. With modules, reflection is blocked by default. `opens` allows specific reflective access while maintaining encapsulation.

**Q3: What is ServiceLoader? How does it enable loose coupling?**

**Answer:** `ServiceLoader` is Java's SPI (Service Provider Interface) mechanism for discovering and loading service implementations at runtime.

**Benefits:**
1. **Loose Coupling**: Consumer depends on interface, not implementation
2. **Extensibility**: Add new implementations without changing consumer code
3. **Plugin Architecture**: Load plugins at runtime

**Example:**
```java
// API module
module com.example.api {
    exports com.example.api;
}

public interface Calculator {
    double calculate(double a, double b);
}

// Implementation module
module com.example.impl {
    requires com.example.api;
    provides com.example.api.Calculator 
        with com.example.impl.AddCalculator;
}

// Consumer module
module com.example.client {
    requires com.example.api;
    uses com.example.api.Calculator;
}

// Usage
ServiceLoader<Calculator> loader = ServiceLoader.load(Calculator.class);
Calculator calc = loader.findFirst().orElseThrow();
```

**Q4: What is jlink? When would you use it?**

**Answer:** `jlink` is a tool for creating custom runtime images containing only the modules your application needs.

**Use cases:**
1. **Microservices**: Smaller Docker images
2. **Desktop Applications**: Reduce download size
3. **Embedded Systems**: Limited storage
4. **Security**: Fewer modules = smaller attack surface

**Example:**
```bash
jlink --module-path $JAVA_HOME/jmods:out \
      --add-modules com.example.app \
      --output custom-jre \
      --launcher app=com.example.app/com.example.app.Main \
      --strip-debug \
      --compress=2
```

**Benefits:**
- **Smaller size**: 30-50MB vs 200MB full JRE
- **Faster startup**: Less code to load
- **Self-contained**: Includes everything needed

**Q5: Explain the difference between Path and File. Why use Path?**

**Answer:**

| Aspect | File (Old IO) | Path (NIO.2) |
|--------|---------------|--------------|
| **Package** | java.io | java.nio.file |
| **Interface** | No | Yes |
| **Functionality** | Limited | Comprehensive |
| **Performance** | Slower | Faster |
| **Platform-specific** | More | Less |

**Why Path is better:**
1. **More Methods**: Files class provides 100+ utility methods
2. **Better Performance**: Optimized native operations
3. **Symbolic Links**: Proper handling
4. **Watch Service**: Monitor file changes
5. **Streams**: Better integration with Stream API

**Example:**
```java
// Old way (java.io.File)
File file = new File("data.txt");
if (file.exists()) {
    FileInputStream fis = new FileInputStream(file);
    // Manual buffer management...
}

// Modern way (java.nio.file)
Path path = Path.of("data.txt");
if (Files.exists(path)) {
    String content = Files.readString(path);  // One line!
}
```

**Q6: What is try-with-resources? How does it improve resource management?**

**Answer:** try-with-resources (Java 7+) is a mechanism for automatic resource management.

**Problems it solves:**
```java
// ❌ Before Java 7 - Error-prone
InputStream in = null;
try {
    in = new FileInputStream("file.txt");
    // Use in
} finally {
    if (in != null) {
        try {
            in.close();  // Can throw exception!
        } catch (IOException e) {
            // What to do here?
        }
    }
}

// ✅ Java 7+ - Clean and safe
try (InputStream in = new FileInputStream("file.txt")) {
    // Use in
}  // Automatically closed, even if exception occurs
```

**How it works:**
1. Resources must implement `AutoCloseable`
2. `close()` called automatically in reverse order
3. Exceptions properly handled (suppressed exceptions)

**Benefits:**
- **No resource leaks**: Always closed
- **Less code**: No manual try-finally
- **Better exception handling**: Primary exception preserved

**Q7: What are qualified exports/opens? When would you use them?**

**Answer:** Qualified exports/opens restrict access to specific modules only.

**Syntax:**
```java
module com.example.library {
    // Export to everyone
    exports com.example.api;
    
    // Export only to specific modules
    exports com.example.internal to com.example.test, com.example.admin;
    
    // Open only to specific modules
    opens com.example.entities to org.hibernate.orm, com.fasterxml.jackson.databind;
}
```

**Use cases:**
1. **Testing**: Expose internal packages only to test module
2. **Framework Integration**: Open entities only to specific frameworks
3. **Partner Modules**: Share internal APIs with trusted modules only
4. **Gradual Migration**: Control access during refactoring

**Example - Test-only access:**
```java
// Production module
module com.example.app {
    exports com.example.api;
    exports com.example.internal to com.example.app.test;  // Test-only
}

// Test module
module com.example.app.test {
    requires com.example.app;
    requires org.junit.jupiter.api;
}
```

### Coding Questions

**Q8: Design a plugin system using ServiceLoader.**

```java
// ========== API Module ==========
module plugin.api {
    exports plugin.api;
}

package plugin.api;

public interface Plugin {
    String getName();
    String getVersion();
    void execute(Context context);
}

public interface Context {
    void log(String message);
    String getProperty(String key);
}

// ========== Plugin Implementation ==========
module plugin.email {
    requires plugin.api;
    provides plugin.api.Plugin with plugin.email.EmailPlugin;
}

package plugin.email;

import plugin.api.*;

public class EmailPlugin implements Plugin {
    @Override
    public String getName() {
        return "Email Plugin";
    }
    
    @Override
    public String getVersion() {
        return "1.0.0";
    }
    
    @Override
    public void execute(Context context) {
        String recipient = context.getProperty("email.recipient");
        context.log("Sending email to: " + recipient);
        // Email sending logic
    }
}

// ========== Application ==========
module plugin.app {
    requires plugin.api;
    uses plugin.api.Plugin;
}

package plugin.app;

import plugin.api.*;
import java.util.*;

public class PluginManager {
    private final Map<String, Plugin> plugins = new HashMap<>();
    
    public PluginManager() {
        loadPlugins();
    }
    
    private void loadPlugins() {
        ServiceLoader.load(Plugin.class).forEach(plugin -> {
            plugins.put(plugin.getName(), plugin);
            System.out.println("Loaded: " + plugin.getName() + 
                             " v" + plugin.getVersion());
        });
    }
    
    public void executePlugin(String name, Context context) {
        Plugin plugin = plugins.get(name);
        if (plugin == null) {
            throw new IllegalArgumentException("Plugin not found: " + name);
        }
        plugin.execute(context);
    }
    
    public List<Plugin> getAllPlugins() {
        return new ArrayList<>(plugins.values());
    }
}
```

**Q9: Implement a file backup utility using NIO.2.**

```java
import java.nio.file.*;
import java.io.IOException;
import java.time.*;
import java.time.format.*;
import java.util.stream.*;

public class FileBackupUtility {
    
    public static void backup(Path sourceDir, Path backupDir) throws IOException {
        // Create timestamped backup directory
        String timestamp = LocalDateTime.now()
            .format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss"));
        Path targetDir = backupDir.resolve("backup_" + timestamp);
        
        System.out.println("Creating backup: " + targetDir);
        
        // Copy directory tree
        Files.walk(sourceDir)
            .forEach(source -> {
                try {
                    Path target = targetDir.resolve(sourceDir.relativize(source));
                    
                    if (Files.isDirectory(source)) {
                        Files.createDirectories(target);
                    } else {
                        Files.copy(source, target, 
                            StandardCopyOption.COPY_ATTRIBUTES);
                    }
                    
                    System.out.println("Backed up: " + source.getFileName());
                    
                } catch (IOException e) {
                    System.err.println("Failed to backup: " + source);
                    e.printStackTrace();
                }
            });
        
        System.out.println("Backup complete!");
    }
    
    public static void cleanupOldBackups(Path backupDir, int keepCount) 
            throws IOException {
        try (Stream<Path> stream = Files.list(backupDir)) {
            List<Path> backups = stream
                .filter(Files::isDirectory)
                .filter(p -> p.getFileName().toString().startsWith("backup_"))
                .sorted(Comparator.comparing(p -> {
                    try {
                        return Files.getLastModifiedTime(p);
                    } catch (IOException e) {
                        return FileTime.fromMillis(0);
                    }
                }))
                .collect(Collectors.toList());
            
            // Delete oldest backups
            while (backups.size() > keepCount) {
                Path oldest = backups.remove(0);
                deleteDirectory(oldest);
                System.out.println("Deleted old backup: " + oldest);
            }
        }
    }
    
    private static void deleteDirectory(Path directory) throws IOException {
        if (Files.exists(directory)) {
            Files.walk(directory)
                .sorted(Comparator.reverseOrder())
                .forEach(path -> {
                    try {
                        Files.delete(path);
                    } catch (IOException e) {
                        System.err.println("Failed to delete: " + path);
                    }
                });
        }
    }
    
    public static void main(String[] args) throws IOException {
        Path sourceDir = Path.of("project");
        Path backupDir = Path.of("backups");
        
        Files.createDirectories(backupDir);
        
        // Create backup
        backup(sourceDir, backupDir);
        
        // Keep only 5 most recent backups
        cleanupOldBackups(backupDir, 5);
    }
}
```

**Q10: Create a module that uses both qualified exports and services.**

```java
// ========== API Module ==========
module data.api {
    exports data.api;
}

package data.api;

public interface DataProcessor {
    String process(String data);
}

// ========== Implementation Module ==========
module data.impl {
    requires data.api;
    
    // Public API
    exports data.impl.api;
    
    // Internal utilities (exposed only to test)
    exports data.impl.internal to data.impl.test;
    
    // Service implementation (not exported!)
    provides data.api.DataProcessor with data.impl.JsonProcessor;
}

package data.impl.api;

import data.api.DataProcessor;
import java.util.ServiceLoader;

public class DataProcessorFactory {
    public static DataProcessor getProcessor() {
        return ServiceLoader.load(DataProcessor.class)
            .findFirst()
            .orElseThrow(() -> new IllegalStateException("No processor found"));
    }
}

package data.impl.internal;

// Only accessible to test module
public class ValidationUtils {
    public static boolean isValidJson(String data) {
        return data != null && data.trim().startsWith("{");
    }
}

package data.impl;

import data.api.DataProcessor;
import data.impl.internal.ValidationUtils;

// Not exported! Only available via ServiceLoader
class JsonProcessor implements DataProcessor {
    @Override
    public String process(String data) {
        if (!ValidationUtils.isValidJson(data)) {
            throw new IllegalArgumentException("Invalid JSON");
        }
        return data.toUpperCase();  // Example processing
    }
}

// ========== Test Module ==========
module data.impl.test {
    requires data.impl;
    requires org.junit.jupiter.api;
}

package data.impl.test;

import data.impl.internal.ValidationUtils;  // Can access!
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class ValidationUtilsTest {
    @Test
    void testValidation() {
        assertTrue(ValidationUtils.isValidJson("{\"key\":\"value\"}"));
        assertFalse(ValidationUtils.isValidJson("not json"));
    }
}
```

---

## Real-World Scenarios

### Scenario 1: Microservice with Custom Runtime

**Problem:** Build a microservice with minimal Docker image size.

**Solution:**

```dockerfile
# Stage 1: Build
FROM maven:3.8-openjdk-17 AS build

WORKDIR /app
COPY pom.xml .
COPY src ./src

# Build modular JAR
RUN mvn clean package

# Create custom runtime
RUN jlink \
    --module-path /opt/java/jmods:target/libs:target/app.jar \
    --add-modules com.example.microservice \
    --output /opt/jre \
    --strip-debug \
    --compress=2 \
    --no-header-files \
    --no-man-pages

# Stage 2: Runtime
FROM debian:11-slim

COPY --from=build /opt/jre /opt/jre
COPY --from=build /app/target/app.jar /app/app.jar

ENV PATH="/opt/jre/bin:${PATH}"

EXPOSE 8080
CMD ["java", "-m", "com.example.microservice/com.example.microservice.Main"]

# Result: ~50MB vs 500MB with full JDK
```

### Scenario 2: Enterprise Application with Plugins

**Problem:** Design an extensible application where plugins can be added/removed without recompilation.

**Solution:**

```
enterprise-app/
├── api/              (Core interfaces)
├── core/             (Main application)
├── plugin-email/     (Email plugin)
├── plugin-sms/       (SMS plugin)
└── plugin-slack/     (Slack plugin)
```

See Exercise 8 for complete implementation.

---

## Summary & Best Practices

### Key Takeaways

1. **JPMS Benefits:**
   - Strong encapsulation
   - Reliable configuration
   - Custom runtimes
   - Better security

2. **Module Design:**
   - Export only public API
   - Use `opens` for frameworks
   - Prefer `requires transitive` for API dependencies
   - Use qualified exports for test code

3. **ServiceLoader:**
   - Perfect for plugin architectures
   - Enables loose coupling
   - Runtime discovery of implementations

4. **jlink:**
   - Create custom runtimes
   - Reduce deployment size
   - Faster startup times

5. **NIO.2:**
   - Use `Path` over `File`
   - Leverage `Files` utility methods
   - Stream API for large directories
   - Watch service for file monitoring

6. **Resource Management:**
   - Always use try-with-resources
   - Implement `AutoCloseable` for custom resources
   - Handle exceptions properly

### Common Pitfalls

❌ **Don't:**
- Export internal packages
- Forget `requires` declarations
- Mix old IO and NIO.2 unnecessarily
- Forget to close resources
- Use `File` when `Path` is better

✅ **Do:**
- Design clear module boundaries
- Use ServiceLoader for extensibility
- Use NIO.2 for all file operations
- Always use try-with-resources
- Create custom runtimes with jlink
- Document module dependencies

---

**Congratulations!** You've completed Week 7. You now understand:
- Java Platform Module System (JPMS)
- `exports` vs `opens` and their use cases
- Services with `uses`/`provides` and `ServiceLoader`
- Custom runtime images with `jlink`
- Modern IO with NIO.2 (`Path`, `Files`)
- Resource management with try-with-resources

**Next:** Week 8 - The Concurrency Revolution (Virtual Threads, Structured Concurrency, Scoped Values)
