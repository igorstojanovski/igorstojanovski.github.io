---
layout: post
title: "Run code before and after each test in Junit4"
excerpt: "When I say “before and after” I refer to running the code always, for all tests,
independent whether @Before and @After are present in a test or not."
date: 2018-03-18
tags: [Testing, JUnit 4]
feature_image: __GHOST_URL__/content/images/wordpress/2018/03/874086.png
---

When I say “before and after” I refer to running the code always, for all tests, independent whether **@Before** and **@After** are present in a test or not. I don’t have in mind simply using **@Before**s and **@After**s. That would mean that you need to add the same before and after in each test. Lets say you need to log the start and end times of each test. That sort of stuff.

If you thought about writing a class that implements **@Before** and **@After** methods and then making every test extend that class, then the answer is **no**. Implementing your own **Rule** and then adding it to each test class is a much better approach. JUnit4 even provides *ExternalResource* for that reason. But that still means that you will have one extra variable in each of your classes representing the rule.

But there is one more approach to solving this problem.

# Write your own Runner

If you want to change the way Junit4 runs your tests you need to create your own **Runner**. But that would be just too much work! Luckily, the good people of JUnit gave us a ready implementation open to tweaking, the **BlockJUnit4ClassRunner** class. Now the question is, what exactly needs to be tweaked?!

## The TestRule approach

I wont explain how **TestRule** works, but feel free to find out in the [documentation](https://github.com/junit-team/junit4/wiki/Rules).

In the **BlockJUnit4ClassRunner** there is the following method:

```java
/**
 * @param target the test case instance
 * @return a list of TestRules that should be applied when executing this
 * test
 */
protected List getTestRules(Object target) {
    List result = getTestClass().getAnnotatedMethodValues(target,
            Rule.class, TestRule.class);

    result.addAll(getTestClass().getAnnotatedFieldValues(target,
            Rule.class, TestRule.class));

    return result;
}
```

And this looks like a good place for to add that custom **TestRule**.

```java
public class MyCustomTestRule implements TestRule {
    public Statement apply(Statement base, Description description) {
        return statement(base, description);
    }

    private Statement statement(final Statement base, final Description description) {
        return new Statement() {
            private long startTime;
            @Override
            public void evaluate() throws Throwable {
                startTime = System.currentTimeMillis();
                System.out.println("START test " + description.getDisplayName() + '.');
                try {
                    base.evaluate();
                } finally {
                    long endTime = System.currentTimeMillis() - startTime;
                    System.out.println("END test " + description.getDisplayName() + " after " + endTime + " millies.");
                }
            }
        };
    }
}
```

All you need now is you own **Runner**. At least that is easy. Notice the custom implementation of the **getTestRules** method I mentioned earlier:

```java
public class UpgradedRunner extends BlockJUnit4ClassRunner {
    public UpgradedRunner(Class<?> klass) throws InitializationError {
        super(klass);
    }

    protected List getTestRules(Object target) {
        List testRules = super.getTestRules(target);
        testRules.add(new NotVeryUsefulRule());
        return testRules;
    }
}
```

This works. It will print when each test starts, and after each test ends, and how long it took to execute. No **@Before** and **@After** needed, anywhere.

## The Befores approach

The approach is the same, only this time, two different methods need to be implemented: **withBefores** and **withAfters**.

```java
protected Statement withBefores(FrameworkMethod method, Object target, Statement statement) {
    Statement next = super.withBefores(method, target, statement);
    return getBeforeStatement(next, target);
}

protected Statement withAfters(FrameworkMethod method, Object target, Statement statement) {
    Statement next = super.withAfters(method, target, statement);
    return getAfterStatement(next, target);
}

private Statement getAfterStatement(final Statement next, final Object target) {
    return new Statement() {
        public void evaluate() throws Throwable {
            try {
                next.evaluate();
            } finally {
                System.out.println("..... and done!");
            }
        }
    };
}

private Statement getBeforeStatement(final Statement next, final Object target) {
    return new Statement() {
        public void evaluate() throws Throwable {
            System.out.println("Here we go ... ");
            next.evaluate();
        }
    };
}
```

Don’t forget to call the super methods otherwise the **@Before** and **@After** methods in each of the tests wont get run.

## The all-together

The two approaches can be combined in one single runner. If we have a simple test:

```java
@Before
public void setup() {
    System.out.println("Before in test.");
}

@Test
public void thisTestShouldBeOK() {

}

@After
public void teardown() {
    System.out.println("After in test.");
}
```

.. then the final output is going to be:

`START test thisTestShouldBeOK(org.programirame.BasicTest).  
Here we go ...  
Before in test.  
After in test.  
..... and done!  
END test thisTestShouldBeOK(org.programirame.BasicTest) after 3 millies.`

# Conclusion

As always in Java, there are more than one ways to achieve the desired outcome. What you choose depends on your needs and wants. It is important to notice that all previously said is obsolete in **JUnit5**. There are no more Rules or Runners. Luckily, in **JUnit5** this is much easier and straightforward to accomplish.