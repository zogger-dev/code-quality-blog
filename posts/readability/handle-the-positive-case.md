# Handle the Positive Case

Writing readable code is an exercise in minimizing the cognitive energy required to navigate a logic flow. One of the most subtle yet effective ways to respect a reader's cognitive budget is to avoid the temptation of the inverted conditional.

Consider a simple process for validating a user's permission to edit a document. It’s easy to fall into a structure that prioritizes the "exception" case:

```java
public void updateDocument(Document doc, User user) {
    if (!user.hasRole(Role.EDITOR)) {
        log.warn("User {} attempted to edit without permissions", user.getId());
    } else {
        doc.updateContent(newContent);
        doc.save();
        log.info("Document {} updated by {}", doc.getId(), user.getId());
    }
}
```

At first glance, this code is functional and correct. But notice the mental tax it imposes on the reader. The very first thing they encounter is a negation—the "not having." To understand the subsequent block, the reader must maintain a mental state of absence. By the time they reach the `else` block—where the primary intent of the method actually lives—they’ve already spent cognitive points on the "unauthorized" case.

We can drastically improve the scanability of this logic by aligning the code flow with the reader's natural expectation: "If I have X, then do Y." By simply handling the positive case first, the "happy path" moves front-and-center.

```java
public void updateDocument(Document doc, User user) {
    if (user.hasRole(Role.EDITOR)) {
        doc.updateContent(newContent);
        doc.save();
        log.info("Document {} updated by {}", doc.getId(), user.getId());
    } else {
        log.warn("User {} attempted to edit without permissions", user.getId());
    }
}
```

The difference is immediate. The "happy path"—the reason this code exists—is now the first thing the reader processes. The `!` at the beginning, which is famously easy to miss during a quick scan, is gone. We’ve turned a logic check into a natural sentence that matches the intent of the feature.

Better abstractions are built on clear intent. When we favor the positive case, we allow the reader to process the primary logic while their mental context is fresh. We reduce the risk of missed negations and treat the reader's cognitive budget with respect. Always favor the positive case; it makes your code's purpose unmistakable.
