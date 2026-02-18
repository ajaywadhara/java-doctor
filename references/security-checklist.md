# Java Security Checklist Reference

Comprehensive security checks for Java applications. Based on OWASP Top 10 and Java-specific vulnerabilities.

## OWASP Top 10 - Java Mapping

### A01:2021 - Broken Access Control

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A01-001 | Missing authorization checks | `@GetMapping("/admin")` without `@PreAuthorize` | Add role-based access control |
| SEC-A01-002 | Insecure direct object reference (IDOR) | `@PathVariable Long id` directly to repository | Validate ownership, use UUIDs |
| SEC-A01-003 | Missing method-level security | Public endpoint that should be admin-only | Add method security |
| SEC-A01-004 | Path traversal | File download using user input directly | Validate and sanitize paths |

**Detection Patterns:**
```java
// ❌ Missing authorization
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).get(); // Anyone can access!
}

// ✅ With Spring Security
@GetMapping("/users/{id}")
@PreAuthorize("@userService.isOwner(#id)")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();
}
```

### A02:2021 - Cryptographic Failures

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A02-001 | Hardcoded encryption keys | `new SecretKeySpec("12345678".getBytes(), "AES")` | Use KeyStore or config |
| SEC-A02-002 | Weak algorithms | `Cipher.getInstance("DES")` | Use AES-256, RSA-2048+ |
| SEC-A02-003 | Hardcoded passwords | `password = "admin123"` | Use environment variables |
| SEC-A02-004 | Missing SSL/TLS | `HttpURLConnection` without SSL | Use HTTPS, configure SSL |

**Detection Patterns:**
```java
// ❌ Hardcoded key
private static final String KEY = "mySecretKey12345";

// ✅ Use configuration
@Value("${encryption.key}")
private String encryptionKey;
```

### A03:2021 - Injection

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A03-001 | SQL Injection | String concatenation in JPQL | Use parameterized queries |
| SEC-A03-002 | NoSQL Injection | Unsanitized input to MongoDB | Validate and sanitize |
| SEC-A03-003 | LDAP Injection | Unsanitized LDAP queries | Escape special characters |
| SEC-A03-004 | Command Injection | `Runtime.exec()` with user input | Avoid, use APIs |
| SEC-A03-005 | XPath Injection | String concatenation in XPath | Use parameterized XPath |

**Detection Patterns:**
```java
// ❌ SQL Injection
@Query("SELECT u FROM User u WHERE u.name = '" + name + "'")
List<User> findByName(String name);

// ✅ Parameterized
@Query("SELECT u FROM User u WHERE u.name = :name")
List<User> findByName(@Param("name") String name);

// ✅ Use Spring Data
List<User> findByName(String name);
```

### A04:2021 - Insecure Design

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A04-001 | Missing rate limiting | No throttling on login | Add rate limiting |
| SEC-A04-002 | Missing business limits | No max transaction amount | Add business rules |
| SEC-A04-003 | Insufficient session management | Long-lived sessions | Expire sessions, regenerate IDs |

### A05:2021 - Security Misconfiguration

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A05-001 | Debug mode enabled in production | `server.error.include-stacktrace=always` | Disable in production |
| SEC-A05-002 | Default credentials | Default admin/admin | Change defaults |
| SEC-A05-003 | Missing security headers | No X-Frame-Options | Add security headers |
| SEC-A05-004 | CORS allowing all origins | `allowedOrigins("*")` | Specific origins only |
| SEC-A05-005 | Verbose error messages | Stack traces in responses | Sanitize errors |

**Detection Patterns:**
```java
// ❌ Permissive CORS
@CrossOrigin(origins = "*") // Allows all!

// ✅ Specific origins
@CrossOrigin(origins = {"https://example.com", "https://app.example.com"})
```

### A06:2021 - Vulnerable and Outdated Components

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A06-001 | Known vulnerable libraries | Using commons-collections < 3.2.2 | Update dependencies |
| SEC-A06-002 | Outdated Spring version | Using Spring 4.x | Upgrade to latest |
| SEC-A06-003 | Snapshot dependencies | `1.0-SNAPSHOT` | Use stable versions |

### A07:2021 - Identification and Authentication Failures

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A07-001 | Weak password policy | No validation | Require complexity |
| SEC-A07-002 | Credential storage | Storing plain passwords | Use BCrypt/Argon2 |
| SEC-A07-003 | Session fixation | Not regenerating session ID on login | Regenerate on login |
| SEC-A07-004 | Missing MFA | Single-factor auth only | Add MFA support |

**Detection Patterns:**
```java
// ❌ Plain text password storage
public void saveUser(User user) {
    user.setPassword(user.getPassword()); // Plain text!
    userRepository.save(user);
}

// ✅ BCrypt hashing
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

public void saveUser(User user) {
    user.setPassword(passwordEncoder.encode(user.getPassword()));
    userRepository.save(user);
}
```

### A08:2021 - Software and Data Integrity Failures

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A08-001 | Deserialization vulnerabilities | `ObjectInputStream` without filter | Use whitelisting |
| SEC-A08-002 | Untrusted deserialization | Jackson with default typing | Disable default typing |
| SEC-A08-003 | Using vulnerable signatures | `SignedObject` without verification | Verify signatures |

