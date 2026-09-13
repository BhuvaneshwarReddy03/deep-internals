## How does PriorityQueue work internally? (Min-heap / Max-heap)
A **`PriorityQueue`** completely throws FIFO away. Instead of serving whoever arrived first, it serves whoever has the **highest priority** (by default, the smallest value).

---

### Min-Heap vs Max-Heap

Under the hood, Java uses a **binary heap** to organize the data:

* **Min-Heap (Java's Default):** The **smallest** element is always kept at the top (root). When you call `poll()` or `peek()`, you always get the minimum element.
* **Max-Heap:** The **largest** element is always kept at the top. When you call `poll()`, you get the maximum element.

To turn Java's `PriorityQueue` into a Max-Heap, pass a reverse comparator:

```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

```

---

### How It Works Internally: Array-Backed Binary Tree

Java doesn't create node objects with left/right pointers for this tree. Instead, it stores the entire tree inside a simple **dynamic array** (`Object[] queue`).

<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/c1f24433-6e7b-44d9-a8ad-a1e8a695de12" />


---

Because a binary heap is a complete binary tree, you find relationships purely with index arithmetic:

* **Left child** of index $i$ is at $2i + 1$
* **Right child** of index $i$ is at $2i + 2$
* **Parent** of index $i$ is at $(i - 1) / 2$

---

### What Happens on `add()` and `poll()`

1. **`add(element)` (Sift-Up):**
* The element is temporarily appended to the end of the array.
* It is compared to its parent. If it is smaller than the parent (in a min-heap), they swap places.
* This continues bubbling up until the heap property is restored.
* **Time Complexity:** **$O(\log n)$**.


2. **`poll()` (Remove Top Element) (Sift-Down):**
* The top element at index `0` (the minimum) is removed to return to you.
* The very last element in the array is moved to index `0`.
* It compares itself with its children and swaps with the smaller child, sinking down until balance is restored.
* **Time Complexity:** **$O(\log n)$**.


3. **`peek()` (Look at Top Element):**
* Simply reads `queue[0]`.
* **Time Complexity:** **$O(1)$**.



---

## I thought priority queue is also storing the elements in sorted order. Is it not? Then how it is it will store? Like basically I want to understand what it's going to do when we are going to add the elements into the queue or like when we provide the collection, what it's going to do? I want to understand. So does it care about only min element of the collection or the elements added in the queue? Does it care about only minimum element? That's it?
**No, a `PriorityQueue` does not store elements in sorted order.** This is one of the most common misunderstandings in Java.

If you print a `PriorityQueue` or loop through it with a `for-each` loop, **the output is not sorted**:

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.add(30);
pq.add(10);
pq.add(20);

System.out.println(pq); 
// Output: [10, 30, 20]  <-- Not [10, 20, 30]!

```

---

### What Does `PriorityQueue` Actually Care About?

It cares about **only one thing**: guaranteeing that the **minimum (or highest-priority) element is always sitting at index `0**`.

It does **not** waste time sorting the rest of the elements. It leaves them **partially ordered** based on a single rule:

> **Heap Property:** Every parent element must be smaller than its children.

There is no rule about whether the left child is smaller than the right child, or whether index `2` is smaller than index `1`.

---

### What Happens Step-by-Step When You Add Elements?

Say you start adding numbers into an empty `PriorityQueue`:

1. **Add `30`:** Array is `[30]`. It's the root.
2. **Add `10`:** Appended at the end `[30, 10]`. It compares `10` to parent `30`. Since `10 < 30`, it swaps them: `[10, 30]`.
3. **Add `20`:** Appended at the end `[10, 30, 20]`. It compares `20` to parent `10`. Since `20 > 10`, it stops!
* The array is `[10, 30, 20]`. It did not bother sorting `30` and `20`.



---

### If It Doesn't Store in Sorted Order, How Does It Work as a Queue?

The "sorting" happens **on-demand when you extract elements using `poll()**`, one at a time:

```java
while (!pq.isEmpty()) {
    System.out.print(pq.poll() + " "); 
}
// Output: 10 20 30 (retrieved in sorted order!)

```

Here is why that works:

1. `poll()` removes the root (`10`) and hands it to you.
2. It takes the last element (`20`) and puts it at the root.
3. It pushes `20` down (sift-down) by comparing it with `30` to make sure the smallest remaining item moves to index `0`.
4. Now `20` is at index `0`. The next `poll()` will give you `20`.

---

### Why Not Just Sort Everything?

Sorting an entire collection takes **$O(n \log n)$**.

`PriorityQueue` avoids doing all that work upfront:

* Inserting takes only **$O(\log n)$** because it only fixes the path up to the root, not the whole collection.
* It only does work to find the next minimum **when you ask for it** via `poll()`.

---

### Summary for an Interview

> "`PriorityQueue` does **not** store elements in sorted order. It uses a min-heap structure that only guarantees that the **minimum element is always at the head (index `0`)**. The remaining elements are only partially ordered to satisfy the heap property. We only get sorted output when we repeatedly call `poll()` to extract the elements one by one."

## So, you said peek is O(1). Whenever we do the peak, we are basically removing, we have to change the top element, right? So, it does restructuring, right? Why it is considered as O(1)?
**`peek()` does NOT remove the element.** That is why it doesn't need to restructure anything.

In Java queues:

* **`peek()`** only **reads / looks at** the head element without removing it. It literally just executes `return queue[0];`, which is a simple array lookup taking **$O(1)$**.
* **`poll()`** is the method that actually **removes and returns** the head element.

Because `poll()` removes the top element, it has to move the last element to index `0` and sink it down (sift-down) to restore the heap property. That restructuring is why **`poll()` takes $O(\log n)$**.

---

### Direct Java Implementation

Here is almost the exact source code inside Java's `PriorityQueue`:

```java
// Just looks at the root -> O(1)
public E peek() {
    return (size == 0) ? null : (E) queue[0];
}

// Actually removes the root and restructures -> O(log n)
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x); // Restructures the heap!
    return result;
}

```

---

### The Two Methods to Keep Distinct

* **`peek()`**: Read-only $\rightarrow$ no restructuring $\rightarrow$ **$O(1)$**.
* **`poll()`**: Remove + restructure (sift-down) $\rightarrow$ **$O(\log n)$**.

## Okay, is it like so first it will replace the 0th index with the last element, then it's going to find the minimum element. Is it like that?

**Yes, exactly.** That is the exact sequence of events during a `poll()` operation:

1. **Grab and replace:** It saves `queue[0]` to return to you, then moves the **very last element** in the array directly into the `0`th index.
2. **Find and swap (Sift-Down):** That newly placed top element is usually much too large to be at the root. So it compares itself with its **two children** (at indices `2i + 1` and `2i + 2`).
3. **Sink down:** It swaps places with the **smaller** of the two children so that the true minimum bubbles up to the top.
4. **Repeat:** It repeats this comparison down the levels of the tree until it is smaller than both of its children (or reaches the bottom).

Because it only travels down a single branch of the binary tree rather than scanning the whole array, this path is at most the height of the tree ($\log_2 n$ steps). That is why the whole restructuring is strictly **$O(\log n)$**.
