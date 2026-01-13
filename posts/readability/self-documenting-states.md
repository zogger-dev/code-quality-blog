# Beyond Booleans: Modeling Feature Flags as Data

Software development is often a battle against the "mystery value." One of the
most aggressive ways intent is obfuscated is by over-relying on primitive
booleans to represent complex state. When we treat domain concepts as simple
toggles, we lose the semantic meaning of the states we are trying to manage.

Nowhere is this more apparent than in the evolution of a `FeatureFlags` system.
What starts as a simple, pragmatic toggle often grows into a massive sequence of
primitives that creates two distinct liabilities: positional fragility and a
high maintenance tax.

## Positional Fragility

Consider a standard `FeatureFlags` implementation defining a massive list of
booleans. The structure itself invites chaos:

```java
public record FeatureFlags(
    boolean enableAsyncProcessing,
    boolean retainFailedRequestLogs,
    boolean strictSchemaValidation,
    // ... many more booleans
    boolean useDarkTheme
) {}
```

In production, populating this object involves a brittle mapping layer that
reads from a configuration source. While the values themselves aren't mysterious
(we can see the config keys), their _positions_ are rigid and unforgiving.

```java
// Production Code
var flags = new FeatureFlags(
    config.getBoolean("async.enabled").orElse(false),
    config.getBoolean("schema.verify").orElse(false),
    config.getBoolean("logs.retain").orElse(true),
    // ... many more lines of mapping
    config.getBoolean("ui.dark_mode").orElse(false)
);
```

A subtle swap of two adjacent boolean parameters—perhaps "logs.retain" and
"schema.verify"—results in a bug that is invisible during review (did you catch
the swap?). The compiler sees only a sequence of `true` and `false`.

## Maintenance Tax

The cost of this design compounds in the test suite, where it levies a heavy tax
in three distinct forms:

1. **Mystery Values:** Tests use literals (`true`, `false`), forcing reviewers
   to count arguments to understand which feature is enabled.
