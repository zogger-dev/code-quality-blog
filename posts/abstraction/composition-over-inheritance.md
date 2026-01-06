# Composition over Inheritance

"Composition over Inheritance" is a classic principle of object-oriented design.
While inheritance is often the first tool developers reach for when sharing
code, it frequently leads to fragile, rigid systems that are difficult to reason
about. This is especially true in test suites, where inheritance can create
"False Polymorphism" that obscures the very behavior being tested.

## The Inheritance Trap

Consider a test suite for different cloud storage providers (AWS, Azure, GCP).
To avoid duplicating test logic, a developer creates an abstract base class:

```java
public abstract class BlobstoreConfigBaseUnitTests {
    @Test
    public void testSerialization() {
        var config = createConfig();
        var map = config.populateMap();
        assertMapMatches(map, getExpectedValues());
    }

    protected abstract BlobstoreConfig createConfig();
    protected abstract Map<String, Object> getExpectedValues();
}
```

This seems efficient. But look at the cost. To understand a single test in a
subclass, the reader has to weave back and forth between files. The
"Arrange-Act-Assert" flow is split across classes. Worse, because AWS and Azure
configs have fundamentally different schemas, the base class forces them into a
"False Polymorphism." We end up with awkward "data-provider" methods just to
satisfy the inheritance structure.

## The Resolution: Composition through Helpers

We can achieve better results by abandoning inheritance in favor of
**Composition**. Instead of a base class that dictates the structure, we use
focused helper methods that can be composed as needed.

```java
public class AwsBlobstoreConfigUnitTests {
    @Test
    public void testSerialization() {
        // Arrange
        var config = new AwsBlobstoreConfig("test-bucket", "us-east-1");

        // Act
        var map = config.populateMap();

        // Assert
        assertAwsMetadata(map, "test-bucket", "us-east-1");
    }
}

// ... helper in a separate utility or local ...
public static void assertAwsMetadata(
        Map<String, Object> map, String bucket, String region) {
    assertThat(MapUtils.getDeep(map, "storage.bucket")).isEqualTo(bucket);
    assertThat(MapUtils.getDeep(map, "storage.region")).isEqualTo(region);
}
```

This design is vastly more readable. The test is self-contained and follows a
clear linear flow. The shared logic is extracted into a meaningful helper
(`assertAwsMetadata`) that can be reused across tests without imposing a rigid
hierarchy.

## The Synthesis

Why is composition the right tool here?

1. **Linear Flow:** Tracing the data flow for a single test doesn't require
   jumping between multiple files. The entire scenario is visible in one place.
2. **Avoids False Polymorphism:** We aren't forcing AWS and Azure into a shared
   contract that they only partially satisfy. We handle their differences
   explicitly while sharing the "plumbing" (like map traversal) through
   composition.
3. **Refactor-Friendly:** Adding a new test scenario or a new provider is
   simple. You aren't fighting a base class's rigid template.

Inheritance should be used when you are verifying a **Behavioral Contract**
(e.g., "all Lists must behave like Lists"). If you are simply trying to share
code or avoid duplication, **Composition** is almost always the better choice.

## The Insight

Don't let the quest for DRY code lead you into the inheritance trap. If your
base class exists solely to share setup or assertion logic, delete it. Use
composition to build focused, self-contained tests that prioritize the reader's
mental flow over code deduplication.
