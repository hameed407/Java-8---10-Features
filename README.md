# Java-8---10-Features
---

## 📘 Java 8 Features in Spring Boot Microservices: Real-World Scenarios & Code

> Use this as a quick reference when working with enterprise Spring Boot apps.

---

### ✅ 1. **Lambda Expressions**

Sure! Here's a **realistic scenario** for the code you provided, written clearly as if it's part of documentation or GitHub notes:

---

### ✅ Scenario: Triggering a Bulk Email Job (One-time Admin Action)

#### 🧩 **Context / Use Case**:

An **admin user** wants to send a promotional email (like a discount or feature update) to **all registered users**. Since sending thousands of emails takes time and involves I/O (email servers), this task shouldn't block the admin's UI or API response.

The admin clicks a "Send Bulk Emails" button on the dashboard → an API is called to trigger the operation **in the background**.

---

Here's a complete working **Spring Boot async email flow** — perfect for GitHub documentation or reference.

---

## ✅ Async Bulk Email Flow in Spring Boot

> This demo shows how to trigger an **email job in the background** using `@Async` in a clean and production-friendly way.

---

### 🧩 Use Case:

An admin clicks **"Send Promo Emails"** → backend triggers email sending to all users → returns instantly (non-blocking) → emails sent in background.

---

## 🗂 Structure

* **Controller** → triggers the operation
* **Service** → performs email logic
* **Async Config** → enables and configures `@Async`
* **MailSender Setup** → basic email config

---

## 1️⃣ `EmailController.java`

```java
@RestController
@RequestMapping("/api/emails")
public class EmailController {

    private final AsyncEmailService asyncEmailService;

    public EmailController(AsyncEmailService asyncEmailService) {
        this.asyncEmailService = asyncEmailService;
    }

    @PostMapping("/trigger-bulk")
    public ResponseEntity<String> triggerBulkEmails() {
        asyncEmailService.sendBulkEmails();  // async call
        return ResponseEntity.accepted().body("📬 Email job triggered");
    }
}
```

---

## 2️⃣ `AsyncEmailService.java`

```java
@Service
public class AsyncEmailService {

    private final UserRepository userRepository;
    private final JavaMailSender mailSender;

    public AsyncEmailService(UserRepository userRepository, JavaMailSender mailSender) {
        this.userRepository = userRepository;
        this.mailSender = mailSender;
    }

    @Async("taskExecutor")
    public void sendBulkEmails() {
        List<User> users = userRepository.findAll();

        for (User user : users) {
            try {
                SimpleMailMessage mail = new SimpleMailMessage();
                mail.setTo(user.getEmail());
                mail.setSubject("🎁 Special Offer Just for You!");
                mail.setText("Hi " + user.getName() + ",\n\nGet 25% OFF on your next order.\n\nRegards,\nYourApp Team");

                mailSender.send(mail);
                System.out.println("✅ Sent to: " + user.getEmail());

            } catch (Exception ex) {
                System.err.println("❌ Failed to send to: " + user.getEmail());
                ex.printStackTrace();
            }
        }

        System.out.println("📨 Bulk email job finished.");
    }
}
```

---

## 3️⃣ `AppConfig.java` (Enable Async + Executor)

```java
@Configuration
@EnableAsync
public class AppConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("AsyncMail-");
        executor.initialize();
        return executor;
    }
}
```

---

## 4️⃣ `application.properties`

```properties
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your_email@gmail.com
spring.mail.password=your_app_password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

---

## 5️⃣ Example `User.java` and `UserRepository.java`

```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;

    private String name;
    private String email;

    // getters/setters
}

public interface UserRepository extends JpaRepository<User, Long> {}
```

---

## ✅ Output Example (in logs)

```
📬 Email job triggered
✅ Sent to: alice@example.com
✅ Sent to: bob@example.com
❌ Failed to send to: invalid@...
📨 Bulk email job finished.
```

---

### ✅ What’s Achieved:

* Non-blocking REST call
* Emails sent in background
* Simple `ThreadPool` for scalability
* Fail-safe with try/catch
* Easy to extend or schedule

---

🧠 Why Is It Needed?
Because @Async uses Spring AOP (Aspect-Oriented Programming) under the hood to wrap the method in a separate thread (from a thread pool), and Spring needs to be explicitly told to enable this behavior.

📦 Where to Place It?
You can place this class in:

Your main application class:

java
Copy
Edit
@SpringBootApplication
@EnableAsync
public class YourApplication {
    public static void main(String[] args) {
        SpringApplication.run(YourApplication.class, args);
    }
}
Or a separate configuration class:

java
Copy
Edit
@Configuration
@EnableAsync
public class AppConfig {}
✅ Best practice: Use option #1 if you want it globally enabled in your app.

🔧 Optional: Customize the Thread Pool (Advanced)
By default, Spring uses a simple thread executor, which isn’t great for heavy production usage.

You can define a custom TaskExecutor like this:

java
Copy
Edit
@Configuration
@EnableAsync
public class AppConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);     // threads always kept alive
        executor.setMaxPoolSize(50);      // max concurrent threads
        executor.setQueueCapacity(100);   // queue before new threads
        executor.setThreadNamePrefix("Async-Thread-");
        executor.initialize();
        return executor;
    }
}
Then annotate your async method like this to use the custom pool:

java
Copy
Edit
@Async("taskExecutor")
public void sendBulkMails() {
    // ...
}

---

### ✅ 2. **Functional Interfaces**

**📌 Scenario:** Define domain-specific logic in a reusable way for filtering, mapping, or validation.

**💡 Why:** Promote clean separation of logic using `Predicate`, `Function`, `Consumer`, etc.

```java
// Service method to get emails of active users
Predicate<User> isActive = User::isActive;
Function<User, String> getEmail = User::getEmail;

