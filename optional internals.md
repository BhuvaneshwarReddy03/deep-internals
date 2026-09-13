## What is the difference between Optional and null? When should Optional be used?

The primary purpose of inventing `Optional` was **API clarity and eliminating unexpected `NullPointerException` (NPE)**.

---

### The Problem With `null`

Before Java 8, when a method returned an object:

```java
public User findUserById(String id) {
    // might return a User, or might return null
}

```

The method signature gives **zero clue** that the value might be missing.

* The developer calling `findUserById("123")` might forget to write `if (user != null)`, immediately call `user.getName()`, and crash in production with a `NullPointerException`.
* To stay safe, codebases ended up cluttered with defensive, deeply nested null checks:
```java
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        City city = address.getCity();
        if (city != null) { ... }
    }
}

```



---

### What `Optional` Solves

`Optional<T>` is a single-value container that either holds a non-null reference or is empty.

```java
public Optional<User> findUserById(String id) { ... }

```

1. **Explicit API Contract:** The method signature explicitly warns you: *"This value might not exist. You must think about the absent case before using it."*
2. **Eliminates Nested Checks:** Provides functional chain methods like `.map()`, `.flatMap()`, `.filter()`, and `.orElse()`:
```java
String cityName = findUserById(id)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown City");

```



---

### Core Differences

| Dimension | `null` | `Optional<T>` |
| --- | --- | --- |
| **Nature** | A primitive literal / absence of a reference. | A wrapper object in heap memory. |
| **Compiler Warning** | None; silent risk of runtime `NullPointerException`. | Enforces handling through the type system. |
| **Memory Overhead** | **Zero** overhead. | **Overhead:** Allocates an extra wrapper object on the heap. |
| **Serialization** | Fully serializable. | **Not `Serializable**`; not intended for field storage. |

---

### When to Use `Optional` (And When NOT To)

**DO Use It:**

* **As a return type for methods** where the result could legitimately be absent (e.g., database finders like `repository.findById(id)`).

**DO NOT Use It:**

* **Never use as method parameters:** Passing `Optional<User>` forces callers to wrap arguments in `Optional.ofNullable()`. Just accept the parameter directly and check it.
* **Never use as class fields:** `Optional` is not `Serializable` and adds unnecessary heap memory overhead.
* **Never wrap collections:** Never return `Optional<List<T>>`. Return an empty list (`Collections.emptyList()`) instead.
* **Never use for primitives:** Use `OptionalInt`, `OptionalLong`, or `OptionalDouble` to avoid autoboxing overhead.

---
