---
title: "State vs. Interaction: The Case Against Brittle Mocks"
date: 2025-01-11
tags: ["testing"]
draft: true
---

Unit testing is often presented as a choice between two competing philosophies: **Interaction Verification** (Mocking) and **State Verification** (Faking). While both have their place, a common pitfall in large-scale systems is the over-reliance on mocks to verify internal implementation details. This leads to the dreaded **Change Detector Test**—a test that fails not because the behavior is broken, but because the implementation changed.

## The Mocking Trap

Consider a service that performs a background task and records its activity to a monitor. Using a mocking framework (like Mockito), a developer might write a test that verifies the interaction:

```java
@Test
public void recordsWriteActivity() {
    DiskMonitor mockMonitor = mock(DiskMonitor.class);
    var service = new ReplicationService(mockMonitor);

    service.performWrite();

    // Verify the interaction
    verify(mockMonitor, times(1)).recordActivity(Activity.WRITE);
}
```

This test is technically correct, but it is fragile. It has coupled the test to the exact method name and the exact number of times it was called. If you refactor the code to call `recordActivity` twice with smaller chunks, or rename the method to `logActivity`, the test fails. You haven't broken the requirement (recording activity), but you've tripped the "change detector."

## The Resolution: High-Fidelity Fakes

We can build more robust tests by focusing on **State Verification** using high-fidelity **Fakes**. A fake is a lightweight, local implementation of an interface that maintains internal state without the complexity of the production dependency.

```java
public class FakeDiskMonitor implements DiskMonitor {
    private final List<Activity> activities = new ArrayList<>();

    @Override
    public void recordActivity(Activity type) {
        activities.add(type);
    }

    public List<Activity> getRecordedActivities() {
        return List.copyOf(activities);
    }
}

@Test
public void recordsWriteActivityWithFake() {
    var fakeMonitor = new FakeDiskMonitor();
    var service = new ReplicationService(fakeMonitor);

    service.performWrite();

    // Verify the state
    assertThat(fakeMonitor.getRecordedActivities()).contains(Activity.WRITE);
}
```

In this version, the test cares only about the *outcome*: "Was a write activity recorded?" It no longer cares about how many times the internal method was called or the exact mechanics of the call.

## The Synthesis

Why is state verification superior for most scenarios?

1.  **Refactor-Friendliness:** Because the test is decoupled from the internal call graph, you can refactor your implementation's private methods and internal logic with complete confidence. If the state is correct, the test passes.
2.  **Exhaustive Context:** Fakes allow you to verify the *totality* of the side effects. A mock usually only verifies what you explicitly tell it to. A fake captures everything, allowing you to catch unexpected side effects that a mock would ignore.
3.  **Real-World Behavior:** High-fidelity fakes can implement complex logic (like dynamic tokens or unique IDs) that is difficult to script with a mock. This catches bugs that mocks often hide, such as race conditions or incorrect state transitions.

Interaction verification (mocking) should be reserved for side effects that cannot be easily observed in state—such as sending an email or making a remote network call. For everything else, the extra investment in building a high-quality fake pays off in a test suite that supports evolution rather than hindering it.

## The Insight

Don't test *how* your code works; test *what* it does. Favor fakes and state verification over mocks and interaction verification. It turns your test suite from a brittle change detector into a robust safety net for refactoring.
