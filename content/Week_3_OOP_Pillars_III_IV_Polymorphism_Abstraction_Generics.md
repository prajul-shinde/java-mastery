# Week 3: OOP Pillars III & IV - Polymorphism, Abstraction & Generics

## 🎯 Learning Objectives
By the end of this week, you will:
- Master polymorphism from fundamentals to modern pattern matching
- Understand compile-time vs runtime polymorphism deeply
- Differentiate between interfaces and abstract classes architecturally
- Write type-safe generic code with confidence
- Apply modern Java features (Pattern Matching, Sealed Classes, Records)
- Crack technical interviews with strong theoretical and practical knowledge

---

# Part 1: Polymorphism - The Art of Many Forms

## 1.1 Understanding Polymorphism Fundamentals

### What is Polymorphism?
**Poly (many) + Morphism (forms)** = Ability of an object to take many forms.

In Java, polymorphism means:
- One interface, multiple implementations
- Parent reference, child object
- Same method name, different behaviors

### Real-World Analogy
Think of a **universal remote control**:
- Same button (play/pause)
- Different devices (TV, DVD player, Music system)
- Different behaviors based on which device you're controlling

---

## 1.2 Types of Polymorphism

### Type 1: Compile-Time Polymorphism (Static Binding)
Also called **Method Overloading** or **Static Polymorphism**

**Definition**: Multiple methods with the same name but different parameters in the same class.

**Resolved**: At compile time by the compiler

```java
/**
 * Example: Calculator with method overloading
 */
public class Calculator {
    
    // Method 1: Add two integers
    public int add(int a, int b) {
        System.out.println("Adding two integers");
        return a + b;
    }
    
    // Method 2: Add three integers
    public int add(int a, int b, int c) {
        System.out.println("Adding three integers");
        return a + b + c;
    }
    
    // Method 3: Add two doubles
    public double add(double a, double b) {
        System.out.println("Adding two doubles");
        return a + b;
    }
    
    // Method 4: Add two strings (concatenation)
    public String add(String a, String b) {
        System.out.println("Concatenating strings");
        return a + b;
    }
}
```

**How the Compiler Decides Which Method to Call:**

```java
public class OverloadingDemo {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        
        // Compiler looks at argument types and count
        calc.add(5, 10);           // Calls add(int, int)
        calc.add(5, 10, 15);       // Calls add(int, int, int)
        calc.add(5.5, 10.5);       // Calls add(double, double)
        calc.add("Hello", "World"); // Calls add(String, String)
        
        // Type promotion example
        calc.add(5, 10.5);         // int promoted to double -> add(double, double)
    }
}
```

**Overloading Rules:**
1. ✅ Different parameter count
2. ✅ Different parameter types
3. ✅ Different parameter order (if types differ)
4. ❌ Return type alone doesn't matter
5. ❌ Access modifier doesn't matter

```java
// ❌ COMPILATION ERROR - Can't overload by return type only
public class InvalidOverloading {
    public int getValue() { return 10; }
    public double getValue() { return 10.5; } // ERROR!
}

// ✅ VALID - Different parameter order
public class ValidOverloading {
    public void process(String text, int count) { }
    public void process(int count, String text) { } // Different order
}
```

---

### Type 2: Runtime Polymorphism (Dynamic Binding)
Also called **Method Overriding** or **Dynamic Polymorphism**

**Definition**: Subclass provides specific implementation of a method already defined in its parent class.

**Resolved**: At runtime by JVM based on the actual object type

```java
/**
 * Example: Payment processing system
 */
class Payment {
    protected double amount;
    
    public Payment(double amount) {
        this.amount = amount;
    }
    
    // Method to be overridden
    public void processPayment() {
        System.out.println("Processing generic payment: $" + amount);
    }
    
    public void printReceipt() {
        System.out.println("Receipt for payment: $" + amount);
    }
}

class CreditCardPayment extends Payment {
    private String cardNumber;
    
    public CreditCardPayment(double amount, String cardNumber) {
        super(amount);
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void processPayment() {
        System.out.println("Processing credit card payment: $" + amount);
        System.out.println("Card ending in: " + cardNumber.substring(cardNumber.length() - 4));
        // Additional credit card specific logic
        validateCard();
        authorizePayment();
    }
    
    private void validateCard() {
        System.out.println("Validating card...");
    }
    
    private void authorizePayment() {
        System.out.println("Authorizing payment...");
    }
}

class PayPalPayment extends Payment {
    private String email;
    
    public PayPalPayment(double amount, String email) {
        super(amount);
        this.email = email;
    }
    
    @Override
    public void processPayment() {
        System.out.println("Processing PayPal payment: $" + amount);
        System.out.println("PayPal account: " + email);
        redirectToPayPal();
    }
    
    private void redirectToPayPal() {
        System.out.println("Redirecting to PayPal...");
    }
}

class CryptoPayment extends Payment {
    private String walletAddress;
    
    public CryptoPayment(double amount, String walletAddress) {
        super(amount);
        this.walletAddress = walletAddress;
    }
    
    @Override
    public void processPayment() {
        System.out.println("Processing crypto payment: $" + amount);
        System.out.println("Wallet: " + walletAddress);
        initiateBlockchainTransaction();
    }
    
    private void initiateBlockchainTransaction() {
        System.out.println("Initiating blockchain transaction...");
    }
}
```

**Runtime Polymorphism in Action:**

```java
public class PaymentProcessor {
    public static void main(String[] args) {
        // Parent reference, child objects - THE ESSENCE OF POLYMORPHISM
        Payment payment1 = new CreditCardPayment(100.00, "1234-5678-9012-3456");
        Payment payment2 = new PayPalPayment(200.00, "user@email.com");
        Payment payment3 = new CryptoPayment(300.00, "0x742d35Cc6634C0532925a3b8");
        
        // Array of different payment types
        Payment[] payments = {payment1, payment2, payment3};
        
        // Process all payments - Each behaves differently at RUNTIME
        for (Payment payment : payments) {
            payment.processPayment();  // JVM decides which method to call
            payment.printReceipt();     // Uses parent implementation
            System.out.println("---");
        }
    }
}
```

**Output:**
```
Processing credit card payment: $100.0
Card ending in: 3456
Validating card...
Authorizing payment...
Receipt for payment: $100.0
---
Processing PayPal payment: $200.0
PayPal account: user@email.com
Redirecting to PayPal...
Receipt for payment: $200.0
---
Processing crypto payment: $300.0
Wallet: 0x742d35Cc6634C0532925a3b8
Initiating blockchain transaction...
Receipt for payment: $300.0
---
```

---

## 1.3 How JVM Resolves Runtime Polymorphism

### Virtual Method Table (VTable)
Every class has a **VTable** that contains pointers to its methods.

```java
class Animal {
    public void sound() { }      // Entry 0 in Animal's VTable
    public void sleep() { }      // Entry 1 in Animal's VTable
}

class Dog extends Animal {
    @Override
    public void sound() { }      // Entry 0 points to Dog's sound()
    // sleep() - Entry 1 inherited from Animal
    public void bark() { }       // Entry 2 - new method
}
```

**At Runtime:**
1. JVM looks at the **actual object type** (not reference type)
2. Checks the object's VTable
3. Invokes the method from the VTable

```java
Animal a = new Dog();  // Reference: Animal, Object: Dog
a.sound();             // JVM checks Dog's VTable -> calls Dog.sound()
```

---

## 1.4 Method Overriding Rules

### The @Override Annotation
**Always use @Override** - it helps catch errors at compile time

```java
class Parent {
    public void display(String message) {
        System.out.println("Parent: " + message);
    }
}

class Child extends Parent {
    @Override
    public void display(String message) {  // Correct override
        System.out.println("Child: " + message);
    }
    
    // ❌ This would be a compile error with @Override
    // @Override
    // public void display(int number) { } // Different parameter - NOT override
}
```

### Override Rules:
1. **Method signature must match exactly** (name + parameters)
2. **Return type** must be same or covariant (subtype)
3. **Access modifier** must be same or more accessible
4. **Cannot override:**
   - `static` methods (hidden, not overridden)
   - `final` methods
   - `private` methods (not inherited)

```java
class Parent {
    public Number getValue() {
        return 42;
    }
    
    protected void process() { }
    
    public static void staticMethod() { }
    
    public final void finalMethod() { }
}

class Child extends Parent {
    // ✅ Covariant return type
    @Override
    public Integer getValue() {  // Integer is subtype of Number
        return 100;
    }
    
    // ✅ More accessible
    @Override
    public void process() { }  // public is more accessible than protected
    
    // ❌ Cannot override static (this is hiding, not overriding)
    public static void staticMethod() { }  // This HIDES parent's method
    
    // ❌ Cannot override final
    // @Override
    // public void finalMethod() { }  // COMPILATION ERROR
}
```

---

## 1.5 Covariant Return Types (Advanced)

Since Java 5, overriding methods can return a **subtype** of the parent's return type.

```java
class Animal {
    public Animal reproduce() {
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    public Dog reproduce() {  // Returns Dog, not Animal
        return new Dog();
    }
}

class Cat extends Animal {
    @Override
    public Cat reproduce() {  // Returns Cat, not Animal
        return new Cat();
    }
}

// Usage
Dog dog = new Dog();
Dog puppy = dog.reproduce();  // No casting needed!
```

