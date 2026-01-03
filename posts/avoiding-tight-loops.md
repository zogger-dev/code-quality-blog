# Avoiding Tight Loops: Stop Spinning Your CPU

**Date:** January 2026
**Topic:** Performance, Concurrency, Efficiency

## The Problem

When a thread needs to wait for a condition to become true—for example, waiting for a "Gate" to open or a queue to have items—it's tempting to write a simple loop that checks the condition repeatedly.

This is known as a **Tight Loop** (or Busy-Wait), and it is a major performance and efficiency anti-pattern.

## The Anti-Pattern: The Tight Loop

```java
public void waitForData() {
    // BAD: Tight loop consumes 100% CPU on this core
    while (!dataReady) {
        // Do nothing, just spin
    }
    processData();
}
```

### Why this is bad:

1.  **CPU Waste:** The thread is actively executing instructions as fast as the CPU allows, doing nothing useful. This wastes energy, generates heat, and steals cycles from other threads that actually have work to do.
2.  **Resource Contention:** Constant polling of a shared variable (like `dataReady`) can cause cache coherency traffic between CPU cores, slowing down other parts of the system.
3.  **Livelock:** In some environments, a tight loop might prevent the thread that is supposed to *set* `dataReady = true` from ever getting a chance to run!

## The Solution: Proper Synchronization Primitives

The goal is to put the waiting thread to **sleep** so it consumes zero CPU, and have the notifying thread **wake it up** when the condition changes.

### 1. Using `wait()` and `notifyAll()`

The classic Java approach using intrinsic locks.

```java
private final Object lock = new Object();
private boolean dataReady = false;

public void waitForData() throws InterruptedException {
    synchronized (lock) {
        while (!dataReady) {
            // GOOD: Thread goes to sleep and releases the lock
            lock.wait();
        }
    }
    processData();
}

public void setDataReady() {
    synchronized (lock) {
        dataReady = true;
        lock.notifyAll(); // Wake up any waiters
    }
}
```

### 2. Using `java.util.concurrent` (Locks and Conditions)

More flexible and readable for complex scenarios.

```java
private final Lock lock = new ReentrantLock();
private final Condition isReady = lock.newCondition();
private boolean dataReady = false;

public void waitForData() throws InterruptedException {
    lock.lock();
    try {
        while (!dataReady) {
            isReady.await(); // Puts thread to sleep efficiently
        }
    } finally {
        lock.unlock();
    }
    processData();
}
```

### 3. Using Higher-Level Abstractions

Often, the best solution is to avoid managing the wait logic yourself entirely. Use high-level concurrent collections:

-   **`BlockingQueue.take()`:** Automatically sleeps until an item is available.
-   **`CompletableFuture.get()`:** Joins the result of an asynchronous task.
-   **`Semaphore.acquire()`:** Waits for a permit to become available.

## Key Takeaways

1.  **Don't Spin:** If you see a `while` loop with an empty body or just a `Thread.onSpinWait()`, you likely have a tight loop.
2.  **Use `wait/notify` or `Condition`:** These allow the OS to manage thread scheduling efficiently.
3.  **Prefer `j.u.c` collections:** They are highly optimized and handle the complex "waiting" logic for you safely.
4.  **Yielding is not enough:** `Thread.yield()` or `Thread.sleep(1)` are better than a pure tight loop, but still less efficient than proper notification patterns.

By replacing tight loops with blocking primitives, you'll improve your application's throughput, reduce latency, and make your system a better "citizen" on the host machine.
