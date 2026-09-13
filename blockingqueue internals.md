What is a BlockingQueue? How is it used in Producer-Consumer problems?
A standard `Queue` fails in producer-consumer systems because it is **passive**:

* If you call `poll()` on an empty `LinkedList` or `ArrayDeque`, it returns `null` immediately.
* If you want a consumer thread to wait for incoming work, you are forced to write a spinning `while(true)` loop (busy waiting, which wastes 100% of the CPU) or write messy `wait()` and `notify()` blocks with synchronized monitors.

A **`BlockingQueue`** handles this coordination out of the box by putting threads into a **sleep/wait state** instead of burning CPU cycles or failing.

---

### What Makes It "Blocking"?

A `BlockingQueue` introduces two blocking methods: **`put()`** and **`take()`**.

| Operation | Producer Calling `put(item)` | Consumer Calling `take()` |
| --- | --- | --- |
| **Condition** | The queue is **Full** (bounded capacity). | The queue is **Empty**. |
| **Standard Queue (`add`/`poll`)** | Throws `IllegalStateException` or returns `false`. | Returns `null` immediately. |
| **BlockingQueue (`put`/`take`)** | **Blocks (waits)** until a consumer takes an item and frees up space. | **Blocks (waits)** until a producer adds an item. |
| **CPU Usage** | $0\%$ — Thread is suspended by the OS. | $0\%$ — Thread is suspended by the OS. |

---

### How It Solves the Producer-Consumer Problem

`BlockingQueue` acts as a thread-safe buffer that naturally synchronizes the speed differences between producers and consumers:

1. **When Consumers Are Faster Than Producers:**
* Consumers consume everything; the queue empties.
* Consumers calling `take()` are automatically suspended.
* As soon as a producer calls `put()`, the queue signals and wakes up a waiting consumer.


2. **When Producers Are Faster Than Consumers (Backpressure):**
* If using a bounded queue (e.g., size 100), producers fill it up.
* The next producer calling `put()` is suspended, stopping the application from blowing up with an `OutOfMemoryError`.
* As soon as a consumer calls `take()`, space opens up, signaling a waiting producer to wake up.



---

### How It Works Internally

Most implementations (like `ArrayBlockingQueue`) use a single `ReentrantLock` with two `Condition` variables:

* `notEmpty`: Consumers await on this condition when `count == 0`.
* `notFull`: Producers await on this condition when `count == items.length`.

When `put()` adds an item, it calls `notEmpty.signal()`. When `take()` removes an item, it calls `notFull.signal()`.

---

### Common Implementations to Mention

* **`ArrayBlockingQueue`**: Bounded buffer backed by an array.
* **`LinkedBlockingQueue`**: Optionally bounded, backed by linked nodes (uses two separate locks for put and take, offering higher concurrency).
* **`SynchronousQueue`**: Zero-capacity queue where each `put()` must wait for a corresponding `take()` (heavily used in `Executors.newCachedThreadPool()`).
