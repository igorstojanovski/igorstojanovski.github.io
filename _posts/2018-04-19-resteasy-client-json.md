---
layout: post
title: "RESTEasy client to send JSON requests"
excerpt: "This is a very short example of how you can use RESTEasy to send JSON requests.
For one of my pet projects, I needed to send requests to my JSON based API."
date: 2018-04-19
tags: [Java, JBoss, RESTful]
feature_image: __GHOST_URL__/content/images/wordpress/2018/04/resteasy_logo_600x.png
---

This is a very short example of how you can use **RESTEasy** to send **JSON** requests. For one of my pet projects, I needed to send requests to my JSON based API. I usually use **Apache**‘s **HttpClient** for simple requests. But, this time I wanted to try some of **JAX-RS** implementations. As for everything else in Java, the selection of implementations is pretty wide with Apache CXF, Restlet, Jersey, and RESTEasy being one of them. RESTEasy is [JBoss’ implementation](http://resteasy.jboss.org) of both JAX-RS 2.0 and JAX-RS 2.1.

[![RESTEasy logo](https://i1.wp.com/igorski.co/wp-content/uploads/2018/04/resteasy_logo_600x.png?resize=600%2C426)](https://i1.wp.com/igorski.co/wp-content/uploads/2018/04/resteasy_logo_600x.png)

# Setting up RESTEasy

Most of the examples you can find are of how to send XML data. For sending JSON you will need the following dependencies:

```java
<properties>
    <resteasy.version>3.5.0.Final</resteasy.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-jackson2-provider</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-client</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
</dependencies>
```

The Jackson 2 provider will be automatically loaded by RESTEasy. There is also a [way](https://stackoverflow.com/a/48658107/2900795) to load providers manually but in this case, you don’t need to do that.

The model for my example is a simple User class. This POJO will be converted to JSON when sending requests, and the other way around when receiving responses.

```java
@Getter
@Setter
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
    private String surname;
    private String username;
}
```

The annotations are from the [Lombok](https://projectlombok.org) and are not related to RESTEasy. They automatically add boilerplate code like like getters, setters, and no-arg constructors among other things.

RESTEasy has a client proxy framework that lets you use JAX-RS annotations to define all the HTTP requests you want to send. All you need to do is to write a Java interface and use the correct JAX-RS annotations on methods and the interface.

```java
@Path("/user")
public interface UserClient {

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    User getUserById(@QueryParam("id") String id);

    @POST
    @Consumes(MediaType.APPLICATION_JSON)
    User createUser(User user);
}
```

# Sending the request

You first need to build the client and create the proxy based on an interface like the one we just created. All you need to do then is invoke the methods of the proxy. The invoked method gets translated to an HTTP request based on how you annotated the method and get sent to the server. In hindsight, RESTEasy uses Apache’s HTTP client but it does all the manual labor for you.

```java
ResteasyClient client = new ResteasyClientBuilder().build();
ResteasyWebTarget webTarget = client.target(UriBuilder.fromPath(ENDPOINT));
UserClient userClient = webTarget.proxy(UserClient.class);

User user = userClient.getUserById("1");

User nextUser = new User();
nextUser.setName("Marisa");
nextUser.setName("Coulter");
nextUser.setUsername("MrsCoulter");

userClient.createUser(user);
```