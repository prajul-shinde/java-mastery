# Week 5: Streams & Functional Programming

## 🎯 Learning Objectives
By the end of this week, you will:
- Master lambda expressions and functional interfaces
- Understand functional programming concepts in Java
- Use Stream API fluently for data processing
- Work with intermediate and terminal operations
- Apply collectors for complex aggregations
- Use Gatherers (JEP 461) for custom operations
- Understand parallel streams and performance
- Avoid common pitfalls with streams
- Crack technical interviews on functional programming

---

# Part 1: Functional Programming Foundations

## 1.1 What is Functional Programming?

### Paradigm Shift

**Imperative (Traditional):**
```java
// HOW to do it - step by step
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> filtered = new ArrayList<>();

for (String name : names) {
    if (name.length() > 3) {
        filtered.add(name.toUpperCase());
    }
}

Collections.sort(filtered);
System.out.println(filtered);
```

**Declarative (Functional):**
```java
// WHAT to do - describe the result
List<String> result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());

System.out.println(result);
```

**Key Principles:**
1. **Immutability** - Data doesn't change
2. **Pure Functions** - No side effects, same input = same output
3. **First-Class Functions** - Functions as values
4. **Higher-Order Functions** - Functions that take/return functions
5. **Declarative** - Express what, not how

---

## 1.2 Lambda Expressions - Anonymous Functions

### Lambda Syntax

```java
/**
 * Lambda Expression Syntax:
 * (parameters) -> expression
 * (parameters) -> { statements; }
 */
public class LambdaSyntax {
    public static void main(String[] args) {
        
        // No parameters
        Runnable r1 = () -> System.out.println("Hello");
        
        // One parameter (parentheses optional)
        Consumer<String> c1 = s -> System.out.println(s);
        Consumer<String> c2 = (s) -> System.out.println(s);
        
        // Multiple parameters (parentheses required)
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        
        // Multiple statements (braces required)
        BiFunction<Integer, Integer, Integer> multiply = (a, b) -> {
            int result = a * b;
            System.out.println("Multiplying: " + a + " * " + b);
            return result;
        };
        
        // Type inference (Java infers types)
        BiFunction<Integer, Integer, Integer> subtract = (a, b) -> a - b;
        
        // Explicit types (rarely needed)
        BiFunction<Integer, Integer, Integer> divide = (Integer a, Integer b) -> a / b;
        
        // Executing lambdas
        r1.run();
        c1.accept("Hello Lambda");
        System.out.println("5 + 3 = " + add.apply(5, 3));
        System.out.println("5 * 3 = " + multiply.apply(5, 3));
    }
}
```

### Before and After Lambdas

```java
/**
 * Evolution: Anonymous classes to Lambdas
 */
public class LambdaEvolution {
    
    // Before Java 8 - Anonymous Class
    public static void beforeJava8() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // Sorting with anonymous class
        Collections.sort(names, new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return s1.length() - s2.length();
            }
        });
    }
    
    // After Java 8 - Lambda
    public static void afterJava8() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // Sorting with lambda (much cleaner!)
        Collections.sort(names, (s1, s2) -> s1.length() - s2.length());
        
        // Or even simpler with method reference
        Collections.sort(names, Comparator.comparing(String::length));
    }
}
```

---

## 1.3 Functional Interfaces - The Foundation

### What is a Functional Interface?

**Functional Interface**: Interface with **exactly one abstract method** (SAM - Single Abstract Method)

```java
/**
 * Custom functional interface
 */
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);  // Single abstract method
    
    // Default and static methods are allowed
    default void printResult(int result) {
        System.out.println("Result: " + result);
    }
    
    static void info() {
        System.out.println("Calculator interface");
    }
}

// Usage
public class FunctionalInterfaceExample {
    public static void main(String[] args) {
        // Lambda implements the single abstract method
        Calculator add = (a, b) -> a + b;
        Calculator multiply = (a, b) -> a * b;
        
        System.out.println("5 + 3 = " + add.calculate(5, 3));
        System.out.println("5 * 3 = " + multiply.calculate(5, 3));
    }
}
```

### Built-in Functional Interfaces (java.util.function)

#### 1. Predicate<T> - Test a condition

```java
import java.util.function.Predicate;

/**
 * Predicate<T>: T -> boolean
 * Tests a condition and returns true/false
 */
public class PredicateExamples {
    public static void main(String[] args) {
        
        // Basic predicate
        Predicate<Integer> isPositive = n -> n > 0;
        Predicate<Integer> isEven = n -> n % 2 == 0;
        Predicate<String> isEmpty = s -> s.isEmpty();
        
        // Testing
        System.out.println("5 is positive? " + isPositive.test(5));      // true
        System.out.println("4 is even? " + isEven.test(4));              // true
        System.out.println("'' is empty? " + isEmpty.test(""));          // true
        
        // Combining predicates
        Predicate<Integer> isPositiveEven = isPositive.and(isEven);
        Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);
        Predicate<Integer> isNotPositive = isPositive.negate();
        
        System.out.println("4 is positive and even? " + isPositiveEven.test(4));  // true
        System.out.println("-2 is positive or even? " + isPositiveOrEven.test(-2)); // true
        System.out.println("-5 is not positive? " + isNotPositive.test(-5));      // true
        
        // Real-world example: Filtering
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");
        
        Predicate<String> startsWithC = s -> s.startsWith("C");
        Predicate<String> longerThan3 = s -> s.length() > 3;
        
        List<String> filtered = names.stream()
            .filter(startsWithC.and(longerThan3))
            .collect(Collectors.toList());
        
        System.out.println("Filtered: " + filtered);  // [Charlie]
    }
}
```

#### 2. Function<T, R> - Transform T to R

```java
import java.util.function.Function;

/**
 * Function<T, R>: T -> R
 * Transforms input of type T to output of type R
 */
public class FunctionExamples {
    public static void main(String[] args) {
        
        // Basic functions
        Function<String, Integer> length = s -> s.length();
        Function<Integer, Integer> square = n -> n * n;
        Function<String, String> uppercase = s -> s.toUpperCase();
        
        // Applying functions
        System.out.println("Length of 'Hello': " + length.apply("Hello"));      // 5
        System.out.println("Square of 5: " + square.apply(5));                  // 25
        System.out.println("Uppercase: " + uppercase.apply("hello"));           // HELLO
        
        // Composing functions
        Function<String, Integer> lengthSquared = length.andThen(square);
        System.out.println("Length of 'Hello' squared: " + lengthSquared.apply("Hello")); // 25
        
        Function<Integer, String> squareToString = square.andThen(Object::toString);
        System.out.println("Square of 5 as String: " + squareToString.apply(5)); // "25"
        
        // compose() - reverse order
        Function<Integer, Integer> addOne = n -> n + 1;
        Function<Integer, Integer> multiplyByTwo = n -> n * 2;
        
        // andThen: first addOne, then multiplyByTwo
        Function<Integer, Integer> f1 = addOne.andThen(multiplyByTwo);
        System.out.println("(5 + 1) * 2 = " + f1.apply(5));  // 12
        
        // compose: first multiplyByTwo, then addOne
        Function<Integer, Integer> f2 = addOne.compose(multiplyByTwo);
        System.out.println("(5 * 2) + 1 = " + f2.apply(5));  // 11
        
        // Real-world example: Data transformation
        List<String> names = Arrays.asList("alice", "bob", "charlie");
        
        Function<String, String> capitalize = s -> s.substring(0, 1).toUpperCase() + s.substring(1);
        
        List<String> capitalized = names.stream()
            .map(capitalize)
            .collect(Collectors.toList());
        
        System.out.println("Capitalized: " + capitalized);  // [Alice, Bob, Charlie]
    }
}
```

#### 3. Consumer<T> - Consume T without return

