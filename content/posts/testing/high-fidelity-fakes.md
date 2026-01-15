---
title: "High-Fidelity Fakes: Mirrors of Reality"
date: 2025-01-12
categories: ["Testing"]
draft: true
---

Unit testing often relies on test doubles to isolate code from external dependencies. While simple mocks and stubs are easy to create, they often suffer from the **Stub Paradox**: they are so predictable that they hide the very bugs you are trying to find. To build a truly resilient system, we must instead invest in **High-Fidelity Fakes**.

## The Stub Paradox

Consider a service that retrieves temporary credentials from a cloud provider. A standard mock-based test might look like this:

```java
@Test
public void usesCachedToken() {
    AzureApi mockApi = mock(AzureApi.class);
    when(mockApi.getToken()).thenReturn(new Token("fixed-token", fixedExpiry));

    var service = new CredentialManager(mockApi);
    service.getToken(); // first call
    service.getToken(); // second call

    verify(mockApi, times(1)).getToken();
}
```

This test is technically correct, but it is dangerously shallow. Because the mock always returns the *same* static value, it cannot catch bugs related to token expiration or dynamic state changes. It doesn't test the logic of your code; it only tests that your code calls a specific method once.

## The Resolution: High-Fidelity Fakes

We can reach a much deeper level of verification by building high-fidelity fakes that implement the *logic* of the production dependency, not just its interface.

```java
public class FakeAzureApi implements AzureApi {
    private final Duration tokenLifespan;
    private int callCount = 0;

    public FakeAzureApi(Duration tokenLifespan) {
        this.tokenLifespan = tokenLifespan;
    }

    @Override
    public Token getToken() {
        callCount++;
        // Return a dynamic token with a unique ID and a calculated expiry
        return new Token("token-" + UUID.randomUUID(), Instant.now().plus(tokenLifespan));
    }
}
```

By using a fake that returns unique IDs and dynamic timestamps, we turn our test into a true simulation. We can now verify not just *how many times* a method was called, but how our code reacts to the *content* of the response.

## The Synthesis

Why are high-fidelity fakes worth the extra investment?

1.  **Catching Edge Cases:** Static stubs rarely fail. High-fidelity fakes, by their nature, introduce variation. They catch race conditions, off-by-one errors in time calculations, and incorrect state transitions that a static mock would ignore.
2.  **Living Documentation:** A well-written fake acts as an executable specification of the dependency. It tells future developers exactly how the external system is expected to behave.
3.  **Refactor Resilience:** Because high-fidelity fakes are implementation-agnostic, your tests remain valid even if you completely rewrite the internal logic of your service.

The quality of your test suite is capped by the quality of your doubles. If your doubles are shallow and static, your tests will be brittle and blind. By investing in high-fidelity fakes that mirror real-system behavior, you turn your test suite into a powerful sandbox for exploring and validating your code’s intent.

## The Insight

Don't settle for shallow stubs. Invest in high-fidelity fakes that implement real-world logic, unique identifiers, and dynamic state. The extra effort pays off in a test suite that actually catches bugs before they reach production.
