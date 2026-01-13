# The Power of Positive Assertions

Writing readable code is an exercise in **clarity**. When a reader opens a file,
their primary goal is to trace the execution path and understand _what happens
next_. Every conditional introduces a fork in this path, requiring the reader to
pause and evaluate which branch is active. One of the most subtle ways we impede
this flow is by forcing the reader to map a negative state to a positive branch.

## The Inversion Tax

Consider a payment processor migrating to a new gateway. The logic relies on a
feature flag to determine which path to take:

```java
public void processPayment(Payment payment) {
    TransactionResult result;

    // The "Mental Check"
    if (!features.isEnabled(Feature.MODERN_PAYMENT_FLOW)) {
        result = legacyGateway.charge(payment);
    } else {
        result = modernGateway.charge(payment);
    }

    // Common logic prevents early return
    auditLog.record(result);
}
```

The single character negation operator (`!`) is notoriously easy to miss while
skimming. To verify which code block executes, the reader must spot this tiny
symbol, pause to evaluate the condition, and then mentally map the `false` state
of the feature to the `true` branch. It turns a simple check into a
double-negative puzzle. By swapping the branches, we can write code that reads
more like a natural sentence.

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

If negation forces a mental tax, does that mean we should _never_ invert a
conditional? Not necessarily. Sometimes we pay the inversion tax to purchase
something even more valuable: **Linearity**. Consider a permission check in a
`DocumentService`. A nested structure buries the core logic behind indentation:

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

A method's indentation should correspond to its complexity. Here, the "happy
path" is indented even though the wrapping `if` is just a gatekeeper, not a
logic branch. By inverting the condition to "eject" the failure case early, we
flatten the structure. The conditional becomes a **Guard Clause**, allowing the
core logic to stay at the root level of the method.

```java
public void publish(Document doc, User user) {
    if (!user.hasRole(Role.EDITOR)) {
        throw new UnauthorizedException("User cannot publish");
    }

    doc.setState(State.PUBLISHED);
    doc.save();
}
```

### Stacking Guards

The value of this pattern multiplies when performing multiple validations. In a
nested world, each check pushes the core logic further to the right, creating an
"Arrowhead" anti-pattern:

```java
public void register(User user) {
    if (user.isValid()) {
         if (!repo.exists(user.id())) {
             // Core logic buried three levels deep
             repo.save(user);
         }
    }
}
```

By stacking guard clauses, we can linearly handle each failure case. The method
reads like a checklist of requirements before the "real work" begins:

```java
public void register(User user) {
    if (!user.isValid()) {
        throw new IllegalArgumentException("Invalid user");
    }

    if (repo.exists(user.id())) {
        throw new EntityExistsException("User already exists");
    }

    repo.save(user);
}
```

## Designing for Positivity

The guard clause above is powerful, but it still relies on that subtle `!`
operator. To determine the exit condition, the reader must still process the
"NOT" symbol. We can align the code closer to our intent by evolving our domain
API to support **Positive Assertions**. Instead of asking "Does the user NOT
have this role?", we could provide a method that explicitly describes the
negative state in positive terms:

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

### The Trade-Off

Critics might argue that adding methods like `lacksRole`, `isMissing`, or
`isUnknown` bloats the API surface with redundant negatives. This is a valid
concern; we shouldn't double the size of every interface just to avoid a `!`.
However, for high-traffic domain objects or critical control flows, the
readability gains outweigh the cost of an extra helper method. By allowing
developers to reach for a positive assertion, we turn complex logic into a
coherent story that matches our intent.
