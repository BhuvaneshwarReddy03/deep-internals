## What is an exception?
An **exception** is an abnormal event or condition that occurs during the execution of a program, disrupting the normal instruction flow.

In Java, an exception is a first-class object that encapsulates the error details: what went wrong, where it happened (the execution stack trace), and the program state at the moment of failure.

---

### The Java Exception Hierarchy

Every error and exception inherits from `java.lang.Throwable`:

```
               Throwable
              /         \
         Exception       Error (e.g., OutOfMemoryError, StackOverflowError)
        /         \
RuntimeException   Checked Exceptions (e.g., IOException, SQLException)
(Unchecked)
(e.g., NullPointerException,
       IndexOutOfBoundsException)

```

1. **`Throwable`:** The root class of the entire error-handling hierarchy.
2. **`Error`:** Severe, irrecoverable system-level issues caused by the JVM environment (e.g., `OutOfMemoryError`, `StackOverflowError`). Applications should **not** attempt to catch these.
3. **`Exception`:** Conditions that a reasonable application might want to catch, handle, or recover from.

---

### The Two Major Types of Exceptions

| Category | Description | Compiler Enforcement | Common Examples |
| --- | --- | --- | --- |
| **Checked Exceptions**<br>

<br>*(Compile-time)* | External failures that a well-written program should anticipate and recover from. | **Mandatory:** Must be caught via `try-catch` or declared via `throws`. | `IOException`, `SQLException`, `ClassNotFoundException` |
| **Unchecked Exceptions**<br>

<br>*(Runtime / `RuntimeException`)* | Programming flaws, logic bugs, or bad API usage. | **Optional:** Compiler does not enforce handling or declaring them. | `NullPointerException`, `ArrayIndexOutOfBoundsException`, `ArithmeticException` |

---

### How Java Handles Exceptions Under the Hood

1. **Throwing:** When a failure occurs, the JVM (or your code via `throw new ...`) instantiates an exception object, capturing the call stack, and halts the current code block.
2. **Propagating:** The JVM traverses backward up the call stack frame by frame looking for a matching `catch` block.
3. **Handling / Termination:** If an enclosing block catches it, execution resumes in the `catch`/`finally` blocks. If it bubbles to the top of `main()` unhandled, the thread terminates and prints the stack trace.

---

## What is the difference between checked and unchecked exceptions?

That was a very solid, technically accurate explanation. You hit all the major points an interviewer listens for:

1. **Hierarchy:** Direct subclasses of `Exception` (Checked) vs. subclasses of `RuntimeException` (Unchecked).
2. **Compiler enforcement:** Mandatory (`try-catch` / `throws`) vs. optional.
3. **Root causes:** External/environmental conditions vs. logic and programming flaws.
4. **Concrete examples:** `IOException` for checked; logic-based bugs for unchecked.

---

### Key Comparison Table for Rapid Recall

| Feature | Checked Exceptions | Unchecked Exceptions |
| --- | --- | --- |
| **Inheritance** | Extends `java.lang.Exception` (excluding `RuntimeException`) | Extends `java.lang.RuntimeException` |
| **Compiler Check** | Checked at **compile-time** (mandates `try-catch` or `throws`) | Ignored by compiler at compile-time |
| **Nature of Problem** | External or environmental failures outside direct code control | Internal programming bugs, logic flaws, or illegal API usage |
| **Recovery Strategy** | Application should anticipate and gracefully recover | Code should be fixed to prevent the condition entirely |
| **Examples** | `IOException`, `SQLException`, `ClassNotFoundException` | `NullPointerException`, `ArrayIndexOutOfBoundsException`, `ArithmeticException` |

---
## What is the difference between Error and Exception?

### Core Differences at a Glance

