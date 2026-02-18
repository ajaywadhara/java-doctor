# Java Bug Patterns Reference

Comprehensive list of common Java bugs with code examples and fixes.

## Null Safety Bugs

### NULL-001: Optional.get() Without Check

**Severity:** CRITICAL

```java
// ❌ CRITICAL: NoSuchElementException risk
User user = userRepository.findById(id).get();

// ✅ FIX: Use orElseThrow()
User user = userRepository.findById(id)
    .orElseThrow(() -> new UserNotFoundException("User not found: " + id));

// ✅ FIX: Use orElse() for default
User user = userRepository.findById(id)
    .orElse(User.GUEST_USER);

// ✅ FIX: Use orElseGet() for lazy default
User user = userRepository.findById(id)
    .orElseGet(() -> createDefaultUser());
```

### NULL-002: Null Check on Primitive Wrapper

**Severity:** ERROR

```java
// ❌ ERROR: Causes NullPointerException
Integer count = getCount();
if (count == null) { // Autoboxing happens here!
    count = 0;
}

// ✅ FIX: Check before autoboxing
Integer count = getCount();
if (count != null) {
    // Use count safely
}

// ✅ FIX: Use OptionalInt for primitives
OptionalInt count = getCountAsOptionalInt();
count.orElse(0);
```

### NULL-003: Returning Null Instead of Empty Collection

**Severity:** WARNING

```java
// ❌ WARNING: Forces null checks everywhere
public List<Order> findOrdersByUser(Long userId) {
    List<Order> orders = orderRepository.findByUserId(userId);
    if (orders.isEmpty()) {
        return null; // BAD!
    }
    return orders;
}

// ✅ FIX: Return empty collection
public List<Order> findOrdersByUser(Long userId) {
    return orderRepository.findByUserId(userId); // JPA returns empty list
}

// ✅ FIX: If filtering
public List<Order> findActiveOrders(Long userId) {
    return orderRepository.findByUserId(userId).stream()
        .filter(Order::isActive)
        .collect(Collectors.toList()); // Never returns null
}
```

### NULL-004: Null Parameter Without Validation

**Severity:** ERROR

```java
// ❌ ERROR: NPE when name is null
public void createUser(String name, String email) {
    User user = new User();
    user.setName(name.toUpperCase()); // NPE!
    user.setEmail(email.toLowerCase());
}

// ✅ FIX: Add null validation
public void createUser(String name, String email) {
    Objects.requireNonNull(name, "Name cannot be null");
    Objects.requireNonNull(email, "Email cannot be null");
    
    User user = new User();
    user.setName(name.toUpperCase());
    user.setEmail(email.toLowerCase());
}

// ✅ FIX: Use @NonNull annotation (with null checker)
public void createUser(@NonNull String name, @NonNull String email) {
    // If using Spring, @NonNull is enforced
}
```

### NULL-005: Chained Method Calls Without Null Guards

**Severity:** ERROR

```java
// ❌ ERROR: NPE at any step
String city = user.getAddress().getCity().toUpperCase();

// ✅ FIX: Use Optional chaining
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");

// ✅ FIX: Use null-safe accessor (Lombok @NonNull on fields)
@Builder
class User {
    @NonNull private Address address;
}
```

---

## Exception Handling Bugs

### EXC-001: Swallowed Exceptions

**Severity:** ERROR

```java
// ❌ ERROR: Exception silently swallowed
try {
    processPayment(order);
} catch (PaymentException e) {
    // Do nothing!
}

// ✅ FIX: Log and handle
try {
    processPayment(order);
} catch (PaymentException e) {
    log.error("Payment failed for order: {}", order.getId(), e);
    throw new OrderProcessingException("Payment failed", e);
}

// ✅ FIX: Recover gracefully
try {
    processPayment(order);
} catch (PaymentException e) {
    log.warn("Payment failed, using fallback for order: {}", order.getId());
    processWithFallback(order);
}
```

### EXC-002: Catching Generic Exception

**Severity:** WARNING

```java
// ❌ WARNING: Too broad
try {
    parseUserInput(input);
} catch (Exception e) {
    log.error("Error", e);
}

// ✅ FIX: Catch specific exceptions
try {
    parseUserInput(input);
} catch (NumberFormatException e) {
    throw new ValidationException("Invalid number format", e);
} catch (IllegalArgumentException e) {
    throw new ValidationException("Invalid input", e);
}
```