---

## 1.6 Modern Java: Pattern Matching for instanceof

### Traditional instanceof (Before Java 16)

```java
// Old way - verbose and error-prone
public void processShape(Object obj) {
    if (obj instanceof Circle) {
        Circle circle = (Circle) obj;  // Explicit cast needed
        double area = Math.PI * circle.radius * circle.radius;
        System.out.println("Circle area: " + area);
    } else if (obj instanceof Rectangle) {
        Rectangle rect = (Rectangle) obj;  // Explicit cast needed
        double area = rect.width * rect.height;
        System.out.println("Rectangle area: " + area);
    }
}
```

### Modern Pattern Matching (Java 16+)

```java
/**
 * Pattern Matching for instanceof - Combines type check and cast
 */
public void processShape(Object obj) {
    // Type test + variable declaration in one line!
    if (obj instanceof Circle c) {
        // 'c' is automatically cast to Circle
        double area = Math.PI * c.radius * c.radius;
        System.out.println("Circle area: " + area);
    } else if (obj instanceof Rectangle r) {
        // 'r' is automatically cast to Rectangle
        double area = r.width * r.height;
        System.out.println("Rectangle area: " + area);
    } else if (obj instanceof Triangle t && t.isValid()) {
        // Can add conditions with && - 't' is in scope
        double area = 0.5 * t.base * t.height;
        System.out.println("Triangle area: " + area);
    }
}
```

**Key Benefits:**
1. **Less verbose** - no explicit casting
2. **Safer** - pattern variable only in scope when type check succeeds
3. **More readable** - intent is clearer

### Pattern Variable Scope

```java
public void demonstrateScope(Object obj) {
    if (obj instanceof String s) {
        System.out.println(s.toUpperCase());  // 's' available here
    }
    // System.out.println(s);  // ERROR: 's' not in scope
    
    // Scope with logical operators
    if (obj instanceof String s && s.length() > 5) {
        System.out.println(s);  // 's' available in entire if block
    }
    
    // Negation - pattern variable NOT available
    if (!(obj instanceof String s)) {
        // System.out.println(s);  // ERROR: 's' not available
    }
}
```

---

## 1.7 Modern Java: Pattern Matching for Switch

### Traditional Switch Limitations

```java
// Old switch - only primitives and Strings
public String getDescription(int code) {
    switch (code) {
        case 1:
            return "Success";
        case 2:
            return "Warning";
        default:
            return "Unknown";
    }
}
```

### Pattern Matching Switch (Java 21+)

```java
record Circle(double radius) { }
record Rectangle(double width, double height) { }
record Triangle(double base, double height) { }

/**
 * Modern switch with pattern matching and type patterns
 */
public double calculateArea(Object shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        
        case Rectangle r -> r.width() * r.height();
        
        case Triangle t -> 0.5 * t.base() * t.height();
        
        // Guarded patterns - conditions after when
        case String s when s.startsWith("circle:") -> {
            double r = Double.parseDouble(s.substring(7));
            yield Math.PI * r * r;
        }
        
        // null case (Java 21+)
        case null -> {
            System.out.println("Shape is null");
            yield 0.0;
        }
        
        default -> {
            System.out.println("Unknown shape: " + shape.getClass().getSimpleName());
            yield 0.0;
        }
    };
}
```

### Guarded Patterns with `when`

```java
public String categorizeNumber(Object obj) {
    return switch (obj) {
        case Integer i when i > 0 -> "Positive integer";
        case Integer i when i < 0 -> "Negative integer";
        case Integer i -> "Zero";
        
        case Double d when d > 0 -> "Positive decimal";
        case Double d when d < 0 -> "Negative decimal";
        case Double d -> "Zero decimal";
        
        case null -> "Null value";
        default -> "Not a number";
    };
}
```

### Null Handling in Pattern Switch

```java
/**
 * Before Java 21: NullPointerException risk
 * After Java 21: Explicit null handling
 */
public void processValue(Object obj) {
    // Java 21+ allows null case
    switch (obj) {
        case null -> System.out.println("Received null");
        case String s -> System.out.println("String: " + s);
        case Integer i -> System.out.println("Integer: " + i);
        default -> System.out.println("Other: " + obj);
    }
}
```

### Real-World Example: Payment Processing

```java
sealed interface PaymentMethod permits CreditCard, DebitCard, Cash, DigitalWallet { }
record CreditCard(String number, String cvv) implements PaymentMethod { }
record DebitCard(String number, String pin) implements PaymentMethod { }
record Cash(double amount) implements PaymentMethod { }
record DigitalWallet(String provider, String accountId) implements PaymentMethod { }

public class ModernPaymentProcessor {
    
    public String processPayment(PaymentMethod payment) {
        return switch (payment) {
            case CreditCard cc when cc.cvv().length() == 3 -> 
                "Processing credit card: " + cc.number();
                
            case CreditCard cc -> 
                "Invalid CVV for credit card";
                
            case DebitCard dc when dc.pin().length() == 4 -> 
                "Processing debit card: " + dc.number();
                
            case DebitCard dc -> 
                "Invalid PIN for debit card";
                
            case Cash cash when cash.amount() > 0 -> 
                "Accepting cash payment: $" + cash.amount();
                
            case Cash cash -> 
                "Invalid cash amount";
                
            case DigitalWallet dw -> 
                "Processing " + dw.provider() + " wallet: " + dw.accountId();
                
            // No default needed - sealed interface covers all cases!
        };
    }
    
    public static void main(String[] args) {
        ModernPaymentProcessor processor = new ModernPaymentProcessor();
        
        PaymentMethod[] payments = {
            new CreditCard("4532-1234-5678-9010", "123"),
            new DebitCard("6011-1234-5678-9010", "1234"),
            new Cash(50.00),
            new DigitalWallet("PayPal", "user@example.com")
        };
        
        for (PaymentMethod payment : payments) {
            System.out.println(processor.processPayment(payment));
        }
    }
}
```

---

## 1.8 Pattern Matching: Complete Type Hierarchy Example

```java
// Shape hierarchy with modern Java
sealed interface Shape permits Circle, Rectangle, Triangle, Composite { }

record Circle(double radius) implements Shape {
    public double area() {
        return Math.PI * radius * radius;
    }
}

record Rectangle(double width, double height) implements Shape {
    public double area() {
        return width * height;
    }
}

record Triangle(double base, double height) implements Shape {
    public double area() {
        return 0.5 * base * height;
    }
}

record Composite(Shape[] shapes) implements Shape {
    public double area() {
        double total = 0;
        for (Shape shape : shapes) {
            total += calculateArea(shape);
        }
        return total;
    }
    
    private double calculateArea(Shape shape) {
        return switch (shape) {
            case Circle c -> c.area();
            case Rectangle r -> r.area();
            case Triangle t -> t.area();
            case Composite comp -> comp.area();
        };
    }
}

public class ShapeProcessor {
    
    // Using instanceof pattern matching
    public void describeShape(Shape shape) {
        if (shape instanceof Circle c) {
            System.out.println("Circle with radius " + c.radius());
        } else if (shape instanceof Rectangle r) {
            System.out.println("Rectangle: " + r.width() + " x " + r.height());
        } else if (shape instanceof Triangle t) {
            System.out.println("Triangle: base=" + t.base() + ", height=" + t.height());
        } else if (shape instanceof Composite comp) {
            System.out.println("Composite with " + comp.shapes().length + " shapes");
        }
    }
    
    // Using switch pattern matching (cleaner!)
    public String getShapeType(Shape shape) {
        return switch (shape) {
            case Circle c -> "Circle";
            case Rectangle r -> "Rectangle";
            case Triangle t -> "Triangle";
            case Composite comp -> "Composite";
        };
    }
    
    // With guards
    public String categorizeBySize(Shape shape) {
        return switch (shape) {
            case Circle c when c.area() < 10 -> "Small circle";
            case Circle c when c.area() < 100 -> "Medium circle";
            case Circle c -> "Large circle";
            
            case Rectangle r when r.area() < 20 -> "Small rectangle";
            case Rectangle r when r.area() < 200 -> "Medium rectangle";
            case Rectangle r -> "Large rectangle";
            
            case Triangle t -> "Triangle of size " + t.area();
            case Composite comp -> "Composite with total area " + comp.area();
        };
    }
}
```

---

# Part 2: Abstraction - Hiding Complexity

## 2.1 Understanding Abstraction Fundamentals

### What is Abstraction?
**Abstraction** means showing only essential features while hiding implementation details.

**Real-World Example:**
- When you drive a car, you use the steering wheel, pedals, and gear shift
- You don't need to know how the engine combustion works
- The interface (controls) is simple, the implementation (engine mechanics) is hidden

### In Programming:
- **What to do** (interface/contract) vs **How to do it** (implementation)
- Users interact with simple methods
- Complex logic is hidden inside

---

## 2.2 Abstract Classes - The Foundation

### What is an Abstract Class?

