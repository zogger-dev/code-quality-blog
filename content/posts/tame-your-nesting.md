---
title: "Tame Your Nesting"
date: 2025-01-03
tags: ["readability"]
draft: true
---

Deeply nested code is a hiding place for technical debt. Every level of
indentation adds a mental "stack frame" that the reader must maintain just to
understand what a single line of code is doing. While modern functional tools
like `Optional` and `Stream` are powerful, they can inadvertently encourage a
style of nesting that is just as opaque as the legacy "if-pyramid."

Take, for example, a transformation designed to turn configuration labels into
metric tags:

```java
var appConfiguredTags =
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
                                  METRIC_LABEL_PREFIX, entry.getKey());
                          return Tag.of(key, entry.getValue());
                        })
                    .collect(Collectors.toUnmodifiableList()))
        .orElseGet(Collections::emptyList);
```

By the time the reader reaches the actual transformation logic (the
`String.format`), they are buried three levels deep in containers and closures.

But the real issue here isn't just the nesting—it's the abstraction itself.
`config.commonLabels()` is returning an `Optional<Map>`, which is a common
anti-pattern that forces every caller to manage two different kinds of
"emptiness": the absence of the map, and the emptiness of the map.

We can tame this nesting by fixing the underlying abstraction. If we follow the
**Null Object Pattern** and ensure `commonLabels()` returns an empty map instead
of an empty `Optional`, we can lift the logic out of the conditional. Combining
this with a simple method extraction (`toMetricTag`), the entire structure
collapses into a single, linear return:

```java
return config.commonLabels().entrySet().stream()
    .map(this::toMetricTag)
    .collect(Collectors.toUnmodifiableList());
```

And the helper method cleanly encapsulates the formatting logic:

```java
private Tag toMetricTag(Map.Entry<String, String> entry) {
    var key = String.format("%s_%s", METRIC_LABEL_PREFIX, entry.getKey());
    return Tag.of(key, entry.getValue());
}
```

The difference is staggering. By choosing a better return type, we eliminate the
need for the `Optional` container entirely. The code moves from a nested puzzle
to a simple, declarative sentence: "map these labels to metric tags."

Better abstractions don't just hide complexity; they eliminate the need for
special handling. When you return an `Optional<Collection>`, you are forcing
your callers to handle specific edge cases that shouldn't exist. By returning an
empty collection, you allow the logic to breathe and the reader to focus on the
intent. Don't hide your logic in the shadows of deep indentation; give it a
better foundation.