| Feature | `Error` | `Exception` |
| --- | --- | --- |
| **Root Cause** | System/JVM-level failures, resource exhaustion | Program logic bugs, bad input, or external failures |
| **Recoverability** | **Irrecoverable**; indicates environment or memory failure | **Recoverable**; application should handle gracefully |
| **Handling Rule** | **Do not catch**; catching can leave the JVM in an unstable state | **Catch and handle** using `try-catch` blocks |
| **Classification** | Always **unchecked** | Split into **Checked** (compile-time) and **Unchecked** (runtime) |
| **Typical Origin** | Thrown almost exclusively by the **JVM** | Thrown by application code, libraries, or JVM |
| **Examples** | `OutOfMemoryError`, `StackOverflowError`, `VirtualMachineError` | `NullPointerException`, `IOException`, `SQLException` |

---

### Key Nuance for an Interview: *Can* You Catch an `Error`?

Interviewers love asking: *"Syntactically, can you write `catch (OutOfMemoryError e)` or `catch (Error e)`?"*

* **Syntactically:** Yes, because `Error` extends `Throwable`, Java syntax permits catching it.
* **Architecturally:** **No, you never should.** If an `OutOfMemoryError` occurs, the JVM heap is exhausted. If you catch it, the JVM cannot guarantee object consistency, thread safety, or garbage collection stability. The application will run in a corrupted, unpredictable state.

---

## Should you catch Throwable?
**No, in general application code you should almost never catch `Throwable`.**

While it is syntactically legal, doing so is considered a severe anti-pattern in Java.

---

### Why Catching `Throwable` Is Dangerous

`Throwable` is the root superclass of both `Exception` and `Error`:

```
          Throwable
         /         \
   Exception        Error (OutOfMemoryError, StackOverflowError, etc.)

```

When you write `catch (Throwable t)`:

1. **You accidentally swallow JVM-level `Error`s:**
It intercepts fatal errors like `OutOfMemoryError`, `StackOverflowError`, `VirtualMachineError`, and `ThreadDeath`.
2. **The JVM is left in an inconsistent/corrupted state:**
Errors indicate that the runtime environment is broken or out of resources. If you catch an `OutOfMemoryError`, locks may be held, objects may be half-initialized, and garbage collection may be failing. Suppressing the error keeps a "zombie" process running rather than letting it crash cleanly.
3. **It masks non-recoverable bugs:**
Normal business logic should only catch conditions it knows how to handle or recover from. Catching `Throwable` hides catastrophic issues from monitoring, orchestration tools (like Kubernetes or systemd), and logs.

---

### The Only Valid Edge Cases

There are only two rare scenarios where catching `Throwable` is acceptable:

* **Top-Level Framework / Container Boundaries:**
In thread pool runners, web server loops (e.g., Tomcat/Netty request loops), or background task executors:
```java
try {
    task.run();
} catch (Throwable t) {
    logger.error("Fatal failure in background task", t);
    // Clean up resources or initiate graceful shutdown
}

```


Here, the framework catches `Throwable` only to **log the fatal incident** or safely shutdown/restart the worker thread, not to pretend the operation succeeded.
* **Logging Just Before Crashing:**
Catching `Throwable` at the absolute root of an entrypoint to ensure the crash is written to an external alerting system before re-throwing or calling `System.exit(1)`.

---

### What You Should Catch Instead

* Catch **specific exceptions** whenever possible:
```java
catch (IOException | SQLException e)

```


* At the top of your controller or service layer, catch **`Exception`**, never `Throwable`:
```java
catch (Exception e) // Catches all checked & unchecked exceptions, lets Errors pass

```



---

## When should you use checked exceptions vs unchecked exceptions?

The question **"When should you use checked vs. unchecked exceptions?"** is **NOT** just about classes you write.

In everyday Java, you don't even need to build a custom exception to "use" exceptions. Java already provides dozens of built-in standard exceptions (`IllegalArgumentException`, `IllegalStateException`, `IOException`, `FileNotFoundException`).

**"Using" an exception simply means choosing which standard Java exception to `throw` or `catch` in your code.**

---

### 1. Using Standard Built-in Exceptions (Zero Custom Classes)

Look at standard Java validation in a regular service:

