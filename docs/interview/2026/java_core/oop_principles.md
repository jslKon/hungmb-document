# OOP Principles in Java

## 1. The Four Pillars of OOP

### Encapsulation

Bundling data (fields) and methods that operate on that data within a single unit (class), and *
*restricting direct access** to the internal state.

```java
public class BankAccount {

  private double balance; // hidden from outside

  public double getBalance() {
    return balance;
  }

  public void deposit(double amount) {
    if (amount <= 0) {
      throw new IllegalArgumentException("Amount must be positive");
    }
    this.balance += amount;
  }
}
```

**Why it matters:**

- Controls how data is accessed and modified (validation in setters).
- Internal representation can change without affecting callers.
- Reduces coupling between components.

**Access Modifiers:**

| Modifier    | Class | Package | Subclass | World |
|-------------|-------|---------|----------|-------|
| `private`   | Yes   | No      | No       | No    |
| (default)   | Yes   | Yes     | No       | No    |
| `protected` | Yes   | Yes     | Yes      | No    |
| `public`    | Yes   | Yes     | Yes      | Yes   |

> **Reference
**: [Controlling Access - Java Tutorials](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)

---

### Inheritance

A class (subclass) acquires properties and behaviors of another class (superclass), enabling **code
reuse** and establishing an **is-a** relationship.

```java
public class Animal {

  protected String name;

  public void eat() {
    System.out.println(name + " is eating");
  }
}

public class Dog extends Animal {

  public void bark() {
    System.out.println(name + " says Woof!");
  }
}
```

**Key points:**

- Java supports **single inheritance** only (one superclass). Use interfaces for multiple type
  inheritance.
- `super` keyword accesses parent class members and constructor.
- Constructor chaining: subclass constructor calls `super()` implicitly or explicitly.
- `final` class cannot be extended. `final` method cannot be overridden.

**When to use:**

- True **is-a** relationship (Dog is an Animal).
- Shared behavior across a family of classes.

**When NOT to use:**

- Just for code reuse — prefer **composition** (has-a) instead.

> **Reference
**: [Inheritance - Java Tutorials](https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html)

---

### Polymorphism

The ability of an object to take **many forms**. Same method call, different behavior depending on
the actual object type.

#### Compile-time Polymorphism (Method Overloading)

Same method name, different parameter lists. Resolved at **compile time**.

```java
public class Calculator {

  public int add(int a, int b) {
    return a + b;
  }

  public double add(double a, double b) {
    return a + b;
  }

  public int add(int a, int b, int c) {
    return a + b + c;
  }
}
```

**Rules:**

- Must differ in parameter type, count, or order.
- Return type alone is NOT sufficient to overload.

#### Runtime Polymorphism (Method Overriding)

Subclass provides its own implementation of a parent method. Resolved at **runtime** via dynamic
dispatch.

```java
public class Shape {

  public double area() {
    return 0;
  }
}

public class Circle extends Shape {

  private double radius;

  @Override
  public double area() {
    return Math.PI * radius * radius;
  }
}

// Runtime polymorphism in action
Shape shape = new Circle(5);
shape.

area(); // calls Circle.area(), not Shape.area()
```

**Overriding rules:**

- Same method signature (name + parameters).
- Return type must be **same or covariant** (subtype).
- Access modifier must be **same or less restrictive**.
- Cannot override `final`, `static`, or `private` methods.
- `@Override` annotation — always use it (catches mistakes at compile time).

> **Reference
**: [Polymorphism - Java Tutorials](https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html)

---

### Abstraction

Hiding implementation details and exposing only the **essential behavior** through abstract classes
and interfaces.

#### Abstract Classes

```java
public abstract class Vehicle {

  protected String brand;

  // Abstract method — no body, must be implemented by subclass
  public abstract void start();

  // Concrete method — shared behavior
  public void stop() {
    System.out.println("Vehicle stopped");
  }
}

public class Car extends Vehicle {

  @Override
  public void start() {
    System.out.println("Car engine starting...");
  }
}
```

#### Interfaces

```java
public interface Drivable {

  void accelerate(int speed);  // implicitly public abstract

  default void brake() {       // default method (Java 8+)
    System.out.println("Braking...");
  }

  static int maxSpeed() {      // static method (Java 8+)
    return 200;
  }
}
```