```java
/**
 * Abstract class - Cannot be instantiated
 * Can have both abstract and concrete methods
 */
public abstract class Animal {
    // Instance variables
    protected String name;
    protected int age;
    
    // Constructor (yes, abstract classes can have constructors!)
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Abstract method - no implementation
    // Subclasses MUST implement this
    public abstract void makeSound();
    
    // Abstract method
    public abstract void move();
    
    // Concrete method - has implementation
    public void sleep() {
        System.out.println(name + " is sleeping...");
    }
    
    // Concrete method
    public void eat(String food) {
        System.out.println(name + " is eating " + food);
    }
    
    // Getter methods
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

### Implementing Abstract Classes

```java
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    // MUST implement abstract methods
    @Override
    public void makeSound() {
        System.out.println(name + " says: Woof! Woof!");
    }
    
    @Override
    public void move() {
        System.out.println(name + " is running on four legs");
    }
    
    // Additional methods specific to Dog
    public void fetch() {
        System.out.println(name + " is fetching the ball!");
    }
}

public class Bird extends Animal {
    private double wingSpan;
    
    public Bird(String name, int age, double wingSpan) {
        super(name, age);
        this.wingSpan = wingSpan;
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " says: Tweet! Tweet!");
    }
    
    @Override
    public void move() {
        System.out.println(name + " is flying with wingspan of " + wingSpan + " cm");
    }
}

public class Fish extends Animal {
    private String waterType;
    
    public Fish(String name, int age, String waterType) {
        super(name, age);
        this.waterType = waterType;
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " makes bubbles: blub blub");
    }
    
    @Override
    public void move() {
        System.out.println(name + " is swimming in " + waterType + " water");
    }
}
```

### Using Abstract Classes

```java
public class ZooManager {
    public static void main(String[] args) {
        // Animal animal = new Animal("Generic", 5);  // ERROR: Cannot instantiate
        
        // But can use as reference type
        Animal[] animals = {
            new Dog("Max", 3, "Golden Retriever"),
            new Bird("Tweety", 1, 15.5),
            new Fish("Nemo", 2, "salt")
        };
        
        // Polymorphism in action
        for (Animal animal : animals) {
            System.out.println("\n=== " + animal.getName() + " ===");
            animal.makeSound();      // Different for each type
            animal.move();           // Different for each type
            animal.sleep();          // Same for all (inherited)
            animal.eat("food");      // Same for all (inherited)
        }
    }
}
```

---

## 2.3 Interfaces - The Pure Contract

### Traditional Interfaces (Before Java 8)

```java
/**
 * Interface - Pure abstraction (before Java 8)
 * All methods are public and abstract by default
 */
public interface Drawable {
    // public abstract by default
    void draw();
    void resize(double scale);
    
    // public static final by default
    int MAX_SIZE = 1000;
}

public class Circle implements Drawable {
    private double radius;
    
    @Override
    public void draw() {
        System.out.println("Drawing a circle with radius: " + radius);
    }
    
    @Override
    public void resize(double scale) {
        radius *= scale;
        System.out.println("Circle resized. New radius: " + radius);
    }
}
```

---

## 2.4 Modern Interfaces (Java 8+)

### Default Methods

```java
/**
 * Interface with default methods (Java 8+)
 * Default methods have implementation in the interface
 */
public interface Vehicle {
    // Abstract methods (must be implemented)
    void start();
    void stop();
    
    // Default method - provides default implementation
    default void honk() {
        System.out.println("Beep! Beep!");
    }
    
    // Default method with implementation
    default void displayInfo() {
        System.out.println("This is a vehicle");
        System.out.println("Type: " + getType());
    }
    
    // Another abstract method
    String getType();
}

public class Car implements Vehicle {
    private String model;
    
    public Car(String model) {
        this.model = model;
    }
    
    @Override
    public void start() {
        System.out.println(model + " engine started");
    }
    
    @Override
    public void stop() {
        System.out.println(model + " engine stopped");
    }
    
    @Override
    public String getType() {
        return "Car: " + model;
    }
    
    // Can optionally override default method
    @Override
    public void honk() {
        System.out.println(model + " honks: HONK HONK!");
    }
}

public class Bicycle implements Vehicle {
    @Override
    public void start() {
        System.out.println("Start pedaling");
    }
    
    @Override
    public void stop() {
        System.out.println("Stop pedaling");
    }
    
    @Override
    public String getType() {
        return "Bicycle";
    }
    
    // Uses default honk() from interface - no override needed
}
```

### Static Methods in Interfaces (Java 8+)

```java
public interface MathOperations {
    // Static method in interface
    static int add(int a, int b) {
        return a + b;
    }
    
    static int multiply(int a, int b) {
        return a * b;
    }
    
    // Static utility method
    static boolean isEven(int number) {
        return number % 2 == 0;
    }
}

// Usage - called on interface name
public class MathDemo {
    public static void main(String[] args) {
        int sum = MathOperations.add(5, 10);
        int product = MathOperations.multiply(5, 10);
        boolean even = MathOperations.isEven(4);
        
        System.out.println("Sum: " + sum);
        System.out.println("Product: " + product);
        System.out.println("Is 4 even? " + even);
    }
}
```

### Private Methods in Interfaces (Java 9+)

```java
/**
 * Private methods in interfaces (Java 9+)
 * Help avoid code duplication in default methods
 */
public interface Logger {
    // Abstract method
    void log(String message);
    
    // Default methods
    default void logInfo(String message) {
        logMessage("INFO", message);
    }
    
    default void logWarning(String message) {
        logMessage("WARNING", message);
    }
    
    default void logError(String message) {
        logMessage("ERROR", message);
    }
    
    // Private method - reusable within interface only
    private void logMessage(String level, String message) {
        String timestamp = java.time.LocalDateTime.now().toString();
        System.out.println("[" + timestamp + "] [" + level + "] " + message);
    }
    
    // Private static method
    private static String formatMessage(String message) {
        return message.toUpperCase();
    }
}

public class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("Log: " + message);
    }
}

// Usage
public class LoggerDemo {
    public static void main(String[] args) {
        Logger logger = new ConsoleLogger();
        logger.logInfo("Application started");
        logger.logWarning("Low memory");
        logger.logError("Connection failed");
    }
}
```

---

## 2.5 Abstract Class vs Interface - The Complete Guide

### Key Differences Table

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| **Instantiation** | Cannot instantiate | Cannot instantiate |
| **Methods** | Abstract + Concrete | Abstract + Default + Static + Private |
| **Variables** | Any (instance, static) | Only `public static final` |
| **Constructor** | Yes | No |
| **Inheritance** | Single inheritance (`extends`) | Multiple inheritance (`implements`) |
| **Access Modifiers** | Any (public, protected, private) | public (methods) |
| **When to use** | "IS-A" relationship | "CAN-DO" relationship |

### Abstract Class Example - IS-A Relationship

```java
/**
 * Abstract class represents "IS-A" relationship
 * A Dog IS AN Animal
 */
public abstract class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Shared behavior
    public void breathe() {
        System.out.println(name + " is breathing");
    }
    
    // Template method pattern
    public final void dailyRoutine() {
        wakeUp();
        eat();
        move();
        sleep();
    }
    
    private void wakeUp() {
        System.out.println(name + " wakes up");
    }
    
    protected abstract void eat();
    protected abstract void move();
    
    public void sleep() {
        System.out.println(name + " goes to sleep");
    }
}
```

### Interface Example - CAN-DO Relationship

```java
/**
 * Interface represents "CAN-DO" relationship
 * A Bird CAN FLY
 * A Fish CAN SWIM
 */
public interface Flyable {
    void fly();
    
    default void takeOff() {
        System.out.println("Taking off...");
        fly();
    }
}

public interface Swimmable {
    void swim();
    
    default void dive() {
        System.out.println("Diving...");
        swim();
    }
}

// A duck can both fly and swim!
public class Duck extends Animal implements Flyable, Swimmable {
    
    public Duck(String name, int age) {
        super(name, age);
    }
    
    @Override
    protected void eat() {
        System.out.println(name + " is eating bread");
    }
    
    @Override
    protected void move() {
        System.out.println(name + " waddles on land");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
    
    @Override
    public void swim() {
        System.out.println(name + " is swimming");
    }
}

// A penguin can swim but NOT fly!
public class Penguin extends Animal implements Swimmable {
    
    public Penguin(String name, int age) {
        super(name, age);
    }
    
    @Override
    protected void eat() {
        System.out.println(name + " is eating fish");
    }
    
    @Override
    protected void move() {
        System.out.println(name + " waddles on land");
    }
    
    @Override
    public void swim() {
        System.out.println(name + " is swimming fast!");
    }
}
```

### When to Use Which?

**Use Abstract Class when:**
1. You want to share code among closely related classes
2. Classes that extend have many common methods/fields
3. You need non-static or non-final fields
4. You need constructors
5. Example: `Animal`, `Vehicle`, `Shape`

**Use Interface when:**
1. Unrelated classes would implement your interface
2. You want to specify behavior without implementation
3. You want multiple inheritance
4. You want to achieve loose coupling
5. Example: `Comparable`, `Serializable`, `Runnable`

---

## 2.6 Real-World Architectural Example

```java
/**
 * Complex example combining abstract classes and interfaces
 * E-commerce payment system
 */

// Base abstraction
public abstract class PaymentMethod {
    protected String accountId;
    protected double transactionFee;
    
