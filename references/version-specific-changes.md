# Java Version-Specific Changes Reference

Guide to feature availability and changes across Java versions 8-21. Use this for version-aware checks.

## Version Overview

| Version | Release Date | LTS | Key Features |
|---------|-------------|-----|--------------|
| Java 8 | 2014-03-18 | Yes | Lambdas, Streams, Optional, new Date API |
| Java 9 | 2017-09-21 | No | Modules (JPMS), jShell, Private methods in interfaces |
| Java 10 | 2018-03-20 | No | Local-Variable Type Inference (var) |
| Java 11 | 2018-09-25 | Yes | String methods, Files methods, HTTP Client |
| Java 12 | 2019-03-19 | No | Switch expressions (preview) |
| Java 13 | 2019-09-17 | No | Text blocks (preview) |
| Java 14 | 2020-03-17 | No | Switch expressions, Records (preview) |
| Java 15 | 2020-09-15 | No | Text blocks, Sealed classes (preview) |
| Java 16 | 2021-03-16 | No | Records, Pattern matching for instanceof |
| Java 17 | 2021-09-14 | Yes | Sealed classes, Records, Pattern matching |
| Java 18 | 2022-03-22 | No | Pattern matching for switch (preview) |
| Java 19 | 2022-09-20 | No | Virtual threads (preview) |
| Java 20 | 2023-03-21 | No | Scoped values, Virtual threads (preview) |
| Java 21 | 2023-09-19 | Yes | Virtual threads, Sequenced collections, Pattern matching for switch |

---

## Java 8 Features

### New APIs

```java
// Stream API
List<String> filtered = list.stream()
    .filter(s -> s.startsWith("A"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// Optional
String result = optional.orElse("default");
String result = optional.orElseThrow(Exception::new);

// New Date/Time API (java.time)
LocalDate today = LocalDate.now();
LocalDateTime now = LocalDateTime.now();
Duration duration = Duration.between(start, end);
Period period = Period.ofYears(2);

// Base64
String encoded = Base64.getEncoder().encodeToString(data);
```

### Language Features

```java
// Lambdas
Runnable r = () -> System.out.println("Hello");

// Method references
list.forEach(System.out::println);

// Default methods in interfaces
interface Interface {
    default void method() { }
}
```

### Rules for Java 8

| Rule ID | Check |
|---------|-------|
| EJ-001 | Use StringBuilder only for loops, simple + is fine |
| EJ-002 | Use == for primitives, equals() for objects |
| EJ-003 | Use @Override always |
| EJ-004 | Use static nested class if no outer reference |
| EJ-005 | Return empty collections, not nulls |
| EJ-006 | Use generics, not raw types |
| EJ-007 | Don't modify collection during iteration |
| EJ-018 | Use Optional properly (orElse, orElseThrow) |
| EJ-020 | Use Stream API where appropriate |
| EJ-021 | Don't use forEach with side effects |
| EJ-022 | Use method references |
| EJ-028 | Use Optional for optional return types |
```

---

## Java 9 Features

### New APIs

```java
// Immutable collections (Java 9+)
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b");
Map<String, Integer> map = Map.of("a", 1, "b", 2);

// Private methods in interfaces
interface Interface {
    default void method() {
        helper(); // Can call private method
    }
    private void helper() { }
}

// Stream improvements
list.stream().takeWhile(predicate)
           .dropWhile(predicate)
           .iterate(0, i -> i + 1)
           .forEach(System.out::println);

// Optional improvements
optional.ifPresentOrElse(action, emptyAction);

// jShell - interactive Java REPL
// /list, /edit, /vars, /methods
```

### Rules for Java 9

| Rule ID | Check |
|---------|-------|
| EJ-016 | Use List.of(), Set.of(), Map.of() for immutable collections |
| EJ-017 | Use immutable collections over defensive copying |
| EJ-011 | Use try-with-resources (improved) |
```

---

## Java 10 Features

### Local-Variable Type Inference

```java
// Use var (Java 10+)
var list = new ArrayList<String>(); // Infers ArrayList<String>
var stream = list.stream();           // Infers Stream<String>

// var with lambdas needs explicit type
var supplier = (Callable<String>) () -> "hello";
// Or: Function<String, String> mapper = s -> s.toUpperCase();
```

### Rules for Java 10

