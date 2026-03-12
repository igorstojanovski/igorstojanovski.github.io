---
layout: post
title: "Publishing to Maven Central is easy"
excerpt: " 
[https://i1.wp.com/igorski.co/wp-content/uploads/2018/07/d8c256cc-e1bf-4e78-83e8-6aeaefce3231-af8ee71d-b511-4bbc-8ff0-c453a24018cd-3645febb-f64d-42ba-8324-7dd2ea2df7fe-logo-large.png]
Sonatype [http"
date: 2018-07-28
tags: [maven, maven central, publish]
feature_image: __GHOST_URL__/content/images/wordpress/2018/07/sonatype-logo.png
---

[![publishing to maven central via OSSRH](https://i1.wp.com/igorski.co/wp-content/uploads/2018/07/d8c256cc-e1bf-4e78-83e8-6aeaefce3231-af8ee71d-b511-4bbc-8ff0-c453a24018cd-3645febb-f64d-42ba-8324-7dd2ea2df7fe-logo-large.png?resize=256%2C256)](https://i1.wp.com/igorski.co/wp-content/uploads/2018/07/d8c256cc-e1bf-4e78-83e8-6aeaefce3231-af8ee71d-b511-4bbc-8ff0-c453a24018cd-3645febb-f64d-42ba-8324-7dd2ea2df7fe-logo-large.png)

[Sonatype](https://www.sonatype.com) logo

A few days ago I needed to publish a library to Maven Central for the first time. After publishing, my plan was to write a tutorial on how it is supposed to be done. But it turned out there was absolutely no need for yet another tutorial. Sonatype’s OSS Repository Hosting ([OSSRH](https://central.sonatype.org/pages/ossrh-guide.html)) does a perfect job at explaining in detail every step of the process. If you’ve ever been afraid that publishing to Maven Central is difficult and complicated, don’t. It is really easy.

Because the OSSRH guide does such a good job I will only explain what didn’t go quite as planned for me.

# Other tutorials on publishing to Maven Central

There are other tutorials out there that explain how to do the same things as the OSSRH guide. I started by following one of those. It advised me to use a Maven parent POM. Only later I found out that that parent POM is deprecated and should not be used.

Some of the tutorials out there are probably outdated. Just stick to the original guideline. I bet they keep it always up to date.

# Problems with the account setup

After my project was created I was notified on [my ticket](https://issues.sonatype.org/browse/OSSRH-41390) that I can now publish my artifacts. Publishing the snapshot went smoothly but then when I tried to release a full version I got an error:

```java
[INFO] Performing remote staging...
[INFO] 
[INFO] * Remote staging into staging profile ID "5b2a80718c9614"
[ERROR] Remote staging finished with a failure: 403 - Forbidden
[ERROR] 
[ERROR] Possible causes of 403 Forbidden:
[ERROR] * you have no permissions to stage against profile with ID "5b2a80718c9614"? Get to Nexus admin...
```

I had no idea if my local credentials setup was wrong or their account setup was wrong. I commented back on the ticket and it turned out it was the latter. They fixed in less than 24 hours.

# Gpg problem

I work on Windows 10. I installed this [here](https://gpg4win.org/download.html) gpg pack. It also installs the Kleopatra GUI which I used for all the steps of the process. Everything worked without a problem. Until the last step of the deploy process that is. My artifacts were not being uploaded and it didn’t say why. It just stopped. It turned out that I missed adding the **maven-gpg-plugin** to my pom. So the reason why Sonatype complained is that my artifacts were not signed.

Make sure you double check you’ve added all you need to your pom.

# Additional pom information

Adding all the plugins and the group id, artifact id, and version to the pom in order to publish, is not enough. You need to add a large list of additional information as well. That includes license, project, developer, environment information etc. If you try to deploy without these your build will fail and you will be notified what you’re missing.

That is how I found about this requirement, the hard way. Although everything is very well explained in this [page](https://central.sonatype.org/pages/requirements.html) of the official guide. So pay attention to the guide, it will save you some time.

# My whole pom

With all that said this is the fully functioning release profile of my pom:

```java
<profile>
            <id>release</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>2.2.1</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.9.1</version>
                        <executions>
                            <execution>
                                <id>attach-javadocs</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.sonatype.plugins</groupId>
                        <artifactId>nexus-staging-maven-plugin</artifactId>
                        <version>1.6.7</version>
                        <extensions>true</extensions>
                        <configuration>
                            <serverId>ossrh</serverId>
                            <nexusUrl>https://oss.sonatype.org/</nexusUrl>
                            <autoReleaseAfterClose>true</autoReleaseAfterClose>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.5</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
```

This is the additional info that was required:

```java
<!-- More Project Information -->
<name>Samebug Junit Extension</name>
<description>JUnit 5 extension for Samebug.</description>
<url>https://github.com/igorstojanovski/SamebugExtension</url>
<inceptionYear>2018</inceptionYear>
<licenses>
    <license>
        <name>The Apache Software License, Version 2.0</name>
        <url>https://www.apache.org/licenses/LICENSE-2.0</url>
    </license>
</licenses>

<developers>
    <developer>
        <id>igorski</id>
        <name>Igor Stojanovski</name>
        <email>igorce@gmail.com</email>
        <url>https://igorski.co</url>
    </developer>
</developers>

<!-- Environment Settings -->
<scm>
    <url>https://github.com/igorstojanovski/SamebugExtension</url>
    <connection>scm:git:https://github.com/igorstojanovski/SamebugExtension.git</connection>
    <developerConnection>scm:git:https://github.com/igorstojanovski/SamebugExtension.git</developerConnection>
</scm>
```

And this is what I needed to add to the settings.xml:

```java
<profiles>
  <profile>
    <id>ossrh</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
      <gpg.executable>gpg</gpg.executable>
      <gpg.passphrase>HERE_GOES_YOUR_PASS_PHRASE</gpg.passphrase>
    </properties>
  </profile>
</profiles>

  <server>
    <id>ossrh</id>
    <username>YOUR_JIRA_USERNAME</username>
    <password>YOUR_JIRA_ACCOUNT</password>
  </server>
```

In the server section, add the username and pass you use to login to Sonatype’s JIRA.

Here you can see [the whole pom](https://github.com/igorstojanovski/SamebugExtension/blob/master/pom.xml) file in GitGub.

# Conclusion

Publishing to Maven Central via OSSRH is fairly easy and well documented. But, make sure you follow that guide down to a T.