    public PaymentMethod(String accountId) {
        this.accountId = accountId;
    }
    
    // Template method - defines algorithm structure
    public final boolean processPayment(double amount) {
        if (!validateAccount()) {
            return false;
        }
        if (!checkFunds(amount)) {
            return false;
        }
        executeTransaction(amount);
        sendConfirmation();
        return true;
    }
    
    // Steps that subclasses must implement
    protected abstract boolean validateAccount();
    protected abstract boolean checkFunds(double amount);
    protected abstract void executeTransaction(double amount);
    
    // Common implementation
    private void sendConfirmation() {
        System.out.println("Payment confirmation sent to " + accountId);
    }
}

// Capabilities (interfaces)
interface Refundable {
    boolean refund(String transactionId, double amount);
    
    default boolean partialRefund(String transactionId, double amount, double percentage) {
        double refundAmount = amount * (percentage / 100);
        return refund(transactionId, refundAmount);
    }
}

interface Recurring {
    void setupRecurring(int days);
    void cancelRecurring();
}

interface SecurePayment {
    boolean authenticate();
    String encrypt(String data);
}

// Concrete implementations
public class CreditCard extends PaymentMethod implements Refundable, SecurePayment {
    private String cardNumber;
    private String cvv;
    
    public CreditCard(String accountId, String cardNumber, String cvv) {
        super(accountId);
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.transactionFee = 2.9; // 2.9%
    }
    
    @Override
    protected boolean validateAccount() {
        System.out.println("Validating credit card...");
        return cardNumber.length() == 16 && cvv.length() == 3;
    }
    
    @Override
    protected boolean checkFunds(double amount) {
        System.out.println("Checking credit limit...");
        return true; // Simplified
    }
    
    @Override
    protected void executeTransaction(double amount) {
        double total = amount + (amount * transactionFee / 100);
        System.out.println("Charging $" + total + " to credit card");
    }
    
    @Override
    public boolean refund(String transactionId, double amount) {
        System.out.println("Refunding $" + amount + " to credit card");
        return true;
    }
    
    @Override
    public boolean authenticate() {
        System.out.println("Authenticating with 3D Secure...");
        return true;
    }
    
    @Override
    public String encrypt(String data) {
        return "ENCRYPTED:" + data;
    }
}

public class BankTransfer extends PaymentMethod implements Recurring {
    private String bankAccount;
    private String routingNumber;
    
    public BankTransfer(String accountId, String bankAccount, String routingNumber) {
        super(accountId);
        this.bankAccount = bankAccount;
        this.routingNumber = routingNumber;
        this.transactionFee = 0.5; // Flat $0.50
    }
    
    @Override
    protected boolean validateAccount() {
        System.out.println("Validating bank account...");
        return bankAccount.length() > 0;
    }
    
    @Override
    protected boolean checkFunds(double amount) {
        System.out.println("Verifying bank balance...");
        return true; // Simplified
    }
    
    @Override
    protected void executeTransaction(double amount) {
        System.out.println("Transferring $" + amount + " from bank");
    }
    
    @Override
    public void setupRecurring(int days) {
        System.out.println("Setting up recurring payment every " + days + " days");
    }
    
    @Override
    public void cancelRecurring() {
        System.out.println("Recurring payment cancelled");
    }
}

// Usage
public class PaymentProcessor {
    public static void main(String[] args) {
        // Abstract class reference, concrete objects
        PaymentMethod cc = new CreditCard("user@email.com", "1234567890123456", "123");
        PaymentMethod bt = new BankTransfer("user@email.com", "98765432", "123456789");
        
        // Process payments
        cc.processPayment(100.00);
        bt.processPayment(200.00);
        
        // Interface capabilities
        if (cc instanceof Refundable refundable) {
            refundable.refund("TXN123", 50.00);
        }
        
        if (bt instanceof Recurring recurring) {
            recurring.setupRecurring(30);
        }
    }
}
```

---

# Part 3: Generics - Type Safety and Reusability

## 3.1 Understanding Generics Fundamentals

### The Problem Generics Solve

**Before Generics (Java < 5):**

```java
// Pre-generics - using Object
public class OldBox {
    private Object content;
    
    public void set(Object content) {
        this.content = content;
    }
    
    public Object get() {
        return content;
    }
}

// Problems:
public class PreGenericsProblems {
    public static void main(String[] args) {
        OldBox box = new OldBox();
        
        // Can put anything
        box.set("Hello");
        box.set(123);        // Oops, overwrote string with integer
        
        // Must cast - risky!
        String text = (String) box.get();  // ClassCastException at RUNTIME!
        
        // No compile-time type checking
        // Errors discovered at runtime - BAD!
    }
}
```

**With Generics (Java 5+):**

```java
// Generic Box - Type-safe
public class Box<T> {
    private T content;
    
    public void set(T content) {
        this.content = content;
    }
    
    public T get() {
        return content;
    }
}

// Benefits:
public class GenericsAdvantages {
    public static void main(String[] args) {
        // Type specified at creation
        Box<String> stringBox = new Box<>();
        stringBox.set("Hello");
        // stringBox.set(123);  // COMPILE ERROR - type safety!
        
        String text = stringBox.get();  // No cast needed!
        
        Box<Integer> intBox = new Box<>();
        intBox.set(123);
        int number = intBox.get();  // No cast needed!
    }
}
```

---

## 3.2 Generic Classes

### Single Type Parameter

```java
/**
 * Generic class with one type parameter
 * Convention: T for Type
 */
public class Container<T> {
    private T item;
    
    public Container(T item) {
        this.item = item;
    }
    
    public T getItem() {
        return item;
    }
    
    public void setItem(T item) {
        this.item = item;
    }
    
    public void displayType() {
        System.out.println("Type: " + item.getClass().getName());
    }
}

// Usage
Container<String> stringContainer = new Container<>("Hello");
Container<Integer> intContainer = new Container<>(42);
Container<Double> doubleContainer = new Container<>(3.14);
```

### Multiple Type Parameters

```java
/**
 * Generic class with multiple type parameters
 * Common conventions:
 * K = Key, V = Value
 * E = Element, T = Type
 */
public class Pair<K, V> {
    private K key;
    private V value;
    
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    public K getKey() { return key; }
    public V getValue() { return value; }
    
    @Override
    public String toString() {
        return "(" + key + ", " + value + ")";
    }
}

// Usage
Pair<String, Integer> pair1 = new Pair<>("Age", 25);
Pair<Integer, String> pair2 = new Pair<>(1, "First");
Pair<String, String> pair3 = new Pair<>("Name", "John");

System.out.println(pair1);  // (Age, 25)
```

### Real-World Generic Class Example

```java
/**
 * Generic Response wrapper for API calls
 */
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private int statusCode;
    
    public ApiResponse(boolean success, String message, T data, int statusCode) {
        this.success = success;
        this.message = message;
        this.data = data;
        this.statusCode = statusCode;
    }
    
    public boolean isSuccess() { return success; }
    public String getMessage() { return message; }
    public T getData() { return data; }
    public int getStatusCode() { return statusCode; }
    
    // Static factory methods for common cases
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data, 200);
    }
    
    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, message, null, 500);
    }
}

// Usage with different data types
class User {
    String name;
    String email;
    
    User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}

class Product {
    String id;
    double price;
    
    Product(String id, double price) {
        this.id = id;
        this.price = price;
    }
}

public class ApiDemo {
    public static void main(String[] args) {
        // Different response types
        ApiResponse<User> userResponse = ApiResponse.success(
            new User("John", "john@example.com")
        );
        
        ApiResponse<List<Product>> productsResponse = ApiResponse.success(
            List.of(new Product("P1", 99.99), new Product("P2", 149.99))
        );
        
        ApiResponse<String> errorResponse = ApiResponse.error("Not found");
    }
}
```

---

## 3.3 Generic Methods

### Generic Method Syntax

```java
public class GenericMethods {
    
    // Generic method - type parameter before return type
    public static <T> void printArray(T[] array) {
        for (T element : array) {
            System.out.print(element + " ");
        }
        System.out.println();
    }
    
    // Generic method with return type
    public static <T> T getMiddleElement(T[] array) {
        return array[array.length / 2];
    }
    
    // Multiple type parameters
    public static <K, V> void printPair(K key, V value) {
        System.out.println(key + " = " + value);
    }
    
    // Generic method in non-generic class
    public static <T extends Comparable<T>> T findMax(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;
    }
}

// Usage
public class GenericMethodDemo {
    public static void main(String[] args) {
        Integer[] intArray = {1, 2, 3, 4, 5};
        String[] strArray = {"A", "B", "C"};
        
        GenericMethods.printArray(intArray);
        GenericMethods.printArray(strArray);
        
        Integer middle = GenericMethods.getMiddleElement(intArray);
        System.out.println("Middle: " + middle);
        
        GenericMethods.printPair("Name", "John");
        GenericMethods.printPair(1, "First");
        
        Integer max = GenericMethods.findMax(10, 20);
        String maxStr = GenericMethods.findMax("Apple", "Banana");
    }
}
```

---

## 3.4 Bounded Type Parameters

### Upper Bounded (`extends`)

```java
/**
 * Upper bound - restricts type to specific class or its subclasses
 * Syntax: <T extends ClassName>
 */