```java
public void setAge(int age) {
    if (age < 0) {
        // You are "using" Java's built-in UNCHECKED exception:
        throw new IllegalArgumentException("Age cannot be negative");
    }
    this.age = age;
}

```

* You didn't create a custom class here.
* You chose to **use** `IllegalArgumentException` (which extends `RuntimeException`).
* Why? Because passing negative age is a **caller bug / logic flaw**. The caller broke the method contract.

Now look at reading a configuration file using built-in Java classes:

```java
// Java's built-in FileReader "uses" a CHECKED exception:
public String loadConfig(String path) throws IOException {
    FileReader reader = new FileReader(path); // throws FileNotFoundException (checked)
    // reading logic...
}

```

* You didn't create a custom class here either.
* Java's creators chose to **use** `IOException` (which extends `Exception`).
* Why? Because a missing file is an **external, environmental failure** that any valid program must anticipate and handle.

---

### 2. Why the Distinction Exists Beyond Custom Classes

When an interviewer asks: *"When do you use checked vs. unchecked?"*, they are asking:

> **"As a developer writing methods and APIs, how do you decide whether a failure should be checked or unchecked?"**

Here is the decision rule using standard Java built-ins:

| Scenario | What you use | Built-in Java Example | Why |
| --- | --- | --- | --- |
| **Client passed bad data** | **Unchecked** | `throw new IllegalArgumentException(...)` | It's a programming bug; caller must fix their code, not catch it. |
| **Object is in wrong state** | **Unchecked** | `throw new IllegalStateException(...)` | Calling `.next()` on an empty Iterator; fixing code is required. |
| **A value is unexpectedly null** | **Unchecked** | `throw new NullPointerException(...)` | Code defect. |
| **External network / disk failure** | **Checked** | `throws IOException` | External system failed; caller must write fallback or retry logic. |
| **Database connection drops** | **Checked** | `throws SQLException` | Environmental failure outside the application's control. |

---

## What is try-with-resources? What problem does AutoCloseable solve?

**`try-with-resources`** (introduced in Java 7) is an exception-handling construct that automatically closes external resources—such as database connections, file streams, or network sockets—when the `try` block exits, whether it finishes normally or throws an exception.

Any object declared inside the parentheses of the `try(...)` statement must implement the **`java.lang.AutoCloseable`** (or `java.io.Closeable`) interface.

---

### The Problem It Solves: The Flaws of Traditional `finally`

Before Java 7, developers had to manually release resources in a `finally` block. This introduced three major problems in production systems:

#### 1. Verbose, Ugly Boilerplate

Closing a resource can itself throw an exception (e.g., `close()` throws `IOException`). Developers had to nest `try-catch` blocks inside the `finally` block:

```java
// Pre-Java 7: Verbose, error-prone, hard to read
FileInputStream fis = null;
try {
    fis = new FileInputStream("app.log");
    fis.read();
} catch (IOException e) {
    // handle read failure
} finally {
    if (fis != null) {
        try {
            fis.close();
        } catch (IOException e) {
            // handle close failure
        }
    }
}

```

#### 2. Exception Masking (Swallowing the Root Cause)

If the `try` block threw an exception (e.g., `fis.read()` failed), execution jumped to `finally`. If `fis.close()` also threw an exception, **the second exception completely erased the first one**. The developer lost the actual root cause of the crash in logs.

#### 3. Resource Leaks

If multiple resources were closed in one `finally` block, an exception thrown by the first `.close()` would abort the block, leaving subsequent resources open and leaking connections or file handles.

---

### How `try-with-resources` Solves These Problems

With `try-with-resources`, the JVM generates the cleanup code behind the scenes:

```java
// Java 7+: Clean, safe, automatic
try (FileInputStream fis = new FileInputStream("app.log")) {
    fis.read();
} catch (IOException e) {
    // Both read() and close() exceptions land here
}

```