| Rule ID | Check |
|---------|-------|
| EJ-012 | var is lowercase, not Var |
| EJ-013 | Use explicit types with lambdas, var can be ambiguous |
| EJ-014 | Use String.isBlank() instead of .trim().isEmpty() |
| EJ-015 | Use Files.readString() instead of readAllLines() |
```

---

## Java 11 Features

### New String Methods

```java
// Java 11 String methods
String blank = "   ";
blank.isBlank();           // true - checks for empty or whitespace only
String lines = "line1\nline2";
lines.lines();             // Stream<String> of lines
"Hello".repeat(3);         // "HelloHelloHello"
"Hello".strip();           // Like trim() but handles Unicode
"Hello".stripLeading();    // Remove leading whitespace
"String".stripTrailing(); // Remove trailing whitespace
```

### New Files Methods

```java
// Java 11 Files methods
String content = Files.readString(path);
Files.writeString(path, content);
long count = Files.mismatch(path1, path2); // -1 if same, else position
```

### HTTP Client (Java 11+)

```java
// New HTTP Client (Java 11+)
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .GET()
    .build();
HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());
```

### Local-Variable Syntax for Lambda

```java
// Java 11 allows var in lambda parameters
list.stream()
    .map((var s) -> s.toUpperCase())  // var allowed
    .collect(Collectors.toList());
```

### Rules for Java 11

| Rule ID | Check |
|---------|-------|
| EJ-014 | Use String.isBlank(), repeat(), strip() |
| EJ-015 | Use Files.readString(), writeString() |
| EJ-013 | var in lambdas (explicit type alternative) |
```

---

## Java 12-13 Features (Preview)

### Switch Expressions (Preview in 12, Standard in 14)

```java
// Traditional switch
int numLetters;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        numLetters = 6;
        break;
    case TUESDAY:
        numLetters = 7;
        break;
    default:
        numLetters = -1;
}

// Java 12+ switch expression
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    default -> -1;
};
```

### Text Blocks (Preview in 13, Standard in 15)

```java
// Java 13+ text blocks
String json = """
    {
        "name": "John",
        "age": 30
    }
    """;

// With string formatting
String html = """
    <html>
        <body>
            <p>Hello, %s</p>
        </body>
    </html>
    """.formatted(name);
```

---

## Java 14-16 Features

### Records (Preview in 14-15, Standard in 16)

```java
// Java 16+ Records (immutable data carriers)
public record User(Long id, String name, String email) {
    // Auto-generates:
    // - Constructor
    // - equals(), hashCode(), toString()
    // - Accessors: id(), name(), email()
    
    // Compact constructor
    public User {
        if (name == null) {
            throw new IllegalArgumentException("Name cannot be null");
        }
        name = name.trim();
    }
    
    // Can add custom methods
    public String displayName() {
        return name.toUpperCase();
    }
}
```

### Pattern Matching for instanceof (Java 16+)

```java
// Before Java 16
if (obj instanceof String) {
    String s = (String) obj;  // Need explicit cast
    System.out.println(s.length());
}

// Java 16+ pattern matching
if (obj instanceof String s) {
    System.out.println(s.length());  // No cast needed!
}

// Can use in longer contexts
if (obj instanceof String s && s.length() > 5) {
    // s is in scope and length > 5
}
```

### Rules for Java 14-16

| Rule ID | Check |
|---------|-------|
| EJ-024 | Use pattern matching for instanceof instead of cast |
| EJ-025 | Use Records for simple data carriers |
| EJ-026 | Use switch expressions instead of switch statements |
| EJ-027 | Use text blocks for multiline strings |
```

---

## Java 17 Features (LTS)

### Sealed Classes

```java
// Java 17+ Sealed classes
public abstract sealed class Shape 
    permits Circle, Rectangle, Square {
    // Only Circle, Rectangle, Square can extend Shape
}

public final class Circle extends Shape { }
public sealed class Rectangle extends Shape { }
public non-sealed class Square extends Shape { }  // Can be extended
```

### Pattern Matching for switch (Preview in 17)

```java
// Java 17+ pattern matching in switch
String result = switch (obj) {
    case Integer i -> "Integer: " + i;
    case String s -> "String: " + s;
    case null -> "null";  // null handling
    default -> "Unknown";
};

