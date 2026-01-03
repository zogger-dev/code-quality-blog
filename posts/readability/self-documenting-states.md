# Self-Documenting States

Software development is often a battle against the "mystery value." We've all encountered code that is technically flawless but practically inscrutable without digging through layers of documentation. One of the most common ways we accidentally obfuscate our code is by over-relying on primitive booleans to represent complex state.

Consider the initialization of a `FeatureFlags` object. In its most primitive form, we might see a constructor that looks like a game of technical Tetris:

```java
var flags = new FeatureFlags(
    true, false, true, true, false, false, true, 
    false, true, true, false, true, false, false, true
);
```

This is the "Boolean Trap" at its most extreme. To understand what any single one of those values means, a reader is forced to navigate to the class definition and manually count arguments. It is a minefield for reviewers and a source of constant frustration for maintainers. One misplaced `true` or `false` can result in a catastrophic production bug that is almost impossible to catch via visual inspection.

A common "improvement" is to use a builder or named parameters, which provides some context but still leaves us fighting the limitations of primitives:

```java
var flags = FeatureFlags.builder()
    .retainFailedIndexDataOnDisk(false)
    .enableAsyncBlobstoreUploads(true)
    .useNewReplicationManager(true)
    .build();
```

While this is better, we are still dealing with "mystery booleans." What does `false` mean for retaining data? Is it "never," or is it "only on success"? We are forcing the reader to translate a binary technical primitive into a domain concept.

The true resolution is to move from "How" to "What" by using domain-specific Enums. Enums transform abstract values into self-documenting states that speak the language of the business.

```java
public enum State {
    ENABLED,
    DISABLED
}

// ...

var flags = FeatureFlags.builder()
    .retainFailedData(State.DISABLED)
    .blobstoreUploads(State.ENABLED)
    .replicationManager(State.ENABLED)
    .build();
```

The shift in readability is profound. We no longer have to ask what `false` represents; the code explicitly states `State.DISABLED`. By embedding intent directly into the type system, we gain more than just clarity. Features rarely stay binary; an Enum can gracefully evolve from "Enabled/Disabled" to "Scheduled" or "Shadow Mode" without a messy refactor. It also prevents the accidental swapping of parameters, as the compiler becomes your first line of defense. 

Don't let your code be a collection of mystery values. Whenever you find yourself reaching for a boolean to represent a mode or a default behavior, consider if an Enum could tell the story better.