```java
import java.util.function.Consumer;

/**
 * Consumer<T>: T -> void
 * Consumes input and performs action (no return)
 */
public class ConsumerExamples {
    public static void main(String[] args) {
        
        // Basic consumers
        Consumer<String> print = s -> System.out.println(s);
        Consumer<String> printUppercase = s -> System.out.println(s.toUpperCase());
        Consumer<Integer> printSquare = n -> System.out.println(n * n);
        
        // Using consumers
        print.accept("Hello");              // Hello
        printUppercase.accept("world");     // WORLD
        printSquare.accept(5);              // 25
        
        // Chaining consumers
        Consumer<String> printBoth = print.andThen(printUppercase);
        printBoth.accept("Hello");
        // Output:
        // Hello
        // HELLO
        
        // Real-world example: Processing list
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        Consumer<String> greet = name -> System.out.println("Hello, " + name);
        names.forEach(greet);
        
        // Or directly
        names.forEach(name -> System.out.println("Hi, " + name));
    }
}
```

#### 4. Supplier<T> - Supply T without input

```java
import java.util.function.Supplier;

/**
 * Supplier<T>: () -> T
 * Supplies/generates value with no input
 */
public class SupplierExamples {
    public static void main(String[] args) {
        
        // Basic suppliers
        Supplier<String> helloSupplier = () -> "Hello";
        Supplier<Double> randomSupplier = () -> Math.random();
        Supplier<LocalDateTime> timeSupplier = () -> LocalDateTime.now();
        
        // Getting values
        System.out.println(helloSupplier.get());    // Hello
        System.out.println(randomSupplier.get());   // Random number
        System.out.println(timeSupplier.get());     // Current time
        
        // Lazy evaluation example
        System.out.println("Before supplier creation");
        Supplier<String> expensiveOperation = () -> {
            System.out.println("Executing expensive operation...");
            // Simulate expensive operation
            return "Result";
        };
        
        System.out.println("Supplier created but not called yet");
        // Operation hasn't executed yet!
        
        System.out.println("Calling supplier...");
        String result = expensiveOperation.get();  // NOW it executes
        System.out.println("Result: " + result);
        
        // Real-world example: Factory pattern
        Supplier<User> userFactory = () -> new User("Guest", "guest@example.com");
        
        User user1 = userFactory.get();
        User user2 = userFactory.get();  // New instance each time
        
        System.out.println("User1: " + user1.getName());
        System.out.println("User2: " + user2.getName());
    }
    
    static class User {
        private String name;
        private String email;
        
        User(String name, String email) {
            this.name = name;
            this.email = email;
        }
        
        String getName() { return name; }
    }
}
```

#### 5. BiFunction, BiConsumer, BiPredicate

```java
import java.util.function.*;

/**
 * Bi-variants: Two parameters
 */
public class BiVariantExamples {
    public static void main(String[] args) {
        
        // BiFunction<T, U, R>: (T, U) -> R
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        BiFunction<String, String, String> concat = (s1, s2) -> s1 + s2;
        
        System.out.println("5 + 3 = " + add.apply(5, 3));
        System.out.println("Concat: " + concat.apply("Hello", "World"));
        
        // BiConsumer<T, U>: (T, U) -> void
        BiConsumer<String, Integer> printInfo = (name, age) -> 
            System.out.println(name + " is " + age + " years old");
        
        printInfo.accept("Alice", 25);
        
        // BiPredicate<T, U>: (T, U) -> boolean
        BiPredicate<String, Integer> isLengthEqual = (str, len) -> str.length() == len;
        
        System.out.println("'Hello' length is 5? " + isLengthEqual.test("Hello", 5));
        
        // Real-world example: Map operations
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        
        // BiConsumer for forEach
        scores.forEach((name, score) -> 
            System.out.println(name + " scored " + score)
        );
        
        // merge() with BiFunction
        scores.merge("Alice", 5, (oldValue, newValue) -> oldValue + newValue);
        System.out.println("Alice's new score: " + scores.get("Alice")); // 100
    }
}
```

#### 6. UnaryOperator and BinaryOperator

```java
import java.util.function.*;

/**
 * Specialized functions where input and output types are same
 */
public class OperatorExamples {
    public static void main(String[] args) {
        
        // UnaryOperator<T>: T -> T (special case of Function<T, T>)
        UnaryOperator<Integer> square = n -> n * n;
        UnaryOperator<String> uppercase = s -> s.toUpperCase();
        
        System.out.println("Square of 5: " + square.apply(5));
        System.out.println("Uppercase: " + uppercase.apply("hello"));
        
        // BinaryOperator<T>: (T, T) -> T (special case of BiFunction<T, T, T>)
        BinaryOperator<Integer> add = (a, b) -> a + b;
        BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
        BinaryOperator<String> concat = (s1, s2) -> s1 + s2;
        
        System.out.println("5 + 3 = " + add.apply(5, 3));
        System.out.println("Max(5, 3) = " + max.apply(5, 3));
        System.out.println("Concat: " + concat.apply("Hello", "World"));
        
        // Real-world example: List transformation
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        
        UnaryOperator<Integer> doubleIt = n -> n * 2;
        List<Integer> doubled = numbers.stream()
            .map(doubleIt)
            .collect(Collectors.toList());
        
        System.out.println("Doubled: " + doubled);  // [2, 4, 6, 8, 10]
        
        // reduce() with BinaryOperator
        int sum = numbers.stream()
            .reduce(0, add);
        
        System.out.println("Sum: " + sum);  // 15
    }
}
```

### Functional Interfaces Summary Table

| Interface | Parameters | Return | Description | Example |
|-----------|------------|--------|-------------|---------|
| **Predicate\<T>** | T | boolean | Test condition | `n -> n > 0` |
| **Function\<T,R>** | T | R | Transform T to R | `s -> s.length()` |
| **Consumer\<T>** | T | void | Consume T | `s -> System.out.println(s)` |
| **Supplier\<T>** | none | T | Supply T | `() -> "Hello"` |
| **BiFunction\<T,U,R>** | T, U | R | Transform T,U to R | `(a,b) -> a+b` |
| **BiConsumer\<T,U>** | T, U | void | Consume T,U | `(k,v) -> map.put(k,v)` |
| **BiPredicate\<T,U>** | T, U | boolean | Test T,U | `(s,len) -> s.length()==len` |
| **UnaryOperator\<T>** | T | T | Transform T to T | `n -> n*2` |
| **BinaryOperator\<T>** | T, T | T | Combine T,T to T | `(a,b) -> a+b` |

---

## 1.4 Method References - Shorthand for Lambdas

### Types of Method References

```java
import java.util.*;
import java.util.function.*;

/**
 * Method References: Compact lambda syntax
 * Four types:
 * 1. Static method reference
 * 2. Instance method reference on specific object
 * 3. Instance method reference on arbitrary object
 * 4. Constructor reference
 */
public class MethodReferenceExamples {
    
    public static void main(String[] args) {
        
        // 1. Static Method Reference: ClassName::staticMethod
        
        // Lambda: n -> Math.abs(n)
        Function<Integer, Integer> abs1 = n -> Math.abs(n);
        Function<Integer, Integer> abs2 = Math::abs;  // Method reference
        
        System.out.println("abs(-5) = " + abs2.apply(-5));
        
        // Lambda: (a, b) -> Integer.compare(a, b)
        Comparator<Integer> comp1 = (a, b) -> Integer.compare(a, b);
        Comparator<Integer> comp2 = Integer::compare;  // Method reference
        
        // Lambda: s -> Integer.parseInt(s)
        Function<String, Integer> parse1 = s -> Integer.parseInt(s);
        Function<String, Integer> parse2 = Integer::parseInt;  // Method reference
        
        
        // 2. Instance Method Reference (Specific Object): object::instanceMethod
        
        String prefix = "Hello, ";
        
        // Lambda: s -> prefix.concat(s)
        Function<String, String> greet1 = s -> prefix.concat(s);
        Function<String, String> greet2 = prefix::concat;  // Method reference
        
        System.out.println(greet2.apply("World"));  // Hello, World
        
        PrintStream out = System.out;
        // Lambda: s -> System.out.println(s)
        Consumer<String> print1 = s -> System.out.println(s);
        Consumer<String> print2 = System.out::println;  // Method reference
        
        
        // 3. Instance Method Reference (Arbitrary Object): ClassName::instanceMethod
        
        // Lambda: s -> s.toUpperCase()
        Function<String, String> upper1 = s -> s.toUpperCase();
        Function<String, String> upper2 = String::toUpperCase;  // Method reference
        
        System.out.println(upper2.apply("hello"));  // HELLO
        
        // Lambda: s -> s.length()
        Function<String, Integer> len1 = s -> s.length();
        Function<String, Integer> len2 = String::length;  // Method reference
        
        // Lambda: (s1, s2) -> s1.compareTo(s2)
        Comparator<String> strComp1 = (s1, s2) -> s1.compareTo(s2);
        Comparator<String> strComp2 = String::compareTo;  // Method reference
        
        
        // 4. Constructor Reference: ClassName::new
        
        // Lambda: () -> new ArrayList<>()
        Supplier<List<String>> listSupp1 = () -> new ArrayList<>();
        Supplier<List<String>> listSupp2 = ArrayList::new;  // Constructor reference
        
        List<String> list = listSupp2.get();
        
        // Lambda: s -> new User(s)
        Function<String, User> userCreator1 = s -> new User(s);
        Function<String, User> userCreator2 = User::new;  // Constructor reference
        
        User user = userCreator2.apply("Alice");
        System.out.println("User: " + user.getName());
        
        // Array constructor reference
        IntFunction<int[]> arrayCreator1 = size -> new int[size];
        IntFunction<int[]> arrayCreator2 = int[]::new;  // Array constructor
        
        int[] arr = arrayCreator2.apply(10);
        System.out.println("Array length: " + arr.length);
    }
    
    static class User {
        private String name;
        User(String name) { this.name = name; }
        String getName() { return name; }
    }
}
```