### EXC-003: Using Generic RuntimeException

**Severity:** WARNING

```java
// ❌ WARNING: Loses context
throw new RuntimeException("Something went wrong");

// ✅ FIX: Use specific exception types
throw new UserNotFoundException("User not found: " + userId);
throw new InvalidPaymentException("Payment declined: " + reason);
throw new ConfigurationException("Missing config: " + propertyName);
```

### EXC-004: Not Using Try-With-Resources

**Severity:** ERROR

```java
// ❌ ERROR: Resource leak
FileInputStream fis = new FileInputStream(file);
try {
    data = fis.read();
} finally {
    fis.close(); // Exception in close() may mask original exception
}

// ✅ FIX: Try-with-resources
try (FileInputStream fis = new FileInputStream(file)) {
    data = fis.read();
}
// Automatically closed, exceptions properly chained

// ✅ FIX: Java 9+ can use effectively final variable
FileInputStream fis = new FileInputStream(file);
try (fis) {
    data = fis.read();
}
```

### EXC-005: Finally Block Returning or Throwing

**Severity:** CRITICAL

```java
// ❌ CRITICAL: Suppresses exception
public User getUser(Long id) {
    try {
        return userRepository.findById(id).orElseThrow();
    } finally {
        return null; // Ignores exception!
    }
}

// ❌ CRITICAL: Throwing in finally masks exception
try {
    process();
} finally {
    throw new RuntimeException("Cleanup failed"); // Masks original!
}

// ✅ FIX: Never return/throw in finally
public User getUser(Long id) {
    try {
        return userRepository.findById(id).orElseThrow();
    } finally {
        cleanup(); // Just cleanup, no return/throw
    }
}
```

---

## Performance Bugs

### PERF-001: N+1 Query Problem

**Severity:** CRITICAL

```java
// ❌ CRITICAL: N+1 queries
List<Order> orders = orderRepository.findAll();
for (Order order : orders) {
    // Each iteration makes a query!
    System.out.println(order.getCustomer().getName());
}

// ✅ FIX: Use JOIN FETCH
@Query("SELECT o FROM Order o JOIN FETCH o.customer")
List<Order> findAllWithCustomers();

// ✅ FIX: Use EntityGraph
@EntityGraph(attributePaths = {"customer", "items"})
List<Order> findAll();

// ✅ FIX: Use Batch Fetching
@BatchSize(size = 100)
@OneToMany(mappedBy = "order")
private List<OrderItem> items;
```

### PERF-002: String Concatenation in Loops

**Severity:** ERROR

```java
// ❌ ERROR: Creates many StringBuilder instances
String result = "";
for (String word : words) {
    result += word + ","; // O(n²) complexity!
}

// ✅ FIX: Use StringBuilder
StringBuilder sb = new StringBuilder();
for (String word : words) {
    sb.append(word).append(",");
}
String result = sb.toString();

// ✅ FIX: Use Java 8+ String.join()
String result = String.join(",", words);

// ✅ FIX: Use Collectors.joining()
String result = words.stream()
    .collect(Collectors.joining(","));
```

### PERF-003: Boxing/Unboxing in Loops

**Severity:** WARNING

```java
// ❌ WARNING: Autoboxing on every iteration
List<Integer> numbers = getNumbers();
long sum = 0;
for (Integer num : numbers) {
    sum += num; // Autoboxes Integer to long!
}

// ✅ FIX: Use primitive stream
List<Integer> numbers = getNumbers();
long sum = numbers.stream()
    .mapToLong(Integer::longValue)
    .sum();

// ✅ FIX: Use IntSummaryStatistics
IntSummaryStatistics stats = numbers.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();
```

### PERF-004: Creating Unnecessary Objects

**Severity:** WARNING

```java
// ❌ WARNING: New String created each time
public void process(String name) {
    String s = new String(name); // Unnecessary!
    // ...
}

// ❌ WARNING: Compiling regex each time
public boolean isValid(String input) {
    Pattern pattern = Pattern.compile("[A-Z]+");
    return pattern.matcher(input).matches();
}

// ✅ FIX: Use String directly
public void process(String name) {
    // Use name directly
}

// ✅ FIX: Compile regex once (static)
private static final Pattern VALID_PATTERN = Pattern.compile("[A-Z]+");

public boolean isValid(String input) {
    return VALID_PATTERN.matcher(input).matches();
}
```

