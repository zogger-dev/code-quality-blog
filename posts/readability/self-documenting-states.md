# Self-Documenting States

Software development is often a battle against the "mystery value." One of the
most aggressive ways intent is obfuscated is by over-relying on primitive
booleans to represent complex state.

Consider the initialization of a `FeatureFlags` object. In its most primitive
form, a constructor might look like a game of technical Tetris:

```java
var flags = new FeatureFlags(
    true, false, true, true, false, false, true,
    false, true, true, false, true, false, false, true
);
```

This is the "Boolean Trap." To understand what any single value means, a
reviewer is forced to find the class definition and manually count arguments.
Even if these values are injected from configuration rather than hardcoded
literals, the structural fragility remains. A subtle swap of two adjacent
boolean parameters can result in a production bug that is invisible during code
review. What does the third `false` mean? Is it for a retention policy? Is it
"never," or is it "only on success"?

The resolution is to elevate the features themselves to a domain-specific
`Feature` enum. This allows for the definition of the default state alongside
the feature itself, ensuring the "truth" is centralized rather than scattered
across constructors.

```java
public enum Feature {
  ENABLE_ASYNC_PROCESSING("enableAsyncProcessing", State.DISABLED),
  RETAIN_FAILED_REQUEST_LOGS("retainFailedRequestLogs", State.DISABLED),
  STRICT_SCHEMA_VALIDATION("strictSchemaValidation", State.ENABLED);
  // ...
}
```

With the defaults captured in the enum, a `withDefaults()` creator can be
exposed to guarantee a safe starting state. Consumers only need to specify the
overrides, moving the API from setting bits to enabling concepts:

```java
var flags = FeatureFlags.withDefaults()
    .enable(Feature.RETAIN_FAILED_REQUEST_LOGS)
    .enable(Feature.ENABLE_ASYNC_PROCESSING)
    .build();
```

Inside the `FeatureFlags` class, this symmetry is maintained by using an
`EnumMap` to track states. When checking if a feature is active, it is compared
against a `State` enum rather than a primitive boolean.

```java
public boolean isEnabled(Feature feature) {
    return featureStates.get(feature) == State.ENABLED;
}
```

While using a boolean map (where `map.get(feature) == true`) would be
functionally equivalent, the comparison `map.get(feature) == State.ENABLED`
arguably offers superior scan-ability. It allows a reader to glance at the
implementation and be confident it is correct without mentally translating a
boolean to a business state. It transforms the question from "is this bit 1?" to
"is this feature in the enabled state?"

By replacing mystery booleans with enums, the code effectively documents itself.
The `Feature` enum becomes the single source of truth for both identity and
default behavior, preventing accidental parameter swapping and ensuring that
intent is always explicit. Ultimately, true readability is defined not by how
much is written _about_ the code, but by the tangible absence of "code comments
explaining what the code does."
