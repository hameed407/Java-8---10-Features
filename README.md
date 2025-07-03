# Java-8---10-Features
---

## 📘 Java 8 Features in Spring Boot Microservices: Real-World Scenarios & Code

> Use this as a quick reference when working with enterprise Spring Boot apps.

---

### ✅ 1. **Lambda Expressions**

**📌 Scenario:** Trigger async tasks, simplify filtering or mapping logic in service layers.

**💡 Why:** Avoid verbose anonymous classes and make code concise and readable.

```java
// Controller triggers background email job
@PostMapping("/trigger-mails")
public ResponseEntity<Void> triggerAsyncMails() {
    new Thread(() -> emailService.sendBulkMails()).start();
    return ResponseEntity.accepted().build();
}
```

**✔ Use in:** Threads, Stream APIs (`filter`, `map`, `sort`), event listeners.

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

