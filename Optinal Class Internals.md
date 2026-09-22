## What is the Optional class? Why was it introduced?

### 1. Why It Was Introduced

Before Java 8, returning `null` to represent "no result found" had fundamental flaws:

* **Implicit & Silent:** A method signature like `User findById(String id)` gave no clue whether it could return `null`. The caller had to guess or read documentation.
* **Defensive Boilerplate:** Code was littered with nested `if (user != null)` checks.
* **The "Billion-Dollar Mistake":** Forgetting a single check led straight to a production `NullPointerException`.

`Optional` transforms this into an **explicit API contract**:

```java
// Clear contract: The caller is forced by the compiler's type system
// to acknowledge and handle the possibility that a User might not exist.
public Optional<User> findById(String id);

```

---

### 2. Core API Methods

#### Creating an `Optional`

* `Optional.of(value)`: Expects a non-null value. Throws `NullPointerException` immediately if `value` is null.
* `Optional.ofNullable(value)`: Returns an `Optional` with the value if non-null, or `Optional.empty()` if null.
* `Optional.empty()`: Explicitly returns an empty instance.

#### Consuming / Extracting Values

* `.orElse(defaultValue)`: Returns the value if present, otherwise returns `defaultValue` (eagerly evaluated).
* `.orElseGet(Supplier<? extends T>)`: Returns the value if present, otherwise invokes the `Supplier` to lazily generate the default.
* `.orElseThrow(Supplier<? extends X>)`: Throws a custom exception if the value is absent.
* `.ifPresent(Consumer<? super T>)`: Executes a `Consumer` action only if a value is present.

#### Functional Transformations

* `.map(Function)`: Transforms the wrapped value.
* `.flatMap(Function)`: Flattens nested `Optional<Optional<T>>` into `Optional<T>`.
* `.filter(Predicate)`: Retains the value only if it matches the predicate; otherwise turns into `Optional.empty()`.

---

### 3. Critical Production Best Practices (Senior-Level Nuances)

Interviewers frequently test whether you know **how NOT to use** `Optional`:

1. **Intended Solely for Method Return Types:**
* Do **not** use `Optional` as class fields or instance variables—`Optional` does not implement `java.io.Serializable`.
* Do **not** use `Optional` as method parameters—it forces callers to wrap values unnecessarily and creates clutter (`foo(Optional.of("bar"))`).


2. **Never Call `.get()` Blindly:**
* Using `if (opt.isPresent()) { opt.get(); }` completely defeats the purpose; it's just `if (x != null)` in disguise. Prefer `.orElse()`, `.orElseGet()`, `.orElseThrow()`, or `.map()`.


3. **`orElse()` vs `orElseGet()`:**
* `orElse(expensiveComputation())` executes the method **eagerly** every single time, even when the `Optional` is present.
* `orElseGet(() -> expensiveComputation())` executes the `Supplier` **lazily** only when the `Optional` is actually empty.


4. **Never Return `null` for an `Optional`:**
* Always return `Optional.empty()`, never `return null;`.



---
> 
> 
>
## Optional.orElse() vs Optional.orElseGet()?

---

### Core Mechanics Comparison

| Feature | `Optional.orElse(T other)` | `Optional.orElseGet(Supplier<? extends T> supplier)` |
| --- | --- | --- |
| **Parameter** | Direct value / object reference (`T`) | Functional interface `Supplier<T>` |
| **Evaluation Strategy** | **Eager** (always evaluated at call time) | **Lazy** (invoked *only* if the `Optional` is empty) |
| **Performance Impact** | Incurs computation cost even if the value is present | Zero computation cost if the value is already present |
| **Best Used For** | Pre-computed, static constants, or cheap literals | Database queries, API calls, expensive allocations |

---

### Code Demonstration: The Eager vs. Lazy Trap

Consider a method that fetches or generates a default user:

```java
public User getDefaultUser() {
    System.out.println("--> Calling expensive DB/Network fallback...");
    return new User("default_user");
}

```

#### Case 1: The Optional is NOT empty (Value is Present)

```java
Optional<User> optionalUser = Optional.of(new User("john_doe"));

// Using orElse:
User user1 = optionalUser.orElse(getDefaultUser());
// Output: "--> Calling expensive DB/Network fallback..."
// Result: getDefaultUser() WAS executed, and the resulting object was thrown away!

// Using orElseGet:
User user2 = optionalUser.orElseGet(() -> getDefaultUser());
// Output: (Nothing logged!)
// Result: The lambda is never invoked because optionalUser contains a value.

```

In the `orElse` case, Java must evaluate the method arguments **before** passing them into `orElse()`. Even though the value was present, the expensive network call ran anyway.

#### Case 2: Unintended Side Effects

If your fallback method writes to a database or produces an audit log:

```java
// Danger: Creates a record in the database even if 'user' is found!
User result = optionalUser.orElse(userRepository.save(new User("fallback")));

```

`orElseGet` prevents this bug completely because the supplier is only executed when the container is empty.

---

### Rule of Thumb

* Use **`orElse(...)`** when you have a **cheap, constant, or already-instantiated literal**:
```java
String name = optionalName.orElse("UNKNOWN");
int count = optionalCount.orElse(0);

```


* Use **`orElseGet(...)`** whenever the default involves a **method call, constructor allocation (`new Object()`), or expensive I/O**:
```java
User user = optionalUser.orElseGet(() -> userRepository.createDefault());
List<Item> items = optionalList.orElseGet(ArrayList::new);

```



---
