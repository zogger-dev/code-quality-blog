# Review Insights 2025 Q4: Blog Series

A collection of engineering insights and best practices derived from code reviews in Q4 2025.

## Architectural Patterns

- [The Gate Pattern vs Event Consumers](./posts/gate-pattern-vs-event-consumers.md)
  *Why "Push" models fail for concurrency control and how "Gates" simplify state management.*
- [Leaking Abstractions via Dependencies](./posts/leaking-abstractions-via-dependencies.md)
  *Why injecting generic primitives (like Gates) breaks encapsulation and how domain-specific wrappers fix it.*

## Coding Best Practices

- [The Null Object Pattern vs Optional Arguments](./posts/null-object-pattern-vs-optional.md)
  *Why passing `Optional<T>` as a parameter is a code smell and how the Null Object Pattern cleans it up.*
- [Handling InterruptedException Correctly](./posts/handling-interrupted-exception.md)
  *Why swallowing interrupts creates zombie threads and how to properly respect the cancellation signal.*
- [Avoiding Tight Loops](./posts/avoiding-tight-loops.md)
  *Why busy-waiting kills performance and how to use proper synchronization primitives.*
- [The Inverted Conditional Anti-pattern](./posts/inverted-conditional-anti-pattern.md)
  *Why starting with a negation increases cognitive load and how to write scanable `if-else` blocks.*

## Testing Patterns

- [Fakes vs Mocks](./posts/fakes-vs-mocks.md)
  *Why interaction-based mocking is brittle and how state-based Fakes lead to robust, refactor-friendly tests.*
- [Test Method Naming](./posts/test-method-naming.md)
  *Why `testSomething` is obsolete and how behavior-oriented naming acts as live documentation.*