1. **Automatic Lifecycle:** The JVM guarantees `close()` is called as soon as the block terminates.
2. **Reverse Order Cleanup:** If multiple resources are opened in the header, they are closed automatically in **reverse order of creation**:
```java
try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement(query)) {
    // ps closes first, then conn closes
}

```


3. **Exception Suppression:** If both the `try` body and the automatic `.close()` throw exceptions, the exception from the `try` body is preserved as the **primary exception**, while the `.close()` exception is attached to it via **`addSuppressed()`**. You can retrieve it using `e.getSuppressed()`.

---

### The Role of `AutoCloseable`

`AutoCloseable` is the single-method functional interface that powers this construct:

```java
public interface AutoCloseable {
    void close() throws Exception;
}

```

* **The Problem It Solves:** It creates a **universal contract** for any resource that holds system resources (sockets, native handles, DB pools, buffers). By having standard classes and custom classes implement this interface, the JVM knows it can safely invoke `.close()` on them without reflection or custom hooks.

```java
// Custom resource implementing AutoCloseable
public class DatabaseClient implements AutoCloseable {
    public void execute() { /* ... */ }

    @Override
    public void close() {
        System.out.println("Connection released back to pool.");
    }
}

// Usage
try (DatabaseClient client = new DatabaseClient()) {
    client.execute();
} // close() is called automatically here

```

---
## What is the best way to handle exceptions in layered applications?
You have the core architectural principle right: **let exceptions bubble up naturally to a centralized global handler instead of catching and swallowing them in intermediate layers.**

However, in an enterprise layered architecture (Controller $\rightarrow$ Service $\rightarrow$ Repository/Data), simply letting *every* raw exception pass through untouched can leak implementation details.

The gold-standard strategy combines **layer boundaries, translation, and centralized handling**.

---

### The 4-Pillar Strategy for Layered Applications

```
[ Repository Layer ]  ──> Translates vendor/SQL errors into Domain/Data Exceptions
         │
[ Service Layer ]     ──> Enforces business rules; throws Business Exceptions (Unchecked)
         │
[ Controller Layer ]  ──> Keeps methods clean (NO try-catch blocks)
         │
[ Global Handler ]    ──> Intercepts exceptions (@ControllerAdvice) & returns structured API response

```

#### 1. Repository / Data Layer: Wrap and Abstract

* **Never let raw infrastructure exceptions leak** into the business tier.
* Catch database-specific exceptions (e.g., `SQLException`, vendor driver errors) and rethrow them as meaningful data access exceptions using **exception chaining**:
```java
// In Repository
try {
    db.execute(...);
} catch (SQLException e) {
    throw new DataAccessException("Failed to query user records", e);
}

```


*(Note: Frameworks like Spring do this automatically via Spring’s `DataAccessException` hierarchy).*

#### 2. Service Layer: Validate and Throw Domain Exceptions

* Do **not** write `try-catch` blocks around business calls unless you have a concrete fallback plan (like a local cache).
* Validate business rules and throw domain-specific **unchecked exceptions**:
```java
// In Service
public User getUser(String id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
}

```



#### 3. Controller Layer: Keep it Lean and Declarative

* **Zero `try-catch` blocks** in controller methods.
* Controllers should focus solely on routing, payload deserialization, and invoking services. Let all exceptions bubble right past them.

#### 4. Global Exception Handler: The Single Boundary

* Use a centralized handler (like Spring's `@RestControllerAdvice` or equivalent framework filter) to catch exceptions at the edge of the system.
* Map specific exception types to precise HTTP status codes and return a standardized JSON error contract (e.g., RFC 7807 Problem Details):
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse body = new ErrorResponse("NOT_FOUND", ex.getMessage(), Instant.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(body);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        // Log the internal error with stack trace for observability
        logger.error("Unhandled exception occurred", ex);
        // Return generic message to client to avoid leaking internals
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred", Instant.now()));
    }
}

