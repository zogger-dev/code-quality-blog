---
title: "Hardcoding Verification: Eliminating Logic from Tests"
date: 2025-01-13
tags: ["testing"]
draft: true
---

Software engineering is about finding abstractions that reduce duplication. We are trained to avoid hardcoded values and seek generalized solutions. However, in the world of testing, this instinct can lead to a dangerous trap: **Reflected Logic**. When you use logic to calculate your expected result, you are no longer testing your code; you are merely testing your ability to write the same logic twice.

## The Reflected Logic Trap

Consider a service that generates a list of hostnames based on a specific naming scheme. A developer might write a test like this to avoid hardcoding:

```java
@Test
public void generatesCorrectHostnames() {
    var clusterId = "cluster-123";
    var result = service.getHostnames(clusterId, 3);

    // Reflected Logic: calculating the expectation using the same rules
    var expected = IntStream.range(0, 3)
        .mapToObj(i -> String.format("%s-node-%d", clusterId, i))
        .toList();

    assertThat(result).isEqualTo(expected);
}
```

This looks clean and "DRY," but it is a **Change Detector Test**. If the naming scheme in production is slightly wrong (e.g., it should have been `node-00` instead of `node-0`), this test will still pass because the expectation is derived from the same flawed logic. You have successfully verified that your test and your code share the same bug.

## The Resolution: Hardcode the Truth

To break the reflection, we must hardcode the expected outcome. A test should represent a concrete scenario with a concrete result.

```java
@Test
public void generatesCorrectHostnames() {
    var clusterId = "cluster-123";
    var result = service.getHostnames(clusterId, 3);

    // Hardcoded Truth: The requirement in its simplest form
    assertThat(result).containsExactly(
        "cluster-123-node-0",
        "cluster-123-node-1",
        "cluster-123-node-2"
    ).inOrder();
}
```

By hardcoding the expected values, we've made the test an independent authority. The reader doesn't need to parse an `IntStream` to know what the result should be; the answer is right there on the page.

## The Synthesis

Why is logic in tests an anti-pattern?

1.  **Tests Need Testing:** Every line of logic in your test suite is a line that you need to debug. If your test logic is complex, how do you know the test itself is correct?
2.  **Obfuscated Intent:** Logic hides the requirement. Hardcoded values highlight it. A reader should be able to look at your assertions and immediately understand the business rule being enforced.
3.  **Brittle Abstractions:** When you use helpers or calculations to derive expectations, your tests become coupled to those helpers. If you change the helper, you might inadvertently change the "truth" for dozens of unrelated tests.

Tests should be as simple as possible. Their job is to verify complex behavior with simple, unmistakable assertions. If your test needs logic to know what the result should be, your test has failed its primary mission of providing clarity.

## The Insight

Don't let the desire for generalized code lead to noisy, reflected tests. For your expectations, trust the power of the literal. Hardcode the truth; it makes your tests more honest, more readable, and far more likely to catch real bugs.
