---
layout: post
title: "Samebug Extension for JUnit5"
excerpt: "I was contacted recently by the guys over at Samebug [https://samebug.io]."
date: 2018-07-30
tags: [JUnit 5, samebug]
feature_image: __GHOST_URL__/content/images/wordpress/2018/07/samebug_logo.jpg
---

I was contacted recently by the guys over at [Samebug](https://samebug.io). We had a few chats about their product and we came to the subject of JUnit5 extensions. At that point, I volunteered to make a Samebug extension, one that will harness Samebug’s power and enrich the error output of tests.

But let’s take it from the top.

[![samebug-logo](https://i1.wp.com/igorski.co/wp-content/uploads/2018/07/samebug_logo.jpg?resize=400%2C400 "JUnit5 Samebug Extension")](https://i1.wp.com/igorski.co/wp-content/uploads/2018/07/samebug_logo.jpg)

Samebug logo

# What is Samebug?

Samebug is a debugging assistant for developers. It analyzes JVM stack traces to provide deep insights, rich context and technical expertise on them. It constantly monitors the web for JVM stack traces and analyzes them meaning it continually improves getting better at identifying bugs and their characteristics. For each category of errors, it either has a solution entered by a user or it links to sites like StackOverflow. By doing that it saves you time. Instead of copying and pasting errors and stack traces, and sifting through search results in Google, you just go to Samebug.

# The Samebug Extension for JUnit5

The extension is very simple. It collects the stack trace of an error in your test and sends it to Samebug. Samebug does its magic and sends back a single result page id. That id is then displayed in the log of the extension as a URL. When you open the link you can see details about your error, similar errors, and possible solutions.

It saves you even more time in that way. You just click a link.

You can find the Samebug extension source code and more details on how to use it on [GitHub](https://github.com/igorstojanovski/SamebugExtension).

## Implementation details

The extension is very simple. It implements a single callback, **AfterEachCallback**. When implementing the **afterEach()** method of the callback, we get the ExtensionContext as a parameter. The **getExecutionException()** method of that class is all we need. According to the documentation, it gets the exception that was thrown during execution of the test or container associated with the ExtensionContext.

```java
public void afterEach(ExtensionContext extensionContext) {
        Optional<Throwable> throwable = extensionContext.getExecutionException();

        if (throwable.isPresent()) {
            CrashResponse crashResponse = samebugProxy.getSamebugRequest(throwable.get());

            if (crashResponse != null && crashResponse.getData() != null) {
                LOGGER.info("Please visit https://nightly.samebug.com/searches/" + crashResponse.getData().getId()
                        + " for more info.");
            }
        }
    }
```

After that, it all comes down to sending the stack trace content to Samebug. For that purpose, it uses the JBoss [RESTEasy](__GHOST_URL__/java/resteasy-client-json/) client.

## Simple Example

You will need to add the dependency first:

```java
<dependency>
    <groupId>co.igorski</groupId>
    <artifactId>samebug-junit-extension</artifactId>
    <version>1.0.0</version>
</dependency>
```

Then add the SamebugExtension.class to your test class as you would add any other extension.

```java
@ExtendWith(SamebugExtension.class)
public class SampleTest {

    @Test
    public void shouldCompareNumbers() {
        assertEquals("2", "3");
    }
}
```

This is the most basic example to demonstrate how the extension works. Running the test will throw an exception telling you that the two values you are trying to compare are not equal. Together with the error, it will also log the Samebug search link:

“Please visit <https://samebug.io/searches/10116787> for more info.”

The id is unique to each search. Even for the same type of error, you will get a different search id. Once you open that page though it will link you to the page of the error pattern. There you can check for possible solutions or find related errors. In the case of this example, the search page will link to the following pattern: <https://samebug.io/error-patterns/490603>

This is, however, a very simple example. It will mostly benefit the absolute beginners. Samebug shines when solutions to errors are not as straightforward as in this example.

# What is next?

The extension in its first version is far from perfect and it has some quirks that need to be ironed out. A 1.0.1 version should come pretty soon. How things progress from there depends on the user base and if Samebug introduces new features that can be used by the extension. One feature I would like to see is for the API to fetch me the solution to the problem so I can display it in the console. Similar to what Mockito has when you mess up the mocking. It displays a short and clear direction on how to fix your problem.

# Disclaimer

I am in no way affiliated with Samebug, the project and the company. They are just some great people I met online with a very cool product I wanted to play around with.