```



---

### What to Avoid in Layered Architectures

* **The "Log and Rethrow" Anti-Pattern:**
```java
// BAD: Logs the same exception 3 times across Repository, Service, and Controller
catch (Exception e) {
    logger.error("Error", e);
    throw e;
}

```


*Rule:* **Either log it or throw it—never do both.** Let the global handler or root boundary log it once.
* **Swallowing Exceptions:** Catching an exception without rethrowing or logging, leaving callers unaware of failures.
* **Leaking Stack Traces to Clients:** Exposing raw internal traces or SQL errors in API responses creates severe security vulnerabilities.

---
## Can you have an empty catch block? Why is it bad practice?

Yes, syntactically Java permits an empty catch block, but it is considered one of the worst anti-patterns in software engineering—commonly called **exception swallowing** or **the black hole anti-pattern**.

Your reasoning is spot on: catching an exception means taking responsibility for it. If you swallow it silently, you destroy the diagnostics needed to fix bugs or trigger fallbacks.

---

### Why an Empty Catch Block Is a Severe Anti-Pattern

```java
// Anti-pattern: Swallowing the exception
try {
    accountService.debit(amount);
} catch (Exception e) {
    // Empty: The program pretends this never failed!
}

```

1. **Destroys Observability & Root Causes:**
The stack trace, error message, and line numbers vanish completely. Monitoring tools (APM, Datadog, ELK) will show zero failures, making production bugs nearly impossible to track down.
2. **Corrupted / Inconsistent State:**
If a database write or file update fails halfway through and the exception is swallowed, the application continues running as if the operation succeeded, leading to corrupted data downstream.
3. **Breaks Caller Assumptions:**
Callers have no idea that the operation failed and will proceed with dependent logic that assumes success.
4. **Violates the Golden Rule:**
Whenever you catch an exception, you must do at least one of three things:
* **Handle & Fallback:** Execute an alternate flow (e.g., read from cache, retry).
* **Wrap & Rethrow:** Chain it into a domain exception and rethrow it (`throw new ServiceException("...", e)`).
* **Log & Abort:** Log the stack trace with sufficient context and return a safe default or terminal status.



---

### Is There Ever a Valid Exception?

There is only one rare scenario where an empty catch block is tolerable: **ignoring an exception that is genuinely expected, harmless, and unrecoverable**.

Even in that rare case, industry standards (and tools like SonarQube / Checkstyle) mandate **naming the variable `ignored**` and documenting the reason with a comment:

```java
try {
    Thread.sleep(100);
} catch (InterruptedException ignored) {
    // Thread interruption during sleep is intentionally ignored here because...
    Thread.currentThread().interrupt(); // Restore interrupted status
}

```

---
## Multi-catch, rethrowing with precise types
In Java 7, two separate but closely related features were introduced to clean up exception handling: **Multi-Catch** and **Rethrowing with More Precise Exception Types** (often called *precise rethrow* or *inclusive typing*).

Together, they eliminate boilerplate without losing type safety or forcing broad `throws` clauses.

---

### 1. Multi-Catch (The Alternative Pipe `|`)

Before Java 7, handling multiple distinct exceptions identically required duplicate `catch` blocks:

```java
// Pre-Java 7: Verbose and repetitive
try {
    process();
} catch (IOException e) {
    logger.error("Failed to process", e);
} catch (SQLException e) {
    logger.error("Failed to process", e);
}

```

With **Multi-Catch**, you combine them into a single block using the pipe (`|`) symbol:

```java
// Java 7+: Clean and concise
try {
    process();
} catch (IOException | SQLException e) {
    logger.error("Failed to process", e);
}

```

#### Key Rules of Multi-Catch

* **Implicitly `final`:** The caught exception parameter `e` is **implicitly final**. You cannot reassign it (e.g., `e = new IOException()` will not compile).
* **Disjoint Types Only (No Subclasses):** The types separated by `|` cannot have an inheritance relationship (subclass/superclass). If one is a subtype of the other, it causes a compilation error:
```java
// COMPILE ERROR: FileNotFoundException is a subclass of IOException
catch (FileNotFoundException | IOException e) { ... }