// With guards
String result = switch (obj) {
    case Integer i when i > 0 -> "Positive: " + i;
    case Integer i -> "Non-positive: " + i;
    default -> "Unknown";
};
```

### Records (Now Standard)

Records are now standard in Java 17.

### Rules for Java 17

| Rule ID | Check |
|---------|-------|
| EJ-023 | Use sealed classes when inheritance needs control |
| EJ-024 | Use pattern matching for instanceof |
| EJ-025 | Use Records for DTOs/immutable data |
| EJ-026 | Use switch expressions |
| EJ-027 | Use text blocks |
```

---

## Java 19-21 Features

### Virtual Threads (Preview in 19-20, Standard in 21)

```java
// Java 21+ Virtual Threads
// Lightweight threads that reduce thread management overhead

// Creating virtual threads
Thread virtualThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running in virtual thread");
});

// Using Executors
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
executor.submit(() -> {
    // Task runs in virtual thread
});

// Don't change existing thread-per-task code, but virtual threads
// provide better throughput for I/O-bound workloads
```

### Sequenced Collections (Java 21)

```java
// Java 21+ Sequenced collections
// New interfaces: SequencedCollection, SequencedSet, SequencedMap

// getFirst(), getLast()
SequencedCollection<String> collection = new ArrayList<>(List.of("a", "b", "c"));
String first = collection.getFirst();  // "a"
String last = collection.getLast();   // "c"

// addFirst(), addLast()
collection.addFirst("z");
collection.addLast("z");

// reversed()
SequencedCollection<String> reversed = collection.reversed();
```

### Pattern Matching for switch (Standard in 21)

Pattern matching for switch is fully standardized in Java 21.

### Rules for Java 19-21

| Rule ID | Check |
|---------|-------|
| EJ-026 | Use switch expressions with pattern matching |
| EJ-030 | Use orElseGet() for lazy default in Optional |
| - | Consider virtual threads for I/O-bound tasks |
| Use SequencedCollection methods |
```

---

## Java 25 Features (LTS - September 2025)

Java 25 is the latest LTS release, featuring 18 JEPs including major concurrency improvements and language enhancements.

### Language Features

```java
// JEP 506: Scoped Values (FINAL)
// Alternative to ThreadLocal with better inheritance behavior
public class ScopedValuesExample {
    private static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();
    
    public static void main(String[] args) {
        // Run with scoped value
        ScopedValue.where(CURRENT_USER, new User("Alice"))
            .run(() -> {
                // User is available here
                System.out.println("Processing: " + CURRENT_USER.get().name());
                callNestedMethod();
            });
    }
    
    private static void callNestedMethod() {
        // Scoped values are inherited automatically
        System.out.println("In nested: " + CURRENT_USER.get().name());
    }
}

// JEP 507: Primitive Types in Patterns (Third Preview)
// Pattern matching with primitives
public class PrimitivePatterns {
    public static void main(String[] args) {
        Object obj = 42;
        
        // Primitive pattern in instanceof (Java 25+)
        if (obj instanceof int i) {
            System.out.println("Integer: " + i * 2);
        }
        
        // Switch with primitive patterns
        String result = switch (obj) {
            case int i -> "int: " + i;
            case long l -> "long: " + l;
            case double d -> "double: " + d;
            default -> "other";
        };
    }
}

// JEP 513: Flexible Constructor Bodies (FINAL)
// Run instance initializers before super()
public class FlexibleConstructor {
    private final int value;
    
    // Instance initializer runs before super() call
    {
        // Can validate/setup before parent constructor
        System.out.println("Instance initializer");
    }
    
    public FlexibleConstructor(int value) {
        super(); // Parent constructor runs after instance initializer
        this.value = value;
    }
}

// JEP 511: Module Import Declarations (Preview)
// Import all packages from a module
import module java.base; // Import all exported packages

public class ModuleImports {
    public static void main(String[] args) {
        // Can use any class from java.base without full imports
        List<String> list = new ArrayList<>();
        Map<String, Integer> map = new HashMap<>();
    }
}
```

### API Enhancements

```java
// JEP 502: Stable Values (Preview)
// Immutable value that can be updated atomically
public class StableValueExample {
    private static final StableValue<Cache> CACHE = StableValue.newInstance();
    
    public static void main(String[] args) {
        // Create initial cache
        Cache initialCache = new Cache();
        CACHE.set(initialCache);
        
        // Update atomically
        CACHE.update(cache -> cache.withNewEntry("key", "value"));
        
        // Access current value
        Cache current = CACHE.get();
    }
}

