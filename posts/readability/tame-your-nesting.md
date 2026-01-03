# Tame Your Nesting!

Deeply nested code is a hiding place for technical debt. Every level of indentation adds a mental "stack frame" that the reader must maintain just to understand what a single line of code is doing. While modern functional tools like `Optional` and `Stream` are powerful, they can inadvertently encourage a style of nesting that is just as opaque as the legacy "if-pyramid."

Take, for example, a transformation designed to turn configuration labels into metric tags:

```java
var mmsConfiguredTags =
    config
        .commonLabels()
        .map(
            tags ->
                tags.entrySet().stream()
                    .map(
                        entry -> {
                          var key =
                              String.format(
                                  "%s_%s",
                                  MMS_CONFIG_METRIC_COMMON_LABELS_PREFIX, entry.getKey());
                          return Tag.of(key, entry.getValue());
                        })
                    .collect(Collectors.toUnmodifiableList()))
        .orElseGet(Collections::emptyList);
```

We’ve used `Optional.map`, `Stream.map`, and a nested lambda. By the time the reader reaches the core logic, they are buried four levels deep in containers and closures. But the real issue here isn't just the nesting—it's the abstraction itself. `config.commonLabels()` is returning an `Optional<Map>`, which forces every caller to manage the "absence" of a collection.

We can tame this nesting by improving the abstraction. If `commonLabels()` followed the Null Object Pattern and returned an empty map instead of an empty `Optional`, the logic collapses into a single, linear flow:

```java
return config.commonLabels().entrySet().stream()
    .map(this::toMetricTag)
    .collect(Collectors.toUnmodifiableList());

// ...

private Tag toMetricTag(Map.Entry<String, String> entry) {
    var key = String.format("%s_%s", MMS_CONFIG_METRIC_COMMON_LABELS_PREFIX, entry.getKey());
    return Tag.of(key, entry.getValue());
}
```

The difference is staggering. By returning an empty collection, we eliminate the need for the `Optional` container entirely. The code moves from a nested puzzle to a simple sentence: "map these labels to metric tags." 

Better abstractions don't just hide complexity; they eliminate the need for it. When you return an `Optional<Collection>`, you are forcing your callers to deal with two different types of "emptiness." By choosing a better return type, you allow the logic to breathe and the reader to focus on the intent. Don't hide your logic in the shadows of deep indentation; give it a better foundation.
