
## SDKMan

```bash
sdk install java 17.0.15-librca
sdk use java 17.0.15-librca
```

## start.spring.io

[Create a new project](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.5.3&packaging=jar&jvmVersion=17&groupId=com.example.modulith&artifactId=simple&name=&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.modulith.simple&dependencies=native,web,actuator,devtools,modulith)

```bash
unzip `~/Downloads/simple.zip`
cd simple
sdk env init
idea .
```
> Open the new project in IntelliJ

## Make a new package

```text
com.example.modulith.simple.hello
```

## Make a new class in that package

```text
HelloController
```

## Update the class

```java
package com.example.modulith.simple.hello;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@ResponseBody
class HelloController {

    @GetMapping("/hello")
    String hello (){
        // refactor and rebuild
        return "Hello World!" ;
    }
}
```

## Start the app and Look around

- [http://localhost:8080/](http://localhost:8080)
- [http://localhost:8080/hello](http://localhost:8080/hello)
- [http://localhost:8080/actuator](http://localhost:8080/actuator)
- [http://localhost:8080/actuator/health](http://localhost:8080/actuator/health)


## More info at runtime

Add properties to get more information

```text
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
management.endpoint.env.show-values=always
management.info.env.enabled=true
management.info.java.enabled=true
management.info.os.enabled=true
management.info.process.enabled=true
management.server.port=8081
server.port=8080
```
>src/main/resources/application.properties

## Restart the app and Look around

- [http://localhost:8080/](http://localhost:8080)
- [http://localhost:8080/hello](http://localhost:8080/hello)
- [http://localhost:8081/actuator](http://localhost:8081/actuator)
- [http://localhost:8081/actuator/health](http://localhost:8081/actuator/health)
- [http://localhost:8081/actuator/info](http://localhost:8081/actuator/info)

## Add Tests and Documentation

Documentation has never been more important than it is today!

More tests results in more confidence!

## Make a Modularity Test

Spring Modulith helps us design for the future.

```java
package com.example.modulith.simple;

import org.junit.jupiter.api.Test;
import org.springframework.modulith.core.ApplicationModules;
import org.springframework.modulith.docs.Documenter;

class ModularityTests {

    ApplicationModules modules = ApplicationModules.of(Application.class);

    @Test
    void verifiesModularStructure() {
        modules.forEach(System.out::println);
        modules.verify();
    }

    @Test
    void createModuleDocumentation() {
        new Documenter(modules).writeDocumentation();
    }

    @Test
    void createPlantUml() {
        new Documenter(modules)
                .writeModulesAsPlantUml()
                .writeIndividualModulesAsPlantUml();
    }

    @Test
    void writeDocumentationSnippets() {

        new Documenter(modules)
                .writeAggregatingDocument();
    }

}
```
> src/test/java/com/example/modulith/simple/ModularityTests.java

## Run all the tests and generate the documentation

Documentation will be in `target/spring-modulith-docs`

## Test the endpoint and the other TDD, Test Driven Documentation

```xml
    <properties>
        <java.version>17</java.version>
        <project.build.outputDirectory>null</project.build.outputDirectory>
        <spring-modulith.version>1.4.0</spring-modulith.version>
        <spring-restdocs.version>3.0.4</spring-restdocs.version>
    </properties>
```
> Update properties

```xml
    <dependency>
      <groupId>org.springframework.restdocs</groupId>
      <artifactId>spring-restdocs-mockmvc</artifactId>
      <scope>test</scope>
    </dependency>
```
> Add dependency

```xml
      <plugin>
        <groupId>org.asciidoctor</groupId>
        <artifactId>asciidoctor-maven-plugin</artifactId>
        <version>2.2.1</version>
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
```
> Add build plugins

## Add doc template

```asciidoc
= Getting Started With Spring REST Docs

This is an example output for a service running at http://localhost:8080/:

operation::hello[]

As you can see the format is very simple, and in fact you always get the same message.
```
> src/main/asciidoc/hello.adoc

## Add a document root that points to the docs

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Test Driven Documentation::The other TDD</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p>Use the API <a href="/hello">here</a></p>
<p>The documentation is published <a href="docs/hello.html">here</a></p>
</body>
</html>
```
> src/main/resources/static/index.html

## Add RestDocs Test

```java
@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
@WebMvcTest(HelloController.class)
class HelloControllerTest {

    private MockMvc mockMvc;

    @Autowired
    private WebApplicationContext context;

    @BeforeEach
    void setUp(WebApplicationContext webApplicationContext, RestDocumentationContextProvider restDocumentation) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                .apply(documentationConfiguration(restDocumentation))
                .build();
    }

    @Test
    public void shouldReturnDefaultMessage() throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("Hello World")))
                .andDo(document("hello"));
    }
}
```

## Lets rebuild everything with documentation

```bash
./mvnw clean package
```


v1
image
deploy

Add logs
Add Loki


Add Eureka Client
Spring Cloud Gateway
- parameterize hello world
add config client
Spring Cloud Config Server
deploy v2 and v1


## Now how do I get a (better) job with what I've learned here

- Rinse, repeat
- Social media and the job market
- "Make your own slides"
- Make it better
- Share