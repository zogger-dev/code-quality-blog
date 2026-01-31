---
title: "The Power of Positive Assertions"
date: 2026-01-31
tags: ["readability"]
draft: false
---

Every `if-else` block tells a story with two chapters. The positive branch runs
when the condition is true. The negative branch, when it's false. Negate the
condition, and they swap places—now the positive branch handles the negative
case, and readers must mentally flip the narrative to follow along.

## The Inversion Tax

Consider a payment processor migrating to a new gateway:

```java
public void processPayment(Payment payment) {
    TransactionResult result;

    if (!features.isEnabled(Feature.MODERN_PAYMENT_FLOW)) {
        result = legacyGateway.charge(payment);
    } else {
        result = modernGateway.charge(payment);
    }

    auditLog.record(result);
}
```

The branches are misaligned. The condition asks "is modern flow enabled?" but
the positive branch handles the legacy case. The reader must spot the `!`
operator, mentally invert the question, and map the negated result back to the
branch. This is the **inversion tax**—the cognitive overhead of processing
misaligned predicates.

The condition asks about modern. The branch handles legacy. The reader must
invert the question to follow the answer.

Swapping the branches aligns the code with its intent:

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

Now the positive branch handles the positive case. When the modern flow is
enabled, we use the modern gateway. The code reads as it thinks.

Misaligned predicates make the reader think twice. Aligned predicates let the
reader think once.

## The Guard Clause Exception

If negation incurs a tax, should we _never_ invert a conditional? Not quite.
**Guard clauses** are the exception—and for good reason.

Consider a permission check in a `DocumentService`:

```java
public void publish(Document doc, User user) {
    if (user.hasRole(Role.EDITOR)) {
        doc.setState(State.PUBLISHED);
        doc.save();
    } else {
        throw new UnauthorizedException("User cannot publish");
    }
}
```

The positive branch handles the happy path, but the structure is awkward. The
core logic is indented behind a gatekeeper condition, and the exception handling
sits in an `else` block that feels like an afterthought.

Inverting the condition creates a **guard clause**:

```java
public void publish(Document doc, User user) {
    if (!user.hasRole(Role.EDITOR)) {
        throw new UnauthorizedException("User cannot publish");
    }

    doc.setState(State.PUBLISHED);
    doc.save();
}
```

Why doesn't the negation feel costly here? Because guard clauses handle
**exceptional situations**, and our mental model already frames them that way.
When we see `if (!condition) throw`, we read it as "bail out if something is
wrong"—the negation aligns with the exceptional nature of the exit. The
inversion tax is minimal because developer expectations match the code's intent.

Guard clauses handle the exceptional, flatten the structure, let the core logic
breathe. The method reads as a checklist: handle edge cases first, then do the
real work.

### Stacking Guards

The value of guard clauses multiplies with multiple validations. Suppose our
`publish` method also needs to check the document's state. Nested conditions
push the core logic deeper, creating the "arrowhead" anti-pattern:

```java
public void publish(Document doc, User user) {
    if (user.hasRole(Role.EDITOR)) {
        if (doc.isDraft()) {
            doc.setState(State.PUBLISHED);
            doc.save();
        } else {
            throw new IllegalStateException("Only drafts can be published");
        }
    } else {
        throw new UnauthorizedException("User cannot publish");
    }
}
```

Stacking guard clauses flattens the structure:

```java
public void publish(Document doc, User user) {
    if (!user.hasRole(Role.EDITOR)) {
        throw new UnauthorizedException("User cannot publish");
    }

    if (!doc.isDraft()) {
        throw new IllegalStateException("Only drafts can be published");
    }

    doc.setState(State.PUBLISHED);
    doc.save();
}
```

The method now reads as a checklist of preconditions. But both guards carry the
inversion tax.

## Designing for Positivity

Can we eliminate these negations? Sometimes—by designing our domain API to
support **positive assertions**. Instead of negating "has role," we provide a
method that describes the guarded state directly:

```java
public void publish(Document doc, User user) {
    if (user.lacksRole(Role.EDITOR)) {
        throw new UnauthorizedException("User cannot publish");
    }

    if (!doc.isDraft()) {
        throw new IllegalStateException("Only drafts can be published");
    }

    doc.setState(State.PUBLISHED);
    doc.save();
}
```

The first guard now reads naturally: _"If the user lacks the editor role..."_

But what about the document check? A document might be in draft, published, or
archived state. There's no single predicate that naturally describes "not
draft"—we'd have to invent something awkward like `isNotDraft()` or
`isUnpublishable()`. Sometimes the domain doesn't lend itself to positive
assertions, and that's okay. The guard clause still earns its keep through
linearity, and `!doc.isDraft()` is clear enough.

### The Trade-Off

Adding methods like `lacksRole` or `isMissing` increases API surface area. We
shouldn't double every interface just to avoid `!`. But for the right cases, the
readability gains outweigh the cost.

The more a class appears in guard clauses, the more a helper method pays off.
The less it appears, the less it's worth the API surface.

When to invest in positive-assertion methods:

- **High-traffic domain objects.** If a class appears in guard clauses across
  the codebase, a helper method pays dividends everywhere.
- **Critical control flows.** Authorization, validation, and error handling
  deserve maximum clarity.
- **Natural negative states.** When the domain has a clear concept for the
  absence (`lacksRole`, `isMissing`, `isUnknown`), model it explicitly.

When to skip them:

- **One-off checks.** A single `!` in an isolated method isn't worth a new API.
- **Awkward inverses.** As we saw with `!doc.isDraft()`, forcing a positive
  assertion can be worse than the negation it replaces.
- **Standard library types.** Don't wrap `!list.isEmpty()` in production code—
  the idiom is well-known and the tax is low. That said, fluent assertion
  libraries like Truth or Hamcrest do provide methods like `isNotEmpty()` for
  the subjects of standard library types. In test code, where assertions are
  plentiful and readability is paramount, these helpers shine. The context
  matters.

---

Negation is not the enemy. Misalignment is. When the code reads as it thinks,
the reader doesn't have to.
