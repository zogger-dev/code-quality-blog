# Self-Documenting States

Software development is often a battle against the "mystery value." We've all seen code that is technically flawless but practically inscrutable without constant context-switching. One of the most aggressive ways we obfuscate our intent is by over-relying on primitive booleans to represent complex state.

Consider the initialization of a `FeatureFlags` object. In its most primitive form, we might find a constructor that looks like an exercise in counting:

```java
var flags = new FeatureFlags(
    true, false, true, true, false, false, true, 
    false, true, true, false, true, false, false, true
);
```

This is the "Boolean Trap." To understand what any single value means, a reviewer is forced to find the class definition and manually match arguments. One misplaced `true` can result in a production bug that is almost invisible to the eye. Even moving to a builder with boolean setters only solves the ordering problem; it doesn't solve the semantic mystery. What does `false` mean for a retention policy? Is it "never," or "only on success"?

The true resolution is to elevate the flags themselves to a domain-specific `Feature` enum. This allows the API to move from describing "How" (setting bits) to "What" (enabling concepts).

```java
var flags = FeatureFlags.builder()
    .enable(Feature.RETAIN_FAILED_DATA)
    .enable(Feature.ASYNC_BLOBSTORE_UPLOADS)
    .build();
```

Inside the `FeatureFlags` class, we can carry this philosophy even further by using an `EnumMap` to track the state of every feature. When we check if a feature is active, we compare it against a `State` enum rather than a primitive boolean.

```java
public boolean isEnabled(Feature feature) {
    return featureStates.get(feature) == State.ENABLED;
}
```

This comparison—`map.get(feature) == State.ENABLED`—is vastly more readable than `map.get(feature) == true`. It leverages the type system to provide a self-documenting check that is checked by the compiler.

By replacing mystery booleans with enums, we transform our code into a living guide. Enums can gracefully evolve from a simple binary to "Phased Rollout" or "Shadow Mode" without a messy refactor. They prevent the accidental swapping of parameters and ensure that your intent is always explicit. Don't let your code be a collection of mystery bits; let your types tell the story.