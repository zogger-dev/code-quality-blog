# Handle the Positive Case

Writing readable code is an exercise in minimizing the cognitive energy required to navigate a logic flow. Every branch and conditional we introduce adds a "mental stack frame" that the reader must maintain. One of the most effective ways to flatten these frames is to avoid the inverted conditional and embrace the power of the guard clause.

Consider a permission check for a document editor. It’s easy to fall into a nested structure where the core logic is held hostage by an `else` block:

```java
public void updateDocument(Document doc, User user) {
    if (!user.hasRole(Role.EDITOR)) {
        log.warn("Unauthorized edit attempt");
    } else {
        // Core logic buried here
        doc.updateContent(newContent);
        doc.save();
    }
}
```

This imposes a mental tax. The reader encounters a negation (`!`) and must hold that state of absence in their mind while scanning for the "real" work. We can improve this by handling the positive case first, aligning the code with the reader's expectation: "If X, then Y."

However, we can go further. By using a guard clause—a negated check that returns early—we can eliminate the `else` block entirely. This keeps the subsequent code at the lowest level of indentation, making the "happy path" a straight line down the page.

```java
public void updateDocument(Document doc, User user) {
    if (!user.hasRole(Role.EDITOR)) {
        log.warn("Unauthorized edit attempt");
        return;
    }

    doc.updateContent(newContent);
    doc.save();
}
```

The guard clause is powerful because it allows the reader to "fire and forget" the exception. Once the check passes, the cognitive load is reset. 

But there is a catch. Guard clauses work best when the negated case is an exit point—a failure or a termination. If you are choosing between two equally valid paths, the positive case should almost always come first. Imagine a system that routes a notification based on a user preference. If you handle the "Email" case only if "SMS" is NOT preferred, you’ve forced the reader to process a negation just to find a valid alternative. 

Better abstractions treat the reader's flow as sacred. Use guard clauses to clear the brush of exceptions early, but when defining the core narrative of your feature, always handle the positive case first. It turns your logic into a story that matches your intent.