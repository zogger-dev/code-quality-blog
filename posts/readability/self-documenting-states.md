# Self-Documenting States

Software development is often a battle against the "mystery value." We've all encountered code that is technically flawless but practically inscrutable without digging through layers of documentation. One of the most common ways we accidentally obfuscate our code is by over-relying on primitive booleans to represent complex state.

Imagine you are looking at a configuration for a suite of feature flags. You see a list of features, each initialized with a simple boolean:

```java
RETAIN_FAILED_INDEX_DATA_ON_DISK("retainFailedIndexDataOnDisk", false),
ENABLE_ASYNC_BLOBSTORE_UPLOADS("enableAsyncBlobstoreUploads", true),
```

What does `false` mean here? Does it mean the feature is "off"? Does it mean it's "not supported"? Does it mean it's "manual only"? While we can guess the intent based on the feature name, the primitive `false` provides no semantic clarity. It is a "mystery boolean" that forces the reader to pause and translate a technical primitive into a domain concept.

We can eliminate this cognitive friction by replacing mystery booleans with domain-specific Enums. Enums transform an abstract value into a self-documenting state that speaks the language of the business.

```java
public enum State {
    ENABLED,
    DISABLED
}

// ...

RETAIN_FAILED_INDEX_DATA_ON_DISK("retainFailedIndexDataOnDisk", State.DISABLED),
ENABLE_ASYNC_BLOBSTORE_UPLOADS("enableAsyncBlobstoreUploads", State.ENABLED),
```

The shift in readability is profound. We no longer have to ask what `false` represents; the code explicitly states `State.DISABLED`. This moves the code from describing the "How" (a bit is flipped) to the "What" (the feature is in a disabled state). 

By embedding intent directly into the type system, we gain more than just clarity. Features rarely stay binary; an Enum can gracefully evolve from "Enabled/Disabled" to "Scheduled" or "Shadow Mode" without a messy refactor. It also provides a level of type safety that primitives cannot match, preventing the accidental swapping of parameters. 

Don't let your code be a collection of mystery values. Whenever you find yourself reaching for a boolean to represent a mode or a default behavior, consider if an Enum could tell the story better. Your future self will thank you.