### PERF-005: HashMap in Concurrent Environment

**Severity:** ERROR

```java
// ❌ ERROR: Race conditions, possible data corruption
private final Map<String, User> users = new HashMap<>();

public void addUser(User user) {
    users.put(user.getId(), user); // Not thread-safe!
}

// ✅ FIX: Use ConcurrentHashMap
private final Map<String, User> users = new ConcurrentHashMap<>();

public void addUser(User user) {
    users.put(user.getId(), user); // Thread-safe
}

// ✅ FIX: Use ConcurrentHashMap compute methods
users.compute(user.getId(), (key, existing) -> {
    if (existing != null) {
        throw new DuplicateUserException(key);
    }
    return user;
});
```

---

## Concurrency Bugs

### CONC-001: Shared Mutable State

**Severity:** CRITICAL

```java
// ❌ CRITICAL: Race condition
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++; // Not atomic!
    }
    
    public int getCount() {
        return count;
    }
}

// ✅ FIX: Use AtomicInteger
public class Counter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}

// ✅ FIX: Use synchronized
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

### CONC-002: Double-Checked Locking (Pre-Java 5)

**Severity:** CRITICAL

```java
// ❌ BROKEN (pre-Java 5): May return uninitialized object
private static Something instance;

public static Something getInstance() {
    if (instance == null) {
        synchronized (Something.class) {
            if (instance == null) {
                instance = new Something(); // May reorder with constructor!
            }
        }
    }
    return instance;
}

// ✅ FIX: Use volatile (Java 5+)
private static volatile Something instance;

public static Something getInstance() {
    if (instance == null) {
        synchronized (Something.class) {
            if (instance == null) {
                instance = new Something();
            }
        }
    }
    return instance;
}

// ✅ FIX: Bill Pugh Singleton (Initialization-on-demand)
private static class SomethingHolder {
    private static final Something INSTANCE = new Something();
}

public static Something getInstance() {
    return SomethingHolder.INSTANCE;
}

// ✅ FIX: Enum Singleton
public enum Something {
    INSTANCE;
}
```

### CONC-003: Starting Thread in Constructor

**Severity:** CRITICAL

```java
// ❌ CRITICAL: This reference escapes before object is fully constructed
public class MyService {
    public MyService() {
        new Thread(() -> {
            // May use partially constructed object!
            doSomething();
        }).start();
    }
}

// ✅ FIX: Use lazy initialization
public class MyService {
    private Thread initThread;
    
    public MyService() {
        // Don't start thread here
    }
    
    @PostConstruct
    public void init() {
        initThread = new Thread(this::doSomething);
        initThread.start();
    }
}
```

### CONC-004: Not Shutting Down ExecutorService

**Severity:** ERROR

```java
// ❌ ERROR: Resource leak - threads may never terminate
public class TaskExecutor {
    private final ExecutorService executor = Executors.newFixedThreadPool(10);
    
    public void submit(Runnable task) {
        executor.submit(task);
    }
}

// ✅ FIX: Add shutdown hook
public class TaskExecutor implements Closeable {
    private final ExecutorService executor = Executors.newFixedThreadPool(10);
    
    @Override
    public void close() {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}

// ✅ Use in try-with-resources
try (TaskExecutor executor = new TaskExecutor()) {
    executor.submit(task);
}
```

---

## Spring Framework Bugs

### SPR-001: @Transactional on Private Method

**Severity:** CRITICAL

```java
// ❌ CRITICAL: Transaction won't work - Spring AOP can't proxy private methods
@Service
public class UserService {
    
    @Transactional
    private void saveUser(User user) {
        userRepository.save(user);
    }
}

// ✅ FIX: Make public
@Transactional
public void saveUser(User user) {
    userRepository.save(user);
}

// ✅ FIX: Self-injection for internal calls
@Service
public class UserService {
    
    @Autowired
    private UserService self; // Self-injection
    
    public void saveUserWrapper(User user) {
        self.saveUser(user); // Calls through proxy
    }
    