### Real-World Method Reference Examples

```java
/**
 * Practical method reference usage
 */
public class MethodReferenceReal {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
        
        // Print each name
        names.forEach(System.out::println);
        
        // Convert to uppercase
        List<String> upper = names.stream()
            .map(String::toUpperCase)
            .collect(Collectors.toList());
        
        // Sort by length
        names.sort(Comparator.comparing(String::length));
        
        // Filter and print
        names.stream()
            .filter(name -> name.length() > 3)
            .forEach(System.out::println);
        
        // Create users
        List<String> userNames = Arrays.asList("Alice", "Bob", "Charlie");
        List<User> users = userNames.stream()
            .map(User::new)  // Constructor reference
            .collect(Collectors.toList());
        
        // Extract emails
        List<String> emails = users.stream()
            .map(User::getEmail)  // Method reference
            .collect(Collectors.toList());
        
        System.out.println("Emails: " + emails);
    }
    
    static class User {
        private String name;
        private String email;
        
        User(String name) {
            this.name = name;
            this.email = name.toLowerCase() + "@example.com";
        }
        
        String getName() { return name; }
        String getEmail() { return email; }
    }
}
```

---

# Part 2: Stream API - Powerful Data Processing

## 2.1 What is a Stream?

### Understanding Streams

```java
/**
 * Stream: Sequence of elements supporting sequential and parallel operations
 * 
 * Key Points:
 * - NOT a data structure (doesn't store data)
 * - Lazy evaluation (operations only execute when needed)
 * - Can be infinite
 * - Consumable (can only be used once)
 */
public class StreamBasics {
    public static void main(String[] args) {
        
        // Creating streams
        
        // 1. From collection
        List<String> list = Arrays.asList("A", "B", "C");
        Stream<String> stream1 = list.stream();
        
        // 2. From array
        String[] array = {"A", "B", "C"};
        Stream<String> stream2 = Arrays.stream(array);
        
        // 3. From values
        Stream<String> stream3 = Stream.of("A", "B", "C");
        
        // 4. Infinite streams
        Stream<Integer> infinite1 = Stream.iterate(0, n -> n + 1);  // 0, 1, 2, 3...
        Stream<Double> infinite2 = Stream.generate(Math::random);    // Random numbers
        
        // 5. From range
        IntStream range1 = IntStream.range(1, 5);        // 1, 2, 3, 4
        IntStream range2 = IntStream.rangeClosed(1, 5);  // 1, 2, 3, 4, 5
        
        // Stream is consumable (can only be used once)
        Stream<String> stream = Stream.of("A", "B", "C");
        stream.forEach(System.out::println);  // Works
        // stream.forEach(System.out::println);  // IllegalStateException!
    }
}
```

---

## 2.2 Stream Operations Overview

### Intermediate vs Terminal Operations

```java
/**
 * Stream Operations:
 * 
 * INTERMEDIATE (return Stream):
 * - filter(), map(), flatMap(), distinct(), sorted(), limit(), skip()
 * - Lazy: don't execute until terminal operation
 * 
 * TERMINAL (return result or void):
 * - forEach(), collect(), reduce(), count(), anyMatch(), allMatch(), 
 *   findFirst(), findAny(), min(), max()
 * - Trigger execution
 */
public class StreamOperationsOverview {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Pipeline: intermediate -> intermediate -> terminal
        long count = numbers.stream()          // Source
            .filter(n -> n % 2 == 0)           // Intermediate (lazy)
            .map(n -> n * n)                   // Intermediate (lazy)
            .peek(n -> System.out.println("Processing: " + n))  // Intermediate
            .count();                          // Terminal (triggers execution)
        
        System.out.println("Count: " + count);
        
        // Nothing happens without terminal operation
        numbers.stream()
            .filter(n -> {
                System.out.println("Filtering: " + n);
                return n % 2 == 0;
            })
            .map(n -> {
                System.out.println("Mapping: " + n);
                return n * n;
            });
        // No output! No terminal operation.
        
        System.out.println("\nWith terminal operation:");
        numbers.stream()
            .filter(n -> {
                System.out.println("Filtering: " + n);
                return n % 2 == 0;
            })
            .map(n -> {
                System.out.println("Mapping: " + n);
                return n * n;
            })
            .forEach(n -> System.out.println("Result: " + n));
    }
}
```

---

## 2.3 Intermediate Operations

### 1. filter() - Select elements

```java
/**
 * filter(Predicate): Select elements that match condition
 */
public class FilterExamples {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Filter even numbers
        List<Integer> evens = numbers.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());
        
        System.out.println("Evens: " + evens);  // [2, 4, 6, 8, 10]
        
        // Multiple filters (chaining)
        List<Integer> result = numbers.stream()
            .filter(n -> n > 3)           // Greater than 3
            .filter(n -> n % 2 == 0)      // Even
            .filter(n -> n < 9)           // Less than 9
            .collect(Collectors.toList());
        
        System.out.println("Filtered: " + result);  // [4, 6, 8]
        
        // Real-world example: Filter users
        List<User> users = Arrays.asList(
            new User("Alice", 25, "alice@example.com"),
            new User("Bob", 17, "bob@example.com"),
            new User("Charlie", 30, "charlie@company.com"),
            new User("David", 22, "david@company.com")
        );
        
        // Adults with company email
        List<User> companyAdults = users.stream()
            .filter(u -> u.getAge() >= 18)
            .filter(u -> u.getEmail().endsWith("@company.com"))
            .collect(Collectors.toList());
        
        System.out.println("Company adults:");
        companyAdults.forEach(u -> System.out.println(u.getName()));
    }
    
    static class User {
        private String name;
        private int age;
        private String email;
        
        User(String name, int age, String email) {
            this.name = name;
            this.age = age;
            this.email = email;
        }
        
        String getName() { return name; }
        int getAge() { return age; }
        String getEmail() { return email; }
    }
}
```

### 2. map() - Transform elements

```java
/**
 * map(Function): Transform each element
 */
public class MapExamples {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("alice", "bob", "charlie");
        
        // Convert to uppercase
        List<String> upper = names.stream()
            .map(String::toUpperCase)
            .collect(Collectors.toList());
        
        System.out.println("Uppercase: " + upper);  // [ALICE, BOB, CHARLIE]
        
        // Get lengths
        List<Integer> lengths = names.stream()
            .map(String::length)
            .collect(Collectors.toList());
        
        System.out.println("Lengths: " + lengths);  // [5, 3, 7]
        
        // Chain transformations
        List<String> result = names.stream()
            .map(String::toUpperCase)
            .map(s -> s + "!")
            .map(s -> "Hello, " + s)
            .collect(Collectors.toList());
        
        System.out.println("Transformed: " + result);
        // [Hello, ALICE!, Hello, BOB!, Hello, CHARLIE!]
        
        // Real-world example: Extract properties
        List<User> users = Arrays.asList(
            new User("Alice", "alice@example.com"),
            new User("Bob", "bob@example.com")
        );
        
        // Extract emails
        List<String> emails = users.stream()
            .map(User::getEmail)
            .collect(Collectors.toList());
        
        System.out.println("Emails: " + emails);
        
        // Transform to DTOs
        List<UserDTO> dtos = users.stream()
            .map(user -> new UserDTO(user.getName(), user.getEmail()))
            .collect(Collectors.toList());
    }
    
    static class User {
        private String name;
        private String email;
        
        User(String name, String email) {
            this.name = name;
            this.email = email;
        }
        
        String getName() { return name; }
        String getEmail() { return email; }
    }
    
    static class UserDTO {
        private String name;
        private String email;
        
        UserDTO(String name, String email) {
            this.name = name;
            this.email = email;
        }
    }
}
```