public class NumberProcessor<T extends Number> {
    private T number;
    
    public NumberProcessor(T number) {
        this.number = number;
    }
    
    // Can call Number methods
    public double getDoubleValue() {
        return number.doubleValue();
    }
    
    public int getIntValue() {
        return number.intValue();
    }
    
    public boolean isPositive() {
        return number.doubleValue() > 0;
    }
}

// Works with Number and its subclasses
NumberProcessor<Integer> intProc = new NumberProcessor<>(42);
NumberProcessor<Double> doubleProc = new NumberProcessor<>(3.14);
NumberProcessor<Long> longProc = new NumberProcessor<>(1000L);

// NumberProcessor<String> strProc = new NumberProcessor<>("Hi");  // ERROR!
```

### Multiple Bounds

```java
/**
 * Multiple bounds - interface & class
 * Syntax: <T extends Class & Interface1 & Interface2>
 */
public class DataProcessor<T extends Comparable<T> & Cloneable> {
    private T data;
    
    public DataProcessor(T data) {
        this.data = data;
    }
    
    // Can use Comparable methods
    public boolean isGreaterThan(T other) {
        return data.compareTo(other) > 0;
    }
    
    // Can use Cloneable (clone)
    public T getCopy() throws CloneNotSupportedException {
        return (T) data.clone();  // Note: clone() implementation needed
    }
}
```

### Real-World Bounded Generics Example

```java
/**
 * Generic repository with persistence
 */
interface Identifiable {
    Long getId();
    void setId(Long id);
}

class BaseEntity implements Identifiable {
    private Long id;
    
    @Override
    public Long getId() { return id; }
    
    @Override
    public void setId(Long id) { this.id = id; }
}

class User extends BaseEntity {
    private String username;
    private String email;
    
    // constructors, getters, setters...
}

class Product extends BaseEntity {
    private String name;
    private double price;
    
    // constructors, getters, setters...
}

/**
 * Generic Repository - works with any Identifiable entity
 */
public class Repository<T extends BaseEntity> {
    private List<T> storage = new ArrayList<>();
    private Long nextId = 1L;
    
    public T save(T entity) {
        if (entity.getId() == null) {
            entity.setId(nextId++);
            storage.add(entity);
        } else {
            // Update existing
            int index = findIndexById(entity.getId());
            if (index >= 0) {
                storage.set(index, entity);
            }
        }
        return entity;
    }
    
    public T findById(Long id) {
        return storage.stream()
            .filter(e -> e.getId().equals(id))
            .findFirst()
            .orElse(null);
    }
    
    public List<T> findAll() {
        return new ArrayList<>(storage);
    }
    
    public void delete(Long id) {
        storage.removeIf(e -> e.getId().equals(id));
    }
    
    private int findIndexById(Long id) {
        for (int i = 0; i < storage.size(); i++) {
            if (storage.get(i).getId().equals(id)) {
                return i;
            }
        }
        return -1;
    }
}

// Usage
Repository<User> userRepo = new Repository<>();
Repository<Product> productRepo = new Repository<>();
```

---

## 3.5 Wildcards - The Flexible Types

### Unbounded Wildcard (`?`)

```java
/**
 * Unbounded wildcard - unknown type
 * Use when you don't care about the type
 */
public class WildcardExample {
    
    // Accepts List of any type
    public static void printList(List<?> list) {
        for (Object element : list) {
            System.out.print(element + " ");
        }
        System.out.println();
    }
    
    // Get size regardless of type
    public static int getSize(List<?> list) {
        return list.size();
    }
}

// Usage
List<Integer> integers = List.of(1, 2, 3);
List<String> strings = List.of("A", "B", "C");
List<Double> doubles = List.of(1.1, 2.2, 3.3);

WildcardExample.printList(integers);
WildcardExample.printList(strings);
WildcardExample.printList(doubles);
```

### Upper Bounded Wildcard (`? extends Type`)

```java
/**
 * Upper bounded wildcard - read-only operations
 * "? extends Type" means any type that IS Type or EXTENDS Type
 */
public class UpperBoundedWildcard {
    
    // Accepts List of Number or any subclass (Integer, Double, etc.)
    public static double sumNumbers(List<? extends Number> numbers) {
        double sum = 0;
        for (Number num : numbers) {
            sum += num.doubleValue();  // Can READ and use Number methods
        }
        return sum;
    }
    
    // Can't add (except null)
    public static void tryToAdd(List<? extends Number> numbers) {
        // numbers.add(10);      // COMPILE ERROR!
        // numbers.add(10.5);    // COMPILE ERROR!
        numbers.add(null);       // Only null is allowed
        
        Number num = numbers.get(0);  // Reading is OK
    }
}

// Usage
List<Integer> integers = List.of(1, 2, 3, 4, 5);
List<Double> doubles = List.of(1.5, 2.5, 3.5);

double sum1 = UpperBoundedWildcard.sumNumbers(integers);
double sum2 = UpperBoundedWildcard.sumNumbers(doubles);

System.out.println("Sum of integers: " + sum1);
System.out.println("Sum of doubles: " + sum2);
```

### Lower Bounded Wildcard (`? super Type`)

```java
/**
 * Lower bounded wildcard - write operations
 * "? super Type" means any type that IS Type or a SUPERCLASS of Type
 */
public class LowerBoundedWildcard {
    
    // Accepts List of Integer or any superclass (Number, Object)
    public static void addIntegers(List<? super Integer> list) {
        list.add(10);        // Can ADD Integer
        list.add(20);        // Can ADD Integer
        list.add(30);        // Can ADD Integer
        
        // Object obj = list.get(0);  // Can only read as Object
        // Integer i = list.get(0);   // COMPILE ERROR!
    }
    
    public static void addNumbers(List<? super Number> list) {
        list.add(10);        // Integer is Number
        list.add(10.5);      // Double is Number
        list.add(10L);       // Long is Number
    }
}

// Usage
List<Number> numbers = new ArrayList<>();
List<Object> objects = new ArrayList<>();

LowerBoundedWildcard.addIntegers(numbers);  // Number is super of Integer
LowerBoundedWildcard.addIntegers(objects);  // Object is super of Integer

System.out.println(numbers);
System.out.println(objects);
```

### PECS Principle: Producer Extends, Consumer Super

```java
/**
 * PECS: Producer Extends, Consumer Super
 * 
 * - Use extends when you GET values (produce/read)
 * - Use super when you PUT values (consume/write)
 */
public class PECSExample {
    
    // PRODUCER - READ from source (extends)
    public static void copy(
        List<? extends Number> source,    // PRODUCER - extends
        List<? super Number> destination  // CONSUMER - super
    ) {
        for (Number num : source) {
            destination.add(num);
        }
    }
    
    public static void main(String[] args) {
        // Source lists (PRODUCERS)
        List<Integer> integers = List.of(1, 2, 3);
        List<Double> doubles = List.of(1.5, 2.5, 3.5);
        
        // Destination lists (CONSUMERS)
        List<Number> numbers = new ArrayList<>();
        List<Object> objects = new ArrayList<>();
        
        copy(integers, numbers);  // OK
        copy(doubles, objects);   // OK
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Objects: " + objects);
    }
}
```

---

## 3.6 Type Erasure - Under the Hood

### What is Type Erasure?

Java implements generics using **type erasure**:
- Generic type information is removed at runtime
- Replaced with raw types or bounds
- Maintains backward compatibility with pre-Java 5 code

```java
// Your code
public class Box<T> {
    private T item;
    public void set(T item) { this.item = item; }
    public T get() { return item; }
}

// After type erasure (what JVM sees)
public class Box {
    private Object item;  // T becomes Object
    public void set(Object item) { this.item = item; }
    public Object get() { return item; }
}
```

### Type Erasure with Bounds

```java
// Your code with bounds
public class NumberBox<T extends Number> {
    private T number;
    public void set(T number) { this.number = number; }
    public T get() { return number; }
}

// After type erasure (what JVM sees)
public class NumberBox {
    private Number number;  // T becomes Number (the bound)
    public void set(Number number) { this.number = number; }
    public Number get() { return number; }
}
```

### Implications of Type Erasure

```java
public class TypeErasureImplications {
    
    // ❌ Cannot create instances of type parameters
    public <T> void create() {
        // T obj = new T();  // COMPILE ERROR!
    }
    
    // ❌ Cannot create arrays of generic types
    public <T> void createArray() {
        // T[] array = new T[10];  // COMPILE ERROR!
        
        // Workaround:
        @SuppressWarnings("unchecked")
        T[] array = (T[]) new Object[10];  // Warning but works
    }
    
    // ❌ Cannot use instanceof with generic types
    public <T> void check(Object obj) {
        // if (obj instanceof T) { }  // COMPILE ERROR!
        
        // if (obj instanceof List<String>) { }  // COMPILE ERROR!
        if (obj instanceof List<?>) { }  // OK - using wildcard
    }
    
    // ❌ Cannot have method overloads that differ only in generics
    public void process(List<String> list) { }
    // public void process(List<Integer> list) { }  // COMPILE ERROR!
    // After erasure, both become List
    
