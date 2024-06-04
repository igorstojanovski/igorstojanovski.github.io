---
layout: post
title: "Harnessing the Power of Ordered Test Execution in Junit5"
excerpt: "Explore JUnit5's test ordering feature: a blessing for integration tests in microservices. Take a glance at its strengths, use cases, and potential pitfalls."
date: 2023-09-18
---

JUnit 5 introduces a controversial feature: the capability to enforce the order in which test methods are executed. At first glance, this might seem contradictory to the essence of unit testing. Indeed, unit tests should be inherently independent, and their sequence should not influence their outcomes.

However, the ordered test method execution in Junit 5 isn't without merit. Its true potential becomes evident within the realm of integration tests. Let's demystify integration tests in the microservices context. The term 'integration test' carries a myriad of interpretations, but within the world of microservices, it commonly refers to a test that encompasses an entire microservice, isolating it from other external services.

Grouping multiple tests of this caliber allows for thorough testing of user journeys and workflows. Yet, setting up integration tests can sometimes become a resource heavy and lengthy process. Think about instances where stubbing external APIs or running and configuring emulators is required. In such scenarios, we might find ourselves recreating nearly identical setups for various integration tests. Junit5's test ordering feature, coupled with other JUnit tools, can alleviate this redundancy.

## An ordered test example

Consider the following test for a hypothetical e-commerce app:

<script src="https://gist.github.com/igorstojanovski/2b23a2f6a8204e9b94e5069a10d18b41.js"></script>

In this example, there are key features to spotlight:

1.  To activate method ordering, the **@TestMethodOrder** annotation is applied to the test class.
2.  The **@Order** annotations determine the execution sequence of individual tests.
3.  For integration tests, it's often beneficial to maintain states between test executions. The **@TestInstance(TestInstance.Lifecycle.PER\_CLASS)** annotation serves this purpose, instructing JUnit to create a singular instance of the test class. This promotes state sharing between tests, either via fields or by maintaining contexts across executions.

### The Added Value of Descriptive Names

Descriptive display names in tests enhance clarity. Even beginners or those without technical backgrounds can quickly understand the test flow's sequence and logic just by glancing at the code or test report. In combination with method ordering they could make your test reports read like nice little stories.

## Conclusion: Moderation is Key

While structured test ordering can boost efficiency, it's vital to tread with care. Over-dependence could present challenges. It is quite OK to have a few of these test classes that cover the most important happy flows. However, bare in mind that shared state is a tricky thing and it can easily result in flaky and hard to debug tests. Don't over do it.
