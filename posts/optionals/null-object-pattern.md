# The Null Object Pattern vs. Optional Arguments

In modern Java, `Optional` has become the standard tool for expressing that a value might be missing. It is a powerful way to avoid the dreaded `NullPointerException` and force callers to handle the empty case. However, there is a common architectural boundary where `Optional` should almost never be used: as a method or constructor argument.

## The Boolean Trap, Revisited

Consider a service that requires an optional monitor to track its progress. It's tempting to express this dependency using `Optional`:

```java
public class ReplicationService {
    private final Optional<DiskMonitor> diskMonitor;

    public ReplicationService(Optional<DiskMonitor> diskMonitor) {
        this.diskMonitor = diskMonitor;
    }

    public void process() {
        // ...
        diskMonitor.ifPresent(monitor -> monitor.recordActivity(Activity.WRITE));
        // ...
    }
}
```

While this appears safe, it forces the `ReplicationService` to constantly "check its pockets." Every time it wants to use the monitor, it has to wrap the logic in a conditional or a closure. This mechanical noise obscures the business logic and spreads the "missingness" concern throughout the class.

Even worse, it forces the caller to decide how to represent "nothing":

```java
var service = new ReplicationService(Optional.empty());
```

This is merely a more verbose version of passing `null`. We haven't improved the abstraction; we've only added a container.

## The Resolution: The Null Object Pattern

Instead of forcing the consumer to manage the absence of a dependency, we can provide a "nothing" implementation that satisfies the interface. This is the **Null Object Pattern**.

```java
public interface DiskMonitor {
    void recordActivity(Activity type);

    static DiskMonitor noop() {
        return type -> {}; // Do nothing
    }
}

public class ReplicationService {
    private final DiskMonitor diskMonitor;

    public ReplicationService(DiskMonitor diskMonitor) {
        // Ensure we always have a valid implementation
        this.diskMonitor = Objects.requireNonNull(diskMonitor);
    }

    public void process() {
        // ...
        diskMonitor.recordActivity(Activity.WRITE);
        // ...
    }
}
```

By providing a `noop()` implementation, we eliminate the need for `Optional` in the constructor and the internal logic. The `ReplicationService` can now treat the `diskMonitor` as a first-class, always-present citizen.

## The Synthesis

Why is the Null Object Pattern superior to `Optional` arguments?

1.  **Cleaner Logic:** It removes the conditional branching from your core logic. The code reads as a linear sequence of actions, regardless of whether a "real" monitor is present.
2.  **Explicit Intent:** A `noop()` implementation explicitly defines what "doing nothing" means for that specific domain concept.
3.  **Simplified Callers:** Callers don't have to wrap their dependencies in `Optional.of()` or `Optional.empty()`. They just provide an implementation.
4.  **Better Composition:** Null objects are easier to compose and chain than `Optional` containers.

`Optional` was designed to be a **return type**—a way for a method to say, "I might not have found what you were looking for." When you use it as an argument, you are usually trying to solve a dependency problem that is better handled by choosing a better default.

## The Insight

Don't force your objects to constantly check if their tools exist. If a dependency is optional, don't pass an `Optional`. Instead, provide a **Null Object** that does nothing safely. It keeps your logic clean, your APIs simple, and your intent explicit.
