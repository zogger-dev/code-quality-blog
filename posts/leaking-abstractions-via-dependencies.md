# Leaking Abstractions via Dependencies: When "Dependency Injection" Goes Wrong

**Date:** January 2026
**Topic:** Architectural Patterns, Dependency Injection, Encapsulation

## The Problem

Dependency Injection (DI) is a cornerstone of modern software engineering. It decouples components and makes testing easier. However, a common trap is injecting "building blocks" or "primitives" rather than domain concepts.

When we inject a low-level utility class into a high-level business component, we often inadvertently force the high-level component to understand the low-level implementation details. This is a **Leaky Abstraction**.

## The Anti-Pattern: Injecting the "Mechanism"

Let's look at a concrete example from a database replication system. We have a `MergePolicy` that needs to stop merging indexes when disk space is low.

We have a generic `Gate` interface (from our previous [Gate Pattern](./gate-pattern-vs-event-consumers.md) post):

```java
public interface Gate {
    boolean isOpen();
}
```

A common mistake is to inject this `Gate` directly into the business logic:

```java
public class DiskUtilizationAwareMergePolicy {
    private final Gate gate;

    // "Mechanism" Injection
    public DiskUtilizationAwareMergePolicy(Gate gate) {
        this.gate = gate;
    }

    public void merge() {
        if (gate.isOpen()) { // Ambiguous! Which gate? Configured how?
            // ... do merge
        }
    }
}
```

### Why this is problematic:

1.  **Ambiguity:** A `Gate` is too generic. Is this the "Replication Gate"? The "Crash Gate"? The "Initial Sync Gate"? The type system doesn't tell us. The developer reading the code has to trace the dependency graph to find out what *kind* of gate was passed in.
2.  **Broken Encapsulation:** The caller (the wiring code) now has to know exactly how to configure a "Disk Utilization Gate" with the correct thresholds (e.g., stop at 90%, resume at 80%) and pass it in. If the `MergePolicy` changes its requirements (e.g., it needs to know *why* it stopped), the signature must change.
3.  **Leaked Implementation:** The `MergePolicy` isn't checking "Is it safe to merge?"; it's checking "Is this gate open?". Semantically, these are different.

## The Solution: Domain-Specific Wrappers

Instead of injecting the *mechanism* (`Gate`), we should inject the *policy* or *state monitor* that wraps it.

We define a domain-specific interface or class that captures the *intent*:

```java
public class ReplicationStateMonitor {
    private final Gate gate;

    // The wrapper owns the mechanism
    public ReplicationStateMonitor(Gate gate) {
        this.gate = gate;
    }

    // Domain-specific method name
    public boolean isSafeToMerge() {
        return gate.isOpen();
    }
}
```

And update our consumer:

```java
public class DiskUtilizationAwareMergePolicy {
    private final ReplicationStateMonitor monitor;

    // Intent Injection
    public DiskUtilizationAwareMergePolicy(ReplicationStateMonitor monitor) {
        this.monitor = monitor;
    }

    public void merge() {
        if (monitor.isSafeToMerge()) {
            // ... do merge
        }
    }
}
```

Alternatively, if the component effectively "owns" the gate logic (e.g., it's the *only* thing that uses these specific disk thresholds), it should construct the gate internally based on a configuration object, rather than accepting a pre-built gate.

```java
public class DiskUtilizationAwareMergePolicy {
    private final Gate gate;

    // Configuration Injection (Factory method or Constructor)
    public DiskUtilizationAwareMergePolicy(DiskMonitor diskMonitor, double stopThreshold) {
        // Implementation detail hidden internally
        this.gate = new HysteresisGate(stopThreshold);
        diskMonitor.register(this.gate);
    }
}
```

## Key Takeaways

1.  **Inject Intent, Not Implementation:** Don't just inject the tool (Gate, Semaphore, Lock); inject an object that describes *how* that tool is being used in this context.
2.  **Semantic Naming:** `monitor.isSafeToMerge()` is infinitely more readable and maintainable than `gate.isOpen()`.
3.  **Better Testing:** You can easily mock `ReplicationStateMonitor` to return `true` or `false` without worrying about the internal hysteresis logic of the underlying `Gate`.

By wrapping generic dependencies in domain-specific types, we restore encapsulation and make our code self-documenting.