    // ❌ Cannot use primitives as type arguments
    public static void main(String[] args) {
        // List<int> numbers;  // ERROR! Use Integer
        List<Integer> numbers = new ArrayList<>();  // OK
    }
}
```

### Bridge Methods (Type Erasure in Inheritance)

```java
class Node<T> {
    private T data;
    
    public void setData(T data) {
        this.data = data;
    }
}

class IntegerNode extends Node<Integer> {
    @Override
    public void setData(Integer data) {
        super.setData(data);
    }
}

// After type erasure, compiler generates a BRIDGE METHOD:
class IntegerNode extends Node {
    // Your method
    public void setData(Integer data) {
        super.setData(data);
    }
    
    // Bridge method generated by compiler
    public void setData(Object data) {
        setData((Integer) data);  // Calls the Integer version
    }
}
```

---

## 3.7 Real-World Generics Examples

### Generic Stack Implementation

```java
/**
 * Generic Stack with proper type safety
 */
public class Stack<T> {
    private List<T> elements = new ArrayList<>();
    
    public void push(T item) {
        elements.add(item);
    }
    
    public T pop() {
        if (isEmpty()) {
            throw new IllegalStateException("Stack is empty");
        }
        return elements.remove(elements.size() - 1);
    }
    
    public T peek() {
        if (isEmpty()) {
            throw new IllegalStateException("Stack is empty");
        }
        return elements.get(elements.size() - 1);
    }
    
    public boolean isEmpty() {
        return elements.isEmpty();
    }
    
    public int size() {
        return elements.size();
    }
    
    @Override
    public String toString() {
        return elements.toString();
    }
}

// Usage
Stack<Integer> intStack = new Stack<>();
intStack.push(1);
intStack.push(2);
intStack.push(3);
System.out.println(intStack.pop());  // 3

Stack<String> strStack = new Stack<>();
strStack.push("A");
strStack.push("B");
System.out.println(strStack.pop());  // B
```

### Generic Pair/Tuple Classes

```java
/**
 * Generic Pair class
 */
public class Pair<F, S> {
    private final F first;
    private final S second;
    
    public Pair(F first, S second) {
        this.first = first;
        this.second = second;
    }
    
    public F getFirst() { return first; }
    public S getSecond() { return second; }
    
    // Static factory method for type inference
    public static <F, S> Pair<F, S> of(F first, S second) {
        return new Pair<>(first, second);
    }
    
    @Override
    public String toString() {
        return "(" + first + ", " + second + ")";
    }
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Pair<?, ?> other)) return false;
        return Objects.equals(first, other.first) && 
               Objects.equals(second, other.second);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(first, second);
    }
}

// Usage
Pair<String, Integer> nameAge = Pair.of("John", 25);
Pair<Integer, String> idName = Pair.of(1, "Admin");

// Triple for 3 elements
public class Triple<F, S, T> {
    private final F first;
    private final S second;
    private final T third;
    
    public Triple(F first, S second, T third) {
        this.first = first;
        this.second = second;
        this.third = third;
    }
    
    public F getFirst() { return first; }
    public S getSecond() { return second; }
    public T getThird() { return third; }
    
    public static <F, S, T> Triple<F, S, T> of(F first, S second, T third) {
        return new Triple<>(first, second, third);
    }
}
```

### Generic Cache Implementation

```java
/**
 * Generic LRU Cache with bounded size
 */
public class Cache<K, V> {
    private final int maxSize;
    private final Map<K, V> storage;
    
    public Cache(int maxSize) {
        this.maxSize = maxSize;
        this.storage = new LinkedHashMap<>(maxSize, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > Cache.this.maxSize;
            }
        };
    }
    
    public void put(K key, V value) {
        storage.put(key, value);
    }
    
    public V get(K key) {
        return storage.get(key);
    }
    
    public boolean containsKey(K key) {
        return storage.containsKey(key);
    }
    
    public void clear() {
        storage.clear();
    }
    
    public int size() {
        return storage.size();
    }
}

// Usage
Cache<String, User> userCache = new Cache<>(100);
Cache<Integer, Product> productCache = new Cache<>(50);
```

---

# Practice Exercises

## Exercise Set 1: Polymorphism Basics

### Exercise 1.1: Shape Calculator
Create a shape hierarchy and calculate areas using polymorphism.

**Requirements:**
1. Abstract class `Shape` with abstract method `calculateArea()`
2. Classes: `Circle`, `Rectangle`, `Triangle`, `Square`
3. Each shape should have appropriate fields
4. Create array of different shapes and calculate total area

```java
// Your solution here
```

### Exercise 1.2: Payment Gateway
Implement a payment system with different payment methods.

**Requirements:**
1. Create abstract class `PaymentGateway`
2. Implement: `CreditCardGateway`, `PayPalGateway`, `UPIGateway`
3. Each gateway has different `processPayment()` logic
4. Add validation specific to each payment type
5. Use polymorphism to process multiple payments

```java
// Your solution here
```

## Exercise Set 2: Pattern Matching

### Exercise 2.1: Vehicle Inspector
Use pattern matching to inspect different vehicle types.

**Requirements:**
1. Sealed interface `Vehicle` with permits: `Car`, `Bike`, `Truck`
2. Each vehicle has different properties
3. Use `instanceof` pattern matching to inspect vehicle
4. Use `switch` pattern matching to categorize vehicles
5. Add guarded patterns for special cases

```java
// Your solution here
```

### Exercise 2.2: JSON Parser Simulator
Use pattern matching to handle different data types.

**Requirements:**
1. Handle: String, Integer, Double, Boolean, null
2. Use switch pattern matching with type patterns
3. Add guards for special values (empty string, negative numbers)
4. Format output based on type

```java
// Your solution here
```

## Exercise Set 3: Abstraction

### Exercise 3.1: File Handler
Create abstraction for file operations.

**Requirements:**
1. Abstract class `FileHandler` with template method
2. Abstract methods: `read()`, `write()`, `validate()`
3. Concrete implementations: `CSVHandler`, `JSONHandler`, `XMLHandler`
4. Interface `Compressible` with default compression methods
5. Some handlers support compression

```java
// Your solution here
```

### Exercise 3.2: Notification Service
Design a notification system using abstraction.

**Requirements:**
1. Interface `NotificationService` with default methods
2. Implementations: `EmailService`, `SMSService`, `PushNotificationService`
3. Interface `Schedulable` for delayed notifications
4. Use private methods in interfaces for shared logic

```java
// Your solution here
```

## Exercise Set 4: Generics Basics

### Exercise 4.1: Generic Box with Methods
Create a generic Box class with utility methods.

**Requirements:**
1. Generic class `Box<T>`
2. Methods: `isEmpty()`, `swap(Box<T> other)`, `equals(Box<T> other)`
3. Static method: `<T> Box<T> of(T item)`
4. Test with different types

```java
// Your solution here
```

### Exercise 4.2: Generic Pair Operations
Implement operations on generic pairs.

**Requirements:**
1. Generic class `Pair<F, S>`
2. Method: `swap()` returns new pair with swapped values
3. Static method: `<T> Pair<T, T> duplicate(T value)`
4. Method: `boolean containsValue(Object value)`

```java
// Your solution here
```

## Exercise Set 5: Bounded Generics

### Exercise 5.1: Comparable Sorter
Create a generic sorter for comparable objects.

**Requirements:**
1. Generic class `Sorter<T extends Comparable<T>>`
2. Methods: `findMin()`, `findMax()`, `sort()`
3. Test with Integer, String, custom class

```java
// Your solution here
```

### Exercise 5.2: Number Statistics
Calculate statistics for numeric collections.

**Requirements:**
1. Class with bounded generic methods
2. Methods: `sum()`, `average()`, `min()`, `max()`
3. Accept `Collection<? extends Number>`
4. Return appropriate numeric types

```java
// Your solution here
```

## Exercise Set 6: Wildcards

### Exercise 6.1: Collection Utilities
Create utility methods using wildcards.

**Requirements:**
1. Method to copy elements: `copy(List<? extends T> src, List<? super T> dest)`
2. Method to print any list: `printList(List<?> list)`
3. Method to add numbers: `addNumbers(List<? super Integer> list, int count)`
4. Method to find in list: `contains(Collection<?> collection, Object item)`

```java
// Your solution here
```

### Exercise 6.2: Generic Repository
Implement a generic repository with wildcards.

**Requirements:**
1. Interface `Repository<T extends Entity>`
2. Methods using wildcards for flexibility
3. Concrete implementation
4. Demonstrate PECS principle

```java
// Your solution here
```

## Exercise Set 7: Advanced Integration

### Exercise 7.1: Event Handler System
Combine all concepts in an event handling system.

**Requirements:**
1. Abstract class `Event` with subclasses
2. Interface `EventHandler<T extends Event>` with default methods
3. Use pattern matching to handle different events
4. Generic `EventBus<T extends Event>` class
5. Support for event filtering with wildcards

```java
// Your solution here
```

### Exercise 7.2: Data Pipeline
Create a generic data processing pipeline.

**Requirements:**
1. Interface `Processor<I, O>` for transformation
2. Abstract class `Pipeline<T>` with template method
3. Concrete processors using bounded generics
4. Use pattern matching for error handling
5. Support chaining with proper type safety

```java
// Your solution here
```

---

# Interview Questions & Answers

## Polymorphism Questions

### Q1: What is polymorphism? Explain its types with examples.
**Answer:**
Polymorphism means "many forms." It allows objects to take different forms and behave differently based on context.

**Two types:**

1. **Compile-time (Static) Polymorphism - Method Overloading:**
   - Same method name, different parameters
   - Resolved at compile time
   - Example:
   ```java
   class Calculator {
       int add(int a, int b) { return a + b; }
       double add(double a, double b) { return a + b; }
       int add(int a, int b, int c) { return a + b + c; }
   }
   ```

2. **Runtime (Dynamic) Polymorphism - Method Overriding:**
   - Subclass provides specific implementation
   - Resolved at runtime
   - Example:
   ```java
   class Animal {
       void sound() { System.out.println("Some sound"); }
   }
   class Dog extends Animal {
       @Override
       void sound() { System.out.println("Bark"); }
   }
   Animal a = new Dog();  // Runtime decides to call Dog's sound()
   a.sound();  // Output: Bark
   ```

### Q2: How does JVM resolve method calls in runtime polymorphism?
**Answer:**
JVM uses **Virtual Method Table (VTable)**:

1. Each class has a VTable with method pointers
2. At runtime, JVM checks actual object type (not reference type)
3. Looks up method in object's VTable
4. Invokes the method from the VTable

Example:
```java
Animal a = new Dog();  // Reference: Animal, Object: Dog
a.sound();  // JVM checks Dog object -> finds Dog's sound() in VTable -> calls it
```

The reference type (Animal) determines which methods can be called (compile-time check), but the object type (Dog) determines which implementation runs (runtime resolution).

### Q3: Can we override static methods? Why or why not?
**Answer:**
**No, we cannot override static methods.** They can be **hidden**, not overridden.

**Reasons:**
1. Static methods belong to the class, not objects
2. Resolved at compile time based on reference type
3. No runtime polymorphism for static methods

```java
class Parent {
    static void display() { System.out.println("Parent"); }
}
class Child extends Parent {
    static void display() { System.out.println("Child"); }  // Hiding, not overriding
}

