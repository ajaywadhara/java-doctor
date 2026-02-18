# Spring Best Practices Reference

Spring Framework-specific rules and best practices for Java Doctor.

## Spring Boot Configuration

### SPRING-001: Externalized Configuration

```java
// ❌ Hardcoded values
private static final String API_URL = "https://api.example.com";

// ✅ Use @Value
@Value("${api.url}")
private String apiUrl;

// ✅ Use @ConfigurationProperties (recommended for complex configs)
@Component
@ConfigurationProperties(prefix = "api")
@Data
public class ApiProperties {
    private String url;
    private String key;
    private int timeout;
    private List<String> allowedOrigins;
}

// application.yml
// api:
//   url: https://api.example.com
//   key: ${API_KEY}  # Environment variable
//   timeout: 5000
//   allowedOrigins:
//     - https://example.com
```

---

## Spring Data JPA

### SPRING-002: Transaction Management

```java
// ❌ Missing @Transactional on data modification
@Service
public class UserService {
    public void createUser(User user) {
        userRepository.save(user); // Needs transaction!
    }
}

// ✅ Add @Transactional
@Service
public class UserService {
    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
    }
    
    @Transactional(readOnly = true)
    public List<User> findAll() {
        return userRepository.findAll();
    }
}

// ✅ Specify rollback behavior
@Transactional(rollbackFor = {BusinessException.class})
public void processOrder(Order order) throws BusinessException {
    // ...
}
```

### SPRING-003: Entity Design

```java
// ❌ Exposing JPA entity via REST
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}

// ✅ Use DTO
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public UserDTO getUser(@PathVariable Long id) {
        return userService.getUserDTO(id);
    }
}

@Data
@Builder
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    private LocalDateTime createdAt;
}

// ✅ Or use MapStruct
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDTO toDTO(User user);
}
```

### SPRING-004: Repository Best Practices

```java
// ✅ Use Spring Data methods
public interface UserRepository extends JpaRepository<User, Long> {
    // Basic methods provided by JpaRepository
    // save(), findById(), findAll(), delete(), count(), etc.
    
    // Query methods
    Optional<User> findByEmail(String email);
    List<User> findByActiveTrue();
    List<User> findByAgeGreaterThan(int age);
    
    // Custom query
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmailAddress(@Param("email") String email);
    
    // Native query
    @Query(value = "SELECT * FROM users WHERE email = :email", 
           nativeQuery = true)
    Optional<User> findByEmailNative(@Param("email") String email);
    
    // JOIN FETCH for N+1
    @Query("SELECT u FROM User u LEFT JOIN FETCH u.roles WHERE u.id = :id")
    Optional<User> findByIdWithRoles(@Param("id") Long id);
}
```

---

## Spring Security

### SPRING-005: Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // CSRF
            .csrf(csrf -> csrf.disable()) // For APIs, enable for web
            
            // Authorization
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").authenticated()
                .anyRequest().authenticated()
            )
            
            // Session management
            .sessionManagement(session -> session
                .sessionFixation().migrateSession()
                .maximumSessions(1)
            )
            
            // Headers
            .headers(headers -> headers
                .frameOptions().deny()
                .xssProtection().and()
                .contentSecurityPolicy("default-src 'self'")
            )
            
            // Form login (if using)
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            
            // Logout
            .logout(logout -> logout
                .logoutUrl("/logout")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
            );
        
        return http.build();
    }
}
```

### SPRING-006: Method-Level Security

```java
@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        // Only admins can delete
    }
    
    @PreAuthorize("#username == authentication.principal.username or hasRole('ADMIN')")
    public User getUser(String username) {
        // User can get own profile or admin can get any
    }
    
    @Secured({"ROLE_USER", "ROLE_ADMIN"})
    public void updateProfile(User user) {
        // Either role allowed
    }
    
    @PostFilter("filterObject.owner == authentication.principal.username")
    public List<Post> getUserPosts() {
        // Filters results based on ownership
    }
}
```

---

## Spring Web

### SPRING-007: REST Controller Best Practices

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    
    private final UserService userService;
    private final UserMapper userMapper;
    
    @GetMapping
    public ResponseEntity<List<UserDTO>> getAllUsers(
            @PageableDefault(size = 20) Pageable pageable) {
        return ResponseEntity.ok(userService.findAll(pageable));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(
            @Valid @RequestBody UserRequest request) {
        UserDTO created = userService.create(request);
        return ResponseEntity
            .created(URI.create("/api/users/" + created.getId()))
            .body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserRequest request) {
        return userService.update(id, request)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

### SPRING-008: Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(NotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("VALIDATION_ERROR", ex.getMessage()));
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidationErrors(
            MethodArgumentNotValidException ex) {
        Map<String, String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage
            ));
        return ResponseEntity.badRequest().body(errors);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}
```

---

## Spring Testing

### SPRING-009: Test Configuration

```java
// ❌ Using deprecated
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceTest { }

// ✅ Using current
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;
    
    @Test
    void shouldCreateUser() { }
}

// ✅ Slice tests for faster testing
@DataJpaTest
class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindByEmail() { }
}

@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturnUser() throws Exception { }
}

@AutoConfigureMockMvc
class IntegrationTest { }
```

### SPRING-010: Test Isolation

```java
@SpringBootTest
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class UserServiceIntegrationTest {
    
    @Transactional  // Rolls back after each test
    @Test
    void shouldCreateUser() {
        // Test data is rolled back
    }
}
```

---

## Spring Best Practices Checklist

### Configuration
- [ ] Use @ConfigurationProperties over @Value for complex configs
- [ ] Externalize secrets to environment variables
- [ ] Use profile-specific properties (application-{profile}.yml)
- [ ] Enable validation with @Validated

### Data Access
- [ ] Use @Transactional appropriately
- [ ] Use DTOs, not entities in controllers
- [ ] Avoid N+1 with JOIN FETCH
- [ ] Use readOnly=true for read operations

### Security
- [ ] Disable CSRF for APIs
- [ ] Use method-level security (@PreAuthorize)
- [ ] Implement proper CORS configuration
- [ ] Add security headers

### Web
- [ ] Use proper HTTP status codes
- [ ] Use @Valid for input validation
- [ ] Implement global exception handling
- [ ] Add pagination for collection endpoints

### Testing
- [ ] Use @DataJpaTest for repository tests
- [ ] Use @WebMvcTest for controller tests
- [ ] Use @Transactional for test isolation
- [ ] Use @MockBean to mock dependencies
