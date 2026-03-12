---
layout: post
title: "Thymeleaf Layout Dialect with Spring Boot 2"
excerpt: "The Layout dialect is a dialect for Thymeleaf [http://www.thymeleaf.org] that
allows users to build layouts and reusable templates in order to improve code
reuse."
date: 2018-04-03
tags: [fronted, thymeleaf, spring boot]
feature_image: __GHOST_URL__/content/images/wordpress/2018/04/index_layout-1.png
---

The Layout dialect is a dialect for [Thymeleaf](http://www.thymeleaf.org) that allows users to build layouts and reusable templates in order to improve code reuse. It has a hierarchical approach and it uses the decorator pattern for ‘decorating’ the layout files. The Layout Dialect is a separate project and does not come with Thymeleaf. However, it is open source, available [on GitHub](https://github.com/ultraq/thymeleaf-layout-dialect), it is well documented, and as it looks, well maintained as well.

# Setup

We will need to add the Thymeleaf starter pack to your Spring Boot pom:

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

However, starting from Spring Boot 2 this is no longer enough. The Layout Dialect is not part of the starter and we need to add it on our own:

```java
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
    <version>2.3.0</version>
</dependency>
```

The code examples also use Bootstrap so the webjars need to be added as well:

```java
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>4.0.0</version>
</dependency>
```

As a last step, we need to create the LayoutDialect bean in a @Configuration annotated class.

```java
@Bean
public LayoutDialect layoutDialect() {
    return new LayoutDialect();
}
```

And we are good to go.

# Layout Dialect Example

The [example is going to show](https://github.com/igorstojanovski/basic-spring-boot-example) how we can use the Layout Dialect in order to define layouts for our pages so we can have better code reuse. It demonstrates that with an **index.html** page that uses **layout.html** as its layout. **Layout.html**‘s name is arbitrary and it can be any name at all. There a few more files added but those are just for looks. The

[![resources directory content](https://i0.wp.com/igorski.co/wp-content/uploads/2018/04/resources_layout_structure.png?resize=463%2C228)](https://i0.wp.com/igorski.co/wp-content/uploads/2018/04/resources_layout_structure.png)

Structure of the resource folder. Spring Boot will automatically find all Thymeleaf templates in the resources/templates directory.

## The layout.html

```java
<!DOCTYPE html>
<html>
    <head>
        <title layout:title-pattern="$LAYOUT_TITLE - $CONTENT_TITLE">Igorski.co</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <link th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.min.css}" rel="stylesheet" media="screen" />
    </head>
    <body>
        <div th:replace="fragments/header :: header"> This header content is going to be replaced.</div>

        <div class="container">
            <div class="row">
                <div class="col-2" layout:fragment="sidebar">
                    <h1>This is the layout's sidebar</h1>
                    <p>This content will be replaced if the page
                    using the layout also defines a layout:fragment="sidebar" segment.</p>
                </div>
                <div class="col" layout:fragment="content">
                    <h1>This is the Layout's main section</h1>
                    <p>This content will be replaced if the page
                        using the layout also defines a layout:fragment="content" segment. </p>
                </div>
            </div>
        </div>

        <footer th:insert="fragments/footer :: footer" class="footer">
            This content will remain, but other content will be inserted after it.
        </footer>
    </body>
</html>
```

The layout.html uses two of the five processors introduced by the Layout Dialect. First is the **layout:title-pattern** processor. The title-pattern processor helps the user define a better title for the resulting page. In case of this example it defines the final title as a combination of the page’s title and the layout’s title. It does that using the two special tokens introduced by the Layout Dialect, $LAYOUT\_TITLE and $CONTENT\_TITLE.

Of biggest importance in the layout.html are the two placeholders (or fragments) defined by the **layout:fragment** processor. This processor allows us to define content placeholders in our layouts. The content of these placeholders is going to be later replaced by content from the pages that use the layout. The example defines two different fragments, one for the sidebar and another for the main content. We however, can have as many fragments as we wish, as long as they all have different names.

## The index.html

```java
<!DOCTYPE html>
<html xmlns:layout="http://www.w3.org/1999/xhtml" layout:decorate="~{layouts/layout}">

    <head>
        <title>Home Page</title>
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <link th:href="@{/css/core.css}" rel="stylesheet" media="screen" />
    </head>
    <body>
        <div class="container">
            <div class="row">
                <div class="col-2" layout:fragment="sidebar">
                    <h1>Sidebar</h1>
                    <a href="#">Login</a>
                </div>
                <div class="col" layout:fragment="content">
                    <h1>Welcome to the Index page</h1>
                    <br>This content is replacing the content of the layout:fragment="content"<br>
                    placeholder in layout.html</p>
                </div>
            </div>
        </div>
    </body>
</html>
```

We declare that the **index.html** uses the **layout.html** as its layout with the **layout:decorate** processor. By doing that we declare that index.html will decorate the layout.html file. Here the most important thing is again the use of the **fragment** processor. It specifies the content to be used instead of the content of the layout fragments with the same name. Another thing worth mentioning is the header. In the resulting index.html page we get, after the processing, the header will be a combination of the two headers, one from index.html and the other from the layout. It is going to be combined into one.

Both layout.html and index.html can be viewed in a browser without any processing. But after the processing we can see that the index.html is very different. The content from index.html is used to decorate the layout, and place content inside the layout based on what the layout defines.

Two other elements are present in the example, the header and the footer. However, they use the Thymeleaf Standard Layout processors **th:replace** and **th:insert**. Very similar to these are the last two of the five processors the Layout Dialect introduces, **layout:insert** and **layout:replace**. They more or less do the same thing. Unlike the previous processors we discussed, these two don’t use the hierarchical but the include approach. That is more of a characteristic of the Thymeleaf Standard Layout.

[![index created using the layout dialect](https://i0.wp.com/igorski.co/wp-content/uploads/2018/04/index_layout-1.png?resize=486%2C251 "index created using the layout dialect")](https://i0.wp.com/igorski.co/wp-content/uploads/2018/04/index_layout-1.png)

The final look of the index page. It has both header and footer although none are mentioned in the index.html markup.