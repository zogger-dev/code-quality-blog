---
title: "Domain-Specific Semantics"
date: 2025-01-05
categories: ["Abstraction"]
draft: true
---

Abstractions are the building blocks of a maintainable system. However, not all
abstractions are created equal. One of the most common mistakes in software
design is the use of generic primitives where domain-specific concepts are
required. This leads to "leaking abstractions," where callers are forced to
understand the internal mechanics of a dependency just to use it.

## The Leaking primitive

Consider a system designed to pause replication when disk usage exceeds a
certain threshold. In an early iteration, we might introduce a generic `Gate`
abstraction to represent a simple open/closed state:

```java
public interface Gate {
    boolean isOpen();
    void open();
    void close();
}
```

This seems like a clean, reusable component. But look at how it leaks into the
rest of the system. A `ReplicationManager` now depends on a `Gate`:

```java
public class ReplicationManager {
    private final Gate gate;

    public ReplicationManager(Gate gate) {
        this.gate = gate;
    }

    public void process() {
        if (gate.isOpen()) {
            // replicate data
        }
    }
}
```

The problem is that `Gate` is too generic. The `ReplicationManager` shouldn't
care about "gates"—it cares about whether replication is permitted based on the
current system state. By injecting a `Gate`, we've forced the caller to know
that a "gate" is the mechanism used to control replication. Worse, the caller
now has to decide _which_ gate to provide (a hysteresis gate, a manual gate,
etc.) and how to configure its thresholds.

## Elevating to Domain Concepts

We can fix this by wrapping the generic primitive in a domain-specific
abstraction. Instead of a `Gate`, the `ReplicationManager` should depend on a
`ReplicationStateMonitor`.

```java
public interface ReplicationStateMonitor {
    boolean shouldReplicate();
}
```

This simple change transforms the entire design. The `ReplicationStateMonitor`
provides clear, domain-specific semantics. It hides the implementation details
(the gate, the disk monitoring, the thresholds) behind a single, meaningful
method: `shouldReplicate()`.

The implementation can still use the generic `Gate` internally:

```java
public class DiskBasedReplicationMonitor implements ReplicationStateMonitor {
    private final Gate gate;

    public DiskBasedReplicationMonitor(DiskMonitor monitor, double pause, double resume) {
        this.gate = new HysteresisGate(pause, resume);
        monitor.register(this.gate);
    }

    @Override
    public boolean shouldReplicate() {
        return gate.isOpen();
    }
}
```

## The Power of Domain Semantics

By moving from generic primitives to domain-specific semantics, we gain several
key advantages:

1. **Encapsulation:** The internal mechanics of state management (like the use
   of a hysteresis gate) are hidden from consumers.
2. **Readability:** The code reads like a technical requirement.
   `if (monitor.shouldReplicate())` is infinitely more clear than
   `if (gate.isOpen())`.
3. **Testability:** We can easily provide fake implementations of
   `ReplicationStateMonitor` for unit tests without having to mock the complex
   internals of a gate system.
4. **Flexibility:** If we decide to change how we determine replication state
   (e.g., moving from disk monitoring to a manual override), we only need to
   update the monitor implementation. The `ReplicationManager` remains
   untouched.

Better abstractions don't just reduce code duplication; they improve the
language of your system. Whenever you find yourself injecting a generic
primitive, ask if there is a domain concept waiting to be discovered. Don't
speak in "gates" and "bits" when you can speak in "replication" and "state."