### 3. flatMap() - Flatten nested structures

```java
/**
 * flatMap(Function): Flatten stream of streams to single stream
 */
public class FlatMapExamples {
    public static void main(String[] args) {
        
        // Problem: List of lists
        List<List<Integer>> nestedList = Arrays.asList(
            Arrays.asList(1, 2, 3),
            Arrays.asList(4, 5),
            Arrays.asList(6, 7, 8, 9)
        );
        
        // map() gives Stream<Stream<Integer>> (not what we want!)
        // flatMap() gives Stream<Integer> (flattened!)
        
        List<Integer> flattened = nestedList.stream()
            .flatMap(list -> list.stream())  // Flatten
            .collect(Collectors.toList());
        
        System.out.println("Flattened: " + flattened);  // [1, 2, 3, 4, 5, 6, 7, 8, 9]
        
        // Example: Split sentences into words
        List<String> sentences = Arrays.asList(
            "Hello World",
            "Java Streams",
            "Functional Programming"
        );
        
        List<String> words = sentences.stream()
            .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
            .collect(Collectors.toList());
        
        System.out.println("Words: " + words);
        // [Hello, World, Java, Streams, Functional, Programming]
        
        // Real-world example: Get all emails from users
        List<User> users = Arrays.asList(
            new User("Alice", Arrays.asList("alice@work.com", "alice@personal.com")),
            new User("Bob", Arrays.asList("bob@work.com")),
            new User("Charlie", Arrays.asList("charlie@work.com", "charlie@personal.com", "charlie@school.com"))
        );
        
        // Extract all emails (flattened)
        List<String> allEmails = users.stream()
            .flatMap(user -> user.getEmails().stream())
            .collect(Collectors.toList());
        
        System.out.println("All emails: " + allEmails);
        
        // Filter and flatten
        List<String> workEmails = users.stream()
            .flatMap(user -> user.getEmails().stream())
            .filter(email -> email.contains("@work.com"))
            .collect(Collectors.toList());
        
        System.out.println("Work emails: " + workEmails);
    }
    
    static class User {
        private String name;
        private List<String> emails;
        
        User(String name, List<String> emails) {
            this.name = name;
            this.emails = emails;
        }
        
        String getName() { return name; }
        List<String> getEmails() { return emails; }
    }
}
```

### 4. distinct(), sorted(), limit(), skip()

```java
/**
 * Other intermediate operations
 */
public class OtherIntermediateOps {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(5, 2, 8, 2, 9, 1, 5, 3, 8, 4);
        
        // distinct() - Remove duplicates
        List<Integer> unique = numbers.stream()
            .distinct()
            .collect(Collectors.toList());
        
        System.out.println("Unique: " + unique);  // [5, 2, 8, 9, 1, 3, 4]
        
        // sorted() - Natural order
        List<Integer> sorted = numbers.stream()
            .sorted()
            .collect(Collectors.toList());
        
        System.out.println("Sorted: " + sorted);  // [1, 2, 2, 3, 4, 5, 5, 8, 8, 9]
        
        // sorted(Comparator) - Custom order
        List<Integer> reversed = numbers.stream()
            .sorted(Comparator.reverseOrder())
            .collect(Collectors.toList());
        
        System.out.println("Reversed: " + reversed);  // [9, 8, 8, 5, 5, 4, 3, 2, 2, 1]
        
        // limit(n) - Take first n elements
        List<Integer> first5 = numbers.stream()
            .limit(5)
            .collect(Collectors.toList());
        
        System.out.println("First 5: " + first5);  // [5, 2, 8, 2, 9]
        
        // skip(n) - Skip first n elements
        List<Integer> after5 = numbers.stream()
            .skip(5)
            .collect(Collectors.toList());
        
        System.out.println("After 5: " + after5);  // [1, 5, 3, 8, 4]
        
        // Combining operations
        List<Integer> result = numbers.stream()
            .distinct()              // Remove duplicates
            .sorted()                // Sort
            .skip(2)                 // Skip first 2
            .limit(3)                // Take next 3
            .collect(Collectors.toList());
        
        System.out.println("Combined: " + result);  // [3, 4, 5]
        
        // Real-world example: Top 5 products by price
        List<Product> products = Arrays.asList(
            new Product("Laptop", 1200),
            new Product("Mouse", 25),
            new Product("Keyboard", 75),
            new Product("Monitor", 300),
            new Product("Headphones", 150),
            new Product("Webcam", 80),
            new Product("Speaker", 100)
        );
        
        List<Product> top5 = products.stream()
            .sorted(Comparator.comparing(Product::getPrice).reversed())
            .limit(5)
            .collect(Collectors.toList());
        
        System.out.println("Top 5 by price:");
        top5.forEach(p -> System.out.println(p.getName() + ": $" + p.getPrice()));
    }
    
    static class Product {
        private String name;
        private double price;
        
        Product(String name, double price) {
            this.name = name;
            this.price = price;
        }
        
        String getName() { return name; }
        double getPrice() { return price; }
    }
}
```

### 5. peek() - Debug and Side Effects

```java
/**
 * peek(Consumer): Perform action on each element (debugging)
 */
public class PeekExamples {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        
        // Using peek for debugging
        List<Integer> result = numbers.stream()
            .peek(n -> System.out.println("Original: " + n))
            .filter(n -> n % 2 == 0)
            .peek(n -> System.out.println("After filter: " + n))
            .map(n -> n * n)
            .peek(n -> System.out.println("After map: " + n))
            .collect(Collectors.toList());
        
        System.out.println("Final result: " + result);
        
        // Output shows the flow:
        // Original: 1
        // Original: 2
        // After filter: 2
        // After map: 4
        // Original: 3
        // ...
    }
}
```

---

## 2.4 Terminal Operations

### 1. collect() - Gather results

```java
import java.util.stream.Collectors;

/**
 * collect(): Most versatile terminal operation
 */
public class CollectExamples {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");
        
        // Collect to List
        List<String> list = names.stream()
            .filter(n -> n.length() > 3)
            .collect(Collectors.toList());
        
        // Collect to Set
        Set<String> set = names.stream()
            .collect(Collectors.toSet());
        
        // Collect to specific collection
        LinkedList<String> linkedList = names.stream()
            .collect(Collectors.toCollection(LinkedList::new));
        
        // Collect to Map
        Map<String, Integer> nameToLength = names.stream()
            .collect(Collectors.toMap(
                name -> name,              // Key
                name -> name.length()      // Value
            ));
        
        System.out.println("Name to length: " + nameToLength);
        
        // Joining strings
        String joined = names.stream()
            .collect(Collectors.joining(", "));
        
        System.out.println("Joined: " + joined);  // Alice, Bob, Charlie, David, Eve
        
        String withPrefixSuffix = names.stream()
            .collect(Collectors.joining(", ", "[", "]"));
        
        System.out.println("With prefix/suffix: " + withPrefixSuffix);  // [Alice, Bob, ...]
    }
}
```

### 2. reduce() - Combine elements

