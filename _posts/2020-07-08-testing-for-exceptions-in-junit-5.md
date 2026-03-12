---
layout: post
title: "Testing for exceptions in JUnit 5"
excerpt: "Testing for exceptions in JUnit has never been a very straightforward thing."
date: 2020-07-08
tags: [Testing, JUnit 5]
feature_image: __GHOST_URL__/content/images/2020/07/daniel-von-appen-VPWzm4ZBMRw-unsplash--1--1.jpg
---

Testing for exceptions in JUnit has never been a very straightforward thing. I've often seen it misused or done wrong, especially in JUnit 4. I won't go directly into if and how Junit 5 improves this. Let's first remind ourselves how lousy JUnit 4 was when testing for exceptions.

## Testing for exceptions in JUnit 4

There are two ways to test for exceptions in JUnit 4, either using the **Test** annotation itself or using the **ExpectedException** rule.

The Test annotation can get an additional ***expected*** parameter whose value is the class of the exception we are expecting.

```java
    @Test(expected = DummyException.class)
    public void shouldTestForException() throws DummyException {
        ComponentUnderTest component = new ComponentUnderTest();
        component.exceptionThrowingMethod(-1);
    }
```

This has never been very straightforward or clear or obvious. I've almost always used the second approach with the **ExpectedAnnotation** rule.

```java
    @Rule
    public ExpectedException expectedException = ExpectedException.none();

    @Test
    public void shouldUseRuleToTestForException() throws DummyException {
        expectedException.expect(DummyException.class);
        ComponentUnderTest component = new ComponentUnderTest();
        component.exceptionThrowingMethod(-1);
    }
```

This is much better and more flexible. It also offers the possibility to test for the exception cause or message.

## Testing for exceptions in JUnit 5

In Junit 5, however, rules don't exist any more. So, they had to come up with a different way to do this. And they opted for using the assertion API.

```java
    @Test
    void shouldThrowException() {
        ComponentUnderTest componentUnderTest = new ComponentUnderTest();
        DummyException dummyException =
                assertThrows(DummyException.class, () -> componentUnderTest.exceptionThrowingMethod(-1));
        // Use different asserts on the exception object.
    }
```

This approach gives you all the flexibility of using the rule in Junit 4 but without the inconvenience of having to create a rule object.

## Testing for exceptions with AssertJ

The newly introduced assertion in JUnit 5 reminds me quite a lot of the AssertJ fluent API for asserting exceptions.

```java
    @Test
    public void testException() {
        ComponentUnderTest componentUnderTest = new ComponentUnderTest();
        assertThatThrownBy(() -> componentUnderTest.exceptionThrowingMethod(-1))
                .isInstanceOf(DummyException.class)
                .hasMessageContaining("dummy")
                .hasNoCause();
    }
```

It can do all the things the Junit 5 assertion can do and probably even more. As a bonus, it can be used both with Junit 4 and JUnit 5.

## Conclusion

JUnit 4 had two confusing ways for testing for exceptions. Junit 5 has one, much better way which is a real improvement. However, If you are already have AssertJ in your project, use it to test for exceptions as well. It is at least as better as the Junit 5 API and you will have a uniform approach to assertions.

## Links

* Original cover photo by [Daniel von Appen](https://unsplash.com/@daniel_von_appen?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/windows?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
* Source code of the examples on [GitHub](https://github.com/igorstojanovski/junit5-examples)