```



---

### 2. Rethrowing with Precise Exception Types

Before Java 7, if you caught a general `Exception` to do some cross-cutting logic (like logging or metrics) and rethrew it, the compiler forced your method signature to declare `throws Exception`:

```java
// Pre-Java 7: Leaky and imprecise signature
public void doWork() throws Exception { // Forced to declare throws Exception!
    try {
        if (condition) throw new IOException();
        else throw new SQLException();
    } catch (Exception e) {
        logger.error("Encountered error", e);
        throw e; // Compiler saw 'e' as Exception, not the exact types
    }
}

```

This was terrible for callers because they lost the ability to catch the specific checked exceptions (`IOException` or `SQLException`).

#### Java 7's "Precise Rethrow" Solution

From Java 7 onward, the compiler performs static flow analysis:

1. Even if you write `catch (Exception e)`, the compiler checks what **can actually be thrown** from inside the `try` block.
2. If you rethrow `e` without reassigning it, the compiler treats the rethrow as the **precise union of checked exceptions** actually thrown in the `try` block.

```java
// Java 7+: Precise rethrow
public void doWork() throws IOException, SQLException { // Precise signature!
    try {
        if (condition) throw new IOException();
        else throw new SQLException();
    } catch (Exception e) {
        logger.error("Logging before rethrowing", e);
        throw e; // Compiler knows 'e' can ONLY be IOException or SQLException!
    }
}

```

#### The Golden Rule for Precise Rethrow

For precise rethrow to work, the variable `e` must be **effectively final** (you must not reassign `e = ...` inside the catch block). If you reassign `e`, the compiler falls back to treating it as a generic `Exception`, breaking the precise method signature.

---

### Comparison Summary

| Feature | Multi-Catch | Precise Rethrow |
| --- | --- | --- |
| **Syntax** | `catch (TypeA | TypeB e)` | `catch (Exception e) { ... throw e; }` |
| **Method Signature** | Handled locally or declared specifically | Declares exact types (`throws TypeA, TypeB`) |
| **Parameter Mutability** | Explicitly / implicitly `final` | Must be **effectively final** to preserve types |
| **Purpose** | Reduces duplicated handler code | Intercepts general exceptions without losing precise signatures |

---
## Exception wrapping / translation pattern for microservices

The **Exception Translation / Wrapping Pattern** in a microservices architecture is the practice of intercepting low-level technical or downstream service errors at the boundaries of a service, translating them into meaningful domain exceptions, and finally mapping those domain exceptions into a standardized API contract for upstream consumers.

In distributed systems, letting raw errors pass unmanaged creates tight coupling, leaks internal implementation details, and degrades observability.

---

### The 3 Architectural Tiers of Exception Translation

```
[ Downstream Service / DB ]
           │  (e.g., Feign 503, WebClient timeout, SQLException)
           ▼
┌────────────────────────────────────────────────────────┐
│ 1. Boundary Translation (Client / Adapter Layer)       │
│    - Intercept HTTP/gRPC/SQL errors                    │
│    - Wrap into Domain/Infrastructure Exception         │
│    - Preserve original cause (Exception Chaining)      │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 2. Core Domain / Service Layer                         │
│    - Business rules enforce domain constraints         │
│    - Throw clean, unchecked Business Exceptions        │
│      (e.g., InsufficientInventoryException)           │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 3. Edge Translation (Global API Advice / Gateway)      │
│    - Catch Domain Exceptions                           │
│    - Log full stack trace internally with Trace ID     │
│    - Return standardized error payload (RFC 7807)      │
└────────────────────────────────────────────────────────┘

