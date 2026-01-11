# POJO & Lombok Framework: Complete Guide for All Levels

## What is POJO vs Lombok?

### **POJO (Plain Old Java Object)**
A regular Java class with private fields, getters, setters, and constructors - **no framework dependencies**.

### **Lombok Framework** 
A **code generation library** that automatically creates boilerplate code (getters, setters, constructors, etc.) at compile time using annotations.

---

## Core Lombok Annotations - Complete Reference

### **@Data - "Complete POJO Generator"**

**What it is**: Combines @Getter, @Setter, @ToString, @EqualsAndHashCode, and @RequiredArgsConstructor

**Why use it**: Eliminates 90% of boilerplate code for data classes

**When to use it**: Data transfer objects, model classes, configuration classes

**How to define it**:
```java
// Without Lombok (50+ lines)
public class User {
    private String name;
    private String email;
    private int age;
    
    public User() {}
    public User(String name, String email, int age) { /* constructor */ }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    // ... 40+ more lines of getters, setters, equals, hashCode, toString
}

// With Lombok (3 lines!)
@Data
public class User {
    private String name;
    private String email;
    private int age;
}
```

**What @Data generates**:
- All getters: `getName()`, `getEmail()`, `getAge()`
- All setters: `setName()`, `setEmail()`, `setAge()`
- `equals()` and `hashCode()` using all fields
- `toString()` showing all field values
- Constructor for final/required fields

---

### **@Builder - "Fluent Object Construction"**

**What it is**: Generates builder pattern for creating objects with optional parameters

**Why use it**: Readable object creation, handles optional parameters, immutable objects

**When to use it**: Objects with many parameters, configuration classes, complex object creation

**How to define it**:
```java
@Data
@Builder
public class DatabaseConfig {
    private String host;
    private int port;
    private String database;
    private String username;
    
    @Builder.Default
    private int timeout = 30000;  // Default value
    
    @Builder.Default
    private boolean ssl = false;
}

// Usage - Fluent and readable
DatabaseConfig config = DatabaseConfig.builder()
    .host("localhost")
    .port(5432)
    .database("myapp")
    .username("admin")
    .ssl(true)
    // timeout uses default value
    .build();
```

**Advanced Builder Features**:
```java
@Builder
public class ApiRequest {
    private String endpoint;
    private Map<String, String> headers;
    private String body;
    
    // Custom builder method
    public static class ApiRequestBuilder {
        public ApiRequestBuilder addHeader(String key, String value) {
            if (this.headers == null) {
                this.headers = new HashMap<>();
            }
            this.headers.put(key, value);
            return this;
        }
    }
}

// Usage with custom method
ApiRequest request = ApiRequest.builder()
    .endpoint("/api/users")
    .addHeader("Authorization", "Bearer token")
    .addHeader("Content-Type", "application/json")
    .body("{\"name\":\"John\"}")
    .build();
```

---

### **@Getter / @Setter - "Selective Access Control"**

**What it is**: Generates only getter or setter methods for fine-grained control

**Why use it**: Control which methods to generate, create read-only/write-only properties

**When to use it**: When @Data is too much, specific access requirements

**How to define it**:
```java
public class BankAccount {
    @Getter @Setter
    private String accountNumber;  // Read-write
    
    @Getter
    private double balance;  // Read-only
    
    @Setter
    private String pin;  // Write-only
    
    // Manual methods for controlled access
    public void deposit(double amount) {
        this.balance += amount;
    }
    
    public boolean withdraw(double amount) {
        if (balance >= amount) {
            this.balance -= amount;
            return true;
        }
        return false;
    }
}

// Class-level application
@Getter  // All fields get getters
@Setter  // All non-final fields get setters
public class Settings {
    private String theme;
    private boolean darkMode;
    private final String version = "1.0";  // No setter (final)
}
```

**Access Level Control**:
```java
public class SecureData {
    @Getter(AccessLevel.PUBLIC)
    private String publicInfo;
    
    @Getter(AccessLevel.PROTECTED)
    private String protectedInfo;
    
    @Getter(AccessLevel.PACKAGE)
    private String packageInfo;
    
    @Getter(AccessLevel.PRIVATE)
    private String privateInfo;  // Only accessible within class
}
```

---

### **@ToString - "String Representation Control"**

**What it is**: Generates toString() method with customizable output

**Why use it**: Debugging, logging, consistent formatting

**When to use it**: Need toString but not full @Data, custom formatting requirements

**How to define it**:
```java
@ToString
public class Product {
    private String name;
    private double price;
    private String category;
}
// Output: Product(name=Laptop, price=999.99, category=Electronics)

@ToString(exclude = {"password", "secretKey"})
public class User {
    private String username;
    private String email;
    private String password;     // Excluded from toString
    private String secretKey;    // Excluded from toString
}
// Output: User(username=john, email=john@example.com)

@ToString(includeFieldNames = false)
public class Point {
    private int x;
    private int y;
}
// Output: Point(10, 20) instead of Point(x=10, y=20)

@ToString(callSuper = true)
public class Employee extends Person {
    private String employeeId;
    private String department;
}
// Output: Employee(super=Person(name=John, age=30), employeeId=EMP001, department=IT)
```

