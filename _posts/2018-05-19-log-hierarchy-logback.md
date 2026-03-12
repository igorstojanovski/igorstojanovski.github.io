---
layout: post
title: "Log hierarchy in Logback"
excerpt: "Log hierarchy in Logback is based on the log names."
date: 2018-05-19
tags: [Development, Java]
feature_image: __GHOST_URL__/content/images/wordpress/2018/05/AppenderAdditivity.png
---

Log hierarchy in Logback is based on the log names. That is a characteristic that makes lots of interesting and handy features possible. The post looks into a couple of examples how that works.

# How named log hierarchy works

According to the logger documentation, a logger is said to be an ancestor of another logger if its name followed by a dot is a prefix of the descendant logger name. A logger is said to be a parent of a child logger if there are no ancestors between itself and the descendant logger. Or as an example: **org.igorski** is the parent of **org.igorski.one** which in turn is the parent of **org.igorski.one.two**. On top of each hierarchy is the default **root** logger.

[![Log Hierarchy](https://i2.wp.com/igorski.co/wp-content/uploads/2018/05/LogbackHierarchy.png?resize=561%2C211 "log hierarchy")](https://i2.wp.com/igorski.co/wp-content/uploads/2018/05/LogbackHierarchy.png)

This usually corresponds to the package structure of the classes where the logs are used. But it doesn’t have to. You can have all three loggers named as in the diagram in the very same class.

## Log Hierarchy Example

Let us assume that we have a package structure that matches the log naming.

[![Package Hierarchy Logging](https://i0.wp.com/igorski.co/wp-content/uploads/2018/05/Screenshot_20180519_082033.png?resize=321%2C234)](https://i0.wp.com/igorski.co/wp-content/uploads/2018/05/Screenshot_20180519_082033.png)

### Log everything from package two in a file

```java
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>myApp.log</file>
        <encoder>
            <pattern>%date %level [%thread] %logger{10} [%file:%line] %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="org.igorski.two">
        <appender-ref ref="FILE" />
    </logger>

    <root level="info">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

This setup of logback.xml will make everything from package two to be logged in a file. Because **org.igorski.two** is parent of **org.igorski.two.three**, everything from package three will also be logged into that file.

### Log only package two in a file, but not package three

What if we want **org.igorski.two.three** to be logged only on console and not in a file? We will have to add a specific logger for package three:

```java
<logger name="org.igorski.two.three">
    <appender-ref ref="STDOUT" />
</logger>
```

But this doesn’t help. Not only that three is still logged in the file, it is logged in console twice:

```java
08:28:03.079 [main] INFO  org.igorski.Main - Running the examples.
08:28:03.085 [main] INFO  org.igorski.one.One - Creating class ONE
08:28:03.087 [main] INFO  org.igorski.two.Two - Creating class TWO
08:28:03.088 [main] INFO  org.igorski.two.three.Three - Creating class THREE
08:28:03.088 [main] INFO  org.igorski.two.three.Three - Creating class THREE
```

This happens because appenders are additive. If we define appenders for each of the child loggers, then the information that we append in one of the children will be passed along up the logger hierarchy and appended by all ancestors as well.

[![Appender Additivity](https://i1.wp.com/igorski.co/wp-content/uploads/2018/05/AppenderAdditivity.png?resize=651%2C321)](https://i1.wp.com/igorski.co/wp-content/uploads/2018/05/AppenderAdditivity.png)

This is a very useful feature. It basically means that you can assign an appender to the root logger and it will pick up all the info from all the loggers down the line. But if you want to log some of the children in a file, all you need to add a file appender to that child. Thanks to the additivity the information will show up in both places.

However, in our case that causes problems for us. We want the information to be appended only by package **three**. We need to break the additivity. That is easily done by setting the additivity to false:

```java
<logger name="org.igorski.two.three" additivity="false">
    <appender-ref ref="STDOUT" />
</logger>
```

Now whatever is logged from package **three** will be appended only by the **STDOUT** appender and by no other ancestor appenders.

# Log hierarchy and log levels

It is worth noting that log levels are also inherited. If we set **org.igorski** to **TRACE** it will be inherited by all children. Log levels are inherited from the closest immediate ancestor that has log level set. That means that if we set **org.igorski.two** to **INFO** that level will be inherited by package three although the topmost parent is set to **TRACE**. There is a much more comprehensive list of examples in [Logback’s manual](https://logback.qos.ch/manual/architecture.html#effectiveLevel).