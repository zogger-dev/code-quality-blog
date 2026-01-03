# Tame Your Nesting!

Deeply nested code is one of the most effective ways to hide intent. Every level of indentation adds a mental "stack frame" that the reader must maintain just to understand what a single line of code is doing. While modern features like `Optional` and `Stream` are powerful, they can inadvertently encourage a functional style of nesting that is just as opaque as the "if-pyramid" of old.

## The Challenge

Take a look at this snippet designed to transform configuration labels into a list of metric tags.

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

We’ve used `Optional.map`, `Stream.map`, and a nested lambda with a `String.format` and a `Tag.of` call. By the time the reader gets to the core transformation, they are buried four levels deep in nested closures. The "happy path" is obscured by the machinery of the containers.

## The Resolution

We can "tame" this nesting by flattening the logic. The goal is to move from a single complex expression to a series of simple, linear steps.

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

## The Synthesis

Why is the flattened version superior? It’s because it respects the reader’s mental linear flow. 

1.  **Early Returns:** We handle the empty case immediately. The reader can now focus entirely on the transformation logic without worrying about the `Optional` container.
2.  **Method Extraction:** By moving the `String.format` logic into a private helper (`toMetricTag`), we reduce the noise inside the `stream().map()` call. The stream now reads like a sentence: "map these entries to metric tags."
3.  **Readability over "Cleverness":** The first version is "idiomatic" functional Java, but it's hard to debug and hard to scan. The second version is slightly more verbose but infinitely more readable.

Better abstractions don't just hide complexity; they organize it. When you nest deeply, you are effectively saying, "this logic is too small to be named." But often, naming that logic—even just as a separate variable or a private method—is exactly what the reader needs to understand your intent.

## The Insight

Don't let your logic hide in the shadows of deep indentation. Use early returns, extract helper methods, and favor linear flows over complex nested expressions. If your code is more than three levels deep, it's time to tame the nesting.