```java
/**
 * reduce(): Combine elements into single result
 */
public class ReduceExamples {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        
        // Sum using reduce
        Optional<Integer> sum1 = numbers.stream()
            .reduce((a, b) -> a + b);
        
        System.out.println("Sum: " + sum1.get());  // 15
        
        // With identity value (no Optional)
        int sum2 = numbers.stream()
            .reduce(0, (a, b) -> a + b);
        
        System.out.println("Sum with identity: " + sum2);  // 15
        
        // Product
        int product = numbers.stream()
            .reduce(1, (a, b) -> a * b);
        
        System.out.println("Product: " + product);  // 120
        
        // Max
        Optional<Integer> max = numbers.stream()
            .reduce((a, b) -> a > b ? a : b);
        
        System.out.println("Max: " + max.get());  // 5
        
        // Or using Integer::max
        Optional<Integer> max2 = numbers.stream()
            .reduce(Integer::max);
        
        // String concatenation
        List<String> words = Arrays.asList("Java", "Streams", "Are", "Powerful");
        
        String sentence = words.stream()
            .reduce("", (a, b) -> a + " " + b);
        
        System.out.println("Sentence:" + sentence);  // Java Streams Are Powerful
        
        // More efficient for strings
        String sentence2 = words.stream()
            .collect(Collectors.joining(" "));
    }
}
```

### 3. forEach() and forEachOrdered()

```java
/**
 * forEach(): Perform action on each element
 */
public class ForEachExamples {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // forEach
        names.stream()
            .forEach(System.out::println);
        
        // forEachOrdered - maintains order (important for parallel streams)
        names.parallelStream()
            .forEachOrdered(System.out::println);  // Guaranteed order
        
        // forEach with side effects (be careful!)
        List<String> result = new ArrayList<>();
        names.stream()
            .map(String::toUpperCase)
            .forEach(result::add);  // Side effect
        
        System.out.println("Result: " + result);
    }
}
```

### 4. count(), anyMatch(), allMatch(), noneMatch()

```java
/**
 * Matching and counting operations
 */
public class MatchingOps {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // count()
        long count = numbers.stream()
            .filter(n -> n % 2 == 0)
            .count();
        
        System.out.println("Even count: " + count);  // 5
        
        // anyMatch() - at least one matches
        boolean hasEven = numbers.stream()
            .anyMatch(n -> n % 2 == 0);
        
        System.out.println("Has even? " + hasEven);  // true
        
        // allMatch() - all match
        boolean allPositive = numbers.stream()
            .allMatch(n -> n > 0);
        
        System.out.println("All positive? " + allPositive);  // true
        
        // noneMatch() - none match
        boolean noNegative = numbers.stream()
            .noneMatch(n -> n < 0);
        
        System.out.println("No negative? " + noNegative);  // true
        
        // Short-circuiting
        boolean found = Stream.iterate(1, n -> n + 1)  // Infinite stream
            .peek(n -> System.out.println("Testing: " + n))
            .anyMatch(n -> n > 5);  // Stops at 6
        
        System.out.println("Found: " + found);
    }
}
```

### 5. findFirst() and findAny()

```java
/**
 * Finding elements
 */
public class FindingOps {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
        
        // findFirst() - first element
        Optional<String> first = names.stream()
            .filter(n -> n.startsWith("C"))
            .findFirst();
        
        first.ifPresent(System.out::println);  // Charlie
        
        // findAny() - any element (useful for parallel streams)
        Optional<String> any = names.parallelStream()
            .filter(n -> n.length() > 3)
            .findAny();
        
        any.ifPresent(System.out::println);  // Could be any matching element
        
        // Real-world example: Find user by ID
        List<User> users = Arrays.asList(
            new User(1, "Alice"),
            new User(2, "Bob"),
            new User(3, "Charlie")
        );
        
        Optional<User> user = users.stream()
            .filter(u -> u.getId() == 2)
            .findFirst();
        
        user.ifPresent(u -> System.out.println("Found: " + u.getName()));
    }
    
    static class User {
        private int id;
        private String name;
        
        User(int id, String name) {
            this.id = id;
            this.name = name;
        }
        
        int getId() { return id; }
        String getName() { return name; }
    }
}
```

### 6. min() and max()

```java
/**
 * Finding minimum and maximum
 */
public class MinMaxOps {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9, 3);
        
        // min()
        Optional<Integer> min = numbers.stream()
            .min(Comparator.naturalOrder());
        
        System.out.println("Min: " + min.get());  // 1
        
        // max()
        Optional<Integer> max = numbers.stream()
            .max(Comparator.naturalOrder());
        
        System.out.println("Max: " + max.get());  // 9
        
        // Custom comparator
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
        
        Optional<String> shortest = names.stream()
            .min(Comparator.comparing(String::length));
        
        System.out.println("Shortest: " + shortest.get());  // Bob
        
        Optional<String> longest = names.stream()
            .max(Comparator.comparing(String::length));
        
        System.out.println("Longest: " + longest.get());  // Charlie
        
        // Real-world example: Find most expensive product
        List<Product> products = Arrays.asList(
            new Product("Laptop", 1200),
            new Product("Mouse", 25),
            new Product("Keyboard", 75)
        );
        
        Optional<Product> mostExpensive = products.stream()
            .max(Comparator.comparing(Product::getPrice));
        
        mostExpensive.ifPresent(p -> 
            System.out.println("Most expensive: " + p.getName() + " ($" + p.getPrice() + ")")
        );
    }
    
    static class Product {
        private String name;
        private double price;
        
        Product(String name, double price) {
            this.name = name;
            this.price = price;
        }
        
        String getName() { return name; }
        double getPrice() { return price; }
    }
}
```

---

## 2.5 Collectors - Advanced Aggregations

### Basic Collectors

```java
import java.util.stream.Collectors;

/**
 * Collectors: Pre-built reduction operations
 */
public class CollectorsBasic {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");
        
        // toList()
        List<String> list = names.stream()
            .collect(Collectors.toList());
        
        // toSet()
        Set<String> set = names.stream()
            .collect(Collectors.toSet());
        
        // toCollection()
        TreeSet<String> treeSet = names.stream()
            .collect(Collectors.toCollection(TreeSet::new));
        
        // toMap()
        Map<String, Integer> nameToLength = names.stream()
            .collect(Collectors.toMap(
                Function.identity(),    // Key: name itself
                String::length          // Value: length
            ));
        
        System.out.println("Name to length: " + nameToLength);
        
        // joining()
        String joined = names.stream()
            .collect(Collectors.joining(", "));
        
        System.out.println("Joined: " + joined);
        
        // counting()
        long count = names.stream()
            .collect(Collectors.counting());
        
        System.out.println("Count: " + count);
    }
}
```

### Grouping and Partitioning

```java
/**
 * groupingBy() and partitioningBy()
 */
public class GroupingCollectors {
    public static void main(String[] args) {
        
        List<Person> people = Arrays.asList(
            new Person("Alice", 25, "Engineering"),
            new Person("Bob", 30, "Marketing"),
            new Person("Charlie", 25, "Engineering"),
            new Person("David", 35, "Marketing"),
            new Person("Eve", 28, "Engineering")
        );
        
        // groupingBy() - Group by department
        Map<String, List<Person>> byDept = people.stream()
            .collect(Collectors.groupingBy(Person::getDepartment));
        
        System.out.println("By department:");
        byDept.forEach((dept, list) -> {
            System.out.println(dept + ": " + list.size());
        });
        
        // groupingBy() with downstream collector
        Map<String, Long> countByDept = people.stream()
            .collect(Collectors.groupingBy(
                Person::getDepartment,
                Collectors.counting()
            ));
        
        System.out.println("Count by department: " + countByDept);
        
        // groupingBy() - Average age by department
        Map<String, Double> avgAgeByDept = people.stream()
            .collect(Collectors.groupingBy(
                Person::getDepartment,
                Collectors.averagingInt(Person::getAge)
            ));
        
        System.out.println("Average age by department: " + avgAgeByDept);
        
        // groupingBy() - Names by department
        Map<String, List<String>> namesByDept = people.stream()
            .collect(Collectors.groupingBy(
                Person::getDepartment,
                Collectors.mapping(Person::getName, Collectors.toList())
            ));
        
        System.out.println("Names by department: " + namesByDept);
        
        // partitioningBy() - Split into two groups (true/false)
        Map<Boolean, List<Person>> partitioned = people.stream()
            .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));
        
        System.out.println("30 and above: " + partitioned.get(true).size());
        System.out.println("Below 30: " + partitioned.get(false).size());
    }
    
    static class Person {
        private String name;
        private int age;
        private String department;
        
        Person(String name, int age, String department) {
            this.name = name;
            this.age = age;
            this.department = department;
        }
        
        String getName() { return name; }
        int getAge() { return age; }
        String getDepartment() { return department; }
    }
}
```

