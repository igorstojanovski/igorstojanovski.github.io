---
layout: post
title: "Document your API with Spring REST Docs"
excerpt: "A good API documentation is crucial for ease of use of any API."
date: 2019-03-06
tags: [documentation, RESTful, spring, Development]
feature_image: __GHOST_URL__/content/images/wordpress/2019/03/GeneratedDocumentation.png
---

A good API documentation is crucial for ease of use of any API. But documentation is difficult to write and even more difficult to maintain. Spring REST Docs makes sure that process is easy and that your documentation is always up to date.

Let us see that with the help of a short and simple example.

### The sample API

The API that we will document is one for creating teams and adding and removing existing users as members to different teams.

It has 3 different end points:

* POST /api/team/
* POST /api/team/{teamId}/members
* DELETE /api/team/{teamId}/members

The team resource is simple with the most necessary fields:

```java
@Getter
@Setter
@NoArgsConstructor
@Entity
public class Team {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    @ManyToMany
    private Set<User> members;
}
```

## What is Spring REST Docs?

Spring REST Docs is a Spring tool for generating REST API documentation in a test driven manner. The fact that is test driven and integrated in the testing process is what makes it stand apart.

In it’s basis it uses tests to generate documentation snippets. The user can use those snippets in an Asciidoctor file and then generate HTML documentation from that file. Currently REST Docs can be used in tests that use **Spring MVC**’s test framework, **Spring WebFlux**’s **WebTestClient** or **REST Assured 3**.

## Spring REST Docs setup

In this example we are going to use **[MockMVC](__GHOST_URL__/java/spring-boot/mockmvc-test-spring-boot/)** and a Maven project. Based on that we will need the *spring-restdocs-mockmvc* dependency.

```java
		<dependency>
			<groupId>org.springframework.restdocs</groupId>
			<artifactId>spring-restdocs-mockmvc</artifactId>
			<scope>test</scope>
		</dependency>
```

Additionally, we need to add the **asciidoctor-maven-plugin** that will handle the processing of the generated snippets and configure the maven-resources-plugin if we want the documentation packed in the resulting jar file.

```java
	<build>
		<plugins>
			<!-- ... all your other plugins go here as well. -->
			<plugin>
				<groupId>org.asciidoctor</groupId>
				<artifactId>asciidoctor-maven-plugin</artifactId>
				<version>1.5.7.1</version>
				<executions>
					<execution>
						<id>generate-docs</id>
						<phase>prepare-package</phase>
						<goals>
							<goal>process-asciidoc</goal>
						</goals>
						<configuration>
							<backend>html</backend>
							<doctype>book</doctype>
						</configuration>
					</execution>
				</executions>
				<dependencies>
					<dependency>
						<groupId>org.springframework.restdocs</groupId>
						<artifactId>spring-restdocs-asciidoctor</artifactId>
						<version>${spring-restdocs.version}</version>
					</dependency>
				</dependencies>
			</plugin>
			<plugin>
				<artifactId>maven-resources-plugin</artifactId>
				<executions>
					<execution>
						<id>copy-resources</id>
						<phase>prepare-package</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>${project.build.outputDirectory}/static/docs</outputDirectory>
							<resources>
								<resource>
									<directory>${project.build.directory}/generated-docs</directory>
								</resource>
							</resources>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```

## Generating the snippets

As we mentioned, we need a MockMvc based test to generate the documentation. Let us add the test class for the Test resource:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureRestDocs
@AutoConfigureMockMvc
public class TeamTest {
    @Autowired
    private MockMvc mockMvc;
}
```

REST Docs provides the **@AutoConfigureRestDocs** that will do the necessary configuration of the **MockMvc** object. Or, we can also do that manually using a **JUnit 4** rule:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TeamManualMvcConfigurationTest {
    private MockMvc mockMvc = null;
    @Rule
    public JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation();
    @Autowired
    private WebApplicationContext context;

    @Before
    public void beforeEach() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
                .apply(documentationConfiguration(this.restDocumentation))
                .build();
    }
}
```

