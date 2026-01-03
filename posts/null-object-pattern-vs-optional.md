# The Null Object Pattern vs Optional Arguments: Why `Optional<T>` as a Parameter is a Code Smell

**Date:** January 2026
**Topic:** Coding Best Practices, Java, Clean Code

## The Problem

Java 8 introduced `Optional<T>` to cure us of the `NullPointerException` plague. It forces developers to explicitly handle the absence of a value. It is fantastic for **return types**.

However, a common anti-pattern has emerged: **passing `Optional<T>` as method arguments**.

## The Anti-Pattern: `Optional` Parameters

You want to design a component where a dependency is optional. For example, a `BlobstoreClient` that *might* have a `DiskMonitor` to pause downloads if the disk is full.

```java
public class BlobstoreClient {
    private final Optional<DiskMonitor> diskMonitor;

    // BAD: Optional as a parameter
    public BlobstoreClient(Optional<DiskMonitor> diskMonitor) {
        this.diskMonitor = diskMonitor;
    }

    public void download() {
        // Verbose usage
        diskMonitor.ifPresent(m -> m.waitForCapacity());
        // ...
    }
}
```

### Why this smells:

1.  **Caller Burden:** The caller is forced to wrap values.
    ```java
    // What the caller has to write
    new BlobstoreClient(Optional.of(monitor));
    new BlobstoreClient(Optional.empty());
    ```
2.  **It doesn't actually solve `null`:** `Optional` is an object. You can still pass `null` to the constructor!
    ```java
    new BlobstoreClient(null); // Compiles fine, crashes at runtime on 'diskMonitor.ifPresent'
    ```
3.  **Semantic Misuse:** Brian Goetz (Java Language Architect) has stated that `Optional` was intended for library method return types, not for general use as a specialized optional-value type for fields or parameters.

## The Solution: The Null Object Pattern

The best way to handle an optional dependency is often to treat it as *mandatory* but provide a *no-op* implementation. This is the **Null Object Pattern**.

### 1. Define the Interface

```java
public interface DiskMonitor {
    void waitForCapacity();

    // Factory method for the Null Object
    static DiskMonitor noop() {
        return new NoOpDiskMonitor();
    }
}
```

### 2. Implement the Null Object

Create an implementation that does "nothing" gracefully.

```java
class NoOpDiskMonitor implements DiskMonitor {
    @Override
    public void waitForCapacity() {
        // Do nothing. Effectively "infinite" capacity.
        return;
    }
}
```

### 3. Refactor the Client

Now the client code is clean. It doesn't know or care if the monitor is "real" or "no-op".

```java
public class BlobstoreClient {
    private final DiskMonitor diskMonitor;

    // GOOD: Accept the interface directly
    public BlobstoreClient(DiskMonitor diskMonitor) {
        // Fail fast on actual nulls
        this.diskMonitor = Objects.requireNonNull(diskMonitor);
    }

    public void download() {
        // Clean usage - no checks needed
        diskMonitor.waitForCapacity();
        // ...
    }
}
```

### 4. Cleaner Call Sites

```java
// Providing a monitor
new BlobstoreClient(new PeriodicDiskMonitor(...));

// Not providing a monitor
new BlobstoreClient(DiskMonitor.noop());
```

## Key Takeaways

1.  **Avoid `Optional` in parameters:** It adds verbosity without adding safety.
2.  **Use Null Objects:** Replace optional dependencies with polymorphic "do nothing" implementations.
3.  **Simplify Logic:** By ensuring a dependency always exists (even if it does nothing), you remove conditional branching (`ifPresent`, `if (x != null)`) from your business logic.

**Exception:** Method overloading is another valid alternative for optional parameters, especially in public APIs, but the Null Object pattern scales better for dependencies injected into constructors.
