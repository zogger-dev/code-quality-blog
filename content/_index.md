---
title: "Cognitive Load"
---

There's a mass extinction happening in tech blogs. Posts about design patterns, refactoring, and clean code are being quietly abandoned as authors pivot to the gravitational pull of AI. Why write about abstractions when you could be writing about agents?

Here's the uncomfortable truth: the AI revolution doesn't make code quality obsolete. It makes it essential.

## The 10x Ratio

For decades, we've operated under a rough heuristic: engineers read about ten times more code than they write. This ratio is why readability has always mattered more than writability. It's why we optimize for the reader, not the author. It's why "clever" code is an insult.

That 10x ratio is about to explode.

As AI systems write more of our code, the balance shifts dramatically. You might prompt an agent to scaffold a service, generate test fixtures, or implement a migration. In seconds, you're staring at hundreds of lines you didn't write but are now responsible for. The reading-to-writing ratio isn't 10x anymore—it's 50x, 100x, or higher.

If readability mattered when you were reading your own code and your teammates' code, it matters exponentially more when you're reading code written by a system that has no memory of your codebase's conventions, no intuition for your domain, and no taste.

## The Abstraction Gap

Large language models are remarkably good at generating code that works. They are significantly worse at generating code that *fits*.

Ask an AI to implement a feature, and it will produce something functional. But will it respect your existing abstractions? Will it recognize that you already have a `ReplicationStateMonitor` and use it, or will it invent a new `Gate` primitive that leaks implementation details? Will it follow your team's pattern for error handling, or will it introduce a third approach that future readers must reconcile?

This is the abstraction gap. AI systems can follow patterns when explicitly told, but they struggle to discover and honor the implicit architecture of a codebase. They don't see the forest—only the trees in the current context window.

This means humans aren't leaving the loop. We're moving up the stack. Instead of writing every line, we're becoming reviewers, orchestrators, and architects. We set the abstractions. We catch the anti-patterns. We refactor the AI's output until it reads like it belongs.

To do this well, you need a deep vocabulary for what makes code good. You need to recognize the "Validation Monolith" when it appears in generated test code. You need to spot the output parameter anti-pattern when an agent mutates a map instead of returning a result. You need to know when inheritance is being abused and composition would serve better.

## Why This Blog Exists

This blog is a field guide to that vocabulary.

Every post on Cognitive Load anchors to a single thesis: **the most important metric to optimize for in software development is cognitive load**. What slows engineers down is the dissonance between what they *think* and what the code *does*. When you can read and understand code as fast as you can think, you've achieved the ultimate readable codebase.

## The New Skill

In the age of AI, the engineer who can read, evaluate, and reshape generated code at speed has a superpower. The engineer who struggles to parse unfamiliar patterns, who can't articulate why an abstraction feels wrong, who accepts whatever the model produces—that engineer becomes a bottleneck.

Code quality isn't a luxury for teams with spare time. It's the skill that determines whether you can leverage AI effectively or drown in its output.

The reading-to-writing ratio has never been higher. The abstractions have never mattered more. And the need to understand what makes code a joy to read—instead of a puzzle to decode—has never been more urgent.

Welcome to Cognitive Load.

---

## Posts

### Readability

- [The Power of Positive Assertions](/posts/handle-the-positive-case/) `readability`
- [Beyond Booleans: Modeling Feature Flags as Data](/posts/self-documenting-states/) `readability`
- [Tame Your Nesting](/posts/tame-your-nesting/) `readability`
- [The DAMP Principle](/posts/the-damp-principle/) `readability`

### Abstraction

- [Domain-Specific Semantics](/posts/domain-specific-semantics/) `abstraction`
- [The Case Against Generics](/posts/the-case-against-generics/) `abstraction`
- [Composition over Inheritance](/posts/composition-over-inheritance/) `abstraction`
- [Avoiding Output Parameters](/posts/avoiding-output-parameters/) `abstraction`

### Optionals

- [The Null Object Pattern vs. Optional Arguments](/posts/null-object-pattern/) `optionals`
- [Local Scope Efficiency](/posts/local-scope-efficiency/) `optionals`

### Testing

- [State vs. Interaction: The Case Against Brittle Mocks](/posts/state-vs-interaction/) `testing`
- [High-Fidelity Fakes: Mirrors of Reality](/posts/high-fidelity-fakes/) `testing`
- [Hardcoding Verification: Eliminating Logic from Tests](/posts/hardcoding-verification/) `testing`

### System Integrity

- [Concurrency Contracts: Handling InterruptedException](/posts/concurrency-contracts/) `system-integrity`
- [Avoiding Tight Loops: Stop Spinning your CPU](/posts/avoiding-tight-loops/) `system-integrity`
- [Determinism: The Foundation of Distributed Integrity](/posts/determinism/) `system-integrity`