### teeing() - Combine Two Collectors (Java 12+)

```java
/**
 * teeing(): Apply two collectors and merge results
 */
public class TeeingExample {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Get sum and count in one pass
        record Stats(int sum, long count) {}
        
        Stats stats = numbers.stream()
            .collect(Collectors.teeing(
                Collectors.summingInt(Integer::intValue),  // First collector
                Collectors.counting(),                     // Second collector
                (sum, count) -> new Stats(sum, count)      // Merge function
            ));
        
        System.out.println("Sum: " + stats.sum());
        System.out.println("Count: " + stats.count());
        System.out.println("Average: " + (double) stats.sum() / stats.count());
        
        // Another example: Min and Max together
        record MinMax(int min, int max) {}
        
        MinMax minMax = numbers.stream()
            .collect(Collectors.teeing(
                Collectors.minBy(Comparator.naturalOrder()),
                Collectors.maxBy(Comparator.naturalOrder()),
                (min, max) -> new MinMax(min.get(), max.get())
            ));
        
        System.out.println("Min: " + minMax.min());
        System.out.println("Max: " + minMax.max());
    }
}
```

### Statistical Collectors

```java
/**
 * Statistical collectors: summarizingInt, summarizingDouble, etc.
 */
public class StatisticalCollectors {
    public static void main(String[] args) {
        
        List<Product> products = Arrays.asList(
            new Product("Laptop", 1200),
            new Product("Mouse", 25),
            new Product("Keyboard", 75),
            new Product("Monitor", 300)
        );
        
        // IntSummaryStatistics
        IntSummaryStatistics stats = products.stream()
            .collect(Collectors.summarizingInt(p -> (int) p.getPrice()));
        
        System.out.println("Count: " + stats.getCount());
        System.out.println("Sum: " + stats.getSum());
        System.out.println("Min: " + stats.getMin());
        System.out.println("Max: " + stats.getMax());
        System.out.println("Average: " + stats.getAverage());
        
        // DoubleSummaryStatistics
        DoubleSummaryStatistics doubleStats = products.stream()
            .collect(Collectors.summarizingDouble(Product::getPrice));
        
        System.out.println("\nDouble stats:");
        System.out.println("Average: " + doubleStats.getAverage());
    }
    
    static class Product {
        private String name;
        private double price;
        
        Product(String name, double price) {
            this.name = name;
            this.price = price;
        }
        
        String getName() { return name; }
        double getPrice() { return price; }
    }
}
```

---

## 2.6 Gatherers (JEP 461 - Java 22+)

### What are Gatherers?

```java
import java.util.stream.Gatherer;
import java.util.stream.Gatherers;

/**
 * Gatherers: Custom intermediate operations
 * Introduced in JEP 461 (Java 22+)
 * 
 * Allow creating custom stream transformations
 */
public class GatherersIntro {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Built-in gatherer: windowFixed()
        // Process stream in fixed-size windows
        List<List<Integer>> windows = numbers.stream()
            .gather(Gatherers.windowFixed(3))
            .collect(Collectors.toList());
        
        System.out.println("Fixed windows (size 3): " + windows);
        // [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]
        
        // Built-in gatherer: windowSliding()
        // Sliding window
        List<List<Integer>> sliding = numbers.stream()
            .gather(Gatherers.windowSliding(3))
            .collect(Collectors.toList());
        
        System.out.println("Sliding windows (size 3): " + sliding);
        // [[1, 2, 3], [2, 3, 4], [3, 4, 5], ...]
        
        // fold() - Custom accumulation
        Integer sum = numbers.stream()
            .gather(Gatherers.fold(() -> 0, (acc, n) -> acc + n))
            .findFirst()
            .orElse(0);
        
        System.out.println("Sum using fold: " + sum);
        
        // scan() - Running totals
        List<Integer> runningSum = numbers.stream()
            .gather(Gatherers.scan(() -> 0, (acc, n) -> acc + n))
            .collect(Collectors.toList());
        
        System.out.println("Running sum: " + runningSum);
        // [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]
    }
}
```

### Custom Gatherers

```java
/**
 * Creating custom gatherers
 */
public class CustomGatherers {
    
    // Custom gatherer: Take while sum < threshold
    public static <T extends Number> Gatherer<T, ?, T> takeWhileSumLessThan(double threshold) {
        class State {
            double sum = 0;
        }
        
        return Gatherer.ofSequential(
            State::new,  // Initial state
            (state, element, downstream) -> {
                double value = element.doubleValue();
                if (state.sum + value < threshold) {
                    state.sum += value;
                    return downstream.push(element);
                }
                return false;  // Stop processing
            }
        );
    }
    
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        List<Integer> result = numbers.stream()
            .gather(takeWhileSumLessThan(15))
            .collect(Collectors.toList());
        
        System.out.println("Numbers while sum < 15: " + result);
        // [1, 2, 3, 4, 5] (sum = 15)
    }
}
```

### Real-World Gatherers Example

```java
/**
 * Practical gatherer: Batch processing
 */
public class BatchGatherer {
    
    public static void main(String[] args) {
        
        List<String> items = Arrays.asList(
            "Item1", "Item2", "Item3", "Item4", "Item5",
            "Item6", "Item7", "Item8", "Item9", "Item10"
        );
        
        // Process in batches of 3
        items.stream()
            .gather(Gatherers.windowFixed(3))
            .forEach(batch -> {
                System.out.println("Processing batch: " + batch);
                // Simulate batch processing
                processBatch(batch);
            });
    }
    
    private static void processBatch(List<String> batch) {
        System.out.println("  -> Saved " + batch.size() + " items to database");
    }
}
```

---

## 2.7 Parallel Streams

### Introduction to Parallel Streams

```java
/**
 * Parallel Streams: Process elements concurrently
 */
public class ParallelStreamsIntro {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Sequential stream
        numbers.stream()
            .forEach(n -> System.out.println(Thread.currentThread().getName() + ": " + n));
        
        System.out.println("\nParallel stream:");
        
        // Parallel stream
        numbers.parallelStream()
            .forEach(n -> System.out.println(Thread.currentThread().getName() + ": " + n));
        
        // Creating parallel stream
        Stream<Integer> parallelStream1 = numbers.parallelStream();
        Stream<Integer> parallelStream2 = numbers.stream().parallel();
        
        // Back to sequential
        Stream<Integer> sequential = parallelStream1.sequential();
    }
}
```

### When to Use Parallel Streams

```java
/**
 * Performance comparison: Sequential vs Parallel
 */
public class ParallelPerformance {
    public static void main(String[] args) {
        
        List<Integer> largeList = IntStream.rangeClosed(1, 10_000_000)
            .boxed()
            .collect(Collectors.toList());
        
        // Sequential
        long start = System.currentTimeMillis();
        long sum1 = largeList.stream()
            .map(n -> n * 2)
            .reduce(0, Integer::sum);
        long end = System.currentTimeMillis();
        
        System.out.println("Sequential time: " + (end - start) + " ms");
        System.out.println("Sum: " + sum1);
        
        // Parallel
        start = System.currentTimeMillis();
        long sum2 = largeList.parallelStream()
            .map(n -> n * 2)
            .reduce(0, Integer::sum);
        end = System.currentTimeMillis();
        
        System.out.println("Parallel time: " + (end - start) + " ms");
        System.out.println("Sum: " + sum2);
        
        // Note: Parallel is faster for large datasets and CPU-intensive operations
    }
}
```

### ForkJoinPool Mechanics

