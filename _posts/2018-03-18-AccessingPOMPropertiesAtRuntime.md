---
layout: post
title: "Accessing The Pom Properties At Runtime"
date: 2018-18-03
---

Recently, I found myself needing to access specific POM properties during runtime, specifically the version-related properties: artifact id, group id, and version. After a bit of exploration, I found at least three ways to achieve this.

Each approach comes with its pros and cons, so feel free to choose the one that resonates best with your needs.

The manifest of the jar file is the ideal place to locate version info. However, Maven doesn't populate it with this info by default. Here's how you can configure the **maven-jar-plugin** to do it for you:

```xml
<groupId>org.apache.maven.plugins</groupId> <artifactId>maven-jar-plugin</artifactId> <configuration> <archive> <manifest> <mainClass>org.programirame.Main</mainClass> <addDefaultImplementationEntries>true</addDefaultImplementationEntries> </manifest> </archive> </configuration>
```

With the **addDefaultImplementationEntries** set to true, Maven will insert these values into the manifest file:

```bash
Implementation-Title: ${project.name} Implementation-Version: ${project.version} ...
```

Once built, retrieving the info in Java becomes straightforward:

```java
Package mainPackage = Main.class.getPackage(); String version = mainPackage.getImplementationVersion();
```

This approach is simple and direct. However, for POM properties beyond the ones mentioned, it may not be suitable.

### Using Maven Filtering to Access POM Properties

Maven's filtering feature lets you replace resource file variables with the actual POM property values. Start by creating a **pom.properties** file in your resource folder:

```bash
artifactId=${project.artifactId} groupId=${project.groupId}
```

Then, set up the resource section of the build phase:

```xml
<resources> <resource> <directory>src/main/resources</directory> <filtering>true</filtering> <includes> <include>**/pom.properties</include> </includes> </resource> </resources>
```

Post-build, the **pom.properties** in the resulting jar will house the real values. Access them in Java like so:

```java
Properties properties = new Properties(); properties.load(Main.class.getResourceAsStream("/pom.properties"));
```

### The Properties Plugin Approach

Maven offers a specialized plugin for handling POM properties: the **properties-maven-plugin**. Here's how you can use it to write the properties to a **version.properties** file:

```xml
<groupId>org.codehaus.mojo</groupId> <artifactId>properties-maven-plugin</artifactId> <version>1.0.0</version>
```

However, there's a catch: for a property to be written to the file, you must explicitly define it in the POM's properties section
