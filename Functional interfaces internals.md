## What are Lambda Expressions?

### What a Lambda Expression Actually Is

A lambda expression is an **anonymous function**—a block of code that takes in parameters, performs an action, and optionally returns a value, without having a name or being tied to an explicit class declaration.

```java
// Parameters  Arrow  Body
   (a, b)       ->    a + b

```

---

### Anonymous Inner Class vs. Lambda Expression (The Core Interview Distinctions)

Interviewers love to test the internal differences:

| Feature | Anonymous Inner Class | Lambda Expression |
| --- | --- | --- |
| **Under the Hood (JVM)** | Compiles to a separate `.class` file (e.g., `MyService$1.class`). Loaded into memory separately by the ClassLoader. | **No separate `.class` file**. Uses the `invokedynamic` (Indy) bytecode instruction to dynamically create a call site at runtime. Highly memory-efficient. |
| **`this` Keyword Scope** | `this` refers to the **anonymous inner class instance itself**. | `this` is **lexically scoped**—it refers to the **enclosing class** where the lambda is defined. |
| **Applicability** | Can implement interfaces with *any* number of methods, or extend abstract/concrete classes. | Works **only with Functional Interfaces** (interfaces with exactly one abstract method). |
| **Memory Footprint** | Creates a new object instance on the heap every single time it executes. | Can be optimized by the JVM to avoid repeated object allocations. |

---

### Code Comparison

#### 1. Anonymous Inner Class (The Old Way)

```java
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running old way");
    }
};

```

#### 2. Lambda Expression (Java 8+)

```java
Runnable r2 = () -> System.out.println("Running with lambda");

```

---

### The `this` Reference Trap (Classic Trick Question)

```java
public class LambdaScopeTest {
    public void test() {
        Runnable rAnon = new Runnable() {
            @Override
            public void run() {
                System.out.println(this); // Prints: LambdaScopeTest$1@... (the anonymous class instance)
            }
        };

        Runnable rLambda = () -> {
            System.out.println(this); // Prints: LambdaScopeTest@... (the outer enclosing object!)
        };
    }
}

```
## What is method reference syntax (::)?
**Method reference syntax (`::`)** is a shorthand notation introduced in Java 8 used to directly refer to an existing method or constructor by its name without executing it immediately.

Whenever a lambda expression does nothing more than call an existing method with its passed arguments, a method reference replaces that boilerplate with a cleaner, more readable construct:

```java
// Lambda expression:
list.forEach(s -> System.out.println(s));

// Method reference equivalent:
list.forEach(System.out::println);

```

Under the hood, method references are treated exactly like lambda expressions—they compile to target a **Functional Interface** using the `invokedynamic` bytecode instruction.

---

### The 4 Types of Method References

Interviewers frequently ask candidates to enumerate all four types with examples:

| Type | Syntax | Lambda Equivalent | Example |
| --- | --- | --- | --- |
| **1. Static Method** | `ClassName::staticMethod` | `(args) -> ClassName.staticMethod(args)` | `Integer::parseInt` |
| **2. Instance Method of an Existing Object** | `existingObj::instanceMethod` | `(args) -> existingObj.instanceMethod(args)` | `System.out::println` |
| **3. Instance Method of an Arbitrary Object of a Particular Type** | `ClassName::instanceMethod` | `(obj, args) -> obj.instanceMethod(args)` | `String::toUpperCase` |
| **4. Constructor Reference** | `ClassName::new` | `(args) -> new ClassName(args)` | `ArrayList::new` |

---

### Deep Dive into the 4 Types

#### 1. Reference to a Static Method (`ContainingClass::staticMethodName`)

Used when the target functional interface maps directly to a static utility method.

```java
// Lambda:
Function<String, Integer> f1 = s -> Integer.parseInt(s);

// Method Reference:
Function<String, Integer> f2 = Integer::parseInt;

```

#### 2. Reference to an Instance Method of a Specific (Existing) Object (`instance::methodName`)

Used when invoking a method on an object reference that is already defined and in scope.

```java
String prefix = "LOG_";

// Lambda:
Function<String, String> f1 = s -> prefix.concat(s);

// Method Reference:
Function<String, String> f2 = prefix::concat;

```

#### 3. Reference to an Instance Method of an Arbitrary Object of a Particular Type (`ClassName::instanceMethodName`)

This is the one that trips candidates up most often. The **first argument of the lambda becomes the target object** on which the instance method is invoked, and subsequent arguments (if any) are passed as method parameters.

```java
// Notice String is a class name, but toUpperCase() is an instance method!
// The first parameter 's' is the object invoking .toUpperCase()
Function<String, String> f1 = s -> s.toUpperCase();
Function<String, String> f2 = String::toUpperCase;

// Two-argument example:
BiPredicate<String, String> p1 = (s1, s2) -> s1.equalsIgnoreCase(s2);
BiPredicate<String, String> p2 = String::equalsIgnoreCase;

```

#### 4. Reference to a Constructor (`ClassName::new`)

Used to instantiate objects, typically within factory patterns, Stream `map()` operations, or `Supplier` implementations.

```java
// Supplier taking no arguments:
Supplier<List<String>> s1 = () -> new ArrayList<>();
Supplier<List<String>> s2 = ArrayList::new;

// Function taking constructor arguments:
Function<String, Integer> f1 = s -> new Integer(s);
Function<String, Integer> f2 = Integer::new;

```

---

### When Can You Use It?

You can convert a lambda to a method reference **only if**:

1. The lambda simply delegates to an existing method without adding extra operations or modifying arguments.
2. The argument types and return type of the method match the functional interface's abstract method signature.

---
