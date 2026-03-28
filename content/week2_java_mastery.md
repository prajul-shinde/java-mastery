# Week 2: OOP Pillars I & II (Data & Hierarchy) - Java Mastery Guide

## Table of Contents
1. [OOP Fundamentals Review](#oop-fundamentals-review)
2. [Encapsulation](#encapsulation)
3. [Inheritance](#inheritance)
4. [The Module System](#the-module-system)
5. [Practice Exercises](#practice-exercises)
6. [Interview Questions & Answers](#interview-questions--answers)

---

## OOP Fundamentals Review

### The Four Pillars of OOP

Before diving into modern Java features, let's understand the foundational concepts:

**1. Encapsulation**: Bundling data and methods, hiding internal details
**2. Inheritance**: Deriving new classes from existing ones
**3. Polymorphism**: Same interface, different implementations (Week 3)
**4. Abstraction**: Hiding complexity, showing only essentials (Week 3)

### Why OOP?

**Traditional Procedural Approach:**
```java
// Procedural - scattered data and functions
void main() {
    String userName = "John";
    int userAge = 25;
    String userEmail = "john@example.com";
    
    // Functions operate on scattered data
    printUser(userName, userAge, userEmail);
    validateAge(userAge);
}

void printUser(String name, int age, String email) {
    println(name + " - " + age + " - " + email);
}

void validateAge(int age) {
    if (age < 0) println("Invalid age!");
}
```

**Object-Oriented Approach:**
```java
void main() {
    User user = new User("John", 25, "john@example.com");
    user.print();
    user.validateAge();
}

class User {
    private String name;
    private int age;
    private String email;
    
    User(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    void print() {
        println(name + " - " + age + " - " + email);
    }
    
    void validateAge() {
        if (age < 0) println("Invalid age!");
    }
}
```

**Benefits of OOP:**
- **Organization**: Related data and behavior grouped together
- **Reusability**: Classes can be reused across projects
- **Maintainability**: Changes localized to specific classes
- **Security**: Data hiding through encapsulation
- **Scalability**: Easy to extend with inheritance

---

## Encapsulation

### What is Encapsulation?

**Definition**: Bundling data (fields) and methods that operate on that data within a single unit (class), and restricting direct access to some components.

**Core Principles:**
1. **Data Hiding**: Make fields private
2. **Controlled Access**: Provide public methods (getters/setters)
3. **Validation**: Ensure data integrity through methods

### Traditional Java Classes

**Basic Class Structure:**
```java
class Person {
    // Fields (private - data hiding)
    private String name;
    private int age;
    
    // Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Getters (controlled access)
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    // Setters (with validation)
    public void setName(String name) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        this.name = name;
    }
    
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age");
        }
        this.age = age;
    }
    
    // Other methods
    public void introduce() {
        println("Hi, I'm " + name + ", " + age + " years old.");
    }
}

void main() {
    Person p = new Person("Alice", 30);
    p.introduce();
    
    p.setAge(31);  // Controlled modification with validation
    // p.age = -5;  // ❌ Compilation error - field is private
}
```

### Access Modifiers

**Understanding Visibility:**

| Modifier | Same Class | Same Package | Subclass | Everywhere |
|----------|------------|--------------|----------|------------|
| `private` | ✓ | ✗ | ✗ | ✗ |
| default (no modifier) | ✓ | ✓ | ✗ | ✗ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| `public` | ✓ | ✓ | ✓ | ✓ |

**Examples:**
```java
class AccessDemo {
    private int privateField = 1;       // Only within this class
    int defaultField = 2;               // Same package only
    protected int protectedField = 3;   // Same package + subclasses
    public int publicField = 4;         // Everywhere
    
    private void privateMethod() {
        println("Only accessible within this class");
    }
    
    protected void protectedMethod() {
        println("Accessible in subclasses too");
    }
    
    public void publicMethod() {
        println("Accessible everywhere");
        privateMethod();  // Can call private method within same class
    }
}

void main() {
    AccessDemo obj = new AccessDemo();
    // obj.privateField;     // ❌ Compilation error
    // obj.privateMethod();  // ❌ Compilation error
    obj.publicMethod();      // ✓ Works
}
```

### The Problem with Traditional Classes

**Boilerplate Code:**
```java
// To create a simple data holder, you need:
class Point {
    private final double x;  // 1. Fields
    private final double y;
    
    public Point(double x, double y) {  // 2. Constructor
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }  // 3. Getters
    public double getY() { return y; }
    
    @Override
    public boolean equals(Object obj) {  // 4. equals()
        if (this == obj) return true;
        if (!(obj instanceof Point)) return false;
        Point other = (Point) obj;
        return Double.compare(x, other.x) == 0 && 
               Double.compare(y, other.y) == 0;
    }
    
    @Override
    public int hashCode() {  // 5. hashCode()
        return Objects.hash(x, y);
    }
    
    @Override
    public String toString() {  // 6. toString()
        return "Point[x=" + x + ", y=" + y + "]";
    }
}
```

**That's ~30 lines for a simple 2-field data holder!**

---

## Records: Modern Encapsulation

### Introduction to Records

**Records** (Java 14+) are a special kind of class designed for immutable data carriers.

**The Same Point Class as a Record:**
```java
record Point(double x, double y) {}
```

**That's it! 1 line instead of 30!**

### What Records Provide Automatically

```java
record User(String name, int age, String email) {}

void main() {
    User user = new User("Alice", 30, "alice@example.com");
    
    // 1. Automatic getters (no 'get' prefix)
    println(user.name());   // "Alice"
    println(user.age());    // 30
    println(user.email());  // "alice@example.com"
    
    // 2. Automatic toString()
    println(user);  // User[name=Alice, age=30, email=alice@example.com]
    
    // 3. Automatic equals()
    User user2 = new User("Alice", 30, "alice@example.com");
    println(user.equals(user2));  // true (compares values, not references)
    
    // 4. Automatic hashCode()
    println(user.hashCode() == user2.hashCode());  // true
    
    // 5. Immutability - no setters!
    // user.name = "Bob";  // ❌ Compilation error - records are immutable
}
```

### Record Components

**Terminology:**
```java
record Product(String name, double price, int quantity) {}
//             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//                    Record Components
//                    (automatically become private final fields)
```

**What happens behind the scenes:**
```java
// When you write this:
record Product(String name, double price, int quantity) {}

// Java generates (conceptually):
final class Product {
    private final String name;
    private final double price;
    private final int quantity;
    
    public Product(String name, double price, int quantity) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }
    
    public String name() { return name; }
    public double price() { return price; }
    public int quantity() { return quantity; }
    
    // equals(), hashCode(), toString() also generated
}
```

### When to Use Records

**✓ Use Records For:**
- Data Transfer Objects (DTOs)
- Configuration objects
- Database query results
- API responses
- Immutable value objects
- Multiple return values

**✗ Don't Use Records For:**
- Mutable objects (need setters)
- Objects with complex behavior
- Objects that need inheritance (records can't extend classes)
- When you need private constructors

### Compact Constructors

**Validation in Records:**
```java
record User(String name, int age) {
    // Compact constructor - no parameter list!
    User {
        // Validate and normalize data
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Age must be 0-150");
        }
        
        // Normalize name
        name = name.trim();
        
        // No need to assign fields - done automatically!
        // this.name = name;  // Not needed!
        // this.age = age;    // Not needed!
    }
}

void main() {
    User user1 = new User("  Alice  ", 30);
    println(user1.name());  // "Alice" (trimmed)
    
    // User user2 = new User("", 30);  // ❌ IllegalArgumentException
    // User user3 = new User("Bob", -5);  // ❌ IllegalArgumentException
}
```

**Traditional Constructor (less common):**
```java
record Point(double x, double y) {
    // Canonical constructor (full parameter list)
    public Point(double x, double y) {
        if (Double.isNaN(x) || Double.isNaN(y)) {
            throw new IllegalArgumentException("Coordinates cannot be NaN");
        }
        this.x = x;
        this.y = y;
    }
}
```

### Additional Constructors

**Multiple ways to create records:**
```java
record User(String name, int age, String email) {
    // Compact constructor for validation
    User {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name required");
        }
        name = name.trim();
    }
    
    // Alternate constructor - must call canonical constructor
    User(String name, int age) {
        this(name, age, name.toLowerCase() + "@example.com");
    }
    
    // Another alternate constructor
    User(String name) {
        this(name, 18);  // Default age
    }
}

void main() {
    User u1 = new User("Alice", 30, "alice@mail.com");
    User u2 = new User("Bob", 25);  // email auto-generated
    User u3 = new User("Charlie");  // age=18, email auto-generated
    
    println(u1);
    println(u2);
    println(u3);
}
```

### Custom Methods in Records

**Records can have methods:**
```java
record Rectangle(double width, double height) {
    // Compact constructor
    Rectangle {
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Dimensions must be positive");
        }
    }
    
    // Custom methods
    double area() {
        return width * height;
    }
    
    double perimeter() {
        return 2 * (width + height);
    }
    
    boolean isSquare() {
        return width == height;
    }
    
    // Static factory method
    static Rectangle square(double side) {
        return new Rectangle(side, side);
    }
}

void main() {
    Rectangle rect = new Rectangle(5, 3);
    println("Area: " + rect.area());           // 15.0
    println("Perimeter: " + rect.perimeter()); // 16.0
    println("Is square? " + rect.isSquare());  // false
    
    Rectangle sq = Rectangle.square(5);
    println("Is square? " + sq.isSquare());    // true
}
```

### Overriding Generated Methods

**Customizing toString(), equals(), etc.:**
```java
record Money(double amount, String currency) {
    // Custom toString()
    @Override
    public String toString() {
        return String.format("%.2f %s", amount, currency);
    }
    
    // Custom equals - only compare amount
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Money other)) return false;
        return Double.compare(amount, other.amount) == 0;
        // Ignoring currency in comparison!
    }
    
    // If you override equals, override hashCode too
    @Override
    public int hashCode() {
        return Double.hashCode(amount);
    }
}

void main() {
    Money m1 = new Money(100.5, "USD");
    Money m2 = new Money(100.5, "EUR");
    
    println(m1);  // "100.50 USD" (custom toString)
    println(m1.equals(m2));  // true (custom equals - only compares amount)
}
```

### Record Patterns (Deconstruction)

**Pattern Matching with Records (Java 19+):**
```java
record Point(int x, int y) {}

void main() {
    Object obj = new Point(3, 4);
    
    // Traditional instanceof
    if (obj instanceof Point) {
        Point p = (Point) obj;
        int x = p.x();
        int y = p.y();
        println("x=" + x + ", y=" + y);
    }
    
    // Record Pattern - deconstruct in one step!
    if (obj instanceof Point(int x, int y)) {
        println("x=" + x + ", y=" + y);  // x and y are already extracted!
    }
}
```

**In Switch Expressions:**
```java
record Point(int x, int y) {}
record Circle(Point center, double radius) {}
record Rectangle(Point topLeft, Point bottomRight) {}

String describe(Object shape) {
    return switch (shape) {
        case Point(int x, int y) -> 
            "Point at (%d, %d)".formatted(x, y);
        
        case Circle(Point(int cx, int cy), double r) -> 
            "Circle at (%d, %d) with radius %.1f".formatted(cx, cy, r);
        
        case Rectangle(Point(int x1, int y1), Point(int x2, int y2)) -> 
            "Rectangle from (%d, %d) to (%d, %d)".formatted(x1, y1, x2, y2);
        
        case null -> "null shape";
        default -> "Unknown shape";
    };
}

void main() {
    Point p = new Point(5, 10);
    Circle c = new Circle(new Point(0, 0), 5.5);
    Rectangle r = new Rectangle(new Point(0, 0), new Point(10, 20));
    
    println(describe(p));  // Point at (5, 10)
    println(describe(c));  // Circle at (0, 0) with radius 5.5
    println(describe(r));  // Rectangle from (0, 0) to (10, 20)
}
```

### Nested Record Patterns

**Deep deconstruction:**
```java
record Address(String street, String city, String zip) {}
record Person(String name, int age, Address address) {}

void main() {
    Person person = new Person(
        "Alice", 
        30, 
        new Address("123 Main St", "Springfield", "12345")
    );
    
    // Nested pattern matching
    if (person instanceof Person(
        String name, 
        int age, 
        Address(String street, String city, String zip)
    )) {
        println("Name: " + name);
        println("Age: " + age);
        println("Street: " + street);
        println("City: " + city);
        println("Zip: " + zip);
    }
}
```

### Records vs Classes: Complete Comparison

**Use a Record when:**
```java
// Immutable data carrier
record User(String name, int age) {}

// Configuration
record DatabaseConfig(String url, String user, String password) {}

// API Response
record ApiResponse(int status, String message, Object data) {}

// Coordinates/Math
record Point(double x, double y) {}
```

**Use a Class when:**
```java
// Mutable object
class Counter {
    private int count = 0;
    void increment() { count++; }
    int getCount() { return count; }
}

// Complex behavior
class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    void addItem(Item item) { items.add(item); }
    void removeItem(Item item) { items.remove(item); }
    double calculateTotal() { /* complex logic */ }
}

// Need inheritance
class Animal {
    void makeSound() {}
}
class Dog extends Animal {
    @Override void makeSound() { println("Woof!"); }
}
```

### Real-World Record Examples

**1. Database Result:**
```java
record Employee(
    int id, 
    String firstName, 
    String lastName, 
    String department, 
    double salary
) {
    String fullName() {
        return firstName + " " + lastName;
    }
    
    double annualSalary() {
        return salary * 12;
    }
}

void main() {
    // Simulating database query result
    List<Employee> employees = List.of(
        new Employee(1, "John", "Doe", "Engineering", 5000),
        new Employee(2, "Jane", "Smith", "Marketing", 4500)
    );
    
    employees.forEach(emp -> 
        println("%s (%s): $%.2f/year".formatted(
            emp.fullName(), 
            emp.department(), 
            emp.annualSalary()
        ))
    );
}
```

**2. HTTP Request/Response:**
```java
record HttpRequest(
    String method,
    String url,
    Map<String, String> headers,
    String body
) {
    HttpRequest {
        method = method.toUpperCase();
        if (headers == null) headers = Map.of();
    }
}

record HttpResponse(
    int statusCode,
    String statusMessage,
    Map<String, String> headers,
    String body
) {
    boolean isSuccess() {
        return statusCode >= 200 && statusCode < 300;
    }
}

void main() {
    HttpRequest req = new HttpRequest(
        "GET",
        "https://api.example.com/users",
        Map.of("Authorization", "Bearer token123"),
        ""
    );
    
    HttpResponse res = new HttpResponse(
        200,
        "OK",
        Map.of("Content-Type", "application/json"),
        "{\"users\": []}"
    );
    
    println("Request: " + req.method() + " " + req.url());
    println("Response: " + res.statusCode() + " - " + 
            (res.isSuccess() ? "Success" : "Failed"));
}
```

**3. Configuration Management:**
```java
record ServerConfig(
    String host,
    int port,
    boolean ssl,
    int maxConnections,
    int timeoutSeconds
) {
    ServerConfig {
        if (port < 1 || port > 65535) {
            throw new IllegalArgumentException("Invalid port");
        }
        if (maxConnections < 1) {
            throw new IllegalArgumentException("Invalid max connections");
        }
    }
    
    // Factory methods for common configurations
    static ServerConfig development() {
        return new ServerConfig("localhost", 8080, false, 10, 30);
    }
    
    static ServerConfig production() {
        return new ServerConfig("prod.example.com", 443, true, 1000, 60);
    }
    
    String connectionUrl() {
        return (ssl ? "https://" : "http://") + host + ":" + port;
    }
}

void main() {
    ServerConfig dev = ServerConfig.development();
    ServerConfig prod = ServerConfig.production();
    
    println("Dev: " + dev.connectionUrl());
    println("Prod: " + prod.connectionUrl());
}
```

---

## Inheritance

### What is Inheritance?

**Definition**: A mechanism where a new class (subclass/child) derives properties and behaviors from an existing class (superclass/parent).

**Real-World Analogy:**
```
Animal (parent)
├── Dog (child)
├── Cat (child)
└── Bird (child)
```

### Traditional Inheritance

**Basic Inheritance:**
```java
// Parent class (superclass)
class Animal {
    protected String name;
    protected int age;
    
    Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    void eat() {
        println(name + " is eating");
    }
    
    void sleep() {
        println(name + " is sleeping");
    }
    
    void makeSound() {
        println(name + " makes a sound");
    }
}

// Child class (subclass)
class Dog extends Animal {
    private String breed;
    
    Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    @Override
    void makeSound() {
        println(name + " barks: Woof! Woof!");
    }
    
    // Dog-specific method
    void fetch() {
        println(name + " is fetching the ball");
    }
}

class Cat extends Animal {
    Cat(String name, int age) {
        super(name, age);
    }
    
    @Override
    void makeSound() {
        println(name + " meows: Meow!");
    }
    
    // Cat-specific method
    void scratch() {
        println(name + " is scratching");
    }
}

void main() {
    Dog dog = new Dog("Buddy", 3, "Golden Retriever");
    Cat cat = new Cat("Whiskers", 2);
    
    dog.eat();       // Inherited from Animal
    dog.makeSound(); // Overridden in Dog
    dog.fetch();     // Dog-specific
    
    cat.eat();       // Inherited from Animal
    cat.makeSound(); // Overridden in Cat
    cat.scratch();   // Cat-specific
}
```

### The `super` Keyword

**Three uses of `super`:**

```java
class Parent {
    protected int value = 10;
    
    Parent(int value) {
        this.value = value;
    }
    
    void display() {
        println("Parent value: " + value);
    }
}

class Child extends Parent {
    private int value = 20;  // Hides parent's value
    
    Child(int parentValue, int childValue) {
        super(parentValue);  // 1. Call parent constructor
        this.value = childValue;
    }
    
    void display() {
        println("Child value: " + value);        // 20
        println("Parent value: " + super.value); // 2. Access parent field
        super.display();                         // 3. Call parent method
    }
}

void main() {
    Child c = new Child(10, 20);
    c.display();
}
```

### Method Overriding

**Rules for Overriding:**
1. Same method signature (name + parameters)
2. Return type must be same or covariant (subtype)
3. Access level must be same or more accessible
4. Cannot override `final` or `static` methods
5. Use `@Override` annotation (recommended)

```java
class Shape {
    double area() {
        return 0;
    }
    
    final void printInfo() {  // Cannot be overridden
        println("This is a shape");
    }
}

class Circle extends Shape {
    private double radius;
    
    Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    double area() {  // Correct override
        return Math.PI * radius * radius;
    }
    
    // @Override
    // void printInfo() {}  // ❌ Error: cannot override final method
}

void main() {
    Shape s = new Circle(5);
    println("Area: " + s.area());  // Calls Circle's area()
    s.printInfo();                  // Calls Shape's printInfo()
}
```

### Constructor Chaining

**Understanding Constructor Execution Order:**
```java
class GrandParent {
    GrandParent() {
        println("1. GrandParent constructor");
    }
}

class Parent extends GrandParent {
    Parent() {
        super();  // Implicit if not specified
        println("2. Parent constructor");
    }
}

class Child extends Parent {
    Child() {
        super();  // Implicit if not specified
        println("3. Child constructor");
    }
}

void main() {
    new Child();
    // Output:
    // 1. GrandParent constructor
    // 2. Parent constructor
    // 3. Child constructor
}
```

### Multiple Inheritance Problem

**Java doesn't allow multiple class inheritance:**
```java
class A {
    void method() { println("A"); }
}

class B {
    void method() { println("B"); }
}

// class C extends A, B {}  // ❌ Error: Java doesn't support this
// Which method() should C inherit?
```

**Solution: Interfaces (covered in Week 3)**

### The `Object` Class

**Every class inherits from Object:**
```java
class MyClass {
    // Implicitly extends Object
}

// Equivalent to:
// class MyClass extends Object {}

void main() {
    MyClass obj = new MyClass();
    
    // Methods inherited from Object:
    println(obj.toString());   // Object representation
    println(obj.hashCode());   // Hash code
    println(obj.equals(obj));  // Equality check
    println(obj.getClass());   // Class information
}
```

### Inheritance Hierarchy Example

```java
// Level 1: Base class
class Employee {
    protected String name;
    protected double salary;
    
    Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    double calculatePay() {
        return salary;
    }
    
    void displayInfo() {
        println("Name: " + name + ", Salary: $" + salary);
    }
}

// Level 2: Specialized employees
class Manager extends Employee {
    private double bonus;
    
    Manager(String name, double salary, double bonus) {
        super(name, salary);
        this.bonus = bonus;
    }
    
    @Override
    double calculatePay() {
        return salary + bonus;
    }
    
    @Override
    void displayInfo() {
        super.displayInfo();
        println("Bonus: $" + bonus);
    }
}

class Developer extends Employee {
    private int hoursWorked;
    private double hourlyRate;
    
    Developer(String name, int hoursWorked, double hourlyRate) {
        super(name, 0);  // Base salary is 0
        this.hoursWorked = hoursWorked;
        this.hourlyRate = hourlyRate;
    }
    
    @Override
    double calculatePay() {
        return hoursWorked * hourlyRate;
    }
}

// Level 3: Further specialization
class SeniorDeveloper extends Developer {
    private double stockOptions;
    
    SeniorDeveloper(String name, int hours, double rate, double stocks) {
        super(name, hours, rate);
        this.stockOptions = stocks;
    }
    
    @Override
    double calculatePay() {
        return super.calculatePay() + stockOptions;
    }
}

void main() {
    Employee[] employees = {
        new Manager("Alice", 80000, 10000),
        new Developer("Bob", 160, 50),
        new SeniorDeveloper("Charlie", 160, 75, 5000)
    };
    
    for (Employee emp : employees) {
        emp.displayInfo();
        println("Total Pay: $" + emp.calculatePay());
        println();
    }
}
```

---

## Sealed Classes: Controlled Inheritance

### The Problem with Open Inheritance

**Traditional inheritance is unrestricted:**
```java
class Shape {
    double area() { return 0; }
}

// Anyone can extend Shape
class Circle extends Shape { /* ... */ }
class Square extends Shape { /* ... */ }
class Triangle extends Shape { /* ... */ }

// Unexpected extensions
class WeirdShape extends Shape { /* ... */ }
class BrokenShape extends Shape { /* ... */ }
```

**Issues:**
- Can't predict all possible subclasses
- Switch statements incomplete
- Maintenance nightmare
- Security concerns

### Sealed Classes: Explicit Control

**Sealed classes restrict which classes can extend them (Java 17+):**

```java
// Only Circle, Rectangle, and Triangle can extend Shape
sealed class Shape 
    permits Circle, Rectangle, Triangle {
    
    abstract double area();
}

final class Circle extends Shape {
    private final double radius;
    
    Circle(double radius) { this.radius = radius; }
    
    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

final class Rectangle extends Shape {
    private final double width, height;
    
    Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    double area() {
        return width * height;
    }
}

final class Triangle extends Shape {
    private final double base, height;
    
    Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    double area() {
        return 0.5 * base * height;
    }
}

// This won't compile:
// class Pentagon extends Shape {}  // ❌ Error: Shape is sealed

void main() {
    Shape circle = new Circle(5);
    Shape rect = new Rectangle(4, 6);
    Shape triangle = new Triangle(3, 4);
    
    println("Circle area: " + circle.area());
    println("Rectangle area: " + rect.area());
    println("Triangle area: " + triangle.area());
}
```

### Sealed Class Rules

**Permitted subclasses must be one of:**
1. **`final`** - Cannot be extended further
2. **`sealed`** - Can be extended, but also controlled
3. **`non-sealed`** - Opens inheritance again

```java
sealed class Vehicle 
    permits Car, Truck, Motorcycle {}

final class Car extends Vehicle {
    // Cannot be extended
}

sealed class Truck extends Vehicle 
    permits PickupTruck, SemiTruck {
    // Can be extended, but controlled
}

final class PickupTruck extends Truck {}
final class SemiTruck extends Truck {}

non-sealed class Motorcycle extends Vehicle {
    // Anyone can extend Motorcycle
}

class SportBike extends Motorcycle {  // ✓ Allowed
    // non-sealed allows open extension
}
```

### Sealed Interfaces

**Interfaces can also be sealed:**
```java
sealed interface Payment 
    permits CreditCard, DebitCard, Cash {}

record CreditCard(String number, String cvv) implements Payment {}
record DebitCard(String number, String pin) implements Payment {}
record Cash(double amount) implements Payment {}

// record Bitcoin(...) implements Payment {}  // ❌ Error

void main() {
    List<Payment> payments = List.of(
        new CreditCard("1234-5678-9012-3456", "123"),
        new DebitCard("9876-5432-1098-7654", "1234"),
        new Cash(50.00)
    );
    
    for (Payment p : payments) {
        processPayment(p);
    }
}

void processPayment(Payment payment) {
    switch (payment) {
        case CreditCard(String num, String cvv) -> 
            println("Processing credit card: " + num);
        case DebitCard(String num, String pin) -> 
            println("Processing debit card: " + num);
        case Cash(double amount) -> 
            println("Processing cash: $" + amount);
        // No default needed - all cases covered!
    }
}
```

### Exhaustiveness in Pattern Matching

**Sealed classes enable exhaustive switch:**
```java
sealed interface Result 
    permits Success, Failure {}

record Success(String data) implements Result {}
record Failure(String error) implements Result {}

void main() {
    Result result = fetchData();
    
    // Compiler knows all possible types
    String message = switch (result) {
        case Success(String data) -> "Got data: " + data;
        case Failure(String error) -> "Error: " + error;
        // No default needed - compiler verifies all cases covered!
    };
    
    println(message);
}

Result fetchData() {
    return Math.random() > 0.5 
        ? new Success("User data") 
        : new Failure("Network error");
}
```

### Real-World Sealed Class Example

**Expression Evaluator:**
```java
sealed interface Expr 
    permits Const, Add, Multiply {}

record Const(int value) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Multiply(Expr left, Expr right) implements Expr {}

int eval(Expr expr) {
    return switch (expr) {
        case Const(int value) -> value;
        case Add(Expr left, Expr right) -> eval(left) + eval(right);
        case Multiply(Expr left, Expr right) -> eval(left) * eval(right);
    };
}

void main() {
    // (5 + 3) * 2
    Expr expr = new Multiply(
        new Add(new Const(5), new Const(3)),
        new Const(2)
    );
    
    println("Result: " + eval(expr));  // 16
}
```

### Flexible Constructor Bodies (JEP 482)

**New in Java 23: Statements before `super()`**

**The Old Restriction:**
```java
class Parent {
    Parent(String value) {
        println("Parent: " + value);
    }
}

class Child extends Parent {
    Child(String input) {
        // ❌ Error: Can't do anything before super()
        // String processed = input.toUpperCase();
        super(input);  // Must be first statement
    }
}
```

**The New Flexibility (Java 23+):**
```java
class Parent {
    Parent(String value) {
        println("Parent: " + value);
    }
}

class Child extends Parent {
    Child(String input) {
        // ✓ Now allowed: statements before super()
        String processed = input.toUpperCase().trim();
        
        if (processed.isEmpty()) {
            processed = "DEFAULT";
        }
        
        println("Processing: " + processed);
        
        super(processed);  // Can call super() after statements
    }
}

void main() {
    new Child("  hello  ");
    // Output:
    // Processing: HELLO
    // Parent: HELLO
}
```

**Validation Before Construction:**
```java
record User(String name, int age) {
    User {
        // In records, you could always validate
        if (name == null) throw new IllegalArgumentException();
    }
}

class BankAccount {
    private final String accountNumber;
    private final double initialBalance;
    
    BankAccount(String number, double balance) {
        // Now you can validate before calling super()
        if (balance < 0) {
            throw new IllegalArgumentException("Balance cannot be negative");
        }
        
        // Process the data
        String formattedNumber = number.replaceAll("-", "");
        
        // Validate again
        if (formattedNumber.length() != 10) {
            throw new IllegalArgumentException("Invalid account number");
        }
        
        // Now initialize
        this.accountNumber = formattedNumber;
        this.initialBalance = balance;
    }
}
```

### Sealed Classes vs Final Classes

**Comparison:**

| Feature | Final Class | Sealed Class |
|---------|-------------|--------------|
| Can be extended? | No | Yes, but controlled |
| Specify subclasses? | N/A | Yes (permits) |
| Pattern matching | Not exhaustive | Exhaustive |
| Use case | Single implementation | Restricted hierarchy |

```java
// Final class - completely closed
final class ImmutableString {
    // No one can extend
}

// Sealed class - controlled hierarchy
sealed class Result permits Success, Error {
    // Only Success and Error can extend
}
```

---

## The Module System

### What Are Modules?

**Before Modules (Java 8 and earlier):**
- Everything on classpath
- No encapsulation at package level
- "JAR hell" - version conflicts
- Difficult to see dependencies

**With Modules (Java 9+):**
- Explicit dependencies
- Strong encapsulation
- Reliable configuration
- Scalable for large applications

### Basic Module Structure

**Project layout:**
```
myapp/
├── src/
│   └── module-info.java  ← Module descriptor
│   └── com/
│       └── myapp/
│           └── Main.java
```

**module-info.java:**
```java
module com.myapp {
    // This is the module descriptor
}
```

### Module Directives

**1. `requires` - Declare Dependencies:**
```java
module com.myapp {
    requires java.sql;      // Need SQL APIs
    requires java.logging;  // Need logging
}
```

**2. `exports` - Make Packages Available:**
```java
module com.mylib {
    exports com.mylib.api;        // Public API
    // com.mylib.internal not exported - private!
}
```

**3. `opens` - Allow Reflection:**
```java
module com.myapp {
    opens com.myapp.model;  // Allow frameworks to access via reflection
}
```

**4. `provides...with` - Service Provider:**
```java
module com.myapp.impl {
    provides com.myapp.api.Service 
        with com.myapp.impl.ServiceImpl;
}
```

**5. `uses` - Service Consumer:**
```java
module com.myapp {
    uses com.myapp.api.Service;
}
```

### Simple Module Example

**Module 1: Library**
```
mylib/
└── src/
    ├── module-info.java
    └── com/
        └── mylib/
            ├── api/
            │   └── Calculator.java
            └── internal/
                └── CalculatorImpl.java
```

**module-info.java:**
```java
module com.mylib {
    exports com.mylib.api;  // Public API
    // com.mylib.internal is hidden
}
```

**Calculator.java (public API):**
```java
package com.mylib.api;

public interface Calculator {
    int add(int a, int b);
    int subtract(int a, int b);
    
    static Calculator getInstance() {
        return new com.mylib.internal.CalculatorImpl();
    }
}
```

**CalculatorImpl.java (internal):**
```java
package com.mylib.internal;

import com.mylib.api.Calculator;

public class CalculatorImpl implements Calculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }
    
    @Override
    public int subtract(int a, int b) {
        return a - b;
    }
}
```

**Module 2: Application**
```
myapp/
└── src/
    ├── module-info.java
    └── com/
        └── myapp/
            └── Main.java
```

**module-info.java:**
```java
module com.myapp {
    requires com.mylib;  // Depend on library
}
```

**Main.java:**
```java
package com.myapp;

import com.mylib.api.Calculator;

public class Main {
    public static void main(String[] args) {
        Calculator calc = Calculator.getInstance();
        
        System.out.println("5 + 3 = " + calc.add(5, 3));
        System.out.println("10 - 4 = " + calc.subtract(10, 4));
        
        // new com.mylib.internal.CalculatorImpl();  
        // ❌ Error: package com.mylib.internal is not visible
    }
}
```

### Platform Modules

**Java platform is modularized:**
```java
module com.myapp {
    requires java.base;      // Automatic, always present
    requires java.sql;       // JDBC
    requires java.xml;       // XML processing
    requires java.logging;   // Logging
    requires java.desktop;   // Swing/AWT
    requires java.net.http;  // HTTP client
}
```

**Listing modules:**
```bash
java --list-modules
# Output:
# java.base@21
# java.compiler@21
# java.sql@21
# java.xml@21
# ...
```

### Module Benefits

**1. Strong Encapsulation:**
```java
module com.mylib {
    exports com.mylib.api;
    // com.mylib.internal is truly private - not accessible via reflection!
}
```

**2. Reliable Configuration:**
```java
module com.myapp {
    requires com.database;  // Missing? Error at startup, not runtime!
}
```

**3. Better Performance:**
- JVM knows exact dependencies
- Can optimize loading
- Smaller runtime footprint (jlink)

**4. Security:**
- Explicit boundaries
- Controlled access
- No accidental exposure

### Module Exports vs Opens

```java
module com.myapp {
    // exports: Compile-time access
    exports com.myapp.api;
    
    // opens: Runtime reflection access (for frameworks)
    opens com.myapp.model;
    
    // exports to specific module
    exports com.myapp.internal to com.myapp.test;
    
    // opens to specific module
    opens com.myapp.entity to org.hibernate.orm;
}
```

### Unnamed Module

**Code not in a module goes to "unnamed module":**
```java
// Traditional classpath code (no module-info.java)
// Automatically in unnamed module
// Can access all exported packages from named modules
```

### Real-World Module Example: Web Application

```
webapp/
├── api/
│   └── src/
│       ├── module-info.java
│       └── com/webapp/api/
├── core/
│   └── src/
│       ├── module-info.java
│       └── com/webapp/core/
└── web/
    └── src/
        ├── module-info.java
        └── com/webapp/web/
```

**api/module-info.java:**
```java
module com.webapp.api {
    // Just interfaces - no dependencies
    exports com.webapp.api.service;
    exports com.webapp.api.model;
}
```

**core/module-info.java:**
```java
module com.webapp.core {
    requires com.webapp.api;
    requires java.sql;
    
    exports com.webapp.core.service;
    // Implementation details hidden
}
```

**web/module-info.java:**
```java
module com.webapp.web {
    requires com.webapp.api;
    requires com.webapp.core;
    requires java.net.http;
    
    // Web layer implementation hidden
}
```

---

## Practice Exercises

### Exercise 1: Record Basics

Create a `Book` record with validation and custom methods.

**Requirements:**
- Fields: title, author, year, price
- Validation: year 1000-2100, price > 0
- Method: `isClassic()` (published before 1950)
- Method: `discountedPrice(double percentage)`

**Solution:**
```java
record Book(String title, String author, int year, double price) {
    // Compact constructor with validation
    Book {
        if (title == null || title.isBlank()) {
            throw new IllegalArgumentException("Title cannot be blank");
        }
        if (author == null || author.isBlank()) {
            throw new IllegalArgumentException("Author cannot be blank");
        }
        if (year < 1000 || year > 2100) {
            throw new IllegalArgumentException("Year must be 1000-2100");
        }
        if (price <= 0) {
            throw new IllegalArgumentException("Price must be positive");
        }
        
        title = title.trim();
        author = author.trim();
    }
    
    boolean isClassic() {
        return year < 1950;
    }
    
    double discountedPrice(double percentage) {
        if (percentage < 0 || percentage > 100) {
            throw new IllegalArgumentException("Invalid discount percentage");
        }
        return price * (1 - percentage / 100);
    }
    
    // Alternate constructor
    Book(String title, String author, int year) {
        this(title, author, year, 0.0);
    }
}

void main() {
    Book book1 = new Book("1984", "George Orwell", 1949, 15.99);
    Book book2 = new Book("Clean Code", "Robert Martin", 2008, 42.50);
    
    println(book1);
    println("Is classic? " + book1.isClassic());
    println("10% off: $%.2f".formatted(book1.discountedPrice(10)));
    
    println();
    
    println(book2);
    println("Is classic? " + book2.isClassic());
    println("20% off: $%.2f".formatted(book2.discountedPrice(20)));
}
```

### Exercise 2: Record Patterns

Create a system using record patterns to process different types of messages.

**Solution:**
```java
sealed interface Message permits TextMessage, ImageMessage, VideoMessage {}

record TextMessage(String sender, String content, long timestamp) 
    implements Message {}

record ImageMessage(String sender, String url, int width, int height) 
    implements Message {}

record VideoMessage(String sender, String url, int durationSeconds) 
    implements Message {}

void processMessage(Message message) {
    switch (message) {
        case TextMessage(String sender, String content, long timestamp) -> {
            println("Text from %s: %s".formatted(sender, content));
            println("Sent at: " + timestamp);
        }
        
        case ImageMessage(String sender, String url, int w, int h) -> {
            println("Image from %s".formatted(sender));
            println("Dimensions: %dx%d".formatted(w, h));
            println("URL: " + url);
        }
        
        case VideoMessage(String sender, String url, int duration) -> {
            println("Video from %s".formatted(sender));
            println("Duration: %d seconds".formatted(duration));
            println("URL: " + url);
        }
    }
}

void main() {
    List<Message> messages = List.of(
        new TextMessage("Alice", "Hello!", System.currentTimeMillis()),
        new ImageMessage("Bob", "http://example.com/pic.jpg", 1920, 1080),
        new VideoMessage("Charlie", "http://example.com/vid.mp4", 120)
    );
    
    for (Message msg : messages) {
        processMessage(msg);
        println("---");
    }
}
```

### Exercise 3: Inheritance Hierarchy

Create an employee management system with inheritance.

**Solution:**
```java
abstract class Employee {
    protected String name;
    protected String id;
    
    Employee(String name, String id) {
        this.name = name;
        this.id = id;
    }
    
    abstract double calculateSalary();
    
    void displayInfo() {
        println("ID: " + id);
        println("Name: " + name);
        println("Salary: $%.2f".formatted(calculateSalary()));
    }
}

class HourlyEmployee extends Employee {
    private double hourlyRate;
    private int hoursWorked;
    
    HourlyEmployee(String name, String id, double hourlyRate, int hoursWorked) {
        super(name, id);
        this.hourlyRate = hourlyRate;
        this.hoursWorked = hoursWorked;
    }
    
    @Override
    double calculateSalary() {
        double regularPay = Math.min(hoursWorked, 40) * hourlyRate;
        double overtimePay = Math.max(0, hoursWorked - 40) * hourlyRate * 1.5;
        return regularPay + overtimePay;
    }
    
    @Override
    void displayInfo() {
        super.displayInfo();
        println("Type: Hourly");
        println("Rate: $%.2f/hour".formatted(hourlyRate));
        println("Hours: " + hoursWorked);
    }
}

class SalariedEmployee extends Employee {
    private double annualSalary;
    
    SalariedEmployee(String name, String id, double annualSalary) {
        super(name, id);
        this.annualSalary = annualSalary;
    }
    
    @Override
    double calculateSalary() {
        return annualSalary / 12;  // Monthly salary
    }
    
    @Override
    void displayInfo() {
        super.displayInfo();
        println("Type: Salaried");
        println("Annual: $%.2f".formatted(annualSalary));
    }
}

class CommissionEmployee extends Employee {
    private double baseSalary;
    private double commissionRate;
    private double salesAmount;
    
    CommissionEmployee(String name, String id, double baseSalary, 
                       double commissionRate, double salesAmount) {
        super(name, id);
        this.baseSalary = baseSalary;
        this.commissionRate = commissionRate;
        this.salesAmount = salesAmount;
    }
    
    @Override
    double calculateSalary() {
        return baseSalary + (salesAmount * commissionRate);
    }
    
    @Override
    void displayInfo() {
        super.displayInfo();
        println("Type: Commission");
        println("Base: $%.2f".formatted(baseSalary));
        println("Sales: $%.2f".formatted(salesAmount));
        println("Commission: %.1f%%".formatted(commissionRate * 100));
    }
}

void main() {
    List<Employee> employees = List.of(
        new HourlyEmployee("John", "E001", 25.0, 45),
        new SalariedEmployee("Jane", "E002", 60000),
        new CommissionEmployee("Bob", "E003", 2000, 0.05, 50000)
    );
    
    for (Employee emp : employees) {
        emp.displayInfo();
        println("===================");
    }
    
    // Calculate total payroll
    double total = employees.stream()
        .mapToDouble(Employee::calculateSalary)
        .sum();
    
    println("Total Monthly Payroll: $%.2f".formatted(total));
}
```

### Exercise 4: Sealed Class with Pattern Matching

Create a calculator using sealed classes and exhaustive pattern matching.

**Solution:**
```java
sealed interface Expr permits Num, Add, Sub, Mul, Div {}

record Num(double value) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Sub(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}
record Div(Expr left, Expr right) implements Expr {}

double evaluate(Expr expr) {
    return switch (expr) {
        case Num(double value) -> value;
        case Add(Expr left, Expr right) -> evaluate(left) + evaluate(right);
        case Sub(Expr left, Expr right) -> evaluate(left) - evaluate(right);
        case Mul(Expr left, Expr right) -> evaluate(left) * evaluate(right);
        case Div(Expr left, Expr right) -> {
            double divisor = evaluate(right);
            if (divisor == 0) {
                throw new ArithmeticException("Division by zero");
            }
            yield evaluate(left) / divisor;
        }
    };
}

String prettyPrint(Expr expr) {
    return switch (expr) {
        case Num(double value) -> String.valueOf(value);
        case Add(Expr left, Expr right) -> 
            "(%s + %s)".formatted(prettyPrint(left), prettyPrint(right));
        case Sub(Expr left, Expr right) -> 
            "(%s - %s)".formatted(prettyPrint(left), prettyPrint(right));
        case Mul(Expr left, Expr right) -> 
            "(%s * %s)".formatted(prettyPrint(left), prettyPrint(right));
        case Div(Expr left, Expr right) -> 
            "(%s / %s)".formatted(prettyPrint(left), prettyPrint(right));
    };
}

void main() {
    // ((10 + 5) * 2) - (20 / 4)
    Expr expr = new Sub(
        new Mul(
            new Add(new Num(10), new Num(5)),
            new Num(2)
        ),
        new Div(new Num(20), new Num(4))
    );
    
    println("Expression: " + prettyPrint(expr));
    println("Result: " + evaluate(expr));
}
```

### Exercise 5: Complex Record with Nested Records

Create an order management system using records.

**Solution:**
```java
record Address(String street, String city, String state, String zip) {
    Address {
        if (street == null || city == null || state == null || zip == null) {
            throw new IllegalArgumentException("All address fields required");
        }
    }
    
    @Override
    public String toString() {
        return "%s, %s, %s %s".formatted(street, city, state, zip);
    }
}

record Customer(String name, String email, Address address) {
    Customer {
        if (name == null || email == null || address == null) {
            throw new IllegalArgumentException("All customer fields required");
        }
    }
}

record Product(String id, String name, double price) {
    Product {
        if (price <= 0) {
            throw new IllegalArgumentException("Price must be positive");
        }
    }
}

record OrderItem(Product product, int quantity) {
    OrderItem {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
    }
    
    double totalPrice() {
        return product.price() * quantity;
    }
}

record Order(String orderId, Customer customer, List<OrderItem> items) {
    Order {
        if (orderId == null || customer == null || items == null || items.isEmpty()) {
            throw new IllegalArgumentException("Invalid order");
        }
        items = List.copyOf(items);  // Make truly immutable
    }
    
    double subtotal() {
        return items.stream()
            .mapToDouble(OrderItem::totalPrice)
            .sum();
    }
    
    double tax() {
        return subtotal() * 0.08;
    }
    
    double total() {
        return subtotal() + tax();
    }
    
    void printReceipt() {
        println("=".repeat(50));
        println("ORDER #" + orderId);
        println("=".repeat(50));
        println();
        
        println("Customer: " + customer.name());
        println("Email: " + customer.email());
        println("Address: " + customer.address());
        println();
        
        println("Items:");
        println("-".repeat(50));
        for (OrderItem item : items) {
            println("%-30s %2d x $%6.2f = $%8.2f".formatted(
                item.product().name(),
                item.quantity(),
                item.product().price(),
                item.totalPrice()
            ));
        }
        
        println("-".repeat(50));
        println("Subtotal: %35s $%8.2f".formatted("", subtotal()));
        println("Tax (8%%): %35s $%8.2f".formatted("", tax()));
        println("=".repeat(50));
        println("TOTAL: %38s $%8.2f".formatted("", total()));
        println("=".repeat(50));
    }
}

void main() {
    Address addr = new Address(
        "123 Main St",
        "Springfield",
        "IL",
        "62701"
    );
    
    Customer customer = new Customer(
        "John Doe",
        "john@example.com",
        addr
    );
    
    List<OrderItem> items = List.of(
        new OrderItem(new Product("P001", "Laptop", 999.99), 1),
        new OrderItem(new Product("P002", "Mouse", 25.50), 2),
        new OrderItem(new Product("P003", "Keyboard", 75.00), 1)
    );
    
    Order order = new Order("ORD-2024-001", customer, items);
    order.printReceipt();
}
```

### Exercise 6: Combining All Concepts

Create a library management system using records, sealed classes, and inheritance.

**Solution:**
```java
// Sealed interface for library items
sealed interface LibraryItem permits Book, Magazine, DVD {}

record Book(String isbn, String title, String author, int year) 
    implements LibraryItem {
    Book {
        if (isbn == null || title == null || author == null) {
            throw new IllegalArgumentException("Required fields missing");
        }
    }
}

record Magazine(String issn, String title, int issue, String month) 
    implements LibraryItem {}

record DVD(String id, String title, String director, int runtime) 
    implements LibraryItem {}

// Member hierarchy
abstract class Member {
    protected String memberId;
    protected String name;
    protected List<LibraryItem> borrowedItems;
    
    Member(String memberId, String name) {
        this.memberId = memberId;
        this.name = name;
        this.borrowedItems = new ArrayList<>();
    }
    
    abstract int maxBorrowLimit();
    
    boolean canBorrow() {
        return borrowedItems.size() < maxBorrowLimit();
    }
    
    void borrowItem(LibraryItem item) {
        if (!canBorrow()) {
            throw new IllegalStateException("Borrow limit reached");
        }
        borrowedItems.add(item);
    }
    
    void returnItem(LibraryItem item) {
        borrowedItems.remove(item);
    }
    
    void displayInfo() {
        println("Member ID: " + memberId);
        println("Name: " + name);
        println("Borrowed: " + borrowedItems.size() + "/" + maxBorrowLimit());
    }
}

class RegularMember extends Member {
    RegularMember(String id, String name) {
        super(id, name);
    }
    
    @Override
    int maxBorrowLimit() {
        return 3;
    }
}

class PremiumMember extends Member {
    PremiumMember(String id, String name) {
        super(id, name);
    }
    
    @Override
    int maxBorrowLimit() {
        return 10;
    }
}

void main() {
    // Create library items
    LibraryItem book = new Book(
        "978-0-13-468599-1",
        "Effective Java",
        "Joshua Bloch",
        2018
    );
    
    LibraryItem magazine = new Magazine(
        "1234-5678",
        "Java Magazine",
        42,
        "January"
    );
    
    LibraryItem dvd = new DVD(
        "DVD-001",
        "Java Tutorial",
        "John Doe",
        120
    );
    
    // Create members
    Member regular = new RegularMember("M001", "Alice");
    Member premium = new PremiumMember("M002", "Bob");
    
    // Borrow items
    regular.borrowItem(book);
    premium.borrowItem(magazine);
    premium.borrowItem(dvd);
    
    // Display info
    regular.displayInfo();
    println();
    premium.displayInfo();
    
    // Pattern matching on items
    println("\nItem Details:");
    for (LibraryItem item : List.of(book, magazine, dvd)) {
        String details = switch (item) {
            case Book(String isbn, String title, String author, int year) ->
                "Book: '%s' by %s (%d)".formatted(title, author, year);
            case Magazine(String issn, String title, int issue, String month) ->
                "Magazine: '%s' Issue #%d (%s)".formatted(title, issue, month);
            case DVD(String id, String title, String director, int runtime) ->
                "DVD: '%s' directed by %s (%d min)".formatted(title, director, runtime);
        };
        println(details);
    }
}
```

---

## Interview Questions & Answers

### Encapsulation Questions

**Q1: What is encapsulation and why is it important?**

**Answer:**
Encapsulation is the bundling of data (fields) and methods that operate on that data within a single unit (class), while restricting direct access to some components.

**Benefits:**
1. **Data Hiding**: Internal state is hidden from outside
2. **Controlled Access**: Access through methods allows validation
3. **Flexibility**: Can change internal implementation without affecting users
4. **Security**: Prevents unauthorized access
5. **Maintainability**: Changes localized to one place

**Example:**
```java
class BankAccount {
    private double balance;  // Hidden
    
    public void deposit(double amount) {  // Controlled access
        if (amount > 0) {  // Validation
            balance += amount;
        }
    }
    
    public double getBalance() {  // Read-only access
        return balance;
    }
}
```

---

**Q2: What are access modifiers and when should you use each?**

**Answer:**

| Modifier | Visibility | Use Case |
|----------|------------|----------|
| `private` | Same class only | Internal implementation details |
| default | Same package | Package-private utilities |
| `protected` | Package + subclasses | Extension points for inheritance |
| `public` | Everywhere | Public API |

**Guidelines:**
- **Default to `private`**: Expose only what's necessary
- **Use `public` for API**: Methods and classes clients need
- **Use `protected` sparingly**: Only when inheritance requires it
- **Avoid default**: Explicit is better (use `private` or `public`)

```java
class Example {
    private int data;              // Most restrictive
    protected void helperMethod()  // For subclasses
    public void apiMethod()        // Public interface
}
```

---

**Q3: Explain the difference between records and regular classes.**

**Answer:**

| Feature | Record | Class |
|---------|--------|-------|
| Mutability | Immutable (final fields) | Can be mutable |
| Boilerplate | Auto-generated | Must write manually |
| Inheritance | Cannot extend classes | Can extend classes |
| Purpose | Data carriers | General purpose |
| Constructor | Compact constructor | Regular constructor |
| Methods | Auto: getters, equals, hashCode, toString | Must implement |

**When to use Record:**
- Immutable data (DTOs, value objects)
- Simple data carriers
- Configuration objects

**When to use Class:**
- Mutable state needed
- Complex behavior
- Need inheritance
- Encapsulation of complex logic

```java
// Record - concise, immutable
record Point(int x, int y) {}

// Class - flexible, mutable
class Counter {
    private int count = 0;
    void increment() { count++; }
}
```

---

**Q4: What is a compact constructor and how does it work?**

**Answer:**
A compact constructor is a concise way to write constructors in records without explicitly listing parameters.

**Traditional Constructor:**
```java
record User(String name, int age) {
    public User(String name, int age) {
        if (name == null) throw new IllegalArgumentException();
        this.name = name;
        this.age = age;
    }
}
```

**Compact Constructor:**
```java
record User(String name, int age) {
    User {  // No parameter list!
        if (name == null) throw new IllegalArgumentException();
        // Fields assigned automatically
    }
}
```

**How it works:**
1. Parameters are implicit (from record header)
2. Can validate/transform data
3. Field assignment happens automatically after the body
4. Cannot assign fields manually (unless using canonical constructor)

**Common uses:**
- Validation
- Normalization (trim strings, etc.)
- Defensive copying

---

**Q5: How do you handle validation in records?**

**Answer:**
Use compact constructors for validation:

```java
record Email(String address) {
    Email {
        if (address == null || !address.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
        address = address.toLowerCase().trim();
    }
}

record Range(int min, int max) {
    Range {
        if (min > max) {
            throw new IllegalArgumentException("min must be <= max");
        }
    }
    
    boolean contains(int value) {
        return value >= min && value <= max;
    }
}

void main() {
    Email e = new Email("  USER@EXAMPLE.COM  ");
    println(e.address());  // "user@example.com"
    
    Range r = new Range(1, 10);
    println(r.contains(5));  // true
}
```

---

### Inheritance Questions

**Q6: What is inheritance and what problems does it solve?**

**Answer:**
Inheritance is a mechanism where a class (child) derives properties and behaviors from another class (parent).

**Problems it solves:**
1. **Code Reuse**: Don't repeat common code
2. **Extensibility**: Easily add new types
3. **Polymorphism**: Treat different types uniformly
4. **Hierarchy**: Model real-world relationships

**Example:**
```java
class Animal {
    void eat() { println("Eating"); }
    void sleep() { println("Sleeping"); }
}

class Dog extends Animal {
    void bark() { println("Barking"); }  // Additional behavior
}

void main() {
    Dog dog = new Dog();
    dog.eat();    // Inherited
    dog.sleep();  // Inherited
    dog.bark();   // New behavior
}
```

---

**Q7: Explain method overriding vs method overloading.**

**Answer:**

**Overriding** (Runtime Polymorphism):
- Same signature in subclass
- Happens in inheritance
- `@Override` annotation
- Decided at runtime

```java
class Parent {
    void display() { println("Parent"); }
}

class Child extends Parent {
    @Override
    void display() { println("Child"); }
}

void main() {
    Parent obj = new Child();
    obj.display();  // "Child" - runtime decision
}
```

**Overloading** (Compile-time Polymorphism):
- Same name, different parameters
- Can be in same class
- Decided at compile time

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}

void main() {
    Calculator calc = new Calculator();
    calc.add(5, 3);        // First method
    calc.add(5.5, 3.2);    // Second method
    calc.add(1, 2, 3);     // Third method
}
```

---

**Q8: What are the rules for method overriding?**

**Answer:**

**Must Follow:**
1. **Same signature**: Name + parameters
2. **Same or covariant return type**: Can return subtype
3. **Same or more accessible**: Can't reduce visibility
4. **Cannot override `final` methods**
5. **Cannot override `static` methods** (hiding, not overriding)
6. **Can't throw broader checked exceptions**: Can throw narrower or unchecked

```java
class Parent {
    protected Object getValue() { return new Object(); }
    final void finalMethod() {}
    static void staticMethod() {}
}

class Child extends Parent {
    @Override
    public String getValue() {  // ✓ Covariant return, more accessible
        return "value";
    }
    
    // void finalMethod() {}  // ❌ Error: cannot override final
    // void staticMethod() {}  // ⚠️ This is hiding, not overriding
}
```

---

**Q9: Explain the `super` keyword and its uses.**

**Answer:**

The `super` keyword refers to the parent class and has three uses:

**1. Call parent constructor:**
```java
class Parent {
    Parent(String name) {
        println("Parent: " + name);
    }
}

class Child extends Parent {
    Child() {
        super("Child");  // Must be first statement
    }
}
```

**2. Access parent fields:**
```java
class Parent {
    protected int value = 10;
}

class Child extends Parent {
    private int value = 20;
    
    void display() {
        println(value);        // 20 (child's)
        println(super.value);  // 10 (parent's)
    }
}
```

**3. Call parent methods:**
```java
class Parent {
    void display() {
        println("Parent display");
    }
}

class Child extends Parent {
    @Override
    void display() {
        super.display();  // Call parent version
        println("Child display");
    }
}
```

---

**Q10: What is constructor chaining and how does it work?**

**Answer:**
Constructor chaining is the process where constructors call other constructors in a chain.

**Implicit Chaining:**
```java
class GrandParent {
    GrandParent() {
        println("1. GrandParent");
    }
}

class Parent extends GrandParent {
    Parent() {
        // super() is implicit
        println("2. Parent");
    }
}

class Child extends Parent {
    Child() {
        // super() is implicit
        println("3. Child");
    }
}

void main() {
    new Child();
    // Output: 1, 2, 3
}
```

**Explicit Chaining:**
```java
class Employee {
    private String name;
    private double salary;
    
    Employee(String name) {
        this(name, 50000);  // Chain to other constructor
    }
    
    Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
}
```

**Rules:**
- Parent constructor always called first (implicitly or explicitly)
- `super()` or `this()` must be first statement
- Can't call both `super()` and `this()` in same constructor

---

### Sealed Classes Questions

**Q11: What are sealed classes and why were they introduced?**

**Answer:**
Sealed classes restrict which classes can extend them, introduced in Java 17.

**Problem they solve:**
- Uncontrolled inheritance
- Can't predict all subclasses
- Incomplete pattern matching
- API stability

**Example:**
```java
// Before: Anyone can extend
class Shape {}
class UnexpectedShape extends Shape {}  // Oops!

// After: Controlled hierarchy
sealed class Shape permits Circle, Rectangle {}
final class Circle extends Shape {}
final class Rectangle extends Shape {}
// class Triangle extends Shape {}  // ❌ Error!
```

**Benefits:**
1. **Exhaustive pattern matching**: Compiler knows all types
2. **Better maintainability**: Clear hierarchy
3. **API control**: Prevent unexpected extensions
4. **Security**: Prevent malicious subclasses

---

**Q12: What are the requirements for sealed class subtypes?**

**Answer:**
Permitted subtypes must be one of:

**1. `final` - Cannot be extended further:**
```java
sealed class Shape permits Circle {}
final class Circle extends Shape {}
```

**2. `sealed` - Can be extended, but also controlled:**
```java
sealed class Vehicle permits Car, Truck {}
sealed class Truck permits PickupTruck, SemiTruck {}
final class PickupTruck extends Truck {}
final class SemiTruck extends Truck {}
```

**3. `non-sealed` - Opens inheritance again:**
```java
sealed class Animal permits Mammal {}
non-sealed class Mammal extends Animal {}
class Dog extends Mammal {}  // Anyone can extend Mammal
```

**Additional Requirements:**
- Permitted classes must be accessible to sealed class
- Permitted classes must directly extend sealed class
- Must be in same module (or same package if unnamed module)

---

**Q13: How do sealed classes enable exhaustive pattern matching?**

**Answer:**
Sealed classes allow the compiler to verify all cases are handled:

```java
sealed interface Result permits Success, Error {}
record Success(String data) implements Result {}
record Error(String message) implements Result {}

void main() {
    Result result = fetchData();
    
    // Compiler knows all possible types
    String output = switch (result) {
        case Success(String data) -> "Success: " + data;
        case Error(String msg) -> "Error: " + msg;
        // No default needed - compiler verifies completeness!
    };
    
    println(output);
}

Result fetchData() {
    return new Success("data");
}
```

**Without sealed classes:**
```java
interface Result {}  // Not sealed
record Success(String data) implements Result {}
record Error(String message) implements Result {}
record Unknown() implements Result {}  // Can add anytime!

String process(Result r) {
    return switch (r) {
        case Success s -> "success";
        case Error e -> "error";
        // Must have default - can't know all types!
        default -> "unknown";
    };
}
```

---

**Q14: Compare final classes vs sealed classes.**

**Answer:**

| Feature | Final Class | Sealed Class |
|---------|-------------|--------------|
| Can be extended? | No | Yes (controlled) |
| Specify subclasses? | N/A | Yes (permits) |
| Hierarchy | No hierarchy | Controlled hierarchy |
| Pattern matching | Not exhaustive | Exhaustive |

**Final - Completely closed:**
```java
final class String {
    // No one can extend String
}
```

**Sealed - Controlled hierarchy:**
```java
sealed class Payment permits Cash, Card {}
final class Cash extends Payment {}
final class Card extends Payment {}
```

**Use final when:**
- Single implementation
- Security critical (String, Integer, etc.)
- No inheritance needed

**Use sealed when:**
- Known, limited set of subtypes
- Want exhaustive pattern matching
- Domain modeling with variants

---

**Q15: Explain JEP 482 (Flexible Constructor Bodies).**

**Answer:**
JEP 482 allows statements before `super()` call in constructors (Java 23+).

**Before (Java 22 and earlier):**
```java
class Child extends Parent {
    Child(String input) {
        // ❌ Cannot do this
        // String processed = input.toUpperCase();
        
        super(input);  // Must be first
    }
}
```

**After (Java 23+):**
```java
class Child extends Parent {
    Child(String input) {
        // ✓ Can process data first
        String processed = input.toUpperCase().trim();
        
        if (processed.isEmpty()) {
            processed = "DEFAULT";
        }
        
        super(processed);  // Now call parent
    }
}
```

**Benefits:**
- Validate/transform data before parent construction
- More flexible initialization
- Cleaner code

**Restrictions:**
- Cannot access `this` before `super()`
- Cannot use instance fields
- Only local variables and static methods

---

### Module System Questions

**Q16: What problems does the Java module system solve?**

**Answer:**

**Problems Before Modules (Java 8):**
1. **Weak Encapsulation**: Internal APIs accessible via reflection
2. **Classpath Hell**: No clear dependencies, conflicts
3. **Monolithic JRE**: All or nothing
4. **No Explicit Dependencies**: Runtime discovery only

**Solutions with Modules (Java 9+):**
1. **Strong Encapsulation**: Packages truly private
2. **Explicit Dependencies**: Declared in `module-info.java`
3. **Modular JDK**: Use only what you need
4. **Reliable Configuration**: Missing dependencies found at startup

```java
// module-info.java
module com.myapp {
    requires java.sql;      // Explicit dependency
    exports com.myapp.api;  // Public API only
    // com.myapp.internal hidden
}
```

---

**Q17: Explain the structure of `module-info.java`.**

**Answer:**

```java
module com.example.app {
    // Dependencies
    requires java.sql;           // Need this module
    requires java.logging;
    requires transitive java.xml;  // Callers get java.xml too
    
    // Exports
    exports com.example.api;     // Public API
    exports com.example.util to  // Only to specific modules
        com.example.test,
        com.example.debug;
    
    // Reflection
    opens com.example.model;     // Allow reflection access
    opens com.example.entity to  // Only to specific frameworks
        org.hibernate.orm;
    
    // Services
    uses com.example.spi.Service;     // Service consumer
    provides com.example.spi.Service  // Service provider
        with com.example.impl.ServiceImpl;
}
```

**Directives:**
- **requires**: This module depends on another
- **exports**: Make package visible to other modules
- **opens**: Allow reflection access
- **uses**: Consume a service
- **provides...with**: Provide a service implementation

---

**Q18: What's the difference between `exports` and `opens`?**

**Answer:**

**`exports` - Compile-time access:**
```java
module com.library {
    exports com.library.api;  // Can import and use
}

// Other modules can:
import com.library.api.SomeClass;
SomeClass obj = new SomeClass();  // ✓ Works
```

**`opens` - Runtime reflection access:**
```java
module com.app {
    opens com.app.model;  // Frameworks can access via reflection
}

// Frameworks can use reflection:
Class<?> clazz = Class.forName("com.app.model.User");
Object instance = clazz.getDeclaredConstructor().newInstance();  // ✓ Works
```

**Comparison:**

| Feature | exports | opens |
|---------|---------|-------|
| Compile-time access | Yes | No |
| Runtime reflection | No | Yes |
| Use case | Public API | Framework access |

**Common pattern:**
```java
module com.webapp {
    exports com.webapp.api;     // Public API for users
    opens com.webapp.entity to  // JPA entities for Hibernate
        org.hibernate.orm;
}
```

---

**Q19: What is the unnamed module?**

**Answer:**
Code without `module-info.java` goes into the "unnamed module".

**Characteristics:**
- All classpath code
- Can read all exported packages from named modules
- Cannot be read by named modules
- All packages automatically exported (to other unnamed module code)

**Migration path:**
```java
// Step 1: Traditional app (unnamed module)
// - All .jar files on classpath
// - No module-info.java

// Step 2: Start using modules
// - Some libraries have modules
// - Your code still on classpath (unnamed module)
// - Can use modular libraries

// Step 3: Full modularization
// - Add module-info.java to your code
// - Now a named module
// - Explicit dependencies
```

**Example:**
```java
// Your code (classpath - unnamed module)
import java.sql.*;  // Can use java.sql (named module)

public class Main {
    public static void main(String[] args) {
        // Works fine
    }
}
```

---

**Q20: Design a module hierarchy for a real application.**

**Answer: E-commerce Application**

```
ecommerce/
├── api/
│   └── module-info.java
├── core/
│   └── module-info.java
├── persistence/
│   └── module-info.java
└── web/
    └── module-info.java
```

**api/module-info.java:**
```java
module com.shop.api {
    // Just interfaces - no dependencies
    exports com.shop.api.product;
    exports com.shop.api.order;
    exports com.shop.api.user;
}
```

**core/module-info.java:**
```java
module com.shop.core {
    requires com.shop.api;
    requires java.logging;
    
    exports com.shop.core.service;  // Service implementations
    // com.shop.core.internal hidden
}
```

**persistence/module-info.java:**
```java
module com.shop.persistence {
    requires com.shop.api;
    requires java.sql;
    requires transitive com.shop.core;  // Callers get core too
    
    opens com.shop.persistence.entity to  // JPA entities
        org.hibernate.orm;
    
    exports com.shop.persistence.repository;
}
```

**web/module-info.java:**
```java
module com.shop.web {
    requires com.shop.api;
    requires com.shop.core;
    requires com.shop.persistence;
    requires java.net.http;
    
    // Web layer doesn't export anything
}
```

**Dependency Flow:**
```
web → persistence → core → api
                     ↓
                  java.sql
```

---

## Week 2 Summary

### Key Concepts Mastered

**1. Encapsulation**
- Access modifiers (private, protected, public)
- Traditional classes with getters/setters
- Records for immutable data
- Compact constructors
- Record patterns and deconstruction

**2. Inheritance**
- Extends keyword
- Method overriding vs overloading
- The `super` keyword
- Constructor chaining
- Abstract classes

**3. Sealed Classes**
- Controlled inheritance hierarchy
- `permits` clause
- final, sealed, non-sealed subtypes
- Exhaustive pattern matching
- JEP 482: Flexible constructor bodies

**4. Module System**
- `module-info.java` structure
- requires, exports, opens
- Strong encapsulation
- Explicit dependencies
- Unnamed module

### Advanced Patterns

**Combining Records and Sealed Classes:**
```java
sealed interface PaymentMethod 
    permits CreditCard, DebitCard, Cash {}

record CreditCard(String number, String cvv) 
    implements PaymentMethod {}
    
record DebitCard(String number, String pin) 
    implements PaymentMethod {}
    
record Cash(double amount) 
    implements PaymentMethod {}
```

**Nested Record Patterns:**
```java
record Address(String city, String state) {}
record Person(String name, Address address) {}

if (obj instanceof Person(var name, Address(var city, var state))) {
    // All components extracted!
}
```

### Next Steps

**Week 3 Preview:**
- Polymorphism (runtime and compile-time)
- Pattern Matching for `switch` and `instanceof`
- Abstraction with interfaces
- Generics fundamentals

**Recommended Practice:**
1. Build a complete CRUD application using records
2. Create a sealed class hierarchy for a domain model
3. Experiment with record patterns in switch expressions
4. Design a simple module structure
5. Implement inheritance hierarchies with proper encapsulation

---

**Congratulations on completing Week 2! You now have strong OOP fundamentals and understand modern Java's approach to data modeling and class hierarchies.**
