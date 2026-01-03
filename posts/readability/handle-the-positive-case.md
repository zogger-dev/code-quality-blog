# Handle the Positive Case

Writing readable code is an exercise in minimizing the cognitive energy required to navigate a logic flow. One of the most subtle yet effective ways to respect a reader's cognitive budget is to avoid the temptation of the inverted conditional.

Consider a configuration bootstrapper tasked with initializing a disk monitor. It's a common pattern: we only want to set up the full monitoring suite if the configuration is actually present. It's easy to fall into a structure like this:

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

At first glance, this code is functional and correct. But notice the mental tax it imposes on the reader. The very first thing they encounter is a negation—the "not having." To understand the subsequent block, the reader must maintain a mental state of absence. By the time they reach the `else` block—where the primary intent of the method actually lives—they’ve already spent cognitive points on the "missing" case.

We can drastically improve the scanability of this logic by aligning the code flow with the reader's natural expectation: "If I have X, then do Y." By simply handling the positive case first, the "happy path" moves front-and-center.

```java
if (diskMonitorConfig.isPresent()) {
    var config = diskMonitorConfig.get();
    initializeHysteresisGates(config);
    registerWithMonitor(monitor);
} else {
    initializeAlwaysOpenGates();
}
```

The difference is immediate. The `!` at the beginning, which is famously easy to miss during a quick scan, is gone. The reason the code exists is now the first thing the reader processes. 

Better abstractions are built on clear intent. When we favor the positive case, we allow the reader to process the primary logic while their mental context is fresh. We reduce the risk of the "missing bang" bug and turn a logic check into a natural sentence. Always favor the positive case; it respects the reader's flow and makes your code's purpose unmistakable.