#### Abstract Class vs Interface

| Feature      | Abstract Class                        | Interface                                             |
|--------------|---------------------------------------|-------------------------------------------------------|
| Inheritance  | Single (`extends`)                    | Multiple (`implements`)                               |
| Fields       | Instance variables, any access        | `public static final` only                            |
| Constructors | Yes                                   | No                                                    |
| Methods      | Abstract + concrete                   | Abstract + `default` + `static` + `private` (Java 9+) |
| Use case     | Shared state + partial implementation | Contract / capability definition                      |

**Rule of thumb**: Use interfaces to define **what** an object can do. Use abstract classes when
subclasses share **state and behavior**.

> **Reference
**: [Abstract Methods and Classes - Java Tutorials](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html)

---

## 2. SOLID Principles

### S — Single Responsibility Principle (SRP)

> A class should have only **one reason to change**.

```java
// BAD — handles both business logic and persistence
public class UserService {

  public void createUser(User user) { /* validation + business logic */ }

  public void saveToDatabase(User user) { /* SQL query */ }

  public void sendEmail(User user) { /* email logic */ }
}

// GOOD — each class has one responsibility
public class UserService {

  public void createUser(User user) { /* business logic */ }
}

public class UserRepository {

  public void save(User user) { /* persistence */ }
}

public class EmailService {

  public void sendWelcomeEmail(User user) { /* email */ }
}
```

### O — Open/Closed Principle (OCP)

> Software entities should be **open for extension, closed for modification**.

```java
// BAD — must modify existing code for each new shape
public double area(Shape shape) {
  if (shape instanceof Circle c)
    return Math.PI * c.radius * c.radius;
  if (shape instanceof Rectangle r)
    return r.width * r.height;
  // adding Triangle requires modifying this method
}

// GOOD — extend by adding new classes, no modification needed
public abstract class Shape {

  public abstract double area();
}

public class Triangle extends Shape {

  @Override
  public double area() {
    return 0.5 * base * height;
  }
}
```

### L — Liskov Substitution Principle (LSP)

> Subtypes must be **substitutable** for their base types without altering correctness.

```java
// BAD — violates LSP
public class Rectangle {

  public void setWidth(int w) {
    this.width = w;
  }

  public void setHeight(int h) {
    this.height = h;
  }
}

public class Square extends Rectangle {

  @Override
  public void setWidth(int w) {
    this.width = w;
    this.height = w;
  } // surprise!
}

// Code expecting Rectangle breaks:
Rectangle r = new Square();
r.

setWidth(5);
r.

setHeight(10);
assert r.

area() ==50; // FAILS — Square made both sides 10
```

**Test**: if overriding a method changes the expected behavior for callers, LSP is violated.

### I — Interface Segregation Principle (ISP)

> Clients should not be forced to depend on methods they do not use.

```java
// BAD — fat interface
public interface Worker {

  void work();

  void eat();    // robots don't eat

  void sleep();  // robots don't sleep
}

// GOOD — segregated interfaces
public interface Workable {

  void work();
}

public interface Eatable {

  void eat();
}

public interface Sleepable {

  void sleep();
}

public class Robot implements Workable {

  @Override
  public void work() { /* work */ }
}
```

### D — Dependency Inversion Principle (DIP)

> High-level modules should not depend on low-level modules. Both should depend on **abstractions**.

```java
// BAD — tightly coupled to MySQL
public class OrderService {

  private MySQLOrderRepository repo = new MySQLOrderRepository();
}

// GOOD — depends on abstraction
public class OrderService {

  private final OrderRepository repo; // interface

  public OrderService(OrderRepository repo) { // injected
    this.repo = repo;
  }
}
```

This is the foundation of **Dependency Injection** in Spring.

