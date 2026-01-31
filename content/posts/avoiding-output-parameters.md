---
title: "Avoiding Output Parameters"
date: 2025-01-08
tags: ["abstraction"]
draft: true
---

Software design is often a quest for clarity in data flow. We want to be able to
look at a method and understand exactly what it takes in and what it produces.
One of the most effective ways to obscure this flow is the **Output Parameter
anti-pattern**—the practice of passing a mutable object into a method so that
the method can "populate" it with results.

## The Side-Effect Trap

Consider a method designed to populate a configuration map for a blobstore
client:

```java
public void populateConfigMap(Map<String, Object> additionalParams) {
    additionalParams.put("storage.provider", "AWS");
    additionalParams.put("storage.bucket", "test-bucket");
    // ... more mutations ...
}
```

This seems convenient. But from the perspective of a caller, it is a source of
hidden complexity.

```java
var params = new HashMap<String, Object>();
populateConfigMap(params);
// what's in params now?
```

The data flow is opaque. To understand how `params` is being populated, the
reader is forced to inspect the implementation of `populateConfigMap`. Worse,
this pattern relies on **Side Effects**. If multiple methods mutate the same
map, the order of operations becomes critical, and debugging a missing or
incorrect key becomes a nightmare of tracing object references through your call
stack.

## The Resolution: Return the Result

We can achieve a much cleaner design by following a simple rule: **Favor methods
that return results over those that mutate state.**

```java
public Map<String, Object> getConfigMap() {
    var config = new HashMap<String, Object>();
    config.put("storage.provider", "AWS");
    config.put("storage.bucket", "test-bucket");
    return config;
}
```

The difference in the caller's code is immediate:

```java
var params = getConfigMap();
// params is populated and ideally immutable
```

The data flow is now explicit. The reader can see exactly where the
configuration comes from. By returning a new (and ideally immutable) object,
we've made the data flow unidirectional and easy to trace.

## The Synthesis

Why is this a superior abstraction?

1. **Transparency:** The method signature now explicitly declares what it
   produces. It's no longer a "black box" that operates on your objects.
2. **Composability:** Functional patterns like `.map()` and `.flatMap()` work on
   return values, not side effects. By returning results, you enable a more
   declarative, modern coding style.

Better abstractions don't just organize logic; they simplify the mental model of
the system. Whenever you find yourself passing a mutable map or list into a
method to be "filled," ask if the method can simply return the result instead.

## The Insight

Don't let your data hide in side effects. Make your data flow explicit by
favoring return values over output parameters. It makes your code easier to
read, easier to test, and far less prone to the "spooky action at a distance"
caused by unexpected mutations.
