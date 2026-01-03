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

We’ve used `Optional.map`, `Stream.map`, and a nested lambda. By the time the reader reaches the core logic, they are buried four levels deep in containers and closures. The intent—formatting a key and creating a tag—is obscured by the mechanical noise of the stream pipeline.

We can tame this nesting by flattening the logic and moving from a complex nested expression to a series of linear, readable steps. By handling the empty case immediately with an early return and extracting the transformation logic into a named method, the story becomes clear.

```java
var commonLabels = config.commonLabels();
if (commonLabels.isEmpty()) {
    return List.of();
}

return commonLabels.get().entrySet().stream()
    .map(this::toMetricTag)
    .collect(Collectors.toUnmodifiableList());

// ...

private Tag toMetricTag(Map.Entry<String, String> entry) {
    var key = String.format("%s_%s", MMS_CONFIG_METRIC_COMMON_LABELS_PREFIX, entry.getKey());
    return Tag.of(key, entry.getValue());
}
```

The flattened version respects the reader’s mental flow. The early return signals that the "empty" scenario is handled and can be forgotten. Extracting `toMetricTag` reduces the noise and allows the stream to read like a simple sentence: "map these entries to metric tags." 

Better abstractions don't just hide complexity; they organize it. When you nest deeply, you are implicitly saying the logic is too trivial to be named. But often, naming that logic—even just as a separate variable or a private helper—is exactly what the reader needs. Don't hide your logic in the shadows of deep indentation; give it room to breathe.