---

### **@EqualsAndHashCode - "Object Comparison Control"**

**What it is**: Generates equals() and hashCode() methods with field customization

**Why use it**: Proper object comparison, HashMap/HashSet usage, custom equality logic

**When to use it**: Objects in collections, need equals/hashCode but not full @Data

**How to define it**:
```java
@EqualsAndHashCode
public class User {
    private String name;
    private String email;
    private int age;
}
// Uses all fields for equality

@EqualsAndHashCode(exclude = {"lastLoginTime", "sessionId"})
public class UserSession {
    private String userId;
    private String username;
    private Date lastLoginTime;  // Ignored in equality
    private String sessionId;    // Ignored in equality
}

@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Product {
    @EqualsAndHashCode.Include
    private String productId;    // Only this field used for equality
    
    private String name;         // Ignored
    private double price;        // Ignored
}

@EqualsAndHashCode(callSuper = true)
public class Employee extends Person {
    private String employeeId;   // Includes parent class fields too
}
```

---

### **@AllArgsConstructor / @NoArgsConstructor / @RequiredArgsConstructor**

**What they are**: Generate different types of constructors

**Why use them**: Control object creation, dependency injection, framework requirements

**How to define them**:
```java
@NoArgsConstructor  // Default constructor: User()
@AllArgsConstructor // Full constructor: User(name, email, age)
@RequiredArgsConstructor  // Constructor for final/non-null fields only
public class User {
    private String name;
    private String email;
    private final String id;  // Required in @RequiredArgsConstructor
    
    @NonNull
    private String username;  // Required in @RequiredArgsConstructor
    
    private int age;  // Optional
}

// Generated constructors:
// User() - from @NoArgsConstructor
// User(String id, String username) - from @RequiredArgsConstructor  
// User(String name, String email, String id, String username, int age) - from @AllArgsConstructor
```

**Advanced Constructor Control**:
```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)  // Private constructor
@Builder  // Use builder instead of public constructor
public class ImmutableUser {
    private final String name;
    private final String email;
    private final int age;
}

// Only accessible via builder
ImmutableUser user = ImmutableUser.builder()
    .name("John")
    .email("john@example.com")
    .age(30)
    .build();
```

---

### **@Value - "Immutable Objects"**

**What it is**: Creates immutable objects (like @Data but all fields final, no setters)

**Why use it**: Thread-safe objects, functional programming, value objects

**When to use it**: Configuration objects, DTOs that shouldn't change, value types

**How to define it**:
```java
@Value  // Immutable version of @Data
public class Point {
    int x;
    int y;
}

// Generated:
// - Constructor: Point(int x, int y)
// - Getters: getX(), getY() 
// - NO setters (immutable)
// - equals(), hashCode(), toString()
// - All fields are final

@Value
@Builder
public class ApiResponse {
    String status;
    String message;
    Object data;
    long timestamp;
    
    @Builder.Default
    long timestamp = System.currentTimeMillis();
}

// Usage - immutable after creation
ApiResponse response = ApiResponse.builder()
    .status("success")
    .message("User created")
    .data(userData)
    .build();

// response.setStatus("error");  // ❌ Compile error - no setters
```

---

### **@Slf4j - "Logging Integration"**

**What it is**: Automatically creates a logger field for the class

**Why use it**: Consistent logging, no manual logger creation, framework integration

**When to use it**: Any class that needs logging capabilities

**How to define it**:
```java
@Slf4j  // Creates: private static final Logger log = LoggerFactory.getLogger(UserService.class);
public class UserService {
    
    public void createUser(String name) {
        log.info("Creating user: {}", name);  // Use 'log' directly
        
        try {
            // Business logic
            log.debug("User creation successful");
        } catch (Exception e) {
            log.error("Failed to create user: {}", name, e);
        }
    }
}

// Other logging annotations:
@Log4j2  // For Log4j2: private static final Logger log = LogManager.getLogger(UserService.class);
@CommonsLog  // For Apache Commons Logging
@Flogger  // For Google Flogger
```

---

## Advanced Lombok Features

### **@Accessors - "Fluent and Chain Methods"**

```java
@Accessors(fluent = true, chain = true)
@Setter
@Getter
public class User {
    private String name;
    private String email;
    private int age;
}

// Usage - fluent and chainable
User user = new User()
    .name("John")      // Instead of setName()
    .email("john@example.com")  // Instead of setEmail()
    .age(30);          // Instead of setAge()

String name = user.name();  // Instead of getName()
```

### **@Delegate - "Composition over Inheritance"**

```java
public class UserService {
    @Delegate
    private DatabaseService database;  // All DatabaseService methods available on UserService
    
    @Delegate(excludes = {List.class})
    private List<String> permissions;  // Delegate List methods except specified
}
```

### **@Cleanup - "Automatic Resource Management"**

