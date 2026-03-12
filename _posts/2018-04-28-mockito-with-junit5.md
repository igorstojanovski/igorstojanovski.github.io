---
layout: post
title: "Using Mockito with JUnit 5"
excerpt: "Since version2.16.3 [https://github.com/mockito/mockito/pull/1221]Mockito has
official support for Junit5."
date: 2018-04-28
tags: [JUnit 5, Testing, Java]
feature_image: __GHOST_URL__/content/images/wordpress/2018/04/Mockit-Junit-5.png
---

Since version [2.16.3](https://github.com/mockito/mockito/pull/1221) Mockito has official support for Junit5. Using Mockito with JUnit now is even easier than before. Previously I kept forgetting what rule I was supposed to use for injecting the mocks, and how I set the strictness again? Plus the extra field in each test class!

[![Mockito with JUnit 5](https://i1.wp.com/igorski.co/wp-content/uploads/2018/04/Mockit-Junit-5.png?resize=300%2C300)](https://i1.wp.com/igorski.co/wp-content/uploads/2018/04/Mockit-Junit-5.png)

# Mockito with JUnit 5 example

The newly introduced extension is not part of the mockito-core package. In order to use it, you will have to add a separate dependency:

```java
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>${mockito.version}</version>
    <scope>test</scope>
</dependency>
```

The extension will initialize all the fields annotated with **@Mock**. If you want to set the strictness of the stubs, use the **MockitoSetting** annotations. In the [sample code](https://github.com/igorstojanovski/testing-examples/blob/master/src/test/java/junit5/mockito/ChatEngineTest.java), I set the strictness to STRTICT\_STUBS although that is already the default.

```java
@ExtendWith(MockitoExtension.class)
@MockitoSettings(strictness = Strictness.STRICT_STUBS)
class ChatEngineTest {

    @Mock
    private MessageSender messageSender;

    @Test
    public void shouldUseMessageSenderToSendMessage() {
        ChatEngine chatEngine = new ChatEngine(messageSender);
        String messageToSend = "Hello!";

        chatEngine.say(messageToSend);
        when(messageSender.send(messageToSend)).thenReturn(OK);

        assertThat(messageSender.send(messageToSend)).isEqualTo(OK);
    }

}
```

## How does the extension work?

I previously spoke about the [JUnit 5 extension model](__GHOST_URL__/java/junit/junit-5-extension-model/), and MockitoExtension is a great example of putting that into practice. It uses three extension points:

* **TestInstancePostProcessor**: to set the test instance.
* **BeforeEachCallback**: to create the Mockito session and initialize all the mocks.
* **AfterEachCallback**: to remove the previously created session.

Nice and easy.