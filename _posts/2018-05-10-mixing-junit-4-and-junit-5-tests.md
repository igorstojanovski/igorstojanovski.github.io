---
layout: post
title: "Mixing JUnit 4 and JUnit 5 tests"
excerpt: "Thanks to the new architecture that JUnit 5 introduces it is possible to run both JUnit 4 and JUnit 5 tests in the same project."
date: 2018-05-10
tags: [JUnit 4, JUnit 5]
feature_image: __GHOST_URL__/content/images/wordpress/2018/05/JUnit5Architecture-1.png
---

Thanks to the new architecture that JUnit 5 introduces it is possible to run both JUnit 4 and JUnit 5 tests in the same project. JUnit 5 consists of three different sub-projects:

![JUnit5Architecture](https://i2.wp.com/igorski.co/wp-content/uploads/2018/05/JUnit5Architecture-1.png?resize=531%2C233)

* **Platform** – to support the implementation of various test engines.
* **Jupiter** – the engine to run **JUnit 5** tests.
* **Vintage** – the engine to run **JUnit 4** tests.

# Maven Setup

Taking this into consideration you will need to add three different dependencies to your project, **junit-platform-launcher**, **junit-jupiter-engine**, and **junit-vintage-engine**.

```java
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>1.2.0</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.2.0</version>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.2.0</version>
</dependency>
```

If you have some very custom requirements, you can always write your own custom engine and include that one as well.

# Running tests with IntelliJ IDEA

For IDEA it is enough for Vintage and Jupiter to be on the classpath.

```java
import org.junit.After;
import org.junit.Before;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class Junit5TestsMixedBefores {
    /**
     * This @Before is NOT going to be executed.
     */
    @Before
    public void before() {
        System.out.println("@Before");
    }

    @BeforeEach
    public void beforeEach() {
        System.out.println("@BeforeEach");
    }

    @Test
    public void simpleJunit5Test() {
        System.out.println("    Simple Junit 5 test.");
    }

    @AfterEach
    public void afterEach() {
        System.out.println("@AfterEach");
    }

    /**
     * This @After is NOT going to be executed.
     */
    @After
    public void after() {
        System.out.println("@After");
    }
}
```

When you run [this example](https://github.com/igorstojanovski/testing-examples/tree/master/src/test/java/org/igorski/junit5/junit4) the @Before and @After methods will not be executed. This is the output that you get:

```java
@BeforeEach
    Simple Junit 5 test.
@AfterEach
```

It is obvious why. In this test class, there is only a single JUnit 5 test. @Before and @After methods run only before and after JUnit 4 tests only. If you add a JUnit 4 test to this same class:

```java
@org.junit.Test
public void simpleJunit4Test() {
    System.out.println("    Simple Junit 4 test.");
}
```

the @Before and @After methods will be run. This is because each engine has its turn. Vintage engine discovers and runs the JUnit 4 tests. Jupiter runs its own discovery and execution phases where it discovers and executes JUnit 5 tests only. So not that you can only run JUnit 4 and JUnit 5 tests in the same project, you can also do that in the same test class.

[![JUnit 4 and JUnit 5 tests in IDEA](https://i2.wp.com/igorski.co/wp-content/uploads/2018/05/IdeaJunitEngines.png?resize=300%2C289)](https://i2.wp.com/igorski.co/wp-content/uploads/2018/05/IdeaJunitEngines.png)

IDEA does a good job at showing which tests were run by each of the engines.

# Running tests with Maven

Running a simple `mnv install` does not run any tests. Maven needs a bit more in order to run the tests:

```java
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.2.0</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

If you run the test now both JUnit 4 and JUnit 5 tests will be executed.

## Using JUnit 4 to run JUnit 5 tests

Today most of the IDEs and build tools support JUnit 5. However, if you have a build system that can run JUnit 4 tests but does not support the JUnit Platform directly, you can still use it to run JUnit 5 tests. That is thanks to the **JUnitPlatform** runner that was introduced for forward-compatibility.

When you annotate a JUnit 5 class with the **@RunWith(JUnitPlatform.class)** it will be run as a JUnit 4 test but with JUnit 5 features. Not all features are supported though. Read more about that in the [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/#running-tests-junit-platform-runner).

### Example

If you run `mnv install` without adding the junit-platform-surefire-provider dependency as in the previous example, it should not run any Junit 5 tests. But if you annotate those tests with **@RunWith(JUnitPlatform.class)** they will run. By adding the annotation they will be treated as JUnit 4 tests.

```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class SimpleJUnit5Test {

    @BeforeEach
    public void beforeEach() {
        System.out.println("@BeforeEach.");
    }

    @Test
    public void shouldExecuteJunit5Methods() {
        System.out.println("    Forward compatible test.");
    }

    @AfterEach
    public void afterEach() {
        System.out.println("@AfterEach.");
    }
}
```

This is a JUnit 5 test but thanks to the JUnitPlatform runner maven will run it together with all other JUnit 4 tests.

```java
Running org.igorski.SimpleJUnit5Test
@BeforeEach.
    Forward compatible test.
@AfterEach.
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.31 sec
```