2. **Irrelevant Details:** A test focused on "Async Processing" must still
   provide values for "Dark Mode," violating the principle of
   [Including Only Relevant Details](https://testing.googleblog.com/2023/10/include-only-relevant-details-in-tests.html).
3. **Blast Radius:** Adding a single new flag breaks the constructor signature,
   forcing you to update hundreds of unrelated tests.

```java
// Test Code: Paying the Tax
var flags = new FeatureFlags(
    false, true, true, false, false, true, false, // Mystery noise
    true,  // The only relevant flag for the current test
    false, true, true, false, false, true, false  // Mystery noise
);
```

## The Builder Pattern

The standard remedy for "Constructor Bloat" is the Builder Pattern. It replaces
positional arguments with named methods, allowing us to specify only the
relevant flags and rely on defaults for the rest.

```java
// Test Code: Using a Builder
var flags = FeatureFlags.builder()
    .asyncEnabled(true)
    .build();
```

This addresses "Mystery Values" and "Irrelevant Details," but it fails to fully
eliminate the "Maintenance Tax." The builder still relies on the underlying
`FeatureFlags` class having a hardcoded field for every single flag. Adding a
new feature still requires modifying the class, the builder, and the mapping
logic—the "Blast Radius" remains.

Frameworks like Lombok can automate the generation of these builders (e.g., via
`@Builder`), reducing the manual effort of maintaining the builder class.
However, this only addresses the _boilerplate_ tax, not the _architectural_ tax.

```java
FeatureFlags.builder()
    .asyncEnabled(config.getBoolean("async.enabled").orElse(false))
    .retainLogs(config.getBoolean("logs.retain").orElse(true))
    .strictSchema(config.getBoolean("schema.verify").orElse(false))
    .build();
```

As the example above shows, the we still have to manually map the flags to their
configuration keys. And every time we add a new flag, we have to add a new line
in the mapping logic. Additionally, if we want to report the flags to metrics,
we have to add a new line for every new flag.

```java
public void reportFlagsToMetrics(FeatureFlags flags) {
    // Every new flag requires a new line here to ensure visibility
    metrics.report("feature.async.enabled", flags.isAsyncEnabled());
    metrics.report("feature.logs.retain", flags.isRetainLogs());
    metrics.report("feature.schema.verify", flags.isStrictSchema());
}
```

## Modeling State as Data

The root cause isn't just the long parameter list; it's that we are modeling
_domain concepts_ (features) as _primitive codes_.

The resolution is to elevate the features themselves to a domain-specific
`Feature` enum. This allows us to centralize the "truth" of the system—including
names, configuration keys, and default states—into a single definition.

Note how we name the feature as a noun (`ASYNC_PROCESSING`) rather than a
command (`ENABLE_...`), treating it as a concept rather than an instruction.

```java
public enum Feature {
  ASYNC_PROCESSING("async.enabled", State.DISABLED),
  RETAIN_FAILED_REQUEST_LOGS("logs.retain", State.DISABLED),
  STRICT_SCHEMA_VALIDATION("schema.strict", State.ENABLED);
  // ...

  private final String configKey;
  private final State defaultState;
}
```

## The Maintainability Bonus

By replacing mystery booleans with a proper domain model, we gain
**Iterability**. Our `FeatureFlags` container collapses from a massive list of
fields into a simple wrapper around an `EnumMap`.

This architecture satisfies the **Open-Closed Principle (OCP)**: we can add new
features without modifying the container class or its factory logic.

```java
public class FeatureFlags {
    private final EnumMap<Feature, State> states;

    // The "Blast Radius" is now zero. We iterate over the Enum to populate the map.
    // No code changes required in this class when adding a new feature.
    public static FeatureFlags from(Configuration config) {
        var states = new EnumMap<Feature, State>(Feature.class);
        for (Feature feature : Feature.values()) {
             boolean isEnabled = config.getBoolean(feature.configKey)
                                       .orElse(feature.defaultState == State.ENABLED);
             states.put(feature, isEnabled ? State.ENABLED : State.DISABLED);
        }
        return new FeatureFlags(states);
    }

    public boolean isEnabled(Feature feature) {
        return states.getOrDefault(feature, feature.defaultState) == State.ENABLED;
    }
}
```

With our Enum model, the code becomes data-driven. We can iterate over the
"Truth" to guarantee that our observability is always in sync with our
implementation without manual maintenance:

```java
// Zero-Touch Observability
for (Feature feature : Feature.values()) {
    metrics.emit(
        "feature." + feature.configKey,
        flags.isEnabled(feature)
    );
}
```

### The Reward

| Liability                | Record (Booleans)         | Builder Pattern                | Enum Model              |
| :----------------------- | :------------------------ | :----------------------------- | :---------------------- |
| **Positional Fragility** | 🔴 High                   | 🟡 Medium                      | 🟢 None (Named keys)    |
| **Maintenance Tax**      | 🔴 High (Map every field) | 🟡 Medium (Update builder)     | 🟢 Low (Iterate values) |
| **Blast Radius**         | 🔴 High (Break signature) | 🟡 Medium (Observability code) | 🟢 Zero (OCP Compliant) |

## Future Proofing

Modeling implementation details as data also allows us to evolve the definition
of "State" without breaking every call site. Imagine we need a third state,
`CONTROLLED`, to support gradual rollouts.

In the boolean world, this would require refactoring every `boolean` field into
a complex object. With our Enum-based model, we simply add the new state to the
`State` enum and update the logic in one place. The rest of the application
remains unaware of the change.

The transition from primitive booleans to a data-driven Enum model is more than
a refactor for readability; it is a shift in architectural philosophy. By
treating features as first-class domain concepts rather than positional
arguments, we eliminate the "mystery value" and turn a brittle, high-maintenance
component into a self-documenting system.

When your configuration is modeled as data, your system becomes **Open-Closed**:
adding a new capability requires touching only the definition of the feature
itself, leaving your factories, tests, and observability logic untouched. Don't
let your state hide behind anonymous primitives—elevate it, iterate it, and let
it document itself.
