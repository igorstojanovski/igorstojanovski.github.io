---
layout: post
title: "My 100 Days of Code Challenge Project"
excerpt: "Last September I decided to start a 100 days of code challenge for reasons
explained here."
date: 2019-01-26
tags: [100 days of challenge, central committee, blog]
feature_image: __GHOST_URL__/content/images/wordpress/2019/01/StackTrace.png
---

Last September I decided to start a **100 days of code challenge** for reasons explained here. Although I had to prematurely end the challenge at **day 76** I am very happy with the result and what I managed to do in that time. And out of those 76 days **Central Committee** came out as a result.

## Central Committee

The initial idea for my **100 Days of Code Challenge** was to create a test reporting web application for **JUnit** tests. I am happy to say that with minor changes I managed to create exactly what I had planned.

As someone that was born in a country of the Europe’s socialist block, the name came naturally. Central Committee was the common designation of a standing administrative body of communist parties. And everyone snitched to the central committee. It knew everything that was going on, especially about what was working and what was failing in the country.

### Features

**Central Committee** is envisioned as a test logging framework. These are the current features it has:

* The client, or as I like to call it a **snitcher**, sends events from the tests to the server.
* The server collects those test events, builds a test run model based on that and saves all necessary data in a database.
* User then can:
  + View all the runs.
  + Also view all the live runs.
  + View all tests in the run and their status (passed, failed, running).
  + See the stack trace in case a test failed.
  + View the history of a single test. The test path (full class path plus method name) is used as a unique identifier for a test.
  + Define collections of test.
* The only implemented **snitcher** is for **JUnit 5** and its based on the **TestExecutionListener** listener.

There are some features I plan to implement in the very near future:

* **JUnit 5.4-RC1** introduces a new **TestWatcher** extension API which I need to implement either as an addition to the listener. That will allow for a more granular control over which test classes are being logged.
* Central Committee needs to save all the logs and needs to be able to replay a test model from the event logs.
* While the test run is still active the test run model is being built in memory. That saves me a lot of trips to the database. However it needs to be stored in a distributed way. Currently it’s a simple HashMap.
* A user should be able to define alarms for a collection of tests. The alarm should, let’s say, notify the user in some special way if any of the tests fail.

### Architecture

![](https://i0.wp.com/igorski.co/wp-content/uploads/2019/01/CentralCommittee-_v2.png?ssl=1)

The web application is a single monolith with a rather simple architecture.

For each test event the client sends a request to a **RESTful** service. The services then create internal events which they post to a single topic. Using [polymorphic deserialization](__GHOST_URL__/java/polymorphic-deserialization-jackson/) the events are deserialized from the queue to their real type and then handled by their matching handlers.

The handlers build the internal data model of the run. The handler for the event that marks the end of the run that saves everything to the database. The **Vaadin** based GUI then reads from that database to

#### Technology stack

Central Committee stack:

* Vaadin 11
* Apache Kafka (+Zookeeper)
* Spring Boot
* PostgreSQL
* Docker for running the DB, Zookeeper and Kafka

**Kafka** was probably a complete overkill. I could have used something more lightweight or not use messaging at all. But I’ve always liked **Kafka** and since I never had the chance to work with it at work, I wanted to try it out.

Probably I complicated a lot of the architecture without any need. Although, let’s not forget that as part of the **100 Days Of Code Challenge** the main point was learning and trying out new things. That is exactly what I did here.

### Current state of the app

Unfortunately the application has very little test coverage. The architecture also needs simplifications and improvements. It doesn’t have a CI setup. So in an ideal World, I would completely trash this one and start all over.

I am not going to do that though. I will continue working on this one and improving it. Working greenfield is easy. Working on brownfield crappy, messed up projects is what I find challenging.

You can find the source code on [GitHub](https://github.com/igorstojanovski/Central-Committee).

## Screenshots

* ![](https://i0.wp.com/igorski.co/wp-content/uploads/2019/01/CollectionOfTests.png?fit=1024%2C598&ssl=1)
* ![](https://i0.wp.com/igorski.co/wp-content/uploads/2019/01/CreateProjects.png?fit=1024%2C502&ssl=1)
* ![](https://i1.wp.com/igorski.co/wp-content/uploads/2019/01/ListOfRuns.png?fit=1024%2C439&ssl=1)
* ![](https://i1.wp.com/igorski.co/wp-content/uploads/2019/01/Login.png?ssl=1)
* ![](https://i2.wp.com/igorski.co/wp-content/uploads/2019/01/ManageUsers.png?fit=1024%2C443&ssl=1)
* ![](https://i1.wp.com/igorski.co/wp-content/uploads/2019/01/StackTrace.png?fit=1024%2C679&ssl=1)