    @Transactional
    public void saveUser(User user) {
        userRepository.save(user);
    }
}
```

### SPR-002: Returning JPA Entity

**Severity:** WARNING

```java
// ❌ WARNING: Exposes internal structure, lazy loading issues
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userRepository.findById(id).orElseThrow(); // Returns JPA entity!
    }
}

// ✅ FIX: Use DTO
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    public UserDTO getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}

@Data
public class UserDTO {
    private Long id;
    private String name;
    private String email;
}
```

### SPR-003: @Value on Static Field

**Severity:** CRITICAL

```java
// ❌ CRITICAL: Value won't be injected - field is set before Spring processes it
@Component
public class Config {
    @Value("${api.key}")
    private static String API_KEY; // Won't work!
}

// ✅ FIX: Use setter injection with @PostConstruct
@Component
public class Config {
    private static String API_KEY;
    
    @Value("${api.key}")
    public void setApiKey(String apiKey) {
        Config.API_KEY = apiKey;
    }
}

// ✅ FIX: Use @ConfigurationProperties
@Component
@ConfigurationProperties(prefix = "api")
@Data
public class ApiConfig {
    private String key;
    
    public static String getKey() {
        return API_KEY;
    }
}
```

---

## API Design Bugs

### API-001: Using Verbs in REST URLs

**Severity:** WARNING

```java
// ❌ WARNING: Using verbs
@GetMapping("/getUsers")
@GetMapping("/users/getAll")
@PostMapping("/createUser")
@PostMapping("/users/saveUser")
@DeleteMapping("/deleteUser")

// ✅ FIX: Use nouns, HTTP methods
@GetMapping("/users")           // Get all users
@GetMapping("/users/{id}")       // Get specific user
@PostMapping("/users")           // Create user
@PutMapping("/users/{id}")       // Update user
@DeleteMapping("/users/{id}")   // Delete user
```

### API-002: Not Using Proper HTTP Status Codes

**Severity:** WARNING

```java
// ❌ WARNING: Always returning 200
@PostMapping("/users")
public ResponseEntity<?> createUser(@RequestBody UserRequest request) {
    User user = userService.create(request);
    return ResponseEntity.ok(user); // Should be 201 for created!
}

// ✅ FIX: Return proper status codes
@PostMapping("/users")
public ResponseEntity<UserDTO> createUser(@RequestBody @Valid UserRequest request) {
    User user = userService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(toDTO(user));
}

@GetMapping("/users/{id}")
public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}
```

---

## Best Practice Bugs

### BP-001: Magic Numbers/Strings

**Severity:** WARNING

```java
// ❌ WARNING: Magic numbers
if (user.getAge() < 18) {
    throw new ValidationException("Too young");
}
if (order.getStatus() == 2) { // What is 2?
    processOrder(order);
}

// ✅ FIX: Use constants or enums
public class OrderStatus {
    public static final int PENDING = 1;
    public static final int PROCESSING = 2;
    public static final int COMPLETED = 3;
}

// ✅ Better: Use enums
public enum OrderStatus {
    PENDING, PROCESSING, COMPLETED
}

if (order.getStatus() == OrderStatus.PROCESSING) {
    processOrder(order);
}
```

### BP-002: Using == for Object Comparison

**Severity:** WARNING

```java
// ❌ WARNING: Comparing object identity, not equality
String status = getStatus();
if (status == "ACTIVE") { // May fail due to new String objects
    // ...
}

// ✅ FIX: Use equals()
if ("ACTIVE".equals(status)) { // Safe - literal first
    // ...
}

// ✅ FIX: Use Objects.equals() for null safety
if (Objects.equals(status, "ACTIVE")) {
    // ...
}
```

### BP-003: Not Using @Data/@Value Appropriately

**Severity:** WARNING

```java
// ❌ WARNING: @Data on JPA entity (equals/hashCode issues)
@Entity
@Data // BAD for JPA entities!
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}

// ✅ FIX: Use @Entity, @Getter @Setter for entities
@Entity
@Getter
@Setter
@NoArgsConstructor
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    @OneToMany(mappedBy = "user")
    @EqualsAndHashCode.Exclude // Exclude to prevent issues
    private List<Order> orders;
}

// ✅ FIX: Use @Value for DTOs (immutable)
@Value
public class UserDTO {
    Long id;
    String name;
    String email;
}
```

---

This reference document covers the most common Java bugs. Use this alongside the main SKILL.md to identify and fix issues in Java codebases.
