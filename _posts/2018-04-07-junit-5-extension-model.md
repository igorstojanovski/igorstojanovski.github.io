---
layout: post
title: "JUnit 5 extension model"
excerpt: "JUnit 4 extension model is a bit scary."
date: 2018-04-07
tags: [JUnit 5, Testing, Java]
feature_image: __GHOST_URL__/content/images/wordpress/2018/04/JUnit5Extensions.png
---

**JUnit 4** extension model is a bit scary. You have **@Rule**, **@ClassRule**, and different **Runner** implementations. Very often you need to use all of them at once. On top of that you will probably end up with multiple rules that you will need to chain properly. Doable, but scary for beginners. Luckily, JUnit 5 offers a much better solution to [that conundrum](__GHOST_URL__/2018/03/18/run-stuff-before-and-after-each-test-in-junit4/) with a “single, coherent concept”: the **JUnit 5 Extension model**. Its not perfect, but in my opinion it is a vast improvement.

# Creating extensions

Tests can be extended at various points in the test execution lifecycle. For each of those points JUnit 5 provides appropriate interfaces. You can make your extension implement one or more of these interfaces:

* **BeforeAllCallback**
* **BeforeEachCallback**
* **BeforeTestExecutionCallback**
* **AfterTestExecutionCallback**
* **AfterEachCallback**
* **AfterAllCallback**

I’ve added a very basic code (to [GitHub](https://github.com/igorstojanovski/junit5-examples)) to demonstrate this. A few extensions and a couple of tests. The following code shows a basic extensions that implements all of the previously mentioned interfaces and simply prints out when each implemented method is called.

```java
public class CompleteExtension implements 
        BeforeAllCallback,BeforeEachCallback,BeforeTestExecutionCallback,
        AfterTestExecutionCallback, AfterEachCallback, AfterAllCallback {

    @Override
    public void afterAll(ExtensionContext extensionContext) {
        System.out.println("    After all the tests finish.");
    }

    @Override
    public void afterEach(ExtensionContext extensionContext) {
        System.out.println("    After each test.");
    }

    @Override
    public void afterTestExecution(ExtensionContext extensionContext) {
        System.out.println("    After test execution.");
    }

    @Override
    public void beforeAll(ExtensionContext extensionContext) {
        System.out.println("    Before all tests start");
    }

    @Override
    public void beforeEach(ExtensionContext extensionContext) {
        System.out.println("    Before each test.");
    }

    @Override
    public void beforeTestExecution(ExtensionContext extensionContext) {
        System.out.println("    Before test execution starts.");
    }
}
```

As you can see in each implemented method you have access to the **ExtensionContext**. This class encapsulates the context in which the current test or container is being executed. You can use it to access the test method, class, unique id etc. Unfortunately, here you don’t get an access to the **TestIdentifier** object like you do in a **TestExecutionLister** implementation. I am not quite sure why JUnit 5 has it this way.

# Registering extensions

[![JUnit Lifecycle callbacks](https://i1.wp.com/igorski.co/wp-content/uploads/2018/04/JUnit5Extensions.png?resize=281%2C671)](https://i1.wp.com/igorski.co/wp-content/uploads/2018/04/JUnit5Extensions.png)

Test Lifecycle Callbacks

You can register a JUnit 5 extension declaratively via the **@ExtendWith** annotation.

```java
@ExtendWith(CompleteExtension.class)
public class SimpleSingleExtensionTest {

    @BeforeAll
    public static void beforeAll() {
        System.out.println("@BeforeAll");
    }

    @BeforeEach
    public void beforeEach() {
        System.out.println("@BeforeEach");
    }

    @Test
    public void shouldRunTheTest() {
        System.out.println("From inside the test.");
    }

    @AfterEach
    public void afterEach() {
        System.out.println("@AfterEach");
    }

    @AfterAll
    public static void afterAll() {
        System.out.println("@AfterAll");
    }
}
```

After running this single test the following is printed out:

```java
    Before all tests start
@BeforeAll
    Before each test.
@BeforeEach
    Before test execution starts.
From inside the test.
    After test execution.
@AfterEach
    After each test.
@AfterAll
    After all the tests finish.
```

You can also register extensions programmatically, on field level, using the **@RegisterExtension** annotation. And, a third option is to use Java’s **ServiceLoader** mechanism.

## Registering multiple extensions

If you want, you can also register multiple extensions. You can use multiple @ExtendWith annotations.

```java
@ExtendWith(AroundTestExecutionOne.class)
@ExtendWith(AroundTestExecutionTwo.class)
```

Or, list more extensions in the same annotation.

```java
@ExtendWith({AroundTestExecutionOne.class, AroundTestExecutionTwo.class})
```

The order in which you declare the annotation will be the order in which they will execute.

```java
@ExtendWith({AroundTestExecutionOne.class, AroundTestExecutionTwo.class})
public class SimpleMultipleExtensionTest {

    @BeforeAll
    public static void beforeAll() {
        System.out.println("@BeforeAll");
    }

    @BeforeEach
    public void beforeEach() {
        System.out.println("@BeforeEach");
    }

    @Test
    public void shouldRunTheTest() {
        System.out.println("From inside the test.");
    }

    @AfterEach
    public void afterEach() {
        System.out.println("@AfterEach");
    }

    @AfterAll
    public static void afterAll() {
        System.out.println("@AfterAll");
    }
}
```

Consequently, the printout from the execution of this test will be:

```java
@BeforeAll
@BeforeEach
    Before Test execution - ONE
    Before Test execution - TWO
From inside the test.
    After Test execution - TWO
    After Test execution - ONE
@AfterEach
@AfterAll
```

## JUnit 5 Extension model and dynamic tests

Not long ago I wrote about [dynamic test in JUnit 5](__GHOST_URL__/2018/03/23/dynamic-tests-in-junit-5/). The **@BeforeEach** and **@AfterEach** are all called once per **@TestFactory**. That means that the **BeforeTestExecutionCallback** and **AfterTestExecutionCallback** callbacks of an extension will be called once per **@TestFactory**. And that is maybe one of the main downsides of the dynamic tests.

# Conclusion

The JUnit 5 extension model makes it much easier to extend the tests and the way they run. It is simple, easy to use, and straightforward. In my opinion, it is the single good enough reason to do the switch to JUnit 5. On the other hand, it is a huge breaking change. If your testing framework depends heavily on rules and runners, then you are into some remodeling. However, it is well worth the effort.