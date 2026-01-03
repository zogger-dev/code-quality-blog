# Handle the Positive Case

Writing readable code is often about minimizing the cognitive energy required to understand a logic flow. One of the most subtle yet effective ways to achieve this is by avoiding the **Inverted Conditional anti-pattern**.

## The Tension

Consider this snippet from a configuration bootstrapper. It’s tasked with initializing a disk monitor, but only if the configuration exists.

```java
if (!diskMonitorConfig.isPresent()) {
    // default behavior if config is missing
    initializeAlwaysOpenGates();
} else {
    // core logic
    var config = diskMonitorConfig.get();
    initializeHysteresisGates(config);
    registerWithMonitor(monitor);
}
```

At first glance, this code seems fine. But notice the first thing the reader encounters: a negation (`!`). To understand the subsequent block, the reader must maintain a mental state of "not having something." By the time they reach the `else` block—where the primary logic actually lives—they’ve already spent cognitive points on the "missing" case.

## The Resolution

We can drastically improve the scanability of this code by simply handling the positive case first. This aligns the code flow with the reader's natural expectation: "If I have X, then do Y."

```java
if (diskMonitorConfig.isPresent()) {
    var config = diskMonitorConfig.get();
    initializeHysteresisGates(config);
    registerWithMonitor(monitor);
} else {
    initializeAlwaysOpenGates();
}
```

The difference is immediate. The "happy path"—the reason this code exists—is front-and-center. The `!` at the beginning, which is famously easy to miss during a quick scan, is gone. 

## The Synthesis

Why does this matter? Better abstractions are built on clear intent. When we use inverted conditionals, we force the reader to process the "exception" or "default" case before the "intent." This adds a layer of mental indirection that compounds as methods grow in complexity.

By favoring the positive case, we:
1.  **Reduce Cognitive Load:** The reader processes the primary logic while their mental context is fresh.
2.  **Improve Scanability:** A quick glance reveals the "if something exists" condition, which is easier to grasp than "if something does *not* exist."
3.  **Prevent "Missing Bang" Bugs:** Negation symbols are tiny and easy to overlook in code reviews. Handling the positive case eliminates this risk entirely for the primary block.

## The Insight

Always handle the positive case first. It makes your code flow naturally, reduces the risk of missed negations, and respects the reader's cognitive budget.