```

---

### 1. Boundary Translation (Adapter / Downstream Layer)

When calling external dependencies (REST APIs, message brokers, databases), client libraries produce vendor-specific errors (e.g., `FeignException`, `WebClientResponseException`, `MongoException`).

**Anti-Pattern:** Letting `FeignException.NotFound` propagate directly to the controller. The business logic becomes tightly coupled to the fact that HTTP was used.

**Best Practice:** Translate it immediately inside an `ErrorDecoder` or repository adapter using exception chaining:

```java
// OpenFeign custom error decoder example
public class PaymentServiceErrorDecoder implements ErrorDecoder {

    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400 -> {
                return new InvalidPaymentRequestException("Invalid payment payload sent to gateway");
            }
            case 404 -> {
                return new PaymentAccountNotFoundException("Payment account does not exist");
            }
            case 503, 504 -> {
                // Translated into a domain retryable exception
                return new DownstreamServiceUnavailableException(
                    "Payment provider temporarily unavailable", 
                    response.status()
                );
            }
            default -> {
                return new ThirdPartyIntegrationException(
                    "Unhandled payment gateway error: " + response.status()
                );
            }
        }
    }
}

```

---

### 2. Domain Representation (Core Service Layer)

In the service layer, business rules throw domain-specific, unchecked exceptions that carry structured metadata:

```java
public class OrderService {

    public Order createOrder(OrderRequest request) {
        try {
            paymentClient.charge(request.getPaymentDetails());
        } catch (DownstreamServiceUnavailableException ex) {
            // Chaining and translating into a local business exception
            throw new OrderProcessingFailedException("Could not process order due to billing outage", ex);
        }

        return orderRepository.save(new Order(request));
    }
}

```

---

### 3. Edge Translation (Global Handler to RFC 7807)

At the entry point of the microservice, a centralized global exception handler maps internal exceptions into the industry-standard **RFC 7807 (Problem Details for HTTP APIs)** format.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(OrderProcessingFailedException.class)
    public ResponseEntity<ProblemDetail> handleOrderFailure(
            OrderProcessingFailedException ex, 
            HttpServletRequest request) {

        // 1. Log the full cause chain internally with distributed tracing context
        log.error("Order processing failed for URI: {}", request.getRequestURI(), ex);

        // 2. Build clean RFC 7807 response for the client (no leaked stack traces)
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
                HttpStatus.BAD_GATEWAY, 
                ex.getMessage()
        );
        problem.setTitle("Order Processing Failure");
        problem.setType(URI.create("https://api.myorg.com/errors/order-failed"));
        problem.setProperty("timestamp", Instant.now());

        return ResponseEntity.status(HttpStatus.BAD_GATEWAY).body(problem);
    }
}

```

---

### Why Exception Translation Is Mandatory in Microservices

| Concern | Without Translation | With Translation Pattern |
| --- | --- | --- |
| **Architectural Coupling** | Downstream vendor changes break upstream API contracts. | Boundary adapters isolate internal changes from the external contract. |
| **Security** | Raw stack traces and internal endpoints leak to consumers. | Edge handler returns sanitized RFC 7807 payloads. |
| **Resilience & Circuit Breakers** | Circuit breakers cannot tell client errors (4xx) from server outages (5xx). | Exceptions are categorized into **Retryable** vs **Non-Retryable** domain types. |
| **Observability** | Root causes get lost if caught and rethrown without chaining. | Full stack trace + downstream errors remain preserved via `e.getCause()`. |

---

### Key Production Rules

1. **Classify by Recoverability:** Split translated exceptions into two clear buckets:
* **Transient / Retryable** (e.g., `503 Service Unavailable`, `429 Too Many Requests`, timeouts) $\rightarrow$ trigger retry policies or circuit breakers.
* **Permanent / Non-Retryable** (e.g., `400 Bad Request`, `404 Not Found`, validation) $\rightarrow$ fail fast and alert the caller.


2. **Always Chain the Cause:** When wrapping downstream errors, pass the original exception into `super(message, cause)` to preserve distributed tracing headers and error details in logs.
3. **Propagate Trace IDs:** Always include the distributed trace ID (e.g., OpenTelemetry / Micrometer `traceId`) in the translated error response payload so client developers and server operators can correlate the failure across services.

---
