# Track: Review Insights 2025 Q4

**Goal:** Convert high-value code review comments from Q4 2025 into educational blog posts.

**Source Material:**
- PR #4830 (Refactor PeriodicDiskMonitor)
- PR #4649 (Pause FCIS downloads)
- PR #4433 (Improve FCIS error reporting)

## Phase 1: Architectural Patterns

- [x] **Blog Post: The "Gate" Pattern vs Event Consumers** [commit: 2476b6d]
    - Context: PR #4649 (PeriodicDiskMonitor refactor discussion).
    - Concept: Using a Hysteresis Gate to signal state changes vs pushing events to consumers.
    - Key Insight: Gates isolate concurrency concerns and state management from the observer.
- [x] **Blog Post: Leaking Abstractions via Dependencies** [commit: 02fae87]
    - Context: PR #4830 (Gate dependency injection).
    - Concept: Injecting a generic `Gate` forces callers to understand its configuration.
    - Key Insight: Components should own their "gates" (implementation detail) or depend on a domain-specific wrapper (e.g., `ReplicationStateMonitor`).

## Phase 2: Coding Best Practices

- [x] **Blog Post: The Null Object Pattern vs Optional Arguments** [commit: 7e22dfa]
    - Context: PR #4649 (`Optional<DiskMonitor>`), PR #4433, PR #4830.
    - Concept: Passing `Optional` as an argument is a code smell.
    - Key Insight: Use a `NoOp` implementation (Null Object Pattern) instead of `Optional`.
- [x] **Blog Post: Handling InterruptedException Correctly** [commit: cd00bc9]
    - Context: PR #4830.
    - Concept: Swallowing interrupts or failing to restore the flag.
    - Key Insight: Always call `Thread.currentThread().interrupt()` if you can't propagate the exception.
- [x] **Blog Post: Avoiding Tight Loops** [commit: ec00bc9]
    - Context: PR #4830.
    - Concept: Spinning CPU while waiting for a condition.
    - Key Insight: Use `await/signal` patterns or blocking queues instead of `while(!condition)`.
- [x] **Blog Post: Inverted Conditional Anti-pattern** [commit: f619175]
    - Context: PR #4830.
    - Concept: `if (!condition) { ... } else { ... }`.
    - Key Insight: Handle the positive case first for better readability.

## Phase 3: Testing Patterns

- [x] **Blog Post: Fakes vs Mocks** [commit: 1e6388b]
    - Context: PR #4649.
    - Concept: Mocking internal interactions (brittle) vs Faking state (robust).
    - Key Insight: Test behavior against state, not implementation details.
- [ ] **Blog Post: Test Method Naming**
    - Context: PR #4649.
    - Concept: `testPauseDownloads` vs `pauseDownloads_whenDiskFull_blocks`.
    - Key Insight: Names should describe the scenario and expected outcome.