**Detection Patterns:**
```java
// ❌ Unsafe deserialization
ObjectInputStream ois = new ObjectInputStream(inputStream);
Object obj = ois.readObject(); // Dangerous!

// ✅ With filter (Java 9+)
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "com.example.*;java.base/*;!*");
ObjectInputStream ois = new ObjectInputStream(inputStream);
ois.setObjectInputFilter(filter);
Object obj = ois.readObject();

// ❌ Jackson default typing
objectMapper.enableDefaultTyping();

// ✅ Disable default typing
objectMapper.disableDefaultTyping();
```

### A09:2021 - Security Logging and Monitoring Failures

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A09-001 | Not logging security events | No login attempt logging | Log auth events |
| SEC-A09-002 | Sensitive data in logs | `log.info("Password: " + pwd)` | Mask sensitive data |
| SEC-A09-003 | No monitoring for attacks | No alerting on multiple failed logins | Add monitoring |
| SEC-A09-004 | Insufficient log detail | Generic error messages | Include context |

**Detection Patterns:**
```java
// ❌ Logging sensitive data
log.info("User login: {} with password: {}", username, password);

// ✅ Mask sensitive data
log.info("User login attempt for: {}", username);
// Or use a mask utility
log.info("Card number: {}", maskCardNumber(cardNumber)); // ****1234
```

### A10:2021 - Server-Side Request Forgery (SSRF)

| Rule ID | Issue | Detection | Fix |
|---------|-------|-----------|-----|
| SEC-A10-001 | URL from user input | `new URL(userInput)` | Validate URL allowlist |
| SEC-A10-002 | Accessing internal services | No network restrictions | Segment network |

**Detection Patterns:**
```java
// ❌ SSRF vulnerability
@GetMapping("/fetch")
public String fetch(@RequestParam String url) {
    return restTemplate.getForObject(url, String.class); // Dangerous!
}

// ✅ With validation
@GetMapping("/fetch")
public String fetch(@RequestParam String url) {
    URL validatedUrl = new URL(url);
    if (!allowedHosts.contains(validatedUrl.getHost())) {
        throw new SecurityException("Host not allowed");
    }
    return restTemplate.getForObject(url, String.class);
}
```

---

## Java-Specific Security Rules

### Spring Security Rules

```java
// ✅ Proper Spring Security configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()
                .antMatchers("/api/public/**").permitAll()
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .sessionManagement()
                .sessionFixation().migrateSession()
            .and()
            .headers()
                .frameOptions().deny()
                .xss().and()
            .and()
            .build();
    }
}
```

### Input Validation

```java
// ✅ Validate all inputs
public class UserValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return UserRequest.class.equals(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        UserRequest request = (UserRequest) target;
        
        // Email validation
        if (!EmailValidator.getInstance().isValid(request.getEmail())) {
            errors.rejectValue("email", "invalid.email");
        }
        
        // Password complexity
        if (!isValidPassword(request.getPassword())) {
            errors.rejectValue("password", "weak.password");
        }
    }
    
    private boolean isValidPassword(String password) {
        return password != null 
            && password.length() >= 8
            && password.matches(".*[A-Z].*")
            && password.matches(".*[a-z].*")
            && password.matches(".*[0-9].*");
    }
}
```

### XXE Prevention

```java
// ❌ XXE vulnerable
@PostMapping("/parse")
public String parseXml(@RequestBody String xml) throws Exception {
    DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
    DocumentBuilder db = dbf.newDocumentBuilder();
    Document doc = db.parse(new InputSource(new StringReader(xml)));
    return doc.getDocumentElement().getTextContent();
}

// ✅ XXE protected
@PostMapping("/parse")
public String parseXml(@RequestBody String xml) throws Exception {
    DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
    dbf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
    dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
    
    DocumentBuilder db = dbf.newDocumentBuilder();
    Document doc = db.parse(new InputSource(new StringReader(xml)));
    return doc.getDocumentElement().getTextContent();
}
```

---

## Hardcoded Secrets Detection

Common patterns to detect:

```java
// ❌ Hardcoded secrets (scan for these patterns)
private static final String API_KEY = "sk-1234567890";
private static final String PASSWORD = "admin123";
private static final String SECRET = "mySecretKey";
String token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
String connectionString = "jdbc:mysql://root:root@localhost/db";

// ✅ Should be externalized
@Value("${api.key}")
private String apiKey;

@Value("${database.password}")
private String password;

// ✅ Or use secrets manager
@Value("${aws.secretsmanager.secret}")
private String secret;
```

---

## Secure Coding Guidelines

1. **Validate input** - All user input must be validated
2. **Encode output** - Encode data for the output context
3. **Use parameterized queries** - Never concatenate strings in queries
4. **Use prepared statements** - For SQL, use PreparedStatement
5. **Hash passwords** - Use BCrypt, Argon2, or PBKDF2
6. **Use SSL/TLS** - All communication should be encrypted
7. **Log securely** - Never log sensitive information
8. **Use secure random** - Use SecureRandom, not Random
9. **Minimize exposure** - Limit what you expose
10. **Keep dependencies updated** - Regularly update libraries
