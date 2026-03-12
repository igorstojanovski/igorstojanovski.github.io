---
layout: post
title: "Continuous Integration using Bitbucket and Gradle"
excerpt: "Example of how to build a CI pipeline for a Spring Boot application using Bitbucket, Gradle, Snyk, SonarCloud, Postman and Heroku."
date: 2021-03-28
tags: [CI/CD]
feature_image: __GHOST_URL__/content/images/2021/03/sirisvisual-TYhNHKWnf2c-unsplash-1.jpg
---

In this article I will share my experience on creating a Continuous Integration pipeline for a Gradle project using Bitbucket, Snyk, SonarCloud, Postman and Heroku. In my previous post I wrote about what CI/CD is and why it is a very important practice. Please [read that article](__GHOST_URL__/what-is-ci-cd/) as well if you have any doubts that you need a continuous integration pipeline.

## Pipeline structure

There is no particular reason why I chose Bitbucket. I've used **Travis CI**, **GitHub** and **Jenkins** quite extensively before. This time I wanted to try something new.

I use a very simple Spring Boot application as a sample project. The goal the pipeline is to cover a necessary set of checks to consider the application ready to be released or deployed. This is the set of steps the pipeline contains and the tools used for each step:

* Build - [Gradle](https://gradle.org)
* Unit and integration tests - JUnit 5
* Test coverage -  [JaCoCo](https://www.eclemma.org/jacoco/)
* Security vulnerabilities in dependencies - [Snyk](https://snyk.io)
* Static and security code analysis - [SonarCloud](__GHOST_URL__/p/923cfb14-c535-45f4-883f-e89d2776a93e/https./sonarcloud.io)
* Deployment for the purpose of testing - [Heroku](https://www.heroku.com)
* End-to-end tests - [Postman](https://www.postman.com)

![](__GHOST_URL__/content/images/2021/03/CiCdBitBuckeet.png)

This diagram depicts the different steps in the pipeline. Most of them will run in parallel.

As part of the article I will just explain what I did to make these tools run in the pipeline and how I configured them. I won't go into details about what those tools are, how they work or how to use them in all possible ways.

## Bitbucket pipelines

Bitbucket expects to find the pipeline configuration in the root of the project in the **bitbucket-pipelines.yml** file. Each pipeline must contain at least one **step** and each step must contain at least one **script** section. One pipeline can have a maximum of 100 steps and those steps can be run in parallel if needed.

```java
image: openjdk:8

pipelines:
    default:
        - step:
              script:
                  - ./gradlew build
```

This is the minimal pipeline to build a JDK8 Gradle project.

The file needs to have one default but it can have multiple other pipeline definitions. That allows you to define different pipelines per code branch or pull request. Which is quite a nice thing to have. In Jenkins you need multiple files. Yuck!

The neat thing about Bitbucket pipelines is that each step is run into a separate **Docker** container. You can use one of the predefined images or you can define your own. The only restriction is that it has to be pushed to Docker Hub. You can even use a different container for each step if you need to. Or, you can define one global image that will be used for all steps containers. This makes the Bitbucket pipelines very flexible.

If you have ever used any other CI servers that support pipeline-as-code, like **Travis CI**, **Jenkins** or **Gitlab**, this whole approach won't be anything new for you.

### Dependency Cache

Bitbucket offers the possibility to cache the build dependencies between pipeline steps. That is a great benefit because with Gradle a lot of time is spent on downloading the dependencies.

### The build container

In order to be able to run all the tools I planed to use in the pipeline I needed a custom container that had a few more additional things on top of JDK 11:

* Latest version of Gradle
* Snyk CLI
* Postman CLI
* Heroku CLI

I could have broken up this in at least two different containers, one with Snyk CLI, the other with Postman CLI and then use them in separate steps in the pipeline. However, for the purpose of simplicity, I decided to install everything in a single image and use that as the global build container.

## Gradle for build automation

In case you are not from the Java universe, Gradle is a build automation tool for Java. These days you basically have two options for build automation, Apache Maven or Gradle. The main upside of Gradle is at the same time it's biggest downside. It is very flexible, too flexible at times. That allows you to mess things up big time but it also gives you a lot of power if you know what you are doing.

> I would recommend to rely on Gradle tasks as much as possible. In an ideal case, each pipeline script section would be a call to a Gradle task. If any setup of the environment is needed that should be handled in the Docker image. The main benefit of this is that **you are not tightly coupled to a particular CI server and can easily migrate** to another or even use multiple at the same time.

In my example I don't fully rely on this approach. I use a mix, mostly because I wanted to try different approaches. I use Gradle mainly to build the project, run the tests and generate test coverage reports.

## SonarCloud for code quality scans

**SonarCloud** is the cloud version of **SonarQube**. It is free for public projects an it offers integration with Bitbucket. All you need to do is add and configure the SonarQube plugin. This is well explained in the [SonarQube documentation](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-gradle/). I don't think there is any need for me to reiterate. And I already have a separate post about setting up [SonarQube with the sonar scanner](__GHOST_URL__/sonarqube-scans-using-jenkins-declarative-pipelines/) as an alternative to using the Gradle plugin.

With the plugin configured the only thing left is to add the **SONAR\_TOKEN** variable in BitBucket. This and other variables you can add in the pipeline edit screen.

![](__GHOST_URL__/content/images/2021/03/BitBucketVariables.png)

You can generate the SonarCloud token from the [security section](https://sonarcloud.io/account/security/) of your SonarCloud account. Before that, you need to login to SonarCloud using your Bitbucket account but that is straightforward.

SonarCloud will do static code analysis analysis and based on that it will calculate and track your technical debt. In addition to that, it can also upload test coverage reports. All the findings will be visible on the SonarCloud portal.

### Generating JaCoCo test coverage

SonarCloud works best with JaCoCo coverage reports. To be honest, I don't even know that there is any other option for Java these days. I've explained how to setup SonarQube in a [previous post](__GHOST_URL__/generating-junit-test-coverage-using-gradle-and-jacoco/). In short, you need to add the JaCoCo plugin to the plugins section of your build.gradle file and turn on XML coverage report generation.

```java
plugins {
    id 'jacoco'
}
test {
    finalizedBy jacocoTestReport
}

jacocoTestReport {
    dependsOn test
}

jacocoTestReport {
    reports {
        xml.enabled true
        csv.enabled false
        html.destination file("${buildDir}/jacocoHtml") // not really necessary
    }
}
```

With this in place, running  **gradle build** will build the project, run the tests and generate the coverage reports. This report will be then used by SonarCloud to display coverage data.

## Snyk for dependency vulnerability checks

[Snyk](https://snyk.io) is a solution that seamlessly and proactively finds and fixes vulnerabilities and license violations in open source dependencies and Docker images. And these folks are a beast of a company. They just raised 300$ million at a $4.7 billion valuation. Insane!! For us as regular users that is a good thing. It shows that they are good at what they do and they let open source projects do scans for free.

Snyk has a Gradle plugin but the plugin requires the Snyk CLI installed. Now, since I have to have the CLI I decided not to overburden my build script with another plugin and use the cli directly.

The CLI offers two main commands to perform a scan:

* **test**: to test local project for vulnerabilities
* **monitor**: to snapshot and continuously monitor your project.

For the pipeline I thought that **test** is more suitable and that is what I used in the script step.

When the test command finds vulnerabilities, it will generate a report in the console, fail the build, and with that fail the whole pipeline. Now, you won't be able to always fix all the vulnerabilities. Some of them maybe you'd like to ignore. You can do that using the ignore command with the ID of the issue you want to ignore:

```java
snyk ignore --id=SNYK-JAVA-COMH2DATABASE-31685
```

Make sure you run the commit in the root of the project. That will add the *.snyk* file that you will need to commit as part of your code. Next time the pipeline runs with *.snyk* present, Snyk will ignore all the vulnerabilities listed in that file.

## Postman for end to end tests

The last checks in this pipeline implementation are the end-to-end tests implemented using **Postman**. You will need to use the Postman GUI to create the tests. Once they are created you need to export them and add the exported .json file to your project.

You can then run the tests from the command line using [newman](https://www.npmjs.com/package/newman), the cli companion for Postman:

```java
newman run mycollection.json
```

The tests do need to run against a running application. That means, you will need to deploy the application somewhere first and run the tests later. I tried two different approaches for this. First, I just ran the Spring Boot application using Gradle. But, in that case, the Postman tests needed to run against localhost. The second option, which I opted for in the end, was to do a real deployment of the application to Heroku.

### Heroku for deployments

Heroku is a platform as a service (PaaS) that enables developers to build, run, and operate applications entirely in the cloud. You will need to create an account and then you will get a very basic environment where you can do deployments for free. There are at least three different options for how to deploy a Spring Boot application to Heroku, using git, using a built jar or using Docker containers if you have your application containerized. I opted for the second option, using the executable ("uber", "fat") jar and the Heroku CLI.

The Heroku [documentation](https://devcenter.heroku.com/articles/deploying-executable-jar-files#using-the-heroku-java-cli-plugin) explains how to use the CLI pretty well. After following the instructions and creating my Heroku application, all that I needed to run in the pipeline was the deploy command:

```java
heroku deploy:jar build/libs/spring-boot-example-0.0.1-SNAPSHOT.jar --app $HEROKU-APP-NAME
```

For this to work you need to add the HEROKU-APP-NAME variable and value to Bitbucket.

But for this command to work, I needed to prepare a few things first. I added two additional files to my project. The first one is the Procfile where the command that executes the file is defined:

```java
web: java -Dserver.port=$PORT -jar build/libs/spring-boot-example-0.0.1-SNAPSHOT.jar
```

Contents of Procfile

This also takes care of the port problem. The second file is *system.properties* and this one is needed to set the required Java version. I am using JDK 11 and Heroku still runs with JDK 8 as default.

```java
java.runtime.version=11
```

Contents of system.properties

The last thing I needed to do is set the **HEROKU\_API\_KEY** variable in Bitbucket with the key provided by Heroku as the value. Without doing this the Heroku CLI cannot properly authenticate.

## Putting everything together

All the steps I just described that are needed to run these tools are completely pipeline (and with that CI server) agnostic. You can run all these commands locally on your machine. The only thing Bitbucket specific is how the pipeline steps are structured. This is how the file looked like in the end, with all the steps defined:

```java
image: igorskico/spring-boot-example-ci:1.0.3

clone:
    depth: full

definitions:
    caches:
        sonar: ~/.sonar/cache  # Caching SonarCloud artifacts will speed up your build
    steps:
        - step: &build-test-sonarcloud
              name: Build, test and analyze on SonarCloud
              caches:
                  - gradle
                  - sonar
              script:
                  - gradle build sonarqube
              artifacts:
                  - build/libs/**

pipelines:
    default:
        - step:
              name: Build
              caches:
                  - gradle
              script:
                  - chmod +x gradlew
                  - gradle build -x check
        - parallel:
              - step:
                    name: Tests
                    caches:
                        - gradle
                    script:
                        - gradle test
              - step:
                    name: Postman
                    caches:
                        - gradle
                    script:
                        - gradle build -x check
                        - heroku deploy:jar build/libs/spring-boot-example-0.0.1-SNAPSHOT.jar --app $HEROKU-APP-NAME
                        - newman run src/test/resources/postman.json
              - step:
                    name: Security Scan
                    script:
                        - pipe: atlassian/git-secrets-scan:0.4.3
        - parallel:
              - step:
                    name: Snyk
                    script:
                        - wget https://github.com/snyk/snyk/releases/download/v1.466.0/snyk-linux
                        - chmod +x snyk-linux
                        - chmod +x gradlew
                        - gradle wrapper --gradle-version 6.8.3 --distribution-type all
                        - ./snyk-linux auth $SNYK_TOKEN
                        - ./snyk-linux test -d --all-projects --json-file-output=snyk-test-output.json
                        - pipe: snyk/snyk-scan:0.4.6
                          variables:
                              SNYK_TOKEN: $SNYK_TOKEN
                              SNYK_TEST_JSON_INPUT: "snyk-test-output.json"
              - step: *build-test-sonarcloud
```

The pipeline will get trigger each time you merge code in your repository. When that happens, the Bibucket GUI gives a nice visual representation of which steps ran, were red or green, ran in parallel in sequentially and so on.

![](__GHOST_URL__/content/images/2021/03/image_2021-03-28_231445.png)

With that, the pipeline is complete and functional. A logical next step would be continuous deployment. And since I already have a deployment step to Heroku, I'd just need to to create a new Heroku application and add an additional step in the pipeline.

And as easy as that, you can have CI/CD for your project.

## Final thoughts

In the course of writing this post and building my pipeline I ran into a couple of issues that I consider downsides to using Bitbucket.

### Build minutes

The biggest problem I ran into was the lack of build minutes. You get 50 free build minutes per month. That is way too little, especially when you are at the beginning and you are setting up your pipeline and you need to try a lot of different things. I ended up spending my minutes in just two days.

Since I needed to finish my article I decided to pay. And it seemed cheap, 3$ per developer per month. What Atlassian fails to mention is that the minimal amount of developers that you can subscribe for is 5. Even you are a single person you still need to pay $15 per month. I was willing to forgive them those $15 dollars and I decided to pay after all. Again, not so easy. Two of my cards got declined, I tried both a debit and a credit card. I couldn't even add the cards on file. On explanation that I came up with was that probably they needed a corporate credit card. A company name was a mandatory field in the billing form.

If that is really the case then, it is a real pity. People can't really utilize this for open source projects. Because if they do, they will need more than 50 build minutes but they can't pay for it.

### Community

Bitbucket simply doesn't have as big of a community as Github. If you get stuck, you are probably left on your own. When I tweeted about one of the problems I ran into the Bitbucket did immediately send me an answer. But they just referred me to their support forum where I posted but in the end I ended up answering my own question.

### Not all is bad

Not at all! Apart from the two bigger downsides and a GUI that is a bit finicky, the experience I had with Bitbucket was quite OK. I found everything well documented and the pipeline-as-code file has a clean, simple and straightforward structure. The fact that I could use my own containers for the pipelines steps was a huge plus and thanks to the dependency caching options, the execution times were quite short.

Now, I did use an extremely small project. But, thanks to those 50 minutes limitation I am not sure I would dare to move some of my larger projects over from Github.

## Links

* [Source Code](https://bitbucket.org/igorstojanovski/spring-boot-example/src/master/)
* [Postman CLI](https://www.npmjs.com/package/newman)
* Original photo by [Sirisvisual](https://unsplash.com/@sirisvisual?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/pipeline?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
* [Bitbucket pipelines documentation](https://support.atlassian.com/bitbucket-cloud/docs/configure-bitbucket-pipelinesyml/)