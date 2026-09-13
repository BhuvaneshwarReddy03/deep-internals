## Explain the difference between Iterator and ListIterator.
---

### Key Differences

| Feature | `Iterator` | `ListIterator` |
| --- | --- | --- |
| **Applicability** | Works on **any `Collection**` (`List`, `Set`, `Queue`). | Works **only on `List**` implementations (`ArrayList`, `LinkedList`). |
| **Direction** | **One-way only** (forward). | **Bi-directional** (forward and backward). |
| **Traversal Methods** | `hasNext()`, `next()` | `hasNext()`, `next()`, `hasPrevious()`, `previous()` |
| **Index Access** | Cannot tell you the current index. | Has `nextIndex()` and `previousIndex()`. |
| **Modification Operations** | Can only **read** and **remove** (`remove()`). | Can **read**, **remove**, **add** (`add()`), and **replace/set** (`set()`). |

---
## for each on ArrayList or LinkedList uses ListIterator by default or do we have to explicitly mention the ListIterator?
A `for-each` loop **always uses standard `Iterator**`, even on `ArrayList` and `LinkedList`. It **never** uses `ListIterator` by default.

---

### How the Compiler Handles `for-each`

Under the hood, Java’s enhanced `for-each` loop is syntax sugar for any object that implements `Iterable<T>`.

When you write:

```java
List<String> list = new ArrayList<>();

for (String s : list) {
    System.out.println(s);
}

```

The Java compiler rewrites it into bytecode using `iterator()`:

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    System.out.println(s);
}

```

Because `Iterable` defines only the method `iterator()`, the `for-each` loop can only ever invoke standard `Iterator`.

---

### You Must Explicitly Request `ListIterator`

If you want the extra power of `ListIterator` (going backwards, modifying via `set()`, or checking indexes), you must call `list.listIterator()` manually in code:

```java
ListIterator<String> listIt = list.listIterator();

// Moving backward from the end:
ListIterator<String> reverseIt = list.listIterator(list.size());
while (reverseIt.hasPrevious()) {
    System.out.println(reverseIt.previous());
}

```

---
## What is the difference between Fail-Fast and Fail-Safe iterators? Give examples.

The core difference comes down to **how they handle modifications made to the collection while you are iterating over it**.

---

### Core Differences

| Feature | Fail-Fast Iterator | Fail-Safe (Weakly Consistent) Iterator |
| --- | --- | --- |
| **Behavior on Modification** | Throws **`ConcurrentModificationException`** immediately if modified during iteration. | **Does not throw** an exception; continues iterating safely. |
| **How It Operates** | Operates directly on the **actual collection data**. | Operates on a **clone / snapshot** or a weakly consistent view of the data. |
| **Modification Tracking** | Checks an internal counter called **`modCount`** on every step. | Does not rely on strict `modCount` matching. |
| **Memory & Overhead** | Fast, uses no extra memory copy. | Incurs memory/performance overhead (especially if copying underlying arrays). |
| **Examples** | `ArrayList`, `LinkedList`, `HashSet`, `HashMap` (standard `java.util` collections). | `CopyOnWriteArrayList`, `ConcurrentHashMap` (`java.util.concurrent` collections). |

---

### How Fail-Fast Works (The `modCount` Mechanism)

Standard collections maintain an internal variable:

```java
protected transient int modCount = 0;

```

1. Every time you call `add()` or `remove()` directly on the collection, `modCount` increments by `1`.
2. When you create an `Iterator`, it saves a local copy: `expectedModCount = modCount`.
3. On every `next()` call, it checks:
```java
if (modCount != expectedModCount)
    throw new ConcurrentModificationException();

```



If you alter the list from outside the iterator while looping, the numbers mismatch and it fails immediately.

---

### How Fail-Safe Works

Instead of looking at the live structure under strict lock:

* **`CopyOnWriteArrayList`**: Creates an exact copy (snapshot) of the array whenever a write occurs. The iterator keeps reading the old snapshot, so it never sees the mid-loop modification or crashes.
* **`ConcurrentHashMap`**: Uses a *weakly consistent* iterator that reflects the state of the map at or since creation, without locking or throwing exceptions.

---