```java
public void readFile(String filename) throws IOException {
    @Cleanup FileInputStream in = new FileInputStream(filename);
    @Cleanup FileOutputStream out = new FileOutputStream("output.txt");
    
    // Automatically calls in.close() and out.close() at end of scope
    // Even if exception occurs
}
```

---

## Real-World Examples

### **E-commerce Product System**
```java
@Data
@Builder
@Slf4j
public class Product {
    private String productId;
    private String name;
    private String description;
    private BigDecimal price;
    private String category;
    private List<String> tags;
    
    @Builder.Default
    private boolean active = true;
    
    @Builder.Default
    private LocalDateTime createdAt = LocalDateTime.now();
    
    public void activate() {
        this.active = true;
        log.info("Product activated: {}", productId);
    }
    
    public void deactivate() {
        this.active = false;
        log.info("Product deactivated: {}", productId);
    }
}

@Value
@Builder
public class ProductSearchRequest {
    String category;
    BigDecimal minPrice;
    BigDecimal maxPrice;
    List<String> tags;
    
    @Builder.Default
    int page = 0;
    
    @Builder.Default
    int size = 20;
}
```

### **User Management System**
```java
@Data
@Builder
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class User {
    @EqualsAndHashCode.Include
    private String userId;  // Only userId used for equality
    
    private String firstName;
    private String lastName;
    private String email;
    private String phone;
    
    @ToString.Exclude  // Don't show in toString
    private String password;
    
    @Builder.Default
    private boolean active = true;
    
    @Builder.Default
    private Set<Role> roles = new HashSet<>();
    
    public String getFullName() {
        return firstName + " " + lastName;
    }
}

@Value
@Builder
public class Role {
    String name;
    String description;
    Set<Permission> permissions;
}
```

---

## Dagger vs Lombok Comparison

| Aspect | **Dagger** | **Lombok** |
|--------|------------|------------|
| **Purpose** | Dependency Injection | Code Generation |
| **Framework Type** | Runtime DI Framework | Compile-time Code Generator |
| **Primary Use** | Object wiring and lifecycle | Boilerplate code elimination |
| **When Applied** | Runtime object creation | Compile-time code generation |
| **Main Annotations** | @Inject, @Component, @Module | @Data, @Builder, @Getter, @Setter |

### **Key Differences:**

#### **Dagger - Dependency Management**
```java
// Dagger manages HOW objects are created and connected
@Component(modules = UserModule.class)
public interface AppComponent {
    UserService getUserService();  // Dagger creates and wires dependencies
}

public class UserService {
    @Inject  // Dagger injects dependencies
    public UserService(DatabaseService db, EmailService email) {
        // Dagger provides db and email instances
    }
}
```

#### **Lombok - Code Generation**
```java
// Lombok generates WHAT code exists in your classes
@Data  // Lombok generates getters, setters, equals, hashCode, toString
@Builder  // Lombok generates builder pattern code
public class User {
    private String name;
    private String email;
    // Lombok adds ~50 lines of boilerplate code here
}
```

### **They Work Together:**
```java
@Data  // ← Lombok generates getters/setters
@Builder  // ← Lombok generates builder pattern
public class DatabaseConfig {
    private String url;
    private String username;
    private String password;
}

@Module  // ← Dagger module for dependency injection
public class DatabaseModule {
    @Provides  // ← Dagger provides this dependency
    @Singleton
    public DatabaseConfig provideDatabaseConfig() {
        return DatabaseConfig.builder()  // ← Using Lombok-generated builder
            .url("jdbc:postgresql://localhost:5432/app")
            .username("admin")
            .password("secret")
            .build();
    }
}

public class UserRepository {
    private final DatabaseConfig config;
    
    @Inject  // ← Dagger injects the dependency
    public UserRepository(DatabaseConfig config) {  // ← Lombok-generated object
        this.config = config;
        String url = config.getUrl();  // ← Using Lombok-generated getter
    }
}
```

### **Summary:**
- **Lombok**: Makes your classes cleaner by generating boilerplate code
- **Dagger**: Makes your application architecture cleaner by managing dependencies
- **Together**: Clean classes + clean dependency management = maintainable applications

**Use Lombok for**: Data classes, POJOs, reducing boilerplate
**Use Dagger for**: Dependency injection, object lifecycle management, testing

Both frameworks complement each other perfectly in modern Java applications! 🚀

---
### **🎯 Next Priority Learning:**

#### **1. UNO Framework Integration**
- UnoSyncLambdaHandler base class
- UnoRequest and UnoResponse patterns
- UnoInvocationContext usage

#### **2. AWS Lambda Patterns**
- Handler method signatures
- Context object usage
- Error handling in Lambda environment

#### **3. S3 Integration (if needed)**
- AmazonS3 client usage
- S3 URI parsing and operations
- Error handling for S3 operations

#### **4. Logging Best Practices**
- @Slf4j annotation usage
- Structured logging patterns
- Error logging with context

#### **5. Testing Patterns**
- Mockito for unit testing
- Integration testing strategies
- Test data setup patterns
