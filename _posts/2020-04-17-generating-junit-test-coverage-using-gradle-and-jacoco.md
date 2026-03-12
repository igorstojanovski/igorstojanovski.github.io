---
layout: post
title: "JUnit test coverage reports using Gradle and JaCoCo"
excerpt: "JaCoCo is one the most used tools for generating coverage reports for JUnit tests."
date: 2020-04-17
tags: [Testing, JUnit 5, Gradle]
feature_image: __GHOST_URL__/content/images/2020/04/JaCoCoReport-1.PNG
---

JaCoCo is one the most used tools for generating coverage reports for JUnit tests. One of the reasons for it being so used is it's seamless integration with tools like Jenkins, SonarQube, Maven and Gradle.

Let us take a look how easy it easy to generate a JaCoCo report using Gradle.

### Use the Gradle JaCoCo plugin

One of the benefits of using JaCoCo is that the plugin is a first class citizen of Gradle and that makes things very easy.

In order to generate the reports all you need to do is add the jacoco plugin:

```java
plugins {
    id 'java'
    id 'jacoco'
}
```

When you run the test task it will create the reports but in the JaCoCo binary format: *test.exec*. When the java plugin is applied, like in this case, it will create the jacocoTestReport task. This task can be used to generate the reports in a non-binary format, html or xml. However, it does not depend on the testTask so you will have to run both.

```java
gradle test jacocoTestReport
```

This will, by default, generate html reports in the *build/reports/jacoco/* folder. The reports look something similar to this with the possibility to drill down and check coverage by package, class, line and branch:

![](__GHOST_URL__/content/images/2020/04/JaCoCoReport.PNG)

In case you want to generate the reports in a different location, XML or CSV format, you can do that by configuring the *jacocoTestReport* plugin:

```java
jacocoTestReport {
    reports {
        xml.enabled true
        csv.enabled true
        html.destination file("${buildDir}/reports/html/jacoco")
        xml.destination file("${buildDir}/reports/xml/jacoco")
        csv.destination file("${buildDir}/reports/csv/jacoco")
    }
}
```

This however, will generate reports only based on the *test.exec* file. If the test task is named differently, the *.exec* file will also be named differently and this task won't generate any reports. I have a [different post](__GHOST_URL__/join-jacoco-reports/) that explains how to generate reports for all test tasks and join them into a single overall report.

That is the basics of it. I've done the same thing with Maven as well and it is more difficult and confusing. Point goes to Gradle for this one.

### Sources

* <https://docs.gradle.org/current/userguide/jacoco_plugin.html>
* <https://www.eclemma.org/jacoco/>