# The Code Quality Guide

A living, opinionated platform for sharing synthesized engineering wisdom, focusing on **Readability** and **Powerful Abstractions**.

## Development

```bash
# Start development server
hugo server -D

# Build static site
hugo
```

## Module: Readability & Flow
- [The Power of Positive Assertions](./content/posts/handle-the-positive-case.md)
- [Beyond Booleans: Modeling Feature Flags as Data](./content/posts/self-documenting-states.md)
- [Tame Your Nesting](./content/posts/tame-your-nesting.md)
- [The DAMP Principle](./content/posts/the-damp-principle.md)

## Module: The Art of Abstraction
- [Domain-Specific Semantics](./content/posts/domain-specific-semantics.md)
- [The Case Against Generics](./content/posts/the-case-against-generics.md)
- [Composition over Inheritance](./content/posts/composition-over-inheritance.md)
- [Avoiding Output Parameters](./content/posts/avoiding-output-parameters.md)

## Module: The Case Against Optionals
- [The Null Object Pattern vs. Optional Arguments](./content/posts/null-object-pattern.md)
- [Local Scope Efficiency](./content/posts/local-scope-efficiency.md)

## Module: Robust Testing
- [State vs. Interaction: The Case Against Brittle Mocks](./content/posts/state-vs-interaction.md)
- [High-Fidelity Fakes: Mirrors of Reality](./content/posts/high-fidelity-fakes.md)
- [Hardcoding Verification: Eliminating Logic from Tests](./content/posts/hardcoding-verification.md)

## Module: System Integrity
- [Concurrency Contracts: Handling InterruptedException](./content/posts/concurrency-contracts.md)
- [Avoiding Tight Loops: Stop Spinning your CPU](./content/posts/avoiding-tight-loops.md)
- [Determinism: The Foundation of Distributed Integrity](./content/posts/determinism.md)
