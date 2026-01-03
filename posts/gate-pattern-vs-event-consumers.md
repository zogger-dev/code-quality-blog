# The Gate Pattern vs Event Consumers: Managing Shared State in Concurrent Systems

**Date:** January 2026
**Topic:** Architectural Patterns, Concurrency

## The Problem

In complex distributed systems, multiple subsystems often contend for shared resources. A classic example is disk usage. In a database system like MongoDB (or its sidecar `mongot`), various background processes—index replication, initial syncs, large file downloads—need to pause if the disk becomes too full to prevent a crash.

The challenge is: **How do we coordinate these independent subsystems to respect a global resource limit without tightly coupling them?**

## The Anti-Pattern: Event Consumers (Callbacks)

A common initial approach is the "Push" model, often implemented as an Event Bus or a Observer pattern with callbacks.

```java
// The "Producer"
class PeriodicDiskMonitor {
    private final List<Consumer<DiskUsage>> consumers = new ArrayList<>();

    public void registerConsumer(Consumer<DiskUsage> consumer) {
        this.consumers.add(consumer);
    }

    public void run() {
        DiskUsage usage = computeDiskUsage();
        // Danger: Executing foreign code on the monitor thread!
        consumers.forEach(c -> c.accept(usage));
    }
}

// The "Consumer"
class Downloader {
    public void onDiskUsageUpdate(DiskUsage usage) {
        if (usage.percent() > 90) {
            this.pause(); // What if this blocks?
        } else {
            this.resume();
        }
    }
}
```

### Why this fails in practice:

1.  **Thread Hijacking:** The `PeriodicDiskMonitor` is usually running on a dedicated, lightweight scheduling thread. If a consumer's callback blocks (e.g., waiting for a lock, IO), it halts the entire monitoring process. The monitor stops monitoring!
2.  **Concurrency Nightmares:** The consumer receives the event on the *monitor's* thread, but likely needs to mutate state owned by the *consumer's* thread. This forces every consumer to implement complex thread-safe logic or re-dispatch the event.
3.  **State Management Complexity:** What if we want to stop at 90% usage but only resume when it drops below 80% (Hysteresis)? Implementing this logic inside every consumer duplicates code and invites bugs.

## The Solution: The Gate Pattern

Instead of pushing events *to* consumers, we can invert the relationship using a **Gate**.

A `Gate` is a state container that encapsulates the decision logic (Open/Closed). The Monitor updates the Gate; the Consumer checks the Gate.

### 1. The Gate Abstraction

First, we define a simple interface for the signal.

```java
public interface Gate {
    boolean isOpen();
    // Optional: void awaitOpen() throws InterruptedException;
}
```

### 2. The Hysteresis Implementation

We implement a gate that handles the "flapping" problem internally. This isolates the state transition logic from both the monitor and the consumer.

```java
public class HysteresisGate implements Gate {
    private final double closeThreshold; // e.g., 0.90
    private final double openThreshold;  // e.g., 0.80
    private volatile boolean isOpen = true;

    public void update(double currentUsage) {
        if (isOpen && currentUsage > closeThreshold) {
            isOpen = false; // Close the gate
        } else if (!isOpen && currentUsage < openThreshold) {
            isOpen = true;  // Re-open the gate
        }
    }

    @Override
    public boolean isOpen() {
        return isOpen;
    }
}
```

### 3. Usage

The `PeriodicDiskMonitor` no longer knows about "Downloaders" or "Replicators". It only knows about "Gates".

```java
class PeriodicDiskMonitor {
    private final List<HysteresisGate> gates = new ArrayList<>();

    public void register(HysteresisGate gate) {
        this.gates.add(gate);
    }

    public void run() {
        double usage = computeDiskUsage();
        // Fast, non-blocking updates
        gates.forEach(gate -> gate.update(usage));
    }
}
```

The subsystems own their gates and check them on *their own terms*.

```java
class Downloader {
    private final Gate diskGate;

    public void downloadFile() {
        // Check the gate before expensive operations
        if (!diskGate.isOpen()) {
            pause();
            return;
        }
        // ... proceed
    }
}
```

## Key Takeaways

1.  **Isolation of Concerns:** The Monitor observes. The Gate decides. The Subsystem reacts.
2.  **Thread Safety:** The Monitor thread only updates a volatile boolean (or atomic reference). It never executes foreign, potentially blocking code.
3.  **Testability:** You can unit test the `HysteresisGate` logic (boundaries, state transitions) completely independently of the disk monitor or the downloader.

By moving from Event Consumers to Gates, we converted a complex concurrency problem into a simple state synchronization problem.
