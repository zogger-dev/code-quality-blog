# The Power of Positive Assertions

Writing readable code is an exercise in minimizing the cognitive energy required
to navigate a logic flow. Every branch and conditional we introduce adds a
"mental stack frame" that the reader must maintain. One of the most subtle ways
we burden this stack is by relying on inverted conditionals and negative
assertions.

To flatten these frames, we should embrace a simple principle: **Focus on what
is, not on what isn't.**

## The Branching Trap

Not all logic can be flattened into a linear path. Sometimes, we genuinely need
to fork execution between two valid paths that merge back together. In these
scenarios, a guard clause (early return) is impossible because shared logic
follows the branch.

Consider a payment processor migrating to a new gateway. The logic relies on a
feature flag to determine which path to take:

```java
public void processPayment(Payment payment) {
    TransactionResult result;

    // The "Mental Stumble"
    if (!features.isEnabled(Feature.MODERN_PAYMENT_FLOW)) {
        result = legacyGateway.charge(payment);
    } else {
        result = modernGateway.charge(payment);
    }

    // Common logic prevents early return
    auditLog.record(result);
}
```

This structure imposes a tax on the reader. They encounter a negation (`!`) and
must mentally invert the condition: _"If NOT modern... okay, that means
legacy."_ While simple here, this inversion becomes a source of bugs as
complexity grows.

We can reduce this load by flipping the conditional to handle the positive case
first. By aligning the code with the direct assertion ("If modern flow is
enabled"), the logic reads like a natural sentence:

```java
public void processPayment(Payment payment) {
    TransactionResult result;

    if (features.isEnabled(Feature.MODERN_PAYMENT_FLOW)) {
        result = modernGateway.charge(payment);
    } else {
        result = legacyGateway.charge(payment);
    }

    auditLog.record(result);
}
```

## The Guard Clause

When the branches do _not_ merge back together (specifically when one branch is
an exit or failure state), we can abandon the `else` block entirely.

Consider a permission check in a `DocumentService`. A nested structure holds the
core logic hostage behind indentation:

```java
public void publish(Document doc, User user) {
    if (user.hasRole(Role.EDITOR)) {
        // Core logic buried here
        doc.setState(State.PUBLISHED);
        doc.save();
    } else {
        throw new UnauthorizedException("User cannot publish");
    }
}
```

We can flatten this using a **Guard Clause**, a check that returns or throws
early. This keeps the "happy path" at the lowest level of indentation:

```java
public void publish(Document doc, User user) {
    if (!user.hasRole(Role.EDITOR)) {
        throw new UnauthorizedException("User cannot publish");
    }

    doc.setState(State.PUBLISHED);
    doc.save();
}
```

## Designing for Positivity

The guard clause above is powerful, but it reintroduces our nemesis: the
negation (`!`). A single character negation operator is easy to miss while
skimming through code quickly and slows down the reader when a surprise jumps
out. The reader must still process the "NOT" symbol to understand the exit
condition.

We can reach the ultimate level of readability by evolving our domain API to
support **Positive Assertions**. Instead of asking "Does the user NOT have this
role?", we could provide a method that explicitly describes the negative state
in positive terms:

```java
public void publish(Document doc, User user) {
    if (user.lacksRole(Role.EDITOR)) {
        throw new UnauthorizedException("User cannot publish");
    }

    doc.setState(State.PUBLISHED);
    doc.save();
}
```

The logic now aligns perfectly with the intent. The negation symbol is gone, and
the code reads exactly as you would describe it in a code review: _"If the user
lacks the editor role, throw an exception."_

## The Trade-Off

Critics might argue that adding methods like `lacksRole`, `isMissing`, or
`isUnknown` bloats the API surface with redundant negatives. This is a valid
concern; we shouldn't double the size of every class just to avoid a `!`.

However, for high-traffic domain objects or critical control flows, the
readability gains outweigh the cost of an extra helper method. By allowing
developers to reach for a positive assertion, we turn complex logic into a
coherent story that matches our intent.
