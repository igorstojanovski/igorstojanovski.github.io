---
layout: post
title: "Polymorphic Deserialization with Jackson"
excerpt: "When serializing POJOs to JSON we lose a very valuable information about the
polymorphic nature of the Java object."
date: 2018-06-06
tags: [JSON]
feature_image: __GHOST_URL__/content/images/wordpress/2018/06/PolymorphicDeserialization.png
---

When serializing POJOs to JSON we lose a very valuable information about the polymorphic nature of the Java object. When we deserialize the same object from JSON we have to know the exact object type and we cannot deserialize to a supertype. Lucky for us, [Jackson](https://github.com/FasterXML/jackson) offers the option of polymorphic deserialization.

# The problem when deserializing JSON to POJO

Let us see what would happen if we try to deserialize to a supertype using Jackson. We have two event classes **TestFinished** and **TestStarted** that extend a basic **Event** class.

```java
@Getter
@Setter
@NoArgsConstructor
public class Event {
    protected long timestamp;
}


@Getter
@Setter
@NoArgsConstructor
public class TestFinished extends Event {
    private String testId;
    private Status status;

    public TestFinished(long timeStamp, String testId, Status status) {
        this.timestamp = timeStamp;
        this.testId = testId;
        this.status = status;
    }
}

@Getter
@Setter
@NoArgsConstructor
public class TestStarted extends Event {
    private String testId;

    public TestStarted(long timestamp, String testId) {
        this.testId = testId;
        this.timestamp = timestamp;
    }
}
```

For the example to work we need to add the Jackson dependency.

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.0.5</version>
</dependency>
```

Serializing and deserializing POJOs and JSONs is easy using the ObjectMapper class.

```java
ObjectMapper mapper = new ObjectMapper();
List<String> jsons = new ArrayList<>();

Event testStarted = new TestStarted(System.currentTimeMillis(), "testId000003");
Event testFinished = new TestFinished(System.currentTimeMillis(), "000001", Status.FAILED);

jsons.add(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(testStarted));
jsons.add(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(testFinished));

List<Event> events = new ArrayList<>();

for(String json : jsons) {
    events.add(mapper.readValue(json, Event.class));
}
```

At first glance, this code should work. We serialize a list of Event objects into JSONs and then we want to deserialize that same list of events and place it into an Event list. But it does not, it throws an exception.

```java
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "testId" (class org.igorski.testing.events.Event), not marked as ignorable (0 known properties: ])
 at [Source: java.io.StringReader@351d0846; line: 2, column: 15] (through reference chain: org.igorski.testing.events.Event["testId"])

  at com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException.from(UnrecognizedPropertyException.java:79)
  at com.fasterxml.jackson.databind.DeserializationContext.reportUnknownProperty(DeserializationContext.java:568)
  at com.fasterxml.jackson.databind.deser.std.StdDeserializer.handleUnknownProperty(StdDeserializer.java:650)
```

It immediately becomes obvious why. In the provided JSON there are fields that are not part of the **Event** class and Jackson has no idea what to do with those. The JSON holds no information about the type of the original class it was serialized from. When we get to the moment of the deserialization Jackson no longer knows anything about the **TestStarted** or **TestFinished** classes.

# Using polymorphic deserialization with Jackson

[![Polymorphic Deserialization](https://i1.wp.com/igorski.co/wp-content/uploads/2018/06/PolymorphicDeserialization.png?resize=521%2C291)](https://i1.wp.com/igorski.co/wp-content/uploads/2018/06/PolymorphicDeserialization.png)

We can keep the information about the type in the serialized JSONs

All we need to fix this is to add that type information in the JSON we are serializing. Jackson allows us to list all possible subtypes with the **@JsonSubTypes** annotation. The  **@JsonTypeInfo**, in addition, is used to fine-tune how that information will be encoded in the JSON.

```java
@JsonTypeInfo(use= JsonTypeInfo.Id.CLASS, include= JsonTypeInfo.As.PROPERTY, property="@class")
@JsonSubTypes({
        @JsonSubTypes.Type(value=TestFinished.class, name = "TestFinished"),
        @JsonSubTypes.Type(value=TestStarted.class, name = "TestStarted"),
})
@Getter
@Setter
@NoArgsConstructor
public class Event {
    protected long timestamp;
}
```

After adding this meta info to the superclass, the former code works. This is because in the serialized JSON now there is info about the type of the object.

```java
{
  "@class" : "org.igorski.testing.events.TestFinished",
  "timestamp" : 1528228848663,
  "testId" : "000001",
  "status" : "FAILED"
}
```

There is also an option to add the info about the subtypes globally but I am not going to show it here. How to use that option and all of its pros and cons you can find in the [official documentation](https://github.com/FasterXML/jackson-docs/wiki/JacksonPolymorphicDeserialization).