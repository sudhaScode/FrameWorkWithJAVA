# Dagger Dependency Injection: Complete Guide for All Levels

## Table of Contents
- [Beginner Level](#beginner-level)
- [Intermediate Level](#intermediate-level)
- [Advanced Level](#advanced-level)
- [Professional Level](#professional-level)

---

## Beginner Level

### What is Dependency Injection?

**Simple Analogy**: Imagine ordering food at a restaurant
- **Without DI**: You go to the kitchen, find ingredients, cook the meal yourself
- **With DI**: You order from the menu, the kitchen prepares everything and serves it to you

### The Problem Dagger Solves

```java
// Manual dependency creation (BAD)
public class OrderService {
    private PaymentService paymentService;
    private EmailService emailService;
    
    public OrderService() {
        // Hard-coded dependencies - difficult to test and maintain
        this.paymentService = new PaymentService("stripe-key", "production");
        this.emailService = new EmailService("smtp.gmail.com", 587);
    }
}
```

```java
// Dagger dependency injection (GOOD)
public class OrderService {
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    @Inject
    public OrderService(PaymentService paymentService, EmailService emailService) {
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
}
```

### Core Annotations - Detailed Breakdown

#### @Inject - "I need this dependency"

**What it is**: An annotation that tells Dagger "this class needs these dependencies to work"

**Why use it**: Instead of manually creating dependencies, let Dagger provide them automatically

**When to use it**: 
- On constructors that need dependencies
- On fields that should be injected (not recommended for beginners)
- On methods that should receive dependencies (rare)

**How to define it**:
```java
public class CoffeeShop {
    private final CoffeeMaker coffeeMaker;
    private final PaymentProcessor paymentProcessor;
    
    @Inject  // ← This tells Dagger: "I need these two things"
    public CoffeeShop(CoffeeMaker coffeeMaker, PaymentProcessor paymentProcessor) {
        this.coffeeMaker = coffeeMaker;
        this.paymentProcessor = paymentProcessor;
    }
}
```

**Rules for @Inject**:
- Only ONE constructor per class can have @Inject
- All parameters must be available in Dagger's object graph
- Constructor must be public or package-private

#### @Provides - "Here's the recipe to create this object"

**What it is**: An annotation on methods that tells Dagger how to create specific objects

**Why use it**: When you need custom logic to create objects (configuration, third-party libraries, complex setup)

**When to use it**:
- Creating objects that need configuration
- Wrapping third-party libraries
- Objects that require complex initialization
- When you can't modify the class to add @Inject

**How to define it**:
```java
@Module
public class CoffeeModule {
    
    @Provides  // ← This method provides/creates a CoffeeMaker
    public CoffeeMaker provideCoffeeMaker() {
        // Custom creation logic
        CoffeeMaker maker = new CoffeeMaker();
        maker.setTemperature(85); // Custom configuration
        maker.setBrand("Italian");
        return maker;
    }
    
    @Provides
    public PaymentProcessor providePaymentProcessor() {
        // Third-party library setup
        return new StripePaymentProcessor("api-key-123", "production-mode");
    }
}
```

**Rules for @Provides**:
- Must be inside a @Module class
- Method parameters become dependencies (Dagger will provide them)
- Return type is what this method provides
- Method name doesn't matter (convention: provide + ClassName)

#### @Module - "This class contains creation recipes"

**What it is**: An annotation that marks a class as containing @Provides methods

**Why use it**: Groups related object creation logic together, keeps code organized

**When to use it**:
- When you have @Provides methods
- To group related dependencies (NetworkModule, DatabaseModule, etc.)
- To organize configuration by feature or layer

**How to define it**:
```java
@Module  // ← This tells Dagger: "This class has recipes for creating objects"
public class AppModule {
    
    @Provides
    public String provideApiKey() {
        return "secret-api-key-123";
    }
    
    @Provides
    public int provideTimeoutSeconds() {
        return 30;
    }
    
    @Provides
    public DatabaseConfig provideDatabaseConfig() {
        return new DatabaseConfig("localhost", 5432, "myapp");
    }
}
```

**Rules for @Module**:
- Can contain multiple @Provides methods
- Can be abstract (for @Binds methods - covered in intermediate)
- Should be focused on single responsibility (one concern per module)
- Must be included in @Component to be used

#### @Component - "Assembly instructions for Dagger"

**What it is**: An interface that tells Dagger what objects you want and which modules to use

**Why use it**: Acts as the bridge between your code and Dagger's generated code

**When to use it**:
- At the root of your application (Application level)
- When you need to get objects from Dagger
- To define the scope and lifetime of your object graph

**How to define it**:
```java
@Component(modules = {CoffeeModule.class, PaymentModule.class})  // ← Use these recipe books
public interface CoffeeShopComponent {
    
    // What you want Dagger to create and give you
    CoffeeShop getCoffeeShop();
    
    // You can have multiple methods
    PaymentProcessor getPaymentProcessor();
    
    // Injection methods (for existing objects)
    void inject(MainActivity activity);
}
```

**Rules for @Component**:
- Must be an interface (not a class)
- Methods define what you want from Dagger
- Must specify modules in the annotation
- Dagger generates implementation (DaggerYourComponentName)

#### @Singleton - "Create only one instance"

**What it is**: A scope annotation that tells Dagger to create only one instance of this object

**Why use it**: For expensive objects that should be shared (database connections, network clients, etc.)

**When to use it**:
- Objects that are expensive to create
- Objects that should maintain state across the app
- Objects that are thread-safe and can be shared

**How to define it**:
```java
@Module
public class NetworkModule {
    
    @Provides
    @Singleton  // ← Create only ONE instance for entire app
    public OkHttpClient provideHttpClient() {
        return new OkHttpClient.Builder()
                .connectTimeout(30, TimeUnit.SECONDS)
                .build();
    }
}

// Also mark the component as @Singleton
@Component(modules = NetworkModule.class)
@Singleton
public interface AppComponent {
    OkHttpClient getHttpClient();
}
```

**Rules for @Singleton**:
- Component must also be marked @Singleton
- Object will live as long as the component lives
- Thread-safe by default (Dagger handles synchronization)
- Use sparingly - not everything needs to be singleton

### Your First Complete Dagger Example

```java
// Step 1: Simple class with no dependencies
public class Printer {
    @Inject
    public Printer() {}  // Empty constructor with @Inject
    
    public void print(String message) {
        System.out.println("Printing: " + message);
    }
}

// Step 2: Class that needs dependencies
public class MessageService {
    private final Printer printer;
    
    @Inject  // Tell Dagger: "I need a Printer"
    public MessageService(Printer printer) {
        this.printer = printer;
    }
    
    public void sendMessage(String msg) {
        printer.print("Message: " + msg);
    }
}

// Step 3: Component (no modules needed - simple @Inject constructors)
@Component
public interface MessageComponent {
    MessageService getMessageService();  // What we want
}

// Step 4: Usage
public class Main {
    public static void main(String[] args) {
        // Dagger generates DaggerMessageComponent
        MessageComponent component = DaggerMessageComponent.create();
        MessageService service = component.getMessageService();
        service.sendMessage("Hello Dagger!");
    }
}
```

**What happens behind the scenes**:
1. Dagger sees you want `MessageService`
2. `MessageService` needs `Printer` (via @Inject constructor)
3. `Printer` has @Inject constructor with no parameters
4. Dagger creates `Printer` first, then `MessageService`
5. Returns fully wired `MessageService`

---

## Intermediate Level

### Scopes and Lifecycle Management - Detailed Breakdown

#### @Singleton - "One instance for entire application"

**What it is**: A built-in scope that ensures only one instance of an object exists throughout the application lifecycle

**Why use it**: 
- Saves memory by reusing expensive objects
- Maintains state across the application
- Improves performance by avoiding repeated object creation

**When to use it**:
- Database connections (expensive to create)
- Network clients (OkHttpClient, Retrofit)
- Caches and repositories
- Configuration objects

**How to define it**:
```java
@Module
public class DatabaseModule {
    
    @Provides
    @Singleton  // ← Only create one database connection
    public DatabaseConnection provideDatabase() {
        return new DatabaseConnection("jdbc:postgresql://localhost:5432/mydb");
    }
}

// Component must also be @Singleton
@Component(modules = DatabaseModule.class)
@Singleton
public interface AppComponent {
    DatabaseConnection getDatabase();
}
```

**Rules for @Singleton**:
- Component must also be annotated with @Singleton
- Object lives as long as the component lives
- Thread-safe (Dagger handles synchronization)
- All dependent objects share the same instance

#### Custom Scopes - "Define your own lifecycle"

**What it is**: User-defined scopes that control object lifecycle beyond singleton

**Why use it**: 
- Control object lifetime for specific features
- Memory management for user sessions
- Scope objects to specific workflows

**When to use it**:
- User login sessions (@UserScope)
- Activity/Fragment lifecycle (@ActivityScope)
- Feature-specific scopes (@FeatureScope)

**How to define it**:
```java
// Step 1: Define the scope annotation
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface UserScope {}

// Step 2: Use it in modules
@Module
public class UserModule {
    @Provides
    @UserScope  // ← Lives as long as user session
    public UserPreferences provideUserPreferences(User user) {
        return new UserPreferences(user.getId());
    }
}

// Step 3: Apply to component
@UserScope
@Component(dependencies = AppComponent.class, modules = UserModule.class)
public interface UserComponent {
    UserProfile getUserProfile();
    
    @Component.Factory
    interface Factory {
        UserComponent create(AppComponent appComponent, @BindsInstance User user);
    }
}
```

**Rules for Custom Scopes**:
- Must be annotated with @Scope
- Component must have the same scope annotation
- Scoped objects can only depend on objects with same or broader scope
- Create new component instance to reset scope

### Qualifiers - Multiple implementations

#### @Qualifier - "Distinguish between similar objects"

**What it is**: Annotations that help Dagger distinguish between different implementations of the same type

**Why use it**: 
- Multiple implementations of same interface
- Different configurations of same object type
- Environment-specific implementations (dev vs prod)

**When to use it**:
- Multiple API endpoints (dev, staging, prod)
- Different database connections
- Feature flags and configurations
- Testing vs production implementations

**How to define it**:
```java
// Step 1: Define qualifier annotations
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface Development {}

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface Production {}

// Step 2: Use in modules
@Module
public class ApiModule {
    @Provides
    @Development  // ← This provides dev API URL
    public String provideDevApiUrl() {
        return "https://api-dev.example.com";
    }
    
    @Provides
    @Production  // ← This provides prod API URL
    public String provideProdApiUrl() {
        return "https://api.example.com";
    }
    
    @Provides
    public ApiService provideApiService(@Development String apiUrl) {
        return new ApiService(apiUrl);
    }
}

// Step 3: Use in injection
public class NetworkService {
    private final String devUrl;
    private final String prodUrl;
    
    @Inject
    public NetworkService(@Development String devUrl, 
                         @Production String prodUrl) {
        this.devUrl = devUrl;
        this.prodUrl = prodUrl;
    }
}
```

**Rules for Qualifiers**:
- Must be annotated with @Qualifier
- Use @Retention(RetentionPolicy.RUNTIME)
- Apply to both @Provides methods and injection points
- Can have parameters for more specific qualification

#### @Named - "Built-in string-based qualifier"

**What it is**: A built-in qualifier that uses strings to distinguish objects

**Why use it**: Simpler than creating custom qualifiers for basic cases

**When to use it**: Simple string-based differentiation (not recommended for complex cases)

**How to define it**:
```java
@Module
public class ConfigModule {
    @Provides
    @Named("api_key")  // ← String-based qualifier
    public String provideApiKey() {
        return "secret-key-123";
    }
    
    @Provides
    @Named("base_url")
    public String provideBaseUrl() {
        return "https://api.example.com";
    }
}

public class ApiClient {
    @Inject
    public ApiClient(@Named("api_key") String apiKey,
                    @Named("base_url") String baseUrl) {
        // Use qualified dependencies
    }
}
```

### Advanced Injection Patterns

#### Lazy<T> - "Create only when needed"

**What it is**: A wrapper that delays object creation until first access

**Why use it**: 
- Avoid expensive object creation during app startup
- Break circular dependencies
- Improve performance for rarely used objects

**When to use it**:
- Expensive objects that might not be used
- Objects with circular dependencies
- Performance-critical startup paths

**How to define it**:
```java
public class ExpensiveService {
    private final Lazy<DatabaseConnection> lazyDatabase;
    private final Lazy<NetworkClient> lazyNetwork;
    
    @Inject
    public ExpensiveService(Lazy<DatabaseConnection> lazyDatabase,
                           Lazy<NetworkClient> lazyNetwork) {
        this.lazyDatabase = lazyDatabase;
        this.lazyNetwork = lazyNetwork;
        // Objects not created yet!
    }
    
    public void doWork() {
        // Database only created when first accessed
        DatabaseConnection db = lazyDatabase.get();
        db.query("SELECT * FROM users");
        
        // Network client created only if needed
        if (needsNetwork()) {
            NetworkClient client = lazyNetwork.get();
            client.makeRequest();
        }
    }
}
```

**Rules for Lazy<T>**:
- No special module configuration needed
- Object created on first .get() call
- Subsequent .get() calls return same instance
- Thread-safe by default

#### Provider<T> - "Create new instance each time"

**What it is**: A factory that creates a new instance every time .get() is called

**Why use it**: 
- When you need multiple instances of the same type
- Factory pattern implementation
- Objects that shouldn't be reused

**When to use it**:
- Creating multiple instances of non-singleton objects
- Factory methods that need fresh instances
- Objects with mutable state

**How to define it**:
```java
public class UserFactory {
    private final Provider<User> userProvider;
    
    @Inject
    public UserFactory(Provider<User> userProvider) {
        this.userProvider = userProvider;
    }
    
    public User createNewUser(String name, String email) {
        User user = userProvider.get();  // Creates NEW User instance each time
        user.setName(name);
        user.setEmail(email);
        return user;
    }
    
    public List<User> createMultipleUsers(int count) {
        List<User> users = new ArrayList<>();
        for (int i = 0; i < count; i++) {
            users.add(userProvider.get());  // Each call creates new instance
        }
        return users;
    }
}
```

**Rules for Provider<T>**:
- No special module configuration needed
- Each .get() call creates new instance (unless object is scoped)
- Useful for factory patterns
- Can be combined with scopes

### Conditional Binding

#### Conditional Object Creation - "Different objects for different conditions"

**What it is**: Creating different implementations based on runtime conditions

**Why use it**: 
- Environment-specific implementations
- Feature flags and A/B testing
- Debug vs release builds

**When to use it**:
- Different behavior for debug/release
- Feature toggles
- Environment-specific services

**How to define it**:
```java
@Module
public class ConfigModule {
    
    @Provides
    public ApiService provideApiService() {
        if (BuildConfig.DEBUG) {
            return new MockApiService();  // Fake data for development
        } else {
            return new RealApiService();  // Real API calls for production
        }
    }
    
    @Provides
    public Logger provideLogger() {
        if (isProduction()) {
            return new ProductionLogger();  // Logs to server
        } else {
            return new DebugLogger();       // Logs to console
        }
    }
    
    private boolean isProduction() {
        return "production".equals(System.getProperty("environment"));
    }
}
```

**Rules for Conditional Binding**:
- Logic goes inside @Provides methods
- Consider using qualifiers for cleaner separation
- Be careful with conditions that might change at runtime
- Test both branches thoroughly

---

## Advanced Level

### Component Architecture - Detailed Breakdown

#### Component Dependencies - "Components that depend on other components"

**What it is**: A way to make one component depend on another component, creating a hierarchy

**Why use it**: 
- Separate concerns into different components
- Share common dependencies across multiple components
- Create modular architecture with clear boundaries

**When to use it**:
- App-level component with feature-level components
- Shared services across multiple features
- Testing with different component configurations

**How to define it**:
```java
// Step 1: Base component with shared dependencies
@Component(modules = NetworkModule.class)
@Singleton
public interface NetworkComponent {
    OkHttpClient okHttpClient();
    Retrofit retrofit();
}

// Step 2: Dependent component
@Component(dependencies = NetworkComponent.class, modules = UserModule.class)
@UserScope
public interface UserComponent {
    UserRepository userRepository();
    UserService userService();
    
    // Factory to create component with dependency
    @Component.Factory
    interface Factory {
        UserComponent create(NetworkComponent networkComponent);
    }
}

// Step 3: Usage
public class Application {
    private NetworkComponent networkComponent;
    private UserComponent userComponent;
    
    public void onCreate() {
        // Create base component
        networkComponent = DaggerNetworkComponent.create();
        
        // Create dependent component
        userComponent = DaggerUserComponent.factory()
                .create(networkComponent);
    }
}
```

**Rules for Component Dependencies**:
- Dependent component can access exposed methods from dependency
- Dependency component must expose what dependent needs
- Scopes must be compatible (dependent scope ≤ dependency scope)
- No circular dependencies between components

#### Subcomponents - "Child components with inherited dependencies"

**What it is**: Components that inherit all dependencies from their parent component

**Why use it**: 
- Extend parent component with additional features
- Create scoped child components
- Better performance than component dependencies

**When to use it**:
- Feature modules that extend app functionality
- Activity/Fragment scoped components
- User session components

**How to define it**:
```java
// Step 1: Define subcomponent
@Subcomponent(modules = UserModule.class)
@UserScope
public interface UserSubcomponent {
    UserProfile userProfile();
    UserSettings userSettings();
    
    // Factory for creating subcomponent
    @Subcomponent.Factory
    interface Factory {
        UserSubcomponent create(@BindsInstance User user);
    }
}

// Step 2: Parent component includes subcomponent
@Component(modules = AppModule.class)
@Singleton
public interface AppComponent {
    // Expose subcomponent factory
    UserSubcomponent.Factory userSubcomponentFactory();
    
    // Other app-level dependencies
    DatabaseService databaseService();
    NetworkService networkService();
}

// Step 3: Usage
public class UserManager {
    private final AppComponent appComponent;
    private UserSubcomponent userSubcomponent;
    
    public void loginUser(User user) {
        // Create user-scoped subcomponent
        userSubcomponent = appComponent.userSubcomponentFactory()
                .create(user);
        
        UserProfile profile = userSubcomponent.userProfile();
    }
    
    public void logoutUser() {
        // Destroy user scope by nullifying reference
        userSubcomponent = null;
    }
}
```

**Rules for Subcomponents**:
- Inherits all parent dependencies automatically
- Can have narrower scope than parent
- Parent must declare subcomponent factory
- More efficient than component dependencies

### Advanced Binding Patterns

#### Multibindings - Sets - "Collect multiple implementations"

**What it is**: A way to bind multiple objects into a Set that can be injected

**Why use it**: 
- Plugin architecture with multiple implementations
- Validation systems with multiple validators
- Event handlers or listeners

**When to use it**:
- Multiple implementations of same interface
- Extensible systems (plugins, validators, processors)
- Observer patterns

**How to define it**:
```java
// Step 1: Define implementations
public interface Validator {
    boolean validate(String input);
}

public class EmailValidator implements Validator {
    @Inject
    public EmailValidator() {}
    
    @Override
    public boolean validate(String input) {
        return input.contains("@");
    }
}

public class PasswordValidator implements Validator {
    @Inject
    public PasswordValidator() {}
    
    @Override
    public boolean validate(String input) {
        return input.length() >= 8;
    }
}

// Step 2: Bind into set
@Module
public abstract class ValidationModule {
    @Binds
    @IntoSet  // ← Add this validator to the set
    abstract Validator bindEmailValidator(EmailValidator validator);
    
    @Binds
    @IntoSet  // ← Add this validator to the set
    abstract Validator bindPasswordValidator(PasswordValidator validator);
    
    // Can also use @Provides with @IntoSet
    @Provides
    @IntoSet
    static Validator providePhoneValidator() {
        return input -> input.matches("\\d{10}");
    }
}

// Step 3: Inject the set
public class FormValidator {
    private final Set<Validator> validators;
    
    @Inject
    public FormValidator(Set<Validator> validators) {
        this.validators = validators;  // Contains all bound validators
    }
    
    public boolean validateForm(String input) {
        return validators.stream()
                .allMatch(validator -> validator.validate(input));
    }
}
```

#### Multibindings - Maps - "Key-value collections"

**What it is**: A way to bind multiple objects into a Map with specific keys

**Why use it**: 
- Strategy pattern implementations
- Factory methods with string keys
- Configuration systems

**When to use it**:
- Different implementations based on string keys
- Factory pattern with multiple creators
- Plugin systems with named plugins

**How to define it**:
```java
// Step 1: Define map key annotation
@MapKey
@Retention(RetentionPolicy.RUNTIME)
public @interface ProcessorKey {
    String value();
}

// Step 2: Define implementations
public interface DataProcessor {
    void process(String data);
}

public class JsonProcessor implements DataProcessor {
    @Inject
    public JsonProcessor() {}
    
    @Override
    public void process(String data) {
        // Process JSON data
    }
}

public class XmlProcessor implements DataProcessor {
    @Inject
    public XmlProcessor() {}
    
    @Override
    public void process(String data) {
        // Process XML data
    }
}

// Step 3: Bind into map
@Module
public abstract class ProcessorModule {
    @Binds
    @IntoMap
    @ProcessorKey("json")  // ← Key for this processor
    abstract DataProcessor bindJsonProcessor(JsonProcessor processor);
    
    @Binds
    @IntoMap
    @ProcessorKey("xml")   // ← Key for this processor
    abstract DataProcessor bindXmlProcessor(XmlProcessor processor);
    
    // String-based keys (simpler)
    @Binds
    @IntoMap
    @StringKey("csv")
    abstract DataProcessor bindCsvProcessor(CsvProcessor processor);
}

// Step 4: Inject the map
public class DataProcessorFactory {
    private final Map<String, DataProcessor> processors;
    
    @Inject
    public DataProcessorFactory(Map<String, DataProcessor> processors) {
        this.processors = processors;
    }
    
    public DataProcessor getProcessor(String type) {
        DataProcessor processor = processors.get(type);
        if (processor == null) {
            throw new IllegalArgumentException("Unknown processor type: " + type);
        }
        return processor;
    }
}
```

#### @Binds - "Bind interface to implementation"

**What it is**: An abstract method annotation that binds an interface to its implementation

**Why use it**: 
- More efficient than @Provides for simple interface bindings
- Cleaner code for interface-implementation relationships
- Better performance (no method call overhead)

**When to use it**:
- Binding interfaces to implementations
- When you don't need custom creation logic
- Abstract modules for better organization

**How to define it**:
```java
// Step 1: Interface and implementation
public interface UserRepository {
    User findById(String id);
}

public class DatabaseUserRepository implements UserRepository {
    private final DatabaseService database;
    
    @Inject
    public DatabaseUserRepository(DatabaseService database) {
        this.database = database;
    }
    
    @Override
    public User findById(String id) {
        return database.query("SELECT * FROM users WHERE id = ?", id);
    }
}

// Step 2: Abstract module with @Binds
@Module
public abstract class RepositoryModule {
    
    @Binds  // ← Bind interface to implementation
    abstract UserRepository bindUserRepository(DatabaseUserRepository impl);
    
    @Binds
    abstract OrderRepository bindOrderRepository(DatabaseOrderRepository impl);
    
    // Can mix with @Provides for complex objects
    @Provides
    static DatabaseService provideDatabaseService() {
        return new DatabaseService("jdbc:postgresql://localhost:5432/app");
    }
}
```

**Rules for @Binds**:
- Must be in abstract module
- Method must be abstract
- Return type is the interface, parameter is the implementation
- More efficient than @Provides for simple bindings

### Optional Bindings

#### @BindsOptionalOf - "Optional dependencies"

**What it is**: Allows injecting Optional<T> for dependencies that might not be available

**Why use it**: 
- Optional features that might not be enabled
- Dependencies that might not be available in all configurations
- Graceful degradation when services are unavailable

**When to use it**:
- Analytics services that might be disabled
- Optional features based on build variants
- Services that might fail to initialize

**How to define it**:
```java
// Step 1: Declare optional binding
@Module
public abstract class OptionalModule {
    @BindsOptionalOf
    abstract AnalyticsService optionalAnalytics();
    
    @BindsOptionalOf
    abstract CrashReportingService optionalCrashReporting();
}

// Step 2: Conditionally provide implementation
@Module
public class AnalyticsModule {
    @Provides
    static AnalyticsService provideAnalytics() {
        if (BuildConfig.ANALYTICS_ENABLED) {
            return new FirebaseAnalytics();
        } else {
            // Don't provide - will be Optional.empty()
            throw new RuntimeException("Analytics disabled");
        }
    }
}

// Step 3: Inject optional dependency
public class UserService {
    private final Optional<AnalyticsService> analytics;
    private final Optional<CrashReportingService> crashReporting;
    
    @Inject
    public UserService(Optional<AnalyticsService> analytics,
                      Optional<CrashReportingService> crashReporting) {
        this.analytics = analytics;
        this.crashReporting = crashReporting;
    }
    
    public void trackUserAction(String action) {
        // Use analytics only if available
        analytics.ifPresent(service -> service.track(action));
    }
    
    public void handleError(Exception e) {
        // Report crash only if service available
        crashReporting.ifPresent(service -> service.report(e));
    }
}
```

**Rules for Optional Bindings**:
- Must declare @BindsOptionalOf in a module
- If no implementation provided, Optional.empty() is injected
- If implementation provided, Optional.of(implementation) is injected
- Useful for graceful degradation

### Component Factories and Builders

#### @Component.Factory - "Custom component creation"

**What it is**: An interface that defines how to create a component with runtime parameters

**Why use it**: 
- Pass runtime values to component creation
- Inject application context or configuration
- Create components with specific parameters

**When to use it**:
- Android applications (need Context)
- Configuration that comes from runtime
- Testing with specific parameters

**How to define it**:
```java
@Component(modules = AppModule.class)
@Singleton
public interface AppComponent {
    UserService getUserService();
    
    @Component.Factory
    interface Factory {
        // Create component with runtime parameters
        AppComponent create(@BindsInstance Context context,
                          @BindsInstance @Named("api_key") String apiKey,
                          @BindsInstance boolean debugMode);
    }
}

// Usage
public class Application {
    public void onCreate() {
        AppComponent component = DaggerAppComponent.factory()
                .create(
                    this,                    // Context
                    "secret-api-key-123",   // API key
                    BuildConfig.DEBUG       // Debug mode
                );
    }
}
```

#### @Component.Builder - "Step-by-step component creation"

**What it is**: A builder pattern for creating components with optional parameters

**Why use it**: 
- More flexible than factory
- Optional parameters with defaults
- Step-by-step component configuration

**When to use it**:
- Complex component creation
- Many optional parameters
- When you prefer builder pattern

**How to define it**:
```java
@Component(modules = AppModule.class)
@Singleton
public interface AppComponent {
    UserService getUserService();
    
    @Component.Builder
    interface Builder {
        @BindsInstance
        Builder context(Context context);
        
        @BindsInstance
        Builder apiKey(@Named("api_key") String apiKey);
        
        @BindsInstance
        Builder debugMode(boolean debugMode);
        
        // Optional: can provide custom module instance
        Builder appModule(AppModule appModule);
        
        AppComponent build();
    }
}

// Usage
public class Application {
    public void onCreate() {
        AppComponent component = DaggerAppComponent.builder()
                .context(this)
                .apiKey("secret-api-key-123")
                .debugMode(BuildConfig.DEBUG)
                .build();
    }
}
```

**Rules for Factory vs Builder**:
- **Factory**: All parameters required, single method
- **Builder**: Can have optional parameters, step-by-step creation
- **@BindsInstance**: Binds runtime values into component
- Choose based on complexity and flexibility needs

---

## Professional Level

### Testing Strategies

#### Test Doubles with Dagger
```java
// Production module
@Module
public class ProductionNetworkModule {
    @Provides
    @Singleton
    public ApiService provideApiService() {
        return new RetrofitApiService();
    }
}

// Test module
@Module
public class TestNetworkModule {
    @Provides
    @Singleton
    public ApiService provideApiService() {
        return new MockApiService();
    }
}

// Test component
@Component(modules = TestNetworkModule.class)
@Singleton
public interface TestAppComponent extends AppComponent {
    void inject(MyTest test);
}
```

#### Espresso Testing with Dagger
```java
public class TestApplication extends Application {
    private AppComponent appComponent;
    
    @Override
    public void onCreate() {
        super.onCreate();
        appComponent = DaggerTestAppComponent.builder()
                .testNetworkModule(new TestNetworkModule())
                .build();
    }
    
    public AppComponent getAppComponent() {
        return appComponent;
    }
}
```

### Performance Optimization

#### Lazy Loading Critical Path
```java
@Module
public class PerformanceModule {
    @Provides
    @Singleton
    public Lazy<ExpensiveService> provideExpensiveService(
            Provider<ExpensiveService> provider) {
        return provider::get;
    }
}
```

#### Component Caching
```java
public class ComponentManager {
    private static volatile AppComponent appComponent;
    
    public static AppComponent getAppComponent(Context context) {
        if (appComponent == null) {
            synchronized (ComponentManager.class) {
                if (appComponent == null) {
                    appComponent = DaggerAppComponent.builder()
                            .context(context.getApplicationContext())
                            .build();
                }
            }
        }
        return appComponent;
    }
}
```

### Error Handling and Debugging

#### Component Validation
```java
@Component(modules = {NetworkModule.class, DatabaseModule.class})
@Singleton
public interface AppComponent {
    // Validate all dependencies at compile time
    void validateDependencies();
    
    // Expose for debugging
    Set<Object> getAllSingletons();
}
```

#### Debugging Missing Bindings
```java
// Use @BindsOptionalOf for optional dependencies
@Module
public abstract class OptionalBindingsModule {
    @BindsOptionalOf
    abstract AnalyticsService optionalAnalytics();
}

public class UserService {
    @Inject
    public UserService(Optional<AnalyticsService> analytics) {
        this.analytics = analytics;
    }
    
    public void trackEvent(String event) {
        analytics.ifPresent(service -> service.track(event));
    }
}
```

### Architecture Patterns

#### Clean Architecture with Dagger
```java
// Domain Layer
@Module
public abstract class DomainModule {
    @Binds
    abstract UserRepository bindUserRepository(UserRepositoryImpl impl);
    
    @Binds
    abstract GetUserUseCase bindGetUserUseCase(GetUserUseCaseImpl impl);
}

// Data Layer
@Module
public class DataModule {
    @Provides
    @Singleton
    public UserApi provideUserApi(Retrofit retrofit) {
        return retrofit.create(UserApi.class);
    }
}

// Presentation Layer
@Module
public abstract class PresentationModule {
    @Binds
    @IntoMap
    @ViewModelKey(UserViewModel.class)
    abstract ViewModel bindUserViewModel(UserViewModel viewModel);
}
```

#### Feature Modules
```java
@Module(subcomponents = {
    UserSubcomponent.class,
    OrderSubcomponent.class,
    PaymentSubcomponent.class
})
public class FeatureModule {}

@Subcomponent(modules = UserModule.class)
@FeatureScope
public interface UserSubcomponent {
    UserFragment.Factory userFragmentFactory();
    
    @Subcomponent.Factory
    interface Factory {
        UserSubcomponent create();
    }
}
```

### Advanced Patterns

#### Assisted Injection
```java
public class ReportGenerator {
    private final DatabaseService database;
    private final String reportType;
    
    @AssistedInject
    public ReportGenerator(DatabaseService database, 
                          @Assisted String reportType) {
        this.database = database;
        this.reportType = reportType;
    }
    
    @AssistedFactory
    interface Factory {
        ReportGenerator create(String reportType);
    }
}
```

#### Dynamic Feature Modules
```java
@Module
public class DynamicFeatureModule {
    @Provides
    @IntoMap
    @StringKey("feature_a")
    public FeatureProvider provideFeatureA() {
        return new FeatureAProvider();
    }
    
    @Provides
    @IntoMap
    @StringKey("feature_b")
    public FeatureProvider provideFeatureB() {
        return new FeatureBProvider();
    }
}

public class FeatureManager {
    private final Map<String, FeatureProvider> features;
    
    @Inject
    public FeatureManager(Map<String, FeatureProvider> features) {
        this.features = features;
    }
    
    public Optional<Feature> getFeature(String name) {
        return Optional.ofNullable(features.get(name))
                .map(FeatureProvider::provide);
    }
}
```

---

## Best Practices Summary

### For Beginners
- Start with simple `@Inject` constructors
- Use `@Provides` for complex object creation
- Keep modules focused on single responsibility
- Always use `@Singleton` for expensive objects

### For Intermediates
- Learn scopes and qualifiers for better organization
- Use `Lazy<T>` and `Provider<T>` for performance
- Implement proper testing strategies
- Understand component lifecycle

### For Advanced Users
- Master subcomponents and component dependencies
- Use multibindings for extensible architectures
- Implement assisted injection for runtime parameters
- Design for testability and maintainability

### For Professionals
- Optimize for build time and runtime performance
- Implement comprehensive testing strategies
- Design modular, scalable architectures
- Use advanced patterns like feature modules and dynamic injection

---

## Common Pitfalls and Solutions

| Problem | Solution |
|---------|----------|
| Circular dependencies | Use `Provider<T>` or `Lazy<T>` |
| Missing bindings | Check modules are included in component |
| Scope mismatches | Ensure dependent scopes are compatible |
| Large object graphs | Use subcomponents and lazy loading |
| Slow build times | Minimize component rebuilding, use incremental annotation processing |

---

## Resources for Further Learning

- **Official Documentation**: [Dagger 2 User's Guide](https://dagger.dev/users-guide)
- **Advanced Topics**: [Dagger 2 Android Guide](https://dagger.dev/android)
- **Best Practices**: [Google's Dagger Best Practices](https://developer.android.com/training/dependency-injection/dagger-basics)
- **Community**: [Dagger GitHub Discussions](https://github.com/google/dagger/discussions)

---

*This guide covers Dagger from basic concepts to professional-level patterns. Start with your current level and gradually work your way up as you gain experience with dependency injection concepts.*
---
 ## **Dagger @Component - Complete Deep Dive**

### **What is @Component?**

@Component is the bridge between your code and Dagger's generated code. It's an interface that tells Dagger:
1. What objects you want (provision methods)
2. Where to inject dependencies (injection methods)
3. Which modules to use (dependency recipes)

Think of it as a contract between you and Dagger's code generator.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **How @Component Works - Step by Step**

### **Step 1: You Define the Interface**
java
@Component(modules = {DatabaseModule.class, NetworkModule.class})
@Singleton                                                                                                
public interface AppComponent {                                                                           
    // What you want from Dagger                                                                          
    UserService getUserService();                                                                         
    DatabaseConnection getDatabase();                                                                     
                                                                                                          
    // Where you want Dagger to inject                                                                    
    void inject(MainActivity activity);                                                                   
}                                                                                                         
                                                                                                          

### **Step 2: Dagger Generates Implementation**
When you build your project, Dagger automatically creates:
java
// Generated by Dagger (you don't write this!)
public final class DaggerAppComponent implements AppComponent {                                           
    private DatabaseConnection databaseConnection;                                                        
    private UserService userService;                                                                      
                                                                                                          
    @Override                                                                                             
    public UserService getUserService() {                                                                 
        if (userService == null) {                                                                        
            userService = new UserService(getDatabaseConnection());                                       
        }                                                                                                 
        return userService;                                                                               
    }                                                                                                     
                                                                                                          
    @Override                                                                                             
    public DatabaseConnection getDatabase() {                                                             
        if (databaseConnection == null) {                                                                 
            databaseConnection = DatabaseModule_ProvideDatabaseFactory.provideDatabase();                 
        }                                                                                                 
        return databaseConnection;                                                                        
    }                                                                                                     
                                                                                                          
    @Override                                                                                             
    public void inject(MainActivity activity) {                                                           
        MainActivity_MembersInjector.injectUserService(activity, getUserService());                       
    }                                                                                                     
}                                                                                                         
                                                                                                          

### **Step 3: You Use the Generated Code**
java
public class Application {
    private AppComponent appComponent;                                                                    
                                                                                                          
    public void onCreate() {                                                                              
        // Use Dagger-generated implementation                                                            
        appComponent = DaggerAppComponent.create();                                                       
                                                                                                          
        // Get objects from Dagger                                                                        
        UserService userService = appComponent.getUserService();                                          
    }                                                                                                     
}                                                                                                         
                                                                                                          

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **Types of Component Methods**

### **1. Provision Methods - "Give me this object"**

Purpose: Get fully constructed objects from Dagger

java
@Component(modules = UserModule.class)
public interface AppComponent {                                                                           
                                                                                                          
    // Provision methods - Dagger creates and returns objects                                             
    UserService getUserService();           // Returns UserService instance                               
    DatabaseConnection getDatabase();       // Returns DatabaseConnection instance                        
    List<Validator> getValidators();       // Returns list of validators                                  
                                                                                                          
    // Can have parameters (Dagger will provide them)                                                     
    UserRepository getUserRepository();                                                                   
}                                                                                                         
                                                                                                          

What Dagger generates:
java
@Override
public UserService getUserService() {                                                                     
    // Dagger figures out dependencies and creates UserService                                            
    return new UserService(getDatabase(), getEmailService());                                             
}                                                                                                         
                                                                                                          

### **2. Injection Methods - "Inject into this existing object"**

Purpose: Inject dependencies into objects you've already created

java
@Component(modules = AppModule.class)
public interface AppComponent {                                                                           
                                                                                                          
    // Injection methods - inject into existing objects                                                   
    void inject(MainActivity activity);      // Inject into MainActivity                                  
    void inject(UserFragment fragment);      // Inject into UserFragment                                  
    void inject(BackgroundService service);  // Inject into Service                                       
}                                                                                                         
                                                                                                          

Usage:
java
public class MainActivity extends Activity {
    @Inject UserService userService;  // Will be injected                                                 
    @Inject DatabaseService dbService; // Will be injected                                                
                                                                                                          
    @Override                                                                                             
    protected void onCreate(Bundle savedInstanceState) {                                                  
        super.onCreate(savedInstanceState);                                                               
                                                                                                          
        // Inject dependencies into this activity                                                         
        DaggerAppComponent.create().inject(this);                                                         
                                                                                                          
        // Now userService and dbService are available                                                    
        userService.loadUser();                                                                           
    }                                                                                                     
}                                                                                                         
                                                                                                          

What Dagger generates:
java
@Override
public void inject(MainActivity activity) {                                                               
    // Dagger injects into @Inject fields                                                                 
    activity.userService = getUserService();                                                              
    activity.dbService = getDatabaseService();                                                            
}                                                                                                         
                                                                                                          

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **Complete Real-World Example**

### **Step 1: Define Your Classes**
java
// Service that needs dependencies
public class UserService {                                                                                
    private final DatabaseService database;                                                               
    private final EmailService emailService;                                                              
                                                                                                          
    @Inject                                                                                               
    public UserService(DatabaseService database, EmailService emailService) {                             
        this.database = database;                                                                         
        this.emailService = emailService;                                                                 
    }                                                                                                     
                                                                                                          
    public void createUser(String name, String email) {                                                   
        User user = new User(name, email);                                                                
        database.save(user);                                                                              
        emailService.sendWelcomeEmail(email);                                                             
    }                                                                                                     
}                                                                                                         
                                                                                                          
// Activity that needs injection                                                                          
public class MainActivity extends Activity {                                                              
    @Inject UserService userService;  // Dagger will inject this                                          
    @Inject DatabaseService database; // Dagger will inject this                                          
                                                                                                          
    @Override                                                                                             
    protected void onCreate(Bundle savedInstanceState) {                                                  
        super.onCreate(savedInstanceState);                                                               
                                                                                                          
        // Tell Dagger to inject dependencies                                                             
        ((MyApplication) getApplication()).getAppComponent().inject(this);                                
                                                                                                          
        // Now dependencies are available                                                                 
        userService.createUser("John", "john@example.com");                                               
    }                                                                                                     
}                                                                                                         
                                                                                                          

### **Step 2: Create Module for Complex Objects**
java
@Module
public class AppModule {                                                                                  
                                                                                                          
    @Provides                                                                                             
    @Singleton                                                                                            
    public DatabaseService provideDatabaseService() {                                                     
        return new DatabaseService("jdbc:postgresql://localhost:5432/app");                               
    }                                                                                                     
                                                                                                          
    @Provides                                                                                             
    @Singleton                                                                                            
    public EmailService provideEmailService() {                                                           
        return new EmailService("smtp.gmail.com", 587);                                                   
    }                                                                                                     
}                                                                                                         
                                                                                                          

### **Step 3: Define Component**
java
@Component(modules = AppModule.class)
@Singleton                                                                                                
public interface AppComponent {                                                                           
                                                                                                          
    // Provision methods - get objects from Dagger                                                        
    UserService getUserService();                                                                         
    DatabaseService getDatabaseService();                                                                 
                                                                                                          
    // Injection methods - inject into existing objects                                                   
    void inject(MainActivity activity);                                                                   
    void inject(UserFragment fragment);                                                                   
}                                                                                                         
                                                                                                          

### **Step 4: Use in Application**
java
public class MyApplication extends Application {
    private AppComponent appComponent;                                                                    
                                                                                                          
    @Override                                                                                             
    public void onCreate() {                                                                              
        super.onCreate();                                                                                 
                                                                                                          
        // Create component once                                                                          
        appComponent = DaggerAppComponent.create();                                                       
    }                                                                                                     
                                                                                                          
    public AppComponent getAppComponent() {                                                               
        return appComponent;                                                                              
    }                                                                                                     
}                                                                                                         
                                                                                                          

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **How Dependency Resolution Works**

### **Dagger's Resolution Process**:

1. You request: getUserService()
2. Dagger checks: What does UserService need?
3. Dagger finds: UserService needs DatabaseService + EmailService
4. Dagger creates: DatabaseService (from module)
5. Dagger creates: EmailService (from module)
6. Dagger creates: UserService with both dependencies
7. Dagger returns: Fully constructed UserService

### **Dependency Graph Example**:
getUserService()
    ↓                                                                                                     
UserService needs:                                                                                        
    ├── DatabaseService (@Provides in AppModule)                                                          
    └── EmailService (@Provides in AppModule)                                                             
                                                                                                          
Result: UserService(databaseService, emailService)                                                        
                                                                                                          

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **Component Scopes and Lifecycle**

### **Component Lifecycle**:
java
@Component(modules = AppModule.class)
@Singleton  // ← Component scope                                                                          
public interface AppComponent {                                                                           
    UserService getUserService();                                                                         
}                                                                                                         
                                                                                                          
// Usage                                                                                                  
public class Application {                                                                                
    private AppComponent component;                                                                       
                                                                                                          
    public void onCreate() {                                                                              
        component = DaggerAppComponent.create();  // Component created                                    
                                                                                                          
        UserService service1 = component.getUserService();  // First call                                 
        UserService service2 = component.getUserService();  // Second call                                
                                                                                                          
        // service1 == service2 (same instance due to @Singleton)                                         
    }                                                                                                     
                                                                                                          
    public void onDestroy() {                                                                             
        component = null;  // Component destroyed, all singletons released                                
    }                                                                                                     
}                                                                                                         
                                                                                                          

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **Advanced Component Features**

### **Component with Runtime Parameters**:
java
@Component(modules = AppModule.class)
public interface AppComponent {                                                                           
    UserService getUserService();                                                                         
                                                                                                          
    @Component.Factory                                                                                    
    interface Factory {                                                                                   
        AppComponent create(@BindsInstance Context context,                                               
                          @BindsInstance @Named("api_key") String apiKey);                                
    }                                                                                                     
}                                                                                                         
                                                                                                          
// Usage with runtime values                                                                              
AppComponent component = DaggerAppComponent.factory()                                                     
    .create(this, "secret-api-key-123");                                                                  
                                                                                                          

### **Component Dependencies**:
java
// Base component
@Component(modules = NetworkModule.class)                                                                 
@Singleton                                                                                                
public interface NetworkComponent {                                                                       
    OkHttpClient getHttpClient();                                                                         
}                                                                                                         
                                                                                                          
// Dependent component                                                                                    
@Component(dependencies = NetworkComponent.class, modules = UserModule.class)                             
@UserScope                                                                                                
public interface UserComponent {                                                                          
    UserService getUserService();  // Can use OkHttpClient from NetworkComponent                          
}                                                                                                         
                                                                                                          

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **Key Benefits of @Component**

### **✅ Type Safety**
- Compile-time verification of dependencies
- No runtime dependency resolution failures
- Clear error messages for missing dependencies

### **✅ Performance**
- No reflection at runtime
- Direct method calls in generated code
- Optimal object creation paths

### **✅ Maintainability**
- Clear contract between your code and DI framework
- Easy to see what dependencies are available
- Centralized dependency configuration

### **✅ Testability**
- Easy to create test components with mock modules
- Can inject test doubles for any dependency
- Isolated testing of individual components

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


## **Summary**

@Component is the control center of Dagger:

1. You define what you want (interface methods)
2. Dagger generates how to create it (implementation class)
3. You use the generated code to get dependencies
4. Dagger handles all the complex wiring automatically

It's the bridge that connects your dependency needs with Dagger's dependency provision capabilities! 🚀
