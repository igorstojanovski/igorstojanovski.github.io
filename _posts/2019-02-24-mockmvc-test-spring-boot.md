---
layout: post
title: "Use MockMvc to test Spring Boot applications"
excerpt: "When I need to create and API endpoint I usually like to write an integration
test first that will cover the basic happy scenarios."
date: 2019-02-24
tags: [JUnit 4, mockmvc, spring boot]
feature_image: __GHOST_URL__/content/images/wordpress/2019/02/MockMvc.png
---

When I need to create and API endpoint I usually like to write an integration test first that will cover the basic happy scenarios. By doing this I make sure that everything works well together and that I have the most important aspects in place. I find that the best tool for writing this type of integration tests when working with **Spring Boot** is **MockMvc**.

## What is MockMvc?

MockMvc is a Spring Boot test tool class that lets you test controllers without needing to start an HTTP server. In these tests the application context is loaded and you can test the web layer as if i’s receiving the requests from the HTTP server without the hustle of actually starting it.

#### Versions used in the example

These are the two maven dependencies that are important:

```java
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.2.RELEASE</version>
		<relativePath/>
	</parent>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
```

The spring-boot-starter-parent determines the version of spring-boot-starter-test. The latter one pulls in a bunch of dependencies that are important for writing tests:

* JUnit 4: I’ll explain in a follow up post how to use JUnit 5.
* Spring Test & Spring Boot Test: This brings in the MockMvc class.
* AssertJ
* Hamcrest
* Mockito
* JSONassert
* JsonPath

We will use most if not all of these in the following example.

## MockMvc Example

This a code of our working example:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class UserResourceIT {

    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void shouldCreateUser() throws Exception {
        User createdUser = createUser();
        assertThat(createdUser.getId()).isNotNull();
        assertThat(createdUser.getPassword()).isNullOrEmpty();
    }

    private User createUser() throws Exception {
        User user = new User();
        user.setName("Igor");
        user.setSurname("Stojanovski");
        user.setUsername("igorski");
        user.setPassword("1234#atdk");

        String postValue = OBJECT_MAPPER.writeValueAsString(user);

        MvcResult storyResult = mockMvc.perform(MockMvcRequestBuilders
                .post("/api/user/")
                .contentType(MediaType.APPLICATION_JSON)
                .content(postValue))
                .andExpect(status().isCreated())
                .andDo(print())
                .andReturn();

        return OBJECT_MAPPER.readValue(storyResult.getResponse().getContentAsString(), User.class);
    }
}
```

Let us go step by step and deconstruct this.

### @RunWith(SpringRunner.class)

Since the example uses **JUnit 4** you need to use the appropriate JUnit 4 runner for the occasion.

### @SpringBootTest

**@SpringBootTest** is a Spring Boot alternative for the standard **@ContextConfiguration**. It will use **SpringApplication** to load the **ApplicationContext**. By default, it will not start a server but it will provide a mock environment. However, you can use **webEnvironment** to configure the annotation and to start a server either with a fix or a random port.

### @AutoConfigureMockMvc

**@AutoConfigureMockMvc** will configure the MockMvc object. You can omit this annotation in case you want to do the configuration manually:

```java
    private MockMvc mockMvc;

    @Before
    public void beforeEach() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
                .apply(documentationConfiguration(this.restDocumentation))
                .build();
    }
```

In that case the MockMvc won’t be autowired.

### Posting the request

The test needs to test if posting to the /api/user endpoint will create a new user. Thanks to MockMvc’s fluent interface the code is very easy to understand.

```java
      MvcResult storyResult = mockMvc.perform(MockMvcRequestBuilders
                //We say that we want to do a post
                .post("/api/user/")
                //We declare the content type.  
                .contentType(MediaType.APPLICATION_JSON)
                //At last, we set the content. 
                .content(postValue))
                //Then expect the status of the request to be 201.
                //There are different methods for different HTTP statuses.
                .andExpect(status().isCreated())
                //We can print or log the response.
                .andDo(print())
                // At the end we return the result.
                .andReturn();
```

From the result we can get the response as a String and then do with it as we please. We can also check a long list of things like cookies, headers etc. The post method is part of the **MockMvcRequestBuilders** class. In it you can find methods for the rest of the HTTP verbs, put, get, delete and patch. You use the methods in a similar way as in the example.

## A liter alternative to @SpringBootTest

As we mentioned, **@SpringBootTest** will load the whole context using the **@SpringBootApplication** main class. Instead of having all beans loaded in the context you sometimes might want to mock them and test only a single controller instead. That can be used using @WebMvcTest:

```java
@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserResourceWebMvcTest {

    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();
    @Autowired
    private MockMvc mockMvc;
    @MockBean
    private UserService userService;
    private User testUser;

    @Before
    public void beforeEach() {
        testUser = new User();
        testUser.setName("Igor");
        testUser.setSurname("Stojanovski");
        testUser.setUsername("igorski");
        testUser.setPassword("1234#atdk");

        User createdUser = new User();
        createdUser.setId(101L);

        given(userService.createUser(testUser)).willReturn(createdUser);
    }
    
    // ....
}
```

There is no need to show a test because this doesn’t change how we work with MockMvc. It is the same as in the previous example. What it changes is the way we do the setup. **@WebMvcTest** also takes care to auto configure MockMvc so the **@AutoConfigureMockMvc** is no longer needed.

## Checking the result content

The goal of the test most often is to check whether the returned content is what we expect it to be. If the result content is JSON MockMvc lets you use [JsonPath](https://goessner.net/articles/JsonPath/) matchers:

```java
                .andExpect(jsonPath("$.id", is(notNullValue())))
                .andExpect(jsonPath("$.name", is("Igor")))
                .andExpect(jsonPath("$.surname", is("Stojanovski")))
```

The static sonPath() method is found both in MockRestRequestMatchers and MockMvcResultMatchers. Make sure to import from the latter to avoid confusions.

The JsonPath approach is good because it lets you do everything in one command. Setup the request, send it, assert on the response on every possible way. Very neat. However, you do need a bit of knowledge of JsonPath for more complicated assertions. Because of that I usually opt to use Jackson’s ObjectMapper to transform the JSON in a POJO and then use AssertJ for more standard way of doing assertions.

## Example Code

You can find a full working example with both a post and a get request example in my [GitHub repository](https://github.com/igorstojanovski/basic-spring-boot-example/blob/master/src/test/java/org/igorski/example/UserResourceMockMvcIT.java).