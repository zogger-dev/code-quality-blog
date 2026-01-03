# Fakes vs Mocks: Testing Behavior, Not Implementation

**Date:** January 2026
**Topic:** Testing Patterns, Quality, Java

## The Problem

When writing unit tests for components with dependencies, we often reach for mocking frameworks like Mockito. While powerful, over-reliance on mocks—specifically **interaction-based mocking**—leads to brittle tests that break during refactoring, even if the external behavior remains correct.

## The Anti-Pattern: Brittle Interaction Mocks

Suppose we are testing a `Downloader` that checks a `FileStore` for available space.

```java
@Test
public void shouldNotDownloadIfSpaceLow() {
    FileStore mockStore = mock(FileStore.class);
    // BAD: Mocking the internal interaction details
    when(mockStore.getTotalSpace()).thenReturn(1000L);
    when(mockStore.getUsableSpace()).thenReturn(50L);

    Downloader downloader = new Downloader(mockStore);
    downloader.download("file.zip");

    // BAD: Verifying specific method calls
    verify(mockStore).getUsableSpace();
}
```

### Why this is brittle:

1.  **Refactoring Blocker:** If you rename `getUsableSpace()` to `getFreeSpace()` or change the logic to use a different metric (e.g., percentage), the test fails, even though the `Downloader`'s behavior (stopping the download) is still correct.
2.  **Implementation Leakage:** The test has to know *exactly how* the `Downloader` interacts with the `FileStore` to set up the mock correctly.
3.  **Setup Fatigue:** Complex dependencies require long chains of `when(...).thenReturn(...)` which obscure the intent of the test.

## The Solution: Using Fakes

A **Fake** is a lightweight, working implementation of an interface that uses a simplified mechanism (usually in-memory) to manage state.

### 1. Create the Fake

Instead of mocking method calls, we create a simple implementation that behaves like the real thing but is fully controlled by the test.

```java
public class FakeFileStore implements FileStore {
    private long totalSpace = 1000;
    private long usableSpace = 1000;

    // Helper methods for the test to set state
    public void setUsableSpace(long space) { this.usableSpace = usableSpace; }

    @Override public long getTotalSpace() { return totalSpace; }
    @Override public long getUsableSpace() { return usableSpace; }
}
```

### 2. Test with State, Not Interactions

Now the test becomes a story about **state and behavior**.

```java
@Test
public void shouldNotDownloadIfSpaceLow() {
    FakeFileStore fakeStore = new FakeFileStore();
    // GOOD: Set the state directly
    fakeStore.setUsableSpace(50); 

    Downloader downloader = new Downloader(fakeStore);
    downloader.download("file.zip");

    // GOOD: Assert the OUTCOME (the behavior), not the interaction
    assertFalse(downloader.isDownloading());
}
```

### Why Fakes are superior:

1.  **Refactoring Friendly:** You can change the `Downloader` implementation entirely. As long as it still respects the "low space" state of the `FileStore`, the test remains green.
2.  **Reusable:** The `FakeFileStore` can be used across dozens of different test suites.
3.  **Clarity:** The test setup focus on the "Scenario" (Usable space is 50) rather than the "Mechanism" (When method X is called, return Y).

## When to use which?

-   **Use Mocks** for "Procedures": Verifying that an email was sent or a specific event was logged (side effects).
-   **Use Fakes** for "State Providers": Data stores, configuration services, or resource monitors where your component reacts to the *data* provided.

## Key Takeaways

1.  **Avoid Mocking "Getter" Methods:** If you find yourself mocking methods that just return data, you probably need a Fake.
2.  **Test the "What", not the "How":** Focus on the final result of the operation, not the sequence of internal method calls.
3.  **Invest in Fixtures:** Building a small library of Fakes for your core interfaces will pay dividends in test speed and maintainability.

By switching from Mocks to Fakes, you move from testing the *implementation* to testing the *contract*, leading to a much more robust and flexible test suite.
