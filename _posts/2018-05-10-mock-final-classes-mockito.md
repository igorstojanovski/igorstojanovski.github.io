---
layout: post
title: "Mock final classes with Mockito"
excerpt: "‼️Read the latest 15/10/23 update at the bottom page.

One of the standard Mockito related questions I’ve come across is online is “Can we mock final classes with Mockito?”."
date: 2018-05-10
tags: [mock, Mockito, PowerMock, Testing]
feature_image: https://images.unsplash.com/photo-1558276052-76eca36e8750?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3wxMTc3M3wwfDF8c2VhcmNofDZ8fGFyY2hpdGVjdHVyYWwlMjBtb2RlbHxlbnwwfHx8fDE2OTczMjA4MDN8MA&ixlib=rb-4.0.3&q=80&w=2000
---

‼️

Read the latest 15/10/23 update at the bottom page.

One of the standard Mockito related questions I’ve come across is online is “Can we mock final classes with Mockito?”. The answer to this question since Mockito 2 was introduced is “Yes, we can.”.

## Example code

Since it is a JUnit 5 test is uses the official [MockitoExtension](__GHOST_URL__/java/junit/mockito-with-junit5/). It is supposed to test the following class:

Unfortunately, the PinProvider class is final:

This makes the test fail with the following exception:

```java
org.mockito.exceptions.base.MockitoException: 
Cannot mock/spy class org.igorski.finalexample.PinProvider
Mockito cannot mock/spy because :
 - final class
```

## Configuration to mock final classes

Mockito 2 brings a solution to this problem. The Mockito team still considers this feature as “incubating” though. It is optional and it if you want to use it you will need to turn it on explicitly. In order to do that create the file *src/test/resources/mockito-extensions/org.mockito.plugins.MockMaker* with the following content:

```java
mock-maker-inline
```

After this, the test passes and no exceptions or warnings are thrown. Mission accomplished.

## PowerMock problems

Very often projects use both PowerMock and Mockito. But using **mock-maker-inline** does not play well with **PowerMock**. PowerMock has its own **org.mockito.plugins.MockMaker.** Turning on the Mockito mock-maker-inline feature adds a second MockMaker on the classpath, and that’s a problem.

I experienced these problems first hand. I followed [PowerMock’s suggestion](https://github.com/powermock/powermock/wiki/Mockito?ref=igorski.co#mockito-mock-maker-inline) how to fix but it didn’t work for me. Probably the safest way would be to choose a single approach. Either mock final classes with Mockito or use PowerMock, but not both.

## Update: 15/10/23

Starting with Mockito 5.0.0, mocking final classes has been simplified; no special configuration is required, as the feature is [enabled by default](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#39). To take advantage of this, simply upgrade to Mockito version 5.0.0 or higher.