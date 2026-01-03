# The DAMP Principle

In software engineering, "DRY" (Don't Repeat Yourself) is a fundamental mantra. However, in the world of testing, blind adherence to DRY often leads to a dangerous side effect: test obfuscation. In an attempt to centralize verification and reduce boilerplate, we often inadvertently hide the very scenarios we are trying to test.

Consider a test suite designed to verify that search nodes correctly download indexes from a blobstore. To keep things "DRY," a developer might extract a generic utility method:

```java
@Then("I verify via logs that {int} search nodes performed {string}")
public void verifyLogs(int count, String action) {
    var nodes = getLatestProvisionedNodes(count);
    for (var node : nodes) {
        assertTrue(checkLogForAction(node, action, null, null));
    }
}
```

On the surface, this is efficient. But look at it from the perspective of a reader trying to understand a test failure. When they see `verifyLogs(10, "download")`, what are they actually looking at? What specific log line was expected? Which nodes were checked? The utility has flattened the test scenario into an abstract "action," forcing the reader to guess at the author's intent.

The fundamental property of a good test is that within the local context, a reader must be able to understand the scenario and the assertions well enough to conclude whether the test is correct. Generic "verify" methods destroy this property.

We can solve this by designing test APIs that reduce boilerplate without hiding important details. Imagine an ideal API that reads like a story but hides only the plumbing:

```java
@Then("I verify that the provisioning node downloaded the index")
public void verifyDownload() {
    expect(provisioningNode)
        .toHaveDownloadedIndex()
        .withDefinitionVersion(1);
}
```

In this version, the intent is unmistakable. We are asserting that a specific node performed a specific download with a specific version. The "plumbing"—the log parsing, the regex matching, the node retrieval—is hidden inside the `expect` DSL, but the **assertion details** remain in the local context.

In production code, we use abstractions to hide implementation details. In test code, we must ensure our abstractions highlight behavioral intent. Don't let your quest for DRY code turn your tests into a riddle. Favor descriptive and meaningful phrases that empower the reader to verify the verifier.