// JEP 510: Key Derivation Function API (FINAL)
import java.security.kdf.KeyDerivationFunction;

public class KDFExample {
    public static void main(String[] args) throws Exception {
        KeyDerivationFunction kdf = KeyDerivationFunction.getInstance("HKDF-SHA256");
        kdf.init(new KDFParameterSpec(secretKey, "info".getBytes(), 32));
        byte[] derivedKey = kdf.deriveKey();
    }
}

// JEP 470: PEM Encodings (Preview)
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

// Read PEM files natively
```

### Performance Features

```java
// JEP 521: Generational Shenandoah (FINAL)
// Low-latency garbage collector now generational
// Enable with: -XX:+UseGenerationalShenandoah

// JEP 519: Compact Object Headers (FINAL)
// Compressed object headers for better memory efficiency

// JEP 515: Ahead-of-Time Profiling (FINAL)
// Profile methods ahead of time for faster startup
// Compile with: jaotc --compile-for-all-modules
```

### Rules for Java 25

| Rule ID | Check |
|---------|-------|
| EJ-031 | Use ScopedValue instead of ThreadLocal for better inheritance |
| EJ-032 | Use primitive patterns in instanceof and switch |
| EJ-033 | Consider StableValue for immutable caching |
| EJ-034 | Use flexible constructor bodies |
| EJ-035 | Use module import declarations (preview) |

---

## Version Detection in Code

### Detecting Java Version at Runtime

```java
public class JavaVersion {
    public static void main(String[] args) {
        String version = System.getProperty("java.version");
        System.out.println("Java version: " + version);
        
        int major = Runtime.version().feature();
        System.out.println("Major version: " + major);
        
        // Use for version-specific code
        if (major >= 17) {
            // Java 17+ code
        } else if (major >= 11) {
            // Java 11-16 code
        }
    }
}
```

---

## Quick Version Check Reference

| Version | Can Use |
|---------|---------|
| **Java 8** | Lambdas, Streams, Optional, Date/Time API |
| **Java 9** | List.of(), Set.of(), Map.of(), Private interface methods |
| **Java 10** | var for local variables |
| **Java 11** | String.isBlank(), Files.readString(), HTTP Client |
| **Java 12-13** | Switch expressions (preview), Text blocks (preview) |
| **Java 14-15** | Switch expressions, Records (preview), Text blocks |
| **Java 16** | Records (standard), Pattern matching instanceof |
| **Java 17** | Sealed classes, Full pattern matching |
| **Java 18-20** | Pattern matching improvements, Virtual threads (preview) |
| **Java 21** | Virtual threads, Sequenced collections, Full switch pattern matching |
| **Java 25** | Scoped Values, Primitive Patterns, Stable Values, Flexible Constructors |

---

## Checklist by Version

### Java 8 Project

- [ ] Use lambdas and streams
- [ ] Use Optional for optional values
- [ ] Use Date/Time API (java.time)
- [ ] Use method references where possible
- [ ] Return empty collections, not nulls

### Java 11 Project

- [ ] All Java 8 items
- [ ] Use var for local variables
- [ ] Use String.isBlank(), repeat(), strip()
- [ ] Use Files.readString(), writeString()
- [ ] Use immutable collections: List.of(), Set.of(), Map.of()

### Java 17 Project

- [ ] All Java 11 items
- [ ] Use Records for DTOs
- [ ] Use Sealed classes for controlled inheritance
- [ ] Use Pattern matching for instanceof
- [ ] Use Switch expressions
- [ ] Use Text blocks for multiline strings

### Java 21 Project

- [ ] All Java 17 items
- [ ] Consider Virtual threads for I/O-bound tasks
- [ ] Use SequencedCollection methods: getFirst(), getLast()
- [ ] Use full pattern matching in switch

### Java 25 Project

- [ ] All Java 21 items
- [ ] Use ScopedValue instead of ThreadLocal where appropriate
- [ ] Use primitive patterns in instanceof and switch (JEP 507)
- [ ] Consider StableValue for immutable caching scenarios (JEP 502)
- [ ] Use flexible constructor bodies (JEP 513)
- [ ] Consider module import declarations (JEP 511 - preview)
- [ ] Enable Generational Shenandoah GC for low-latency requirements

