Queue data structure, defining it as a linear, abstract data type (ADT) that follows the FIFO (First-In, First-Out) principle

Operations: The queue supports two primary operations: Enqueue (inserting data at the rear end) and Dequeue (deleting data from the front end).
Logical Representation: A queue is conceptually represented with two pointers, front and rear, which manage the ends of the structure.
Time Complexity: Fundamental queue operations (Enqueue, Dequeue, Peek/Front) run in O(1) constant time

---

## What is a Queue?

Think of a queue like a line of people waiting at a ticket counter. The first person who joins the line is the first person to get the ticket and leave. No one can cut in from the front — new people join at the back, and service happens at the front.

- **Enqueue** → add an item to the **rear** (back) of the queue
- **Dequeue** → remove an item from the **front** of the queue
- **Peek / Front** → just look at the front item without removing it
- **isEmpty** → check if the queue has no elements

---

## Real-Life Examples

| Situation | Queue Behaviour |
|---|---|
| Printer job queue | First document sent is printed first |
| Customer support calls | First caller in line is answered first |
| CPU task scheduling | Tasks are processed in the order they arrive |
| Breadth-First Search (BFS) | Nodes are explored level by level using a queue |

---

## How Front and Rear Pointers Work

```
Initial empty queue:   front = -1, rear = -1

After Enqueue(10):     [ 10 ]          front = 0, rear = 0
After Enqueue(20):     [ 10, 20 ]      front = 0, rear = 1
After Enqueue(30):     [ 10, 20, 30 ]  front = 0, rear = 2
After Dequeue():       [ 20, 30 ]      front = 1, rear = 2  → 10 was removed
After Peek():          20              front still = 1, nothing removed
```

The **front** pointer always points to the element that will be removed next.
The **rear** pointer always points to where the next element will be inserted.

---

## Examples

### Basic Queue Operations in Java (using LinkedList)

Java's built-in `Queue` interface is the easiest way to use a queue:

```java
import java.util.LinkedList;
import java.util.Queue;

Queue<Integer> queue = new LinkedList<>();

// Enqueue — add elements to the rear
queue.add(10);
queue.add(20);
queue.add(30);

System.out.println("Front element: " + queue.peek()); // 10 (not removed)

// Dequeue — remove elements from the front
System.out.println("Removed: " + queue.poll()); // 10
System.out.println("Removed: " + queue.poll()); // 20

System.out.println("Queue now: " + queue); // [30]

System.out.println("Is empty? " + queue.isEmpty()); // false
```

---

### Queue Using an Array (manual implementation)

This shows exactly how a queue works under the hood:

```java
class ArrayQueue {
    int[] data;
    int front, rear, size;

    ArrayQueue(int capacity) {
        data  = new int[capacity];
        front = 0;
        rear  = -1;
        size  = 0;
    }

    // Add element to rear
    void enqueue(int value) {
        if (size == data.length) {
            System.out.println("Queue is full!");
            return;
        }
        rear++;
        data[rear] = value;
        size++;
    }

    // Remove element from front
    int dequeue() {
        if (isEmpty()) {
            System.out.println("Queue is empty!");
            return -1;
        }
        int value = data[front];
        front++;
        size--;
        return value;
    }

    // Look at front without removing
    int peek() {
        if (isEmpty()) return -1;
        return data[front];
    }

    boolean isEmpty() {
        return size == 0;
    }
}

// Usage
ArrayQueue q = new ArrayQueue(5);
q.enqueue(10);
q.enqueue(20);
q.enqueue(30);
System.out.println(q.peek());    // 10
System.out.println(q.dequeue()); // 10
System.out.println(q.dequeue()); // 20
```

---

### Printing All Elements in Order

```java
import java.util.LinkedList;
import java.util.Queue;

Queue<String> customers = new LinkedList<>();
customers.add("Alice");
customers.add("Bob");
customers.add("Charlie");

// Process all customers in order
while (!customers.isEmpty()) {
    System.out.println("Serving: " + customers.poll());
}
// Output:
// Serving: Alice
// Serving: Bob
// Serving: Charlie
```

---

## Types of Queue

| Type | Description |
|---|---|
| Simple Queue | Basic FIFO — insert at rear, remove from front |
| Circular Queue | Rear connects back to front to reuse empty spaces |
| Double-Ended Queue (Deque) | Insert and remove from both ends |
| Priority Queue | Element with highest priority is served first, not the oldest |

---

## Queue vs Stack — Quick Comparison

| Feature | Queue | Stack |
|---|---|---|
| Principle | FIFO (First In, First Out) | LIFO (Last In, First Out) |
| Insert | At the **rear** | At the **top** |
| Remove | From the **front** | From the **top** |
| Real-life | Ticket line | Stack of plates |

---

## Time Complexity

| Operation | Complexity | Notes |
|---|---|---|
| Enqueue (add) | O(1) | Always inserts at the rear |
| Dequeue (remove) | O(1) | Always removes from the front |
| Peek / Front | O(1) | Just reads the front pointer |
| isEmpty | O(1) | Checks if size is zero |
| Search | O(n) | Must scan elements one by one |