Parent p = new Child();
p.display();  // Output: Parent (based on reference type!)

Child c = new Child();
c.display();  // Output: Child
```

### Q4: What is covariant return type? When was it introduced?
**Answer:**
**Covariant return type** (Java 5+) allows overriding method to return a subtype of the return type declared in parent class.

```java
class Animal {
    Animal reproduce() { return new Animal(); }
}
class Dog extends Animal {
    @Override
    Dog reproduce() { return new Dog(); }  // Covariant return
}

Dog dog = new Dog();
Dog puppy = dog.reproduce();  // No casting needed!
```

**Benefits:**
- More specific return types
- No casting required
- Better type safety

### Q5: Explain pattern matching for instanceof with examples.
**Answer:**
Pattern matching (Java 16+) combines type checking and casting in one line.

**Old way:**
```java
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.toUpperCase());
}
```

**New way:**
```java
if (obj instanceof String s) {  // Type check + declaration
    System.out.println(s.toUpperCase());  // No cast needed!
}
```

**With conditions:**
```java
if (obj instanceof String s && s.length() > 5) {
    System.out.println(s);  // s is in scope
}
```

**Pattern variable scope:**
- Only available when type check succeeds
- Not available after if block
- Not available in negated conditions

### Q6: What are the new features in switch expressions for pattern matching?
**Answer:**
Java 21 introduced enhanced switch for pattern matching:

**Features:**
1. **Type patterns**
2. **Guarded patterns** with `when`
3. **null handling**
4. **No fall-through**

```java
String result = switch (obj) {
    case Integer i when i > 0 -> "Positive: " + i;
    case Integer i -> "Non-positive: " + i;
    case String s when s.isEmpty() -> "Empty string";
    case String s -> "String: " + s;
    case null -> "Null value";
    default -> "Unknown";
};
```

**Benefits:**
- More readable
- Exhaustiveness checking with sealed types
- Type-safe
- Can return values

---

## Abstraction Questions

### Q7: What's the difference between abstract class and interface?
**Answer:**

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| **Methods** | Abstract + Concrete | Abstract + Default + Static + Private |
| **Variables** | Any type | Only `public static final` |
| **Constructor** | Yes | No |
| **Inheritance** | Single (`extends`) | Multiple (`implements`) |
| **When to use** | IS-A relationship | CAN-DO capability |

**Example:**
```java
// Abstract class - IS-A
abstract class Animal {  // Dog IS-A Animal
    String name;
    Animal(String name) { this.name = name; }
    abstract void sound();
}

// Interface - CAN-DO
interface Flyable {  // Bird CAN-DO flying
    void fly();
}
```

### Q8: What are default methods in interfaces? Why were they introduced?
**Answer:**
**Default methods** (Java 8) provide implementation in interfaces.

**Purpose:**
- **Interface evolution** - add methods without breaking implementations
- **Multiple inheritance of behavior**

```java
interface Vehicle {
    void start();  // Abstract
    
    default void honk() {  // Default implementation
        System.out.println("Beep!");
    }
}

class Car implements Vehicle {
    public void start() { System.out.println("Car started"); }
    // Can use default honk() or override it
}
```

**Real-world example:**
```java
// Java Collections Framework
interface List<E> {
    // ... existing methods
    
    // Added in Java 8 without breaking existing implementations
    default void sort(Comparator<? super E> c) {
        Collections.sort(this, c);
    }
}
```

### Q9: Can interfaces have private methods? What's their purpose?
**Answer:**
**Yes** (Java 9+). Private methods help avoid code duplication in default methods.

```java
interface Logger {
    default void logInfo(String msg) {
        log("INFO", msg);  // Reuse private method
    }
    
    default void logError(String msg) {
        log("ERROR", msg);  // Reuse private method
    }
    
    private void log(String level, String msg) {  // Shared logic
        System.out.println("[" + level + "] " + msg);
    }
}
```

**Purpose:**
- DRY principle in interfaces
- Encapsulation within interface
- Support default methods

### Q10: Can an abstract class implement an interface without implementing its methods?
**Answer:**
**Yes!** Abstract class doesn't need to implement interface methods. Concrete subclasses must.

```java
interface Flyable {
    void fly();
    void land();
}

abstract class Bird implements Flyable {
    // No implementation needed
    abstract void eat();  // Can add own abstract methods
}

class Eagle extends Bird {
    // Must implement all abstract methods
    public void fly() { System.out.println("Eagle flying"); }
    public void land() { System.out.println("Eagle landing"); }
    public void eat() { System.out.println("Eagle eating"); }
}
```

---

## Generics Questions

### Q11: What problem do generics solve? Explain with example.
**Answer:**
**Problems before generics:**
1. No compile-time type safety
2. Required explicit casting
3. Runtime ClassCastException risk

```java
// Before Generics
List list = new ArrayList();
list.add("Hello");
list.add(123);  // Oops! Mixed types
String s = (String) list.get(1);  // ClassCastException at RUNTIME!

// With Generics
List<String> list = new ArrayList<>();
list.add("Hello");
// list.add(123);  // COMPILE ERROR - type safety!
String s = list.get(0);  // No cast needed
```

**Benefits:**
- Compile-time type safety
- Eliminate casts
- Enable generic algorithms

### Q12: What is type erasure? Why does Java use it?
**Answer:**
**Type erasure** removes generic type information at runtime.

**Process:**
1. Replace type parameters with bounds (or Object)
2. Insert casts where needed
3. Generate bridge methods for polymorphism

```java
// Your code
class Box<T> {
    private T item;
    public void set(T item) { this.item = item; }
}

// After erasure (bytecode)
class Box {
    private Object item;  // T becomes Object
    public void set(Object item) { this.item = item; }
}
```

**Why?**
- **Backward compatibility** with pre-Java 5 code
- **No runtime overhead** for generics
- Single `.class` file for all type parameters

**Implications:**
```java
// Cannot do:
T obj = new T();  // ❌
T[] arr = new T[10];  // ❌
if (obj instanceof T) { }  // ❌
```

### Q13: What's the difference between `List<?>`, `List<Object>`, and raw `List`?
**Answer:**

**`List<?>`** - Unknown type (unbounded wildcard):
```java
List<?> list = new ArrayList<String>();
// Object obj = list.get(0);  // Can only read as Object
// list.add("Hello");  // ❌ Cannot add (except null)
```

**`List<Object>`** - Specifically Object type:
```java
List<Object> list = new ArrayList<Object>();  // Must be Object!
// List<Object> list = new ArrayList<String>();  // ❌ Compile error
list.add("Hello");  // ✅ Can add Object or any subtype
list.add(123);      // ✅
```

**Raw `List`** - No generics (backward compatibility):
```java
List list = new ArrayList();  // ⚠️ Warning: unchecked
list.add("Hello");
list.add(123);  // Can add anything - NO TYPE SAFETY
```

**Summary:**
- `List<?>`: Read-only, unknown type
- `List<Object>`: Read-write, specifically Object
- Raw `List`: No type safety, avoid in new code

### Q14: Explain bounded type parameters with examples.
**Answer:**
**Bounded type parameters** restrict generic types to specific classes or interfaces.

**Upper Bound (`extends`):**
```java
// Only Number or its subclasses
class NumberBox<T extends Number> {
    private T number;
    
