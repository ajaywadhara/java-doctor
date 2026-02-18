# Java Performance Antipatterns Reference

Common performance issues in Java applications with detection and fixes.

## Database Performance

### PERF-DB-001: N+1 Query Problem

**Severity:** CRITICAL

The most common performance issue in Java persistence.

```java
// ❌ N+1: 1 query for orders + N queries for customers
List<Order> orders = orderRepository.findAll();
for (Order order : orders) {
    String customerName = order.getCustomer().getName(); // N additional queries!
}

// ✅ Solution 1: JOIN FETCH
@Query("SELECT o FROM Order o JOIN FETCH o.customer")
List<Order> findAllWithCustomers();

// ✅ Solution 2: EntityGraph
@EntityGraph(attributePaths = {"customer"})
List<Order> findAll();

// ✅ Solution 3: Batch Fetching
@Entity
public class Order {
    @ManyToOne
    @JoinColumn(name = "customer_id")
    @BatchSize(size = 50)
    private Customer customer;
}

// ✅ Solution 4: Projections
public interface OrderWithCustomer {
    Long getId();
    String getCustomerName();
}

List<OrderWithCustomer> findAllWithCustomerName();
```

### PERF-DB-002: EAGER Fetching

**Severity:** ERROR

```java
// ❌ EAGER loads all orders AND all items for each order automatically
@Entity
public class Customer {
    @Id
    private Long id;
    
    @OneToMany(fetch = FetchType.EAGER) // BAD!
    private List<Order> orders;
}

// ✅ Use LAZY
@Entity
public class Customer {
    @Id
    private Long id;
    
    @OneToMany(fetch = FetchType.LAZY) // Default, but explicit is better
    @JoinFetch // EclipseLink
    private List<Order> orders;
}
```

### PERF-DB-003: Missing Database Index

**Severity:** WARNING

```java
// ❌ No index on frequently queried column
@Entity
@Table(name = "users")
public class User {
    @Id
    private Long id;
    
    @Column(unique = true)
    private String email; // Should have index but JPA may not create it
    
    @Column(name = "created_at")
    private LocalDateTime createdAt; // Querying by this often
}

// ✅ Explicit index
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_user_email", columnList = "email"),
    @Index(name = "idx_user_created", columnList = "created_at")
})
public class User {
    // ...
}
```

---

## Memory Performance

### PERF-MEM-001: String Concatenation in Loop

**Severity:** ERROR

```java
// ❌ O(n²) complexity - creates many StringBuilder instances
String result = "";
for (String item : items) {
    result += item + ",";
}

// ✅ StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < items.size(); i++) {
    sb.append(items.get(i));
    if (i < items.size() - 1) {
        sb.append(",");
    }
}
String result = sb.toString();

// ✅ Java 8+ joining
String result = String.join(",", items);

// ✅ Stream API
String result = items.stream()
    .collect(Collectors.joining(","));
```

### PERF-MEM-002: Creating Unnecessary Objects

**Severity:** WARNING

```java
// ❌ Compiling regex every time
public boolean isValidEmail(String email) {
    Pattern pattern = Pattern.compile("^[A-Za-z0-9+_.-]+@(.+)$");
    return pattern.matcher(email).matches();
}

// ✅ Compile once (static)
private static final Pattern EMAIL_PATTERN = 
    Pattern.compile("^[A-Za-z0-9+_.-]+@(.+)$");

public boolean isValidEmail(String email) {
    return EMAIL_PATTERN.matcher(email).matches();
}

// ❌ Creating new ArrayList for every call
public List<String> getNames() {
    return new ArrayList<>(nameList); // Unnecessary copy
}

// ✅ Return unmodifiable or empty
public List<String> getNames() {
    return Collections.unmodifiableList(nameList);
}
```

### PERF-MEM-003: Boxing/Unboxing in Loops

**Severity:** WARNING

```java
// ❌ Autoboxing on every iteration
List<Integer> numbers = getNumbers();
long sum = 0;
for (Integer num : numbers) {
    sum += num; // Autoboxes Integer to long!
}

// ✅ Use primitive streams
long sum = numbers.stream()
    .mapToLong(Integer::longValue)
    .sum();

// ✅ Or use for-each with primitives
for (int num : getPrimitiveArray()) {
    sum += num;
}

// ✅ Or use IntSummaryStatistics
IntSummaryStatistics stats = numbers.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();
```

---

## Collection Performance

### PERF-COL-001: Wrong Collection Type

**Severity:** WARNING

```java
// ❌ Using LinkedList for random access
List<String> list = new LinkedList<>();
for (int i = 0; i < 1000; i++) {
    list.add(i);
}
for (int i = 0; i < 1000; i++) {
    String s = list.get(i); // O(n) for each access!
}

// ✅ Use ArrayList for random access
List<String> list = new ArrayList<>();
// ...

// ❌ Using HashSet when order matters
Set<String> set = new HashSet<>();
// Need insertion order later...

// ✅ Use LinkedHashSet
Set<String> set = new LinkedHashSet<>();
```

### PERF-COL-002: ConcurrentHashMap for Single-Thread

**Severity:** SUGGESTION

```java
// ❌ Overhead of concurrent structures in single thread
Map<String, User> cache = new ConcurrentHashMap<>();

// ✅ Use HashMap in single thread
Map<String, User> cache = new HashMap<>();
```

---

## Threading Performance

### PERF-THR-001: Blocking Operations in Synchronized Block