public List<String> getActiveUserEmails(List<User> users) {
    return users.stream()
        .filter(isActive)
        .map(getEmail)
        .collect(Collectors.toList());
}
```

**✔ Use in:** Validation rules, stream chains, repository filtering logic.

---

### ✅ 3. **Streams API**

**📌 Scenario:** Transform list of entities into DTOs, filter records based on conditions, group/summarize data.

**💡 Why:** Clean, functional, declarative way to handle collections.

```java
// Convert users to DTOs and filter verified users
public List<UserDTO> getVerifiedUserDTOs() {
    return userRepository.findAll().stream()
        .filter(User::isVerified)
        .map(user -> new UserDTO(user.getId(), user.getName()))
        .collect(Collectors.toList());
}
```

**✔ Use in:** DTO mapping, data transformation, filtering lists, aggregation.

---

### ✅ 4. **Optional API**

**📌 Scenario:** Access nested objects (e.g., `user.profile.username`) without null checks.

**💡 Why:** Prevent `NullPointerException` with safe chaining.

```java
// Get username if available, or return 'Guest'
public String resolveUsername(User user) {
    return Optional.ofNullable(user)
        .map(User::getProfile)
        .map(Profile::getUsername)
        .orElse("Guest");
}
```

**✔ Use in:** Configuration values, entity access, avoiding deep null checks.

---

### ✅ 5. **Method References**

**📌 Scenario:** When using the same method repeatedly in lambdas (e.g., DTO constructors, logging).

**💡 Why:** Makes lambda expressions shorter and more readable.

```java
// DTO constructor reference
List<UserDTO> dtos = users.stream()
    .map(UserDTO::new)
    .collect(Collectors.toList());

// Logging
users.forEach(System.out::println);
```

**✔ Use in:** Stream operations, logging, mapping to constructors.

---

### ✅ 6. **Default Methods in Interfaces**

**📌 Scenario:** Audit time or common utility logic in interfaces like `Auditable`, `Retryable`.

**💡 Why:** Avoid creating utility classes for common methods.

```java
public interface Auditable {
    default LocalDateTime now() {
        return LocalDateTime.now();
    }
}

// In service class
public class OrderService implements Auditable {
    public void createOrder(Order order) {
        order.setCreatedAt(now());
    }
}
```

**✔ Use in:** Interfaces for time tracking, retry logic, fallback behaviors.

---

### ✅ 7. **Date & Time API (`java.time`)**

**📌 Scenario:** Replace legacy `Date`, handle expiry, durations, timestamps cleanly.

**💡 Why:** Immutable, clear API for handling dates in microservices.

```java
@Entity
public class Token {
    private LocalDateTime createdAt = LocalDateTime.now();
    private LocalDateTime expiresAt = createdAt.plusMinutes(15);
}
```

**✔ Use in:** Audit fields, session expiry, token generation, job scheduling.

---

### ✅ 8. **Collectors Utility**

**📌 Scenario:** Group users by role, collect emails as comma-separated values.

**💡 Why:** Collectors give expressive power for transformations.

```java
// Group users by role
Map<String, List<User>> byRole = users.stream()
    .collect(Collectors.groupingBy(User::getRole));

// Join emails for CSV export
String csv = users.stream()
    .map(User::getEmail)
    .collect(Collectors.joining(","));
```

**✔ Use in:** Data grouping, statistics, reporting, CSV export.

---

### ✅ 9. **Parallel Streams**

**📌 Scenario:** Parallelize CPU-intensive tasks (e.g., image processing, PDF generation).

**💡 Why:** Utilize multi-core processors in background batch jobs.

```java
// Parallel processing for heavy calculations
public List<Report> generateReports(List<Data> dataset) {
    return dataset.parallelStream()
        .map(this::generate)
        .collect(Collectors.toList());
}
```

> ⚠️ Not recommended for DB or I/O-bound operations.

**✔ Use in:** Report generation, analytics, file processing.

---

### ✅ 10. **CompletableFuture (Async)**

**📌 Scenario:** Send notifications without blocking user request (non-blocking microservice call).

**💡 Why:** Better than `ExecutorService` for simple async tasks.

```java
// In async service
public void notifyUser(Order order) {
    CompletableFuture.runAsync(() -> {
        emailService.sendOrderNotification(order.getId());
    });
}
```

**✔ Use in:** Background email/sms, parallel microservice calls.

---