    public double getDoubleValue() {
        return number.doubleValue();  // Can call Number methods
    }
}

NumberBox<Integer> intBox = new NumberBox<>();  // ✅
NumberBox<Double> doubleBox = new NumberBox<>();  // ✅
// NumberBox<String> strBox = new NumberBox<>();  // ❌
```

**Multiple Bounds:**
```java
// Must extend Number AND implement Comparable
class SortableNumber<T extends Number & Comparable<T>> {
    public T max(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;
    }
}
```

**Lower Bound (`super`):**
```java
// Only Integer or its superclasses (Number, Object)
public void addIntegers(List<? super Integer> list) {
    list.add(10);  // ✅ Can add Integer
}
```

### Q15: What is PECS principle? Explain with examples.
**Answer:**
**PECS: Producer Extends, Consumer Super**

**Rule:**
- Use `extends` when you **GET** (produce/read) values
- Use `super` when you **PUT** (consume/write) values

```java
// PRODUCER - extends (reading)
public void printNumbers(List<? extends Number> numbers) {
    for (Number num : numbers) {  // Reading - OK
        System.out.println(num);
    }
    // numbers.add(10);  // ❌ Cannot write
}

// CONSUMER - super (writing)
public void addIntegers(List<? super Integer> list) {
    list.add(10);  // Writing - OK
    list.add(20);
    // Integer i = list.get(0);  // ❌ Cannot read as Integer
}

// Both together
public void copy(
    List<? extends Number> source,    // PRODUCER
    List<? super Number> destination  // CONSUMER
) {
    for (Number num : source) {
        destination.add(num);
    }
}
```

### Q16: Why can't we create generic array? What's the workaround?
**Answer:**
**Cannot create generic array due to type erasure.**

```java
// ❌ Cannot do
T[] array = new T[10];  // Compile error

// Why?
// After erasure: Object[] array = new Object[10];
// Type safety cannot be guaranteed
```

**Workarounds:**

**1. Use ArrayList:**
```java
List<T> list = new ArrayList<>();  // Preferred
```

**2. Use Array.newInstance (runtime):**
```java
@SuppressWarnings("unchecked")
T[] array = (T[]) Array.newInstance(componentType, size);
```

**3. Use Object array with cast:**
```java
@SuppressWarnings("unchecked")
T[] array = (T[]) new Object[10];  // Warning
```

**Example:**
```java
public class GenericArray<T> {
    private T[] array;
    
    @SuppressWarnings("unchecked")
    public GenericArray(int size) {
        array = (T[]) new Object[size];  // Suppressed warning
    }
    
    public void set(int index, T value) {
        array[index] = value;
    }
    
    public T get(int index) {
        return array[index];
    }
}
```

---

## Advanced Interview Questions

### Q17: What happens if a class implements two interfaces with same default method?
**Answer:**
**Compiler error** - ambiguity. Must override and choose explicitly.

```java
interface A {
    default void display() { System.out.println("A"); }
}

interface B {
    default void display() { System.out.println("B"); }
}

class C implements A, B {
    @Override
    public void display() {
        // Must explicitly choose or provide new implementation
        A.super.display();  // Call A's default
        // OR
        B.super.display();  // Call B's default
        // OR
        System.out.println("C");  // Own implementation
    }
}
```

### Q18: Can you have a generic singleton? How?
**Answer:**
**Yes**, using a generic type parameter in the singleton.

```java
public class GenericSingleton<T> {
    private static GenericSingleton<?> instance;
    private T value;
    
    private GenericSingleton() { }
    
    @SuppressWarnings("unchecked")
    public static <T> GenericSingleton<T> getInstance() {
        if (instance == null) {
            synchronized (GenericSingleton.class) {
                if (instance == null) {
                    instance = new GenericSingleton<T>();
                }
            }
        }
        return (GenericSingleton<T>) instance;
    }
    
    public T getValue() { return value; }
    public void setValue(T value) { this.value = value; }
}

// Usage
GenericSingleton<String> stringSingleton = GenericSingleton.getInstance();
GenericSingleton<Integer> integerSingleton = GenericSingleton.getInstance();
```

**Note:** Due to type erasure, there's only ONE instance for all types!

### Q19: How does covariance work with generics and arrays?
**Answer:**
**Arrays are covariant, Generics are invariant.**

**Arrays - Covariant:**
```java
Object[] objects = new String[10];  // ✅ Compiles
objects[0] = "Hello";  // ✅ OK
objects[1] = 123;  // ❌ ArrayStoreException at RUNTIME!
```

**Generics - Invariant:**
```java
List<Object> objects = new ArrayList<String>();  // ❌ Compile error
// Generics are safer - error caught at compile time
```

**Reason:**
- Arrays: Runtime type checking (slower, less safe)
- Generics: Compile-time type checking (faster, safer)

**Workaround - Use wildcards:**
```java
List<? extends Object> objects = new ArrayList<String>();  // ✅
```

### Q20: What is the diamond problem in multiple inheritance? How does Java solve it?
**Answer:**
**Diamond Problem**: When a class inherits from two classes that have the same method.

```
     A (method m)
    / \
   B   C (both inherit m)
    \ /
     D (which m to inherit?)
```

**Java's Solution:**

**1. Classes:** No multiple inheritance - only single inheritance
```java
class D extends B, C { }  // ❌ Not allowed
```

**2. Interfaces:** Multiple inheritance allowed, but:
```java
interface A {
    default void m() { System.out.println("A"); }
}

interface B extends A {
    default void m() { System.out.println("B"); }
}

interface C extends A {
    default void m() { System.out.println("C"); }
}

class D implements B, C {
    @Override
    public void m() {
        // Must explicitly resolve
        B.super.m();  // OR C.super.m()
    }
}
```

**Resolution Rules:**
1. Class method beats interface default method
2. More specific interface beats less specific
3. Explicit override required if ambiguous

---

## Architectural Design Questions

### Q21: Design a generic event handling system.
**Answer:**
```java
// Event hierarchy
abstract class Event {
    private final String id;
    private final long timestamp;
    
    protected Event(String id) {
        this.id = id;
        this.timestamp = System.currentTimeMillis();
    }
    
    public String getId() { return id; }
    public long getTimestamp() { return timestamp; }
}

// Generic event handler
interface EventHandler<T extends Event> {
    void handle(T event);
    
    default boolean canHandle(Event event) {
        return getSupportedEventType().isInstance(event);
    }
    
    Class<T> getSupportedEventType();
}

// Event bus
class EventBus {
    private Map<Class<? extends Event>, List<EventHandler<?>>> handlers 
        = new HashMap<>();
    
    public <T extends Event> void register(
        Class<T> eventType, 
        EventHandler<T> handler
    ) {
        handlers.computeIfAbsent(eventType, k -> new ArrayList<>())
                .add(handler);
    }
    
    @SuppressWarnings("unchecked")
    public <T extends Event> void publish(T event) {
        List<EventHandler<?>> eventHandlers = handlers.get(event.getClass());
        if (eventHandlers != null) {
            for (EventHandler<?> handler : eventHandlers) {
                ((EventHandler<T>) handler).handle(event);
            }
        }
    }
}
```

### Q22: Design a type-safe builder pattern using generics.
**Answer:**
```java
// Generic builder with fluent API
public class Person {
    private final String name;
    private final int age;
    private final String email;
    
    private Person(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
    }
    
    public static class Builder {
        private String name;
        private int age;
        private String email;
        
        public Builder name(String name) {
            this.name = name;
            return this;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder email(String email) {
            this.email = email;
            return this;
        }
        
        public Person build() {
            // Validation
            if (name == null) throw new IllegalStateException("Name required");
            return new Person(this);
        }
    }
}

// Usage
Person person = new Person.Builder()
    .name("John")
    .age(25)
    .email("john@example.com")
    .build();
```

---

# Week 3 Summary

## Key Takeaways

### Polymorphism
✅ Compile-time (overloading) vs Runtime (overriding)  
✅ Virtual Method Table (VTable) for method resolution  
✅ Pattern matching for `instanceof` (Java 16+)  
✅ Pattern matching for `switch` with guards (Java 21+)  
✅ Covariant return types  

### Abstraction
✅ Abstract classes: partial abstraction with state  
✅ Interfaces: pure contracts with default/static/private methods  
✅ When to use which: IS-A vs CAN-DO  
✅ Multiple inheritance with interfaces  

### Generics
✅ Type safety and code reusability  
✅ Generic classes, methods, and interfaces  
✅ Bounded type parameters (`extends`, `super`)  
✅ Wildcards (`?`, `? extends`, `? super`)  
✅ PECS principle  
✅ Type erasure and its implications  

## Next Steps for Week 4
- Essential APIs (Time API, Optional)
- Collections Framework deep dive
- Sequenced Collections
- HashMap internals

---

**End of Week 3 Material**

Remember: Practice is key! Complete all exercises before moving to Week 4.