> **Reference
**: [SOLID Principles - Wikipedia](https://en.wikipedia.org/wiki/SOLID) | [Robert C. Martin's original paper](https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)

---

## 3. Design Patterns (Most Asked in Interviews)

### Creational Patterns

#### Singleton

Ensure a class has only one instance.

```java
// Thread-safe, lazy initialization (Bill Pugh approach)
public class Singleton {

  private Singleton() {
  }

  private static class Holder {

    static final Singleton INSTANCE = new Singleton();
  }

  public static Singleton getInstance() {
    return Holder.INSTANCE;
  }
}

// Java enum singleton (recommended by Effective Java)
public enum Singleton {
  INSTANCE;

  public void doSomething() {
  }
}
```

#### Factory Method

Define an interface for creating objects, but let subclasses decide which class to instantiate.

```java
public interface Notification {

  void send(String message);
}

public class EmailNotification implements Notification { /* ... */

}

public class SMSNotification implements Notification { /* ... */

}

public class NotificationFactory {

  public static Notification create(String type) {
    return switch (type) {
      case "EMAIL" -> new EmailNotification();
      case "SMS" -> new SMSNotification();
      default -> throw new IllegalArgumentException("Unknown type: " + type);
    };
  }
}
```

#### Builder

Construct complex objects step by step. Especially useful when constructor has many parameters.

```java
public class User {

  private final String name;
  private final String email;
  private final int age;

  private User(Builder builder) {
    this.name = builder.name;
    this.email = builder.email;
    this.age = builder.age;
  }

  public static class Builder {

    private final String name; // required
    private String email;      // optional
    private int age;           // optional

    public Builder(String name) {
      this.name = name;
    }

    public Builder email(String email) {
      this.email = email;
      return this;
    }

    public Builder age(int age) {
      this.age = age;
      return this;
    }

    public User build() {
      return new User(this);
    }
  }
}

User user = new User.Builder("Alice").email("alice@mail.com").age(30).build();
```

> **Tip**: Lombok's `@Builder` generates this automatically.

### Behavioral Patterns

#### Strategy

Define a family of algorithms and make them interchangeable.

```java
public interface SortStrategy {

  void sort(int[] array);
}

public class QuickSort implements SortStrategy {

  @Override
  public void sort(int[] array) { /* quicksort */ }
}

public class Sorter {

  private SortStrategy strategy;

  public void setStrategy(SortStrategy strategy) {
    this.strategy = strategy;
  }

  public void sort(int[] array) {
    strategy.sort(array);
  }
}
```

#### Observer

One-to-many dependency — when one object changes state, all dependents are notified.

```java
public interface EventListener {

  void onEvent(String event);
}

public class EventPublisher {

  private final List<EventListener> listeners = new ArrayList<>();

  public void subscribe(EventListener listener) {
    listeners.add(listener);
  }

  public void publish(String event) {
    listeners.forEach(l -> l.onEvent(event));
  }
}
```

### Structural Patterns

#### Decorator

Add behavior to objects dynamically without modifying them.

```java
public interface Coffee {

  double cost();

  String description();
}

public class BasicCoffee implements Coffee {

  public double cost() {
    return 2.0;
  }

  public String description() {
    return "Basic coffee";
  }
}

public class MilkDecorator implements Coffee {

  private final Coffee coffee;

  public MilkDecorator(Coffee coffee) {
    this.coffee = coffee;
  }

  public double cost() {
    return coffee.cost() + 0.5;
  }

  public String description() {
    return coffee.description() + " + milk";
  }
}

Coffee order = new MilkDecorator(new BasicCoffee()); // $2.50
```

> **Reference
**: [Design Patterns - Refactoring Guru](https://refactoring.guru/design-patterns) | [Effective Java by Joshua Bloch](https://www.oreilly.com/library/view/effective-java/9780134686097/)

---

## 4. Other Important OOP Concepts

### Composition over Inheritance

> "Favor object composition over class inheritance" — Gang of Four

```java
// Instead of: class Car extends Engine
// Use composition:
public class Car {

  private final Engine engine; // Car HAS-A Engine

  public Car(Engine engine) {
    this.engine = engine;
  }

  public void start() {
    engine.start();
  }
}
```

**Why prefer composition:**

- Avoids fragile base class problem.
- More flexible — can change behavior at runtime.
- Avoids deep inheritance hierarchies.

### Immutability

Objects whose state cannot change after creation.

```java
// Java record (Java 16+) — immutable by default
public record Point(int x, int y) {

}

// Traditional immutable class
public final class Money {

  private final BigDecimal amount;
  private final Currency currency;

  public Money(BigDecimal amount, Currency currency) {
    this.amount = amount;
    this.currency = currency;
  }
  // only getters, no setters
}
```

**Benefits**: thread-safe, safe as map keys, predictable.

### Sealed Classes (Java 17+)

Restrict which classes can extend a class or implement an interface.

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {

}

public final class Circle implements Shape {

}

public final class Rectangle implements Shape {

}

public non-sealed class Triangle implements Shape {

} // open for further extension
```

Enables exhaustive `switch` expressions (compiler knows all subtypes).

> **Reference**: [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)

---

## Interview Questions Cheat Sheet

| Question                               | Key Points                                                                                                                                 |
|----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| Encapsulation vs Abstraction           | Encapsulation = hiding data (access modifiers). Abstraction = hiding implementation (interfaces/abstract classes).                         |
| Overloading vs Overriding              | Overloading = compile-time, same name diff params. Overriding = runtime, same signature in subclass.                                       |
| Abstract class vs Interface            | Abstract = partial impl + state. Interface = contract only (but default methods since Java 8).                                             |
| When to use inheritance vs composition | Inheritance = true is-a. Composition = has-a, more flexible, preferred by default.                                                         |
| Why is `String` immutable?             | Thread safety, caching (string pool), security (used in class loading, network), hashCode caching.                                         |
| Explain SOLID with examples            | One principle per answer with a real scenario. Show bad → good refactoring.                                                                |
| Which design patterns have you used?   | Mention patterns used in Spring: Singleton (beans), Factory (BeanFactory), Proxy (AOP), Template Method (JdbcTemplate), Observer (events). |

---

## Deep Dive Resources (by Section)

### 1. Four Pillars of OOP

- [OOP Concepts in Java — Baeldung](https://www.baeldung.com/java-oop)
- [Encapsulation in Java — Baeldung](https://www.baeldung.com/java-encapsulation)
- [Polymorphism in Java — Baeldung](https://www.baeldung.com/java-polymorphism)
- [Abstract Class vs Interface in Java — Baeldung](https://www.baeldung.com/java-interface-vs-abstract-class)
- [Java Access Modifiers — Jenkov](https://jenkov.com/tutorials/java/access-modifiers.html)

### 2. SOLID Principles

- [SOLID Principles in Java — Baeldung](https://www.baeldung.com/solid-principles)
- [A Solid Guide to SOLID Principles — DigitalOcean](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
- [Liskov Substitution Principle Explained — Reflectoring](https://reflectoring.io/lsp-explained/)
- [The Principles of OOD (original Robert C. Martin articles)](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)
- [Dependency Inversion Principle and Spring DI — DZone](https://dzone.com/articles/solid-principles-dependency-inversion-principle)

### 3. Design Patterns

- [Design Patterns (all patterns with Java examples) — Refactoring Guru](https://refactoring.guru/design-patterns/java)
- [Singleton Pattern in Java — Baeldung](https://www.baeldung.com/java-singleton)
- [Factory Method Pattern — Refactoring Guru](https://refactoring.guru/design-patterns/factory-method/java/example)
- [Builder Pattern — Refactoring Guru](https://refactoring.guru/design-patterns/builder/java/example)
- [Strategy Pattern — Refactoring Guru](https://refactoring.guru/design-patterns/strategy/java/example)
- [Observer Pattern — Refactoring Guru](https://refactoring.guru/design-patterns/observer/java/example)
- [Decorator Pattern — Refactoring Guru](https://refactoring.guru/design-patterns/decorator/java/example)
- [Design Patterns in Spring Framework — Baeldung](https://www.baeldung.com/spring-framework-design-patterns)
- [Head First Design Patterns (book)](https://www.oreilly.com/library/view/head-first-design/9781492077992/)

### 4. Other OOP Concepts

- [Composition vs Inheritance — Baeldung](https://www.baeldung.com/java-inheritance-composition)
- [Immutable Objects in Java — Baeldung](https://www.baeldung.com/java-immutable-object)
- [Java Records — Baeldung](https://www.baeldung.com/java-record-keyword)
- [Sealed Classes in Java — Baeldung](https://www.baeldung.com/java-sealed-classes-interfaces)
- [Effective Java (3rd Edition) by Joshua Bloch — Chapter summaries](https://github.com/jbloch/effective-java-3e-source-code)