```java
/**
 * Understanding ForkJoinPool
 */
public class ForkJoinPoolDemo {
    public static void main(String[] args) {
        
        // Default: Uses common ForkJoinPool
        // Parallelism = Runtime.getRuntime().availableProcessors()
        
        int parallelism = ForkJoinPool.commonPool().getParallelism();
        System.out.println("Default parallelism: " + parallelism);
        
        List<Integer> numbers = IntStream.rangeClosed(1, 100).boxed().collect(Collectors.toList());
        
        // Using common pool (default)
        numbers.parallelStream()
            .forEach(n -> System.out.println(Thread.currentThread().getName()));
        
        // Custom ForkJoinPool
        ForkJoinPool customPool = new ForkJoinPool(2);  // 2 threads
        
        try {
            customPool.submit(() -> {
                numbers.parallelStream()
                    .forEach(n -> System.out.println(Thread.currentThread().getName()));
            }).get();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Parallel Stream Pitfalls

```java
/**
 * Common pitfalls with parallel streams
 */
public class ParallelPitfalls {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        
        // PITFALL 1: Shared mutable state (NOT thread-safe!)
        List<Integer> resultList = new ArrayList<>();  // Shared state
        
        numbers.parallelStream()
            .map(n -> n * 2)
            .forEach(resultList::add);  // DANGEROUS! Race condition
        
        System.out.println("Size (might be wrong): " + resultList.size());
        
        // CORRECT: Use collect()
        List<Integer> correct = numbers.parallelStream()
            .map(n -> n * 2)
            .collect(Collectors.toList());  // Thread-safe
        
        System.out.println("Correct size: " + correct.size());
        
        // PITFALL 2: Order is not guaranteed
        System.out.println("\nParallel forEach (random order):");
        numbers.parallelStream()
            .forEach(System.out::println);  // Random order
        
        System.out.println("\nforEachOrdered (maintains order):");
        numbers.parallelStream()
            .forEachOrdered(System.out::println);  // Ordered
        
        // PITFALL 3: Not everything benefits from parallelization
        // Small datasets: overhead > benefit
        List<Integer> small = Arrays.asList(1, 2, 3);
        
        // Sequential is faster for small lists
        small.stream().forEach(System.out::println);  // Better
        
        // PITFALL 4: Blocking operations
        // Don't use parallel streams with blocking I/O
        
        // WHEN TO USE PARALLEL:
        // ✅ Large dataset (thousands+)
        // ✅ CPU-intensive operations
        // ✅ Stateless operations
        // ✅ No shared mutable state
        
        // WHEN NOT TO USE:
        // ❌ Small dataset
        // ❌ I/O-bound operations
        // ❌ Stateful operations
        // ❌ Shared mutable state
    }
}
```

---

# Part 3: Real-World Stream Examples

## 3.1 Data Processing Pipeline

```java
/**
 * Complete data processing example
 */
public class DataProcessingPipeline {
    
    public static void main(String[] args) {
        
        List<Transaction> transactions = Arrays.asList(
            new Transaction("T1", "Alice", 100, "2024-01-15"),
            new Transaction("T2", "Bob", 200, "2024-01-16"),
            new Transaction("T3", "Alice", 150, "2024-01-17"),
            new Transaction("T4", "Charlie", 300, "2024-01-18"),
            new Transaction("T5", "Bob", 250, "2024-01-19"),
            new Transaction("T6", "Alice", 175, "2024-01-20")
        );
        
        // Task: Find total amount per user for transactions > 100
        
        Map<String, Double> totalByUser = transactions.stream()
            .filter(t -> t.getAmount() > 100)
            .collect(Collectors.groupingBy(
                Transaction::getUser,
                Collectors.summingDouble(Transaction::getAmount)
            ));
        
        System.out.println("Total by user (amount > 100): " + totalByUser);
        
        // Task: Top 3 users by transaction amount
        List<String> top3Users = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getUser,
                Collectors.summingDouble(Transaction::getAmount)
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(3)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
        
        System.out.println("Top 3 users: " + top3Users);
        
        // Task: Average transaction amount by user
        Map<String, Double> avgByUser = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getUser,
                Collectors.averagingDouble(Transaction::getAmount)
            ));
        
        System.out.println("Average by user: " + avgByUser);
    }
    
    static class Transaction {
        private String id;
        private String user;
        private double amount;
        private String date;
        
        Transaction(String id, String user, double amount, String date) {
            this.id = id;
            this.user = user;
            this.amount = amount;
            this.date = date;
        }
        
        String getId() { return id; }
        String getUser() { return user; }
        double getAmount() { return amount; }
        String getDate() { return date; }
    }
}
```

## 3.2 Complex Filtering and Transformation

```java
/**
 * Advanced filtering and transformation
 */
public class ComplexFiltering {
    public static void main(String[] args) {
        
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", 30, "Engineering", 80000),
            new Employee("Bob", 25, "Marketing", 60000),
            new Employee("Charlie", 35, "Engineering", 95000),
            new Employee("David", 28, "Sales", 70000),
            new Employee("Eve", 32, "Engineering", 85000)
        );
        
        // Find senior engineers (age > 30, salary > 80000)
        List<Employee> seniorEngineers = employees.stream()
            .filter(e -> e.getDepartment().equals("Engineering"))
            .filter(e -> e.getAge() > 30)
            .filter(e -> e.getSalary() > 80000)
            .collect(Collectors.toList());
        
        System.out.println("Senior Engineers:");
        seniorEngineers.forEach(e -> System.out.println(e.getName()));
        
        // Department-wise salary statistics
        Map<String, DoubleSummaryStatistics> salaryStats = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summarizingDouble(Employee::getSalary)
            ));
        
        System.out.println("\nSalary statistics by department:");
        salaryStats.forEach((dept, stats) -> {
            System.out.println(dept + ":");
            System.out.println("  Average: $" + stats.getAverage());
            System.out.println("  Max: $" + stats.getMax());
            System.out.println("  Min: $" + stats.getMin());
        });
        
        // Salary increase of 10% for engineers
        List<Employee> updated = employees.stream()
            .map(e -> {
                if (e.getDepartment().equals("Engineering")) {
                    return new Employee(
                        e.getName(),
                        e.getAge(),
                        e.getDepartment(),
                        e.getSalary() * 1.10
                    );
                }
                return e;
            })
            .collect(Collectors.toList());
        
        System.out.println("\nAfter 10% raise for engineers:");
        updated.forEach(e -> System.out.println(e.getName() + ": $" + e.getSalary()));
    }
    
    static class Employee {
        private String name;
        private int age;
        private String department;
        private double salary;
        
        Employee(String name, int age, String department, double salary) {
            this.name = name;
            this.age = age;
            this.department = department;
            this.salary = salary;
        }
        
        String getName() { return name; }
        int getAge() { return age; }
        String getDepartment() { return department; }
        double getSalary() { return salary; }
    }
}
```

---

# Practice Exercises

## Exercise Set 1: Lambdas and Functional Interfaces

### Exercise 1.1: Custom Functional Interfaces
Create functional interfaces for:
1. String validator (returns boolean)
2. Number formatter (formats number as string)
3. List processor (processes list and returns result)

### Exercise 1.2: Method References
Convert these lambdas to method references:
```java
Function<String, Integer> f1 = s -> s.length();
Predicate<String> p1 = s -> s.isEmpty();
Consumer<String> c1 = s -> System.out.println(s);
```

## Exercise Set 2: Stream Operations

### Exercise 2.1: Student Grade Processor
Given list of students with grades:
1. Find average grade
2. Count students above average
3. Get top 3 students
4. Group students by grade range (A, B, C)

### Exercise 2.2: Word Frequency Counter
Given text:
1. Split into words
2. Count frequency of each word
3. Find top 10 most frequent words
4. Filter words longer than 5 characters

### Exercise 2.3: Product Inventory
Given products with price and category:
1. Find most expensive product per category
2. Calculate total inventory value
3. Get products needing restock (quantity < 10)
4. Apply 20% discount to electronics

## Exercise Set 3: Collectors

### Exercise 3.1: Employee Analytics
Given employees with department and salary:
1. Group by department
2. Calculate average salary per department
3. Find department with highest average salary
4. Count employees in each department

### Exercise 3.2: Transaction Analysis
Given transactions with user and amount:
1. Total amount per user
2. Users with transactions > $1000
3. Average transaction amount
4. Top 5 spenders

## Exercise Set 4: Parallel Streams

### Exercise 4.1: Performance Testing
Compare sequential vs parallel for:
1. Summing large list (1M+ elements)
2. Filtering and mapping
3. Finding prime numbers

### Exercise 4.2: Parallel Processing Safety
Fix thread-safety issues in provided code.

---

# Interview Questions & Answers

## Functional Programming Questions

### Q1: What is a functional interface? What is @FunctionalInterface?
**Answer:**
**Functional Interface**: Interface with exactly one abstract method (SAM - Single Abstract Method).

**@FunctionalInterface**: Annotation that ensures interface has only one abstract method. Causes compile error if multiple abstract methods are added.

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);  // Single abstract method
    
    // default and static methods are allowed
    default void printResult(int result) {
        System.out.println("Result: " + result);
    }
}
```

