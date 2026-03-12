---
layout: post
title: "Dynamic tests in JUnit 5"
excerpt: "Although not a novelty concept, dynamic tests in the current form were introduced in JUnit 5."
date: 2020-06-17
tags: [JUnit 5, JUnit 4, Testing]
feature_image: __GHOST_URL__/content/images/2020/07/ryoji-iwata-IBaVuZsJJTo-unsplash.jpg
---

Although not a novelty concept, dynamic tests in the current form were introduced in **JUnit 5**. The main goal of dynamic tests is to allow users to generate tests dynamically, at runtime. This opens up possibilities for implementing some interesting data-driven test scenarios.

Unlike a standard **@Test** method, a dynamic test method should be annotated with **@TestFactory** and itself is not a test but a test generator. As such it needs to return **Stream**, **Collection**, **Iterable**, or **Iterator** of a **DynamicNode** implementation.

## **Dynamic tests example**

The [example](https://github.com/igorstojanovski/dynamic-tests-junit5/blob/master/src/test/java/org/igorski/DynamicTestExample.java) generates tests based on a csv file:

```java
test_name,firstaddend,secondaddend,total
first,1,2,3
second,5,6,11
third,10,12,22
```

It is a simple addition test. As the title line of the csv shows, the first value is going to be used as the name of the test and then follow the first and the second addend and the sum value.

```java
@TestFactory
public Stream<DynamicTest> testFactory() {
    InputStream inputFS = getClass().getClassLoader().getResourceAsStream("testData.csv");
    BufferedReader br = new BufferedReader(new InputStreamReader(inputFS));

    return br.lines().skip(1).map(mapCsvLineToDynamicTest());
}

private Function<String, DynamicTest> mapCsvLineToDynamicTest() {
    return s -> {
        final String[] values = s.split(",");
        return dynamicTest(values[0], () -> assertAddition(values));
    };
}

private void assertAddition(String[] values) {
    assertThat(Integer.parseInt(values[1]) + Integer.parseInt(values[2]))
            .isEqualTo(Integer.parseInt(values[3]));
}
```

After running the test factory method the IntelliJ IDEA generates the following output:

![](__GHOST_URL__/content/images/2020/06/DynamicTestOutputIDEA.png)

## **Parameterised tests in JUnit 4 comparison**

Parameterised tests in JUnit 4 are very similar to the dynamic tests. We can achieve the same effect from the previous example with the following code:

```java
@RunWith(Parameterized.class)
public class ParametrizedTestExample {

    @Parameterized.Parameters(name = "{0}")
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {
                { "first", 1, 2, 3 }, { "second", 5, 6, 11 }, { "third", 10, 12, 22 }
        });
    }

    private String description;
    private int first;
    private int second;
    private int total;

    public ParametrizedTestExample(String description, int first, int second, int total) {
        this.description = description;
        this.first = first;
        this.second = second;
        this.total = total;
    }

    @Test
    public void shouldHaveCorrectTotal() {
        assertThat(first + second).isEqualTo(total);
    }
}
```

But this time the IntelliJ output looks differently:

![](__GHOST_URL__/content/images/2020/06/ParameterisedTestJUnit4.png)

With parameterised tests it is as if we have executed three separate test classes. If we had a **@Before** method JUnit 4 would have executed it for each parameter set. Opposite to this when we use a **@TestFactory** it looks as though we have a single test. This is confirmed by the fact that if we have a **@BeforeEach** or **@AfterEach** methods they will be executed once per a **@TestFactory** method.

Another difference is that when you use the Parametrized runner, each test in that class will be run once for each parameter set. If you have 5 parameters sets and 3 tests that will be a total of 15 test executions. Contrary to that, you can have as many **@TestFactory** methods in the same class as you wish. **JUnit 5** will execute each one exactly once.