**Severity:** WARNING

```java
// ❌ Holding lock during I/O
public synchronized void processOrder(Long orderId) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    
    // Database calls while holding lock!
    sendEmail(order.getCustomer().getEmail(), "Order processed");
    
    updateInventory(order.getItems());
}

// ✅ Use concurrent utilities
private final ExecutorService executor = Executors.newFixedThreadPool(10);

public void processOrder(Long orderId) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    
    executor.submit(() -> {
        sendEmail(order.getCustomer().getEmail(), "Order processed");
    });
    
    updateInventory(order.getItems());
}
```

### PERF-THR-002: Not Using Connection Pool

**Severity:** CRITICAL

```java
// ❌ Creating new connection for each request
public User findById(Long id) {
    Connection conn = DriverManager.getConnection(url, user, pass);
    try {
        // query
    } finally {
        conn.close();
    }
}

// ✅ Use connection pool (HikariCP)
@Configuration
public class DataSourceConfig {
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/db");
        config.setUsername("user");
        config.setPassword("pass");
        config.setMaximumPoolSize(10);
        return new HikariDataSource(config);
    }
}
```

---

## I/O Performance

### PERF-IO-001: Not Using Buffered I/O

**Severity:** WARNING

```java
// ❌ Unbuffered I/O
FileInputStream fis = new FileInputStream(file);
int data;
while ((data = fis.read()) != -1) {
    // process
}

// ✅ Buffered I/O
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file))) {
    int data;
    while ((data = bis.read()) != -1) {
        // process
    }
}

// ✅ Even better: Use Files utility (Java 7+)
String content = Files.readString(path);
List<String> lines = Files.readAllLines(path);
```

### PERF-IO-002: Reading Large Files into Memory

**Severity:** WARNING

```java
// ❌ Loading entire file into memory
String content = Files.readString(path);
List<String> lines = Files.readAllLines(path); // OutOfMemoryError for large files!

// ✅ Stream lines
try (Stream<String> lines = Files.lines(path)) {
    lines.filter(line -> line.contains("keyword"))
         .forEach(System.out::println);
}

// ✅ Use BufferedReader
try (BufferedReader reader = Files.newBufferedReader(path)) {
    reader.lines()
        .filter(line -> line.contains("keyword"))
        .forEach(System.out::println);
}
```

---

## Stream Performance

### PERF-STR-001: Expensive Operations in Streams

**Severity:** WARNING

```java
// ❌ Expensive operation in map (called for each element)
List<String> result = items.stream()
    .map(item -> expensiveOperation(item)) // Called every time!
    .collect(Collectors.toList());

// ✅ Filter first, then map
List<String> result = items.stream()
    .filter(Item::isActive)
    .map(this::processItem)
    .collect(Collectors.toList());

// ❌ Using peek() for side effects in production
List<String> result = items.stream()
    .peek(item -> log.debug("Processing: {}", item)) // Not recommended
    .map(this::transform)
    .collect(Collectors.toList());

// ✅ Use forEach for side effects after stream
items.stream()
    .map(this::transform)
    .forEach(result::add);
```

---

## Caching

### PERF-CACHE-001: Not Using Cache

**Severity:** WARNING

```java
// ❌ Database query for every call
public User getUserByEmail(String email) {
    return userRepository.findByEmail(email); // Query every time
}

// ✅ Use Spring Cache
@Service
public class UserService {
    @Cacheable(value = "users", key = "#email")
    public User getUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }
}

@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users");
    }
}

// ✅ Or use Caffeine
@Bean
public Cache<String, User> userCache() {
    return Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(Duration.ofMinutes(10))
        .build();
}
```

---

## Object Creation

### PERF-OBJ-001: Creating Objects with Wrong Scope

**Severity:** WARNING

```java
// ❌ Creating expensive objects in method called frequently
public class OrderService {
    public void processOrder(Order order) {
        ObjectMapper mapper = new ObjectMapper(); // New instance every time!
        String json = mapper.writeValueAsString(order);
        // ...
    }
}

// ✅ Reuse ObjectMapper
public class OrderService {
    private static final ObjectMapper MAPPER = new ObjectMapper();
    
    public void processOrder(Order order) {
        String json = MAPPER.writeValueAsString(order);
        // ...
    }
}
```

---

## Logging Performance

### PERF-LOG-001: String Concatenation in Logger

**Severity:** WARNING

```java
// ❌ String concatenation happens even if log level is off
log.debug("Processing order: " + order.getId() + " for user: " + user.getName());

// ✅ Use parameterized logging
log.debug("Processing order: {} for user: {}", order.getId(), user.getName());

// ✅ Check level first for expensive operations
if (log.isDebugEnabled()) {
    log.debug("Processing expensive data: {}", expensiveToString());
}
```

---

## Performance Checklist

1. [ ] Use JOIN FETCH to avoid N+1 queries
2. [ ] Use LAZY fetching, not EAGER
3. [ ] Add database indexes for frequently queried columns
4. [ ] Use StringBuilder or String.join() for concatenation
5. [ ] Compile regex patterns as static finals
6. [ ] Use primitive streams for numeric operations
7. [ ] Use appropriate collection types
8. [ ] Use connection pooling (HikariCP)
9. [ ] Use buffered I/O or Files utility
10. [ ] Implement caching for frequently accessed data
11. [ ] Use parameterized logging
12. [ ] Reuse expensive objects (ObjectMapper, Pattern, etc.)