Both approaches seemed to work for me.

Next, we add the Team creation test:

```java
    @Test
    public void shouldCreateTeam() throws Exception {
        MvcResult storyResult = mockMvc.perform(MockMvcRequestBuilders
                .post("/api/team/")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\n" +
                        "  \"name\": \"Spaceballs\"\n" +
                        "}"))
                .andExpect(status().isCreated())
                .andDo(print())
                .andReturn();

        Team team = OBJECT_MAPPER.readValue(storyResult.getResponse().getContentAsString(), Team.class);
        assertThat(team.getId()).isNotNull();
    }
```

In order to document the request and response of “POST /api/team”, REST Docs provides the MockMvc **document** action. It can be used to document both the request and response:

```java
                .andDo(document("team",
                        requestFields(
                                fieldWithPath("name")
                                        .description("The name of the team.")
                        ))).
                andDo(document("team",
                        responseFields(
                                fieldWithPath("id")
                                        .description("ID of the team."),
                                fieldWithPath("name")
                                        .description("Name of the team."),
                                fieldWithPath("members")
                                        .description("Empty list of team members.")
                        )))
```

Running this test will generate the documentation snippets in the target folder.

![rest docs output folder](https://i1.wp.com/igorski.co/wp-content/uploads/2019/03/TeamSnippetsFolder.png)

The other two endopoints are interesting because we need to document a list of User objects. Luckily, REST Docs allows that:

```java
MvcResult addMembersResult = mockMvc.perform(MockMvcRequestBuilders
                .post("/api/team/" + team.getId() + "/members")
                .contentType(MediaType.APPLICATION_JSON)
                .content(OBJECT_MAPPER.writeValueAsString(teamMembersRequest)))
                .andExpect(status().isOk())
                .andDo(document("addTeamMembers",
                        requestFields(
                                fieldWithPath("members")
                                        .description("List of member IDs of type long that should be added to the team.")
                        )))
                .andDo(document("addTeamMembers",
                        responseFields(
                                fieldWithPath("id")
                                        .description("ID of the team."),
                                fieldWithPath("name")
                                        .description("Name of the team."),
                                fieldWithPath("members")
                                        .description("List of members in the team."),
                                fieldWithPath("members[].id")
                                        .description("ID of the user."),
                                fieldWithPath("members[].username")
                                        .description("The username of the user."),
                                fieldWithPath("members[].name")
                                        .description("The name of the user."),
                                fieldWithPath("members[].surname")
                                        .description("The surname of the user.")
                        )))
                .andDo(print())
                .andReturn();
```

You can find the whole, more detailed test in the GitHub [repository](https://github.com/igorstojanovski/basic-spring-boot-example/blob/master/src/test/java/org/igorski/example/controllers/TeamControllerTest.java).

### Generating the final documentation

Generating the snippets is half of the job. We still don’t have the final HTML document generated. In order to do that, we need to create a **team.adoc** file in the **/src/main/asciidoc/** folder*.* This file is in the [Asciidoc](https://asciidoctor.org/docs/asciidoc-writers-guide/) plain text documentation syntax. Running **mvn clean install** will generate the html file in the **/target/generated-docs** folder.

![rest docs generated html](https://i0.wp.com/igorski.co/wp-content/uploads/2019/03/GeneratedDocumentation.png?resize=1024%2C647)

## Documentation as test

The thing I like the most with this approach is that the by documenting the API you are testing it at the same time. That means that if you try to document a value that doesn’t exist the test will fail. The test will also fail if there is a value that is not documented. This solves the problem of documentation getting old and stale. **If you document like this you have no other option but to maintain your documentation snippets or your tests will fail.**

I do not reccoment id but, in case you want to avoid the tests from failing you can use the relaxed variant of the snippet methods:

```java
                andDo(document("team",
                        relaxedResponseFields(
                                fieldWithPath("id")
                                        .description("ID of the team."),
                                fieldWithPath("name")
                                        .description("Name of the team."),
                                fieldWithPath("members")
                                        .description("Empty list of team members.")
                        )))
```