**Benefits:**
- Can be implemented with lambda expressions
- Compiler enforces single abstract method rule
- Makes code more readable and concise

### Q2: Difference between map() and flatMap()?
**Answer:**

**map()**: Transforms each element (one-to-one mapping)
```java
List<String> names = Arrays.asList("Alice", "Bob");
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
// [5, 3]
```

**flatMap()**: Flattens nested structures (one-to-many mapping)
```java
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
List<Integer> flattened = nested.stream()
    .flatMap(list -> list.stream())
    .collect(Collectors.toList());
// [1, 2, 3, 4]
```

**When to use:**
- map(): Transform each element to another type
- flatMap(): When each element maps to multiple elements or a stream

### Q3: What's the difference between intermediate and terminal operations?
**Answer:**

**Intermediate Operations:**
- Return Stream (can be chained)
- Lazy (don't execute until terminal operation)
- Examples: filter(), map(), flatMap(), sorted(), distinct()

**Terminal Operations:**
- Return result or void (end of pipeline)
- Trigger execution
- Examples: collect(), forEach(), reduce(), count(), anyMatch()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

numbers.stream()
    .filter(n -> n % 2 == 0)  // Intermediate (lazy)
    .map(n -> n * n)          // Intermediate (lazy)
    .forEach(System.out::println);  // Terminal (executes now)
```

### Q4: Explain lazy evaluation in streams.
**Answer:**

**Lazy Evaluation**: Intermediate operations don't execute until a terminal operation is called.

```java
Stream<Integer> stream = numbers.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);
        return n % 2 == 0;
    })
    .map(n -> {
        System.out.println("Mapping: " + n);
        return n * n;
    });
// Nothing printed yet!

stream.forEach(System.out::println);  // NOW it executes
```

**Benefits:**
- Performance optimization (skip unnecessary work)
- Short-circuiting possible (stop early)
- Fusion optimization (combine operations)

### Q5: What are the main built-in functional interfaces?
**Answer:**

| Interface | Signature | Example |
|-----------|-----------|---------|
| **Predicate\<T>** | T → boolean | `n -> n > 0` |
| **Function\<T,R>** | T → R | `s -> s.length()` |
| **Consumer\<T>** | T → void | `s -> System.out.println(s)` |
| **Supplier\<T>** | () → T | `() -> "Hello"` |
| **UnaryOperator\<T>** | T → T | `n -> n * 2` |
| **BinaryOperator\<T>** | (T,T) → T | `(a,b) -> a+b` |

All have bi-variants: BiPredicate, BiFunction, BiConsumer

### Q6: What's the difference between Collection.forEach() and Stream.forEach()?
**Answer:**

**Collection.forEach()**: External iteration on collection
```java
list.forEach(System.out::println);
```

**Stream.forEach()**: Terminal operation on stream
```java
list.stream().forEach(System.out::println);
```

**Key differences:**
1. **Stream.forEach()** can be part of pipeline with filtering/mapping
2. **Stream** can be parallel: `parallelStream().forEach()`
3. **Collection.forEach()** is simpler for just iteration

**When to use Stream:**
- When you need to filter/map before iteration
- When you want parallel processing
- When building complex pipelines

### Q7: Explain groupingBy() with examples.
**Answer:**

**groupingBy()**: Groups elements by classifier function

**Basic grouping:**
```java
Map<String, List<Person>> byDept = people.stream()
    .collect(Collectors.groupingBy(Person::getDepartment));
// {Engineering=[Alice, Charlie], Marketing=[Bob]}
```

**With downstream collector:**
```java
// Count per department
Map<String, Long> countByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.counting()
    ));
// {Engineering=2, Marketing=1}

// Average salary per department
Map<String, Double> avgSalary = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.averagingDouble(Person::getSalary)
    ));
```

**Multilevel grouping:**
```java
Map<String, Map<Integer, List<Person>>> byDeptAndAge = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.groupingBy(Person::getAge)
    ));
```

### Q8: What are Gatherers (JEP 461)?
**Answer:**

**Gatherers** (Java 22+): Custom intermediate stream operations for advanced transformations.

**Built-in gatherers:**
```java
// Fixed-size windows
List<List<Integer>> windows = numbers.stream()
    .gather(Gatherers.windowFixed(3))
    .collect(Collectors.toList());
// [[1,2,3], [4,5,6], [7,8,9]]

// Sliding windows
List<List<Integer>> sliding = numbers.stream()
    .gather(Gatherers.windowSliding(3))
    .collect(Collectors.toList());
// [[1,2,3], [2,3,4], [3,4,5], ...]

// Running totals
List<Integer> runningSum = numbers.stream()
    .gather(Gatherers.scan(() -> 0, (acc, n) -> acc + n))
    .collect(Collectors.toList());
// [1, 3, 6, 10, 15, ...]
```

**Use cases:**
- Window-based processing
- Custom accumulations
- State transformations
- Batch processing

### Q9: When should you use parallel streams?
**Answer:**

**Use parallel streams when:**
✅ Large dataset (thousands+ elements)  
✅ CPU-intensive operations  
✅ Stateless operations  
✅ No shared mutable state  
✅ Operations are independent  

**Avoid parallel streams when:**
❌ Small dataset (overhead > benefit)  
❌ I/O-bound operations (blocking)  
❌ Shared mutable state  
❌ Order matters (use forEachOrdered)  
❌ Simple operations (overhead not worth it)  

**Example:**
```java
// Good use: Large dataset, CPU-intensive
List<Integer> large = IntStream.range(1, 10_000_000)
    .boxed()
    .collect(Collectors.toList());

long sum = large.parallelStream()
    .map(n -> expensiveComputation(n))
    .reduce(0L, Long::sum);
```

### Q10: What is the ForkJoinPool?
**Answer:**

**ForkJoinPool**: Framework for parallel execution using work-stealing algorithm.

**How it works:**
1. Tasks are divided into subtasks (fork)
2. Subtasks execute in parallel
3. Results are combined (join)
4. Idle threads steal work from busy threads

**Default behavior:**
```java
// Uses common pool
int parallelism = ForkJoinPool.commonPool().getParallelism();
// Typically: number of CPU cores

parallelStream() uses common pool by default
```

**Custom pool:**
```java
ForkJoinPool customPool = new ForkJoinPool(4);  // 4 threads

customPool.submit(() -> {
    list.parallelStream()
        .forEach(process);
}).get();
```

**Characteristics:**
- Work-stealing algorithm (load balancing)
- Designed for recursive divide-and-conquer
- Efficient for CPU-bound tasks

---

# Week 5 Summary

## Key Takeaways

### Functional Programming
✅ Lambda expressions and method references  
✅ Functional interfaces (Predicate, Function, Consumer, Supplier)  
✅ Pure functions and immutability  
✅ Declarative vs imperative programming  

### Stream API
✅ Intermediate operations (filter, map, flatMap, sorted)  
✅ Terminal operations (collect, reduce, forEach, match)  
✅ Lazy evaluation and short-circuiting  
✅ Method chaining and pipelines  

### Collectors
✅ Basic collectors (toList, toSet, toMap)  
✅ Grouping and partitioning  
✅ teeing() for dual collection  
✅ Statistical collectors  

### Advanced Features
✅ Gatherers (JEP 461) for custom operations  
✅ Parallel streams with ForkJoinPool  
✅ Performance considerations  
✅ Thread-safety pitfalls  

---

**End of Week 5 Material**

You've completed the core Java fundamentals! These 5 weeks cover everything needed to write modern, professional Java code and ace technical interviews.

**Next Steps:**
- Practice exercises extensively
- Build real projects using these concepts
- Review interview questions
- Prepare for advanced topics (Spring, Microservices, etc.)
