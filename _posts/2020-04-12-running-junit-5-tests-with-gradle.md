---
layout: post
title: "Running JUnit 5 tests with Gradle"
excerpt: "With Gradle, very little comes out of the box and a lot of the filtering and configuration you need to do on your own."
date: 2020-04-12
tags: [Testing, JUnit 5, Gradle, Java]
feature_image: __GHOST_URL__/content/images/2020/04/aj-robbie-BuQ1RZckYW4-unsplash.jpg
---

I have been using Maven my whole professional life. Fortunately or not, I had to switch to Gradle recently. To call the way things work in Gradle different, would be an understatement.

### Maven based expectations

Мaven differentiates between two different types of tests, unit and integration tests. This differentiation is based on two plugins, surefire and failsafe. They are intended for running unit and integration tests respectively. The way Maven makes a distinction between the different types of tests is based on the name of the test classes. Out of the box, test classes whose fully qualified names match the patterns *\*\*/\*IT.java*, *\*\*/\*IT.java*, *\*\*/\*ITCase.java* will be treated as integration tests by the failsafe plugin.

### How Gradle does it

With Gradle, very little comes out of the box and a lot of the filtering and configuration you need to do on your own. That can be confusing at first especially as a beginner and even more so if you are accustomed to Maven.

Everything regarding running tests in Gradle revolves around the Test task. Before running it you need to configure it by telling the task where it can find the compiled test classes as well as giving it the execution classpath, which should include the classes under test as well as the test library that you’re using. Luckily, when you use the java plugin it will create a dedicated test source set for unit tests as well as a test task of type Test that runs those unit tests.

### Simple example

With all that in mind, this is the minimal Gradle configuration needed to run JUnit 5 tests:

```java
plugins {
    id 'java'
}

group 'co.igorski'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testImplementation('org.junit.jupiter:junit-jupiter:5.6.2')
}

test {
    useJUnitPlatform()    
}
```

We run the tests using the test task:

```java
gradle test
```

This will run every test and print out the result if the run was successful or not. However, it won't print the result for each test separately. In order to do that we need to add a logging configuration:

```java
test {
    useJUnitPlatform()
    testLogging {
        events "passed", "skipped", "failed"
    }
}
```

This will print a log line for each test that was run:

> co.igorski.SimpleCalculatorTest > shouldDivideTwoNumbers() PASSED  
> co.igorski.SimpleCalculatorTest > shouldMultiplyTwoNumbers() PASSED  
> co.igorski.SimpleCalculatorTest > shouldSubstractTwoNumbers() PASSED  
> co.igorski.SimpleCalculatorTest > shouldAddTwoNumbers() PASSED

We can run a specific class, specific testcase method, or even use wildcards:

```java
gradle test --tests co.igorski.SimpleCalculatorTest
gradle test --tests co.igorski.SimpleCalculatorTest.shouldMultiplyTwoNumbers
gradle test --tests co.igorski.SimpleCalculator*.should*
```

### Running integration tests

Before we can run integration tests, we need to answer what integration tests are. Maven has an opinionated approach. Every test class that ends with *\*\*/\*IT.java*, *\*\*/\*IT.java* or *\*\*/\*ITCase.java* is an integration test class. However, Gradle is not very opinionated and doesn't care about integration tests. To Gradle, all tests are equal and it is up to the end-user to filter the tests as she pleases. That filtering can be done on various basis.

### Filtering test cases based on class name

If we want to filter test cases based on test class name, as in Maven, we can use the include option:

```java
test {
    useJUnitPlatform {
        exclude '**/*IT.class'
    }
    testLogging {
        events "passed", "skipped", "failed"
    }
}
```

Actually, it is much more logical if we have different tasks for this:

```java
test {
    useJUnitPlatform {
        exclude '**/*IT.class'
    }
    testLogging {
        events "passed", "skipped", "failed"
    }
}

task integrationTest(type: Test) {
    useJUnitPlatform {
        include '**/*IT.class'
    }
    testLogging {
        events "passed", "skipped", "failed"
    }
}
```

There is a better way to filter tests

Starting from JUnit 5 you can use the Tag annotation on tests and use one or more different tags on class leve.

```java
@Tag("IT")
@Tag("slow")
class SimpleCalculatorIT {
// tests left out becaus of brevity    
}
```

You can now do the filtering based on the tags and define different tasks based on that with potentially intersecting sets of tests.

```java
test {
    useJUnitPlatform {
        excludeTags 'IT', "slow"
    }
    testLogging {
        events "passed", "skipped", "failed"
    }
}

task integrationTest(type: Test) {
    useJUnitPlatform {
        includeTags 'IT'
    }
    testLogging {
        events "passed", "skipped", "failed"
    }
}

task slowTests(type: Test) {
    useJUnitPlatform {
        includeTags 'slow'
    }
    testLogging {
        events "passed", "skipped", "failed"
    }
}
```

### Conclusion

Gradle is hands down powerful. But the fact that it comes with so little out of the box can make it overwhelming at first. Plus, the build scripts can grow and if you overdo it you can spend as much time tending to them as you do to your code.

For example, there is a lot of repetition when the different tasks are defined in the last code example. And this code can be refactored:

```java
task integrationTest(type: LoggedTests) {
    useJUnitPlatform {
        includeTags 'IT'
    }
}

task slowTests(type: LoggedTests) {
    useJUnitPlatform {
        includeTags 'slow'
    }
}

class LoggedTests extends Test {
    LoggedTests() {
        testLogging {
            events "passed", "skipped", "failed"
        }
    }
}
```

And this code works although I have no idea if this is the best possible way to achieve the wanted result of removing duplication. But, that is Gradle for you.

### Sources

* Cover photo by AJ Robbie on Unsplash
* <https://docs.gradle.org/current/dsl/org.gradle.api.tasks.testing.AbstractTestTask.html>
* <https://maven.apache.org/surefire/maven-failsafe-plugin/examples/junit-platform.html>
* <https://stackoverflow.com/questions/1399240/how-do-i-get-my-maven-integration-tests-to-run>
* <https://stackoverflow.com/questions/29948381/how-to-extend-the-behavior-of-a-gradle-task-for-a-new-task-type>
* <https://docs.gradle.org/current/userguide/java_testing.html>
* <https://github.com/igorstojanovski/jenkins-pipeline-as-code/blob/master/build.gradle>