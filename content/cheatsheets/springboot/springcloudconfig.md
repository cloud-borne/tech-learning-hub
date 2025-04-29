---
title: Spring Cloud Config
linktitle: Spring Cloud Config
type: book
date: "2019-05-05T00:00:00+01:00"
tags:
  - SpringBoot
  - Netflix

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

Centralized versioned Configuration Management

<!--more-->

### Overview

Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer’s own laptop, bare metal data centers, and managed cloud platforms such as AWS.

### The Cloud :partly_sunny: Problem

Applications are more than just code, it also **Configuration** :triangular_ruler:

i.e. Connection to resources, URLs to other services, timeouts and so on.

Even before Cloud, in the traditional J2EE world these were mostly **exteranlized** to adjust software behavior.
In the Cloud supporting this exteranlized configuration had it's challenges:

- Need for build/restart of applications to refresh
- No common file system/network share in Cloud
- Large number of configurations. Even larger than the application we are porting if we go by the microservices design pattern and break up the monolith.
- Tight coupling to Cloud Provider if designed incorrectly.

### The Desired Solution (Blue Sky here with no :cloud:)

* Platform/Cloud Independent Solution
  - Language-independent too
* Centralized
  - Or a few discreet sources of our choosing
* Dynamic
  - Ability to update settings while the app is running
* Versioned
  - Same SCM choices we use with software
* Passive
  - Services (Applications) should do most of the work themselves by self-registering

**Spring Cloud Config** provides just that i.e. a server and client-side support for externalized configuration in a distributed system.
Spring Cloud Config employs Server-Client approach for storing the configuration externally at 1 place. Any changes in configuration doesn’t need to deploy the associated services, in-fact these changes are reflected automatically.

![SpringCloud-Config](/images/uploads/spring-cloud-config.PNG)

### Config Server

Source code for a sample Config-Server can be found [here](https://github.com/spring-cloud-samples/configserver)

But it's relatively straight forward to look under the hood and create one ourselves. That way we may be able to learn a few important concepts related to "Spring" configuration in the cloud.

Here's the steps I followed:

1. Create a new Spring Boot application. Name the project "config-server”, and use this value for the Artifact. Use Jar packaging and the latest versions of Java(jdk11), Maven/Gradle and SpringBoot(v2.5.7). We should add spring-boot-starter-security and spring-boot-starter-actuator dependencies. We would be adding the spring-cloud dependencies manually.

2. Edit the POM (or Gradle) file.  Add a “Dependency Management” section (after `<properties>`, before `<dependencies>`) to identify the spring cloud parent POM. "2020.0.4" is the most recent stable version at the time of this writing, but you can generally use the latest stable version available.  

3. Add a dependency for group "org.springframework.cloud" and artifact "spring-cloud-config-server".  You do not need to specify a version -- this is already defined in the spring-cloud-dependencies POM.

    Example:

    ```xml
    <dependencyManagement>
      <dependencies>
    	  <dependency>
    		 <groupId>org.springframework.cloud</groupId>
    		 <artifactId>spring-cloud-dependencies</artifactId>
    		 <version>2020.0.4</version>
    		 <type>pom</type>
    		 <scope>import</scope>
    	  </dependency>
      </dependencies>
    </dependencyManagement>
    ...
    ...
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    ```
4. Edit the main Application class (probably named ConfigServerApplication). Add the @EnableConfigServer to this class.

    ```Java
    @SpringBootApplication
    @EnableConfigServer
    public class ConfigServerApplication {

    	public static void main(String[] args) {
    		SpringApplication.run(ConfigServerApplication.class, args);
    	}

    }
    ```

5. Next we need to add/edit the "Spring" configuration. We can edit the application.yml/application.properties for the following properties:

    **spring.application.name**=config-server
    **spring.cloud.config.server.native.searchLocations**=classpath:/shared
    **spring.config.activate.on-profile**=native

    **spring.security.user.name**=user
    **spring.security.user.password**=${CONFIG_SERVICE_PASSWORD}

    **server.port**=8888

    Here's the complete application.yml:

      ```yml

      spring:
        application:
          name: config-server
        cloud:
          config:
            server:
              native:
                search-locations: classpath:/shared
        config:
          activate:
            on-profile:
            - native
        security:
          user:
            name: user
            password: ${CONFIG_SERVICE_PASSWORD}

      server:
        port: 8888
      ```

{{% callout note %}}

**spring.application.name** configuration has a few different purposes:
1. Registering spring application name to the naming server(Netflix Eureka)
2. Getting configuration from spring cloud configuration by application name and profile.

{{% /callout %}}

{{% callout note %}}

spring.cloud.config supports a host of backends as the source for storing configuration.
Some examples:
- Git (default)
- Subversion
- Filesystem
- Redis
- Vault
- JDBC
- AWS S3
and possibly others in different Cloud platforms.

For this example we would be using the Filesystem approach. Since our project would be checked into GitHub it's also version controlled and auditable. There is a “native” profile in the Config Server that does not use Git but loads the config files from the local classpath or file system (any static URL you want to point to with **spring.cloud.config.server.native.searchLocations**).

Obviously for an Enterprise solution we should consider better options like Vault etc.

{{% /callout %}}

{{% callout note %}}

To use the native profile, we launch the Config Server with **native** profile. In Spring Boot 2.3, you’d use the spring.profiles key to do this. With Spring Boot 2.4, Spring team  decided to change the property to spring.config.activate.on-profile.

Further read [Config file processing in Spring Boot 2.4](https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4)

{{% /callout %}}

{{% callout note %}}

We added the spring security dependency to our Config Server earlier since we wish to have at least Basic Authentication happening between the Config Server and it's clients. More sophisticated means are possible and supported. We parameterized the password and would use an Environment variable while starting Config Server to set it.

{{% /callout %}}

6. If you use Maven, run the following command in a terminal window (in the complete) directory:
    ```    
    ./mvnw spring-boot:run -Dspring-boot.run.profiles=native \
                           -Dspring-boot.run.jvmArguments="-DCONFIG_SERVICE_PASSWORD=********"
    ```

With that you have a fully functional Config Server that meets our
[Desired Solution]({{< ref "/cheatsheets/springboot/springcloudconfig.md#The Desired Solution" >}}) :clap:

Next we will talk about the Config Client.

### Config Client

Essentially if you think about it in a Microservices Architecture, **ALL** services fit the pattern of a **Config Client**, since all of them need some configuration and **Config Server** serves them just that.

Here's the steps I took:

1. Create a new, separate Spring Boot application. Use a latest version of Spring Boot(2.5.7). Name the project "todo-server", and use this value for the Artifact. Add the web dependency. You can make this a JAR project.

2. Open the POM (or Gradle) file and add a “Dependency Management” section (after , before ) to identify the spring cloud parent pom. (You could simply change the parent entries, but most clients will probably be ordinary applications with their own parents)

3. Add a dependency for group "org.springframework.cloud" and artifact "spring-cloud-starter-config”. You do not need to specify a version -- this is already defined in the parent pom in the dependency management section.

4. For the Config Client, it could be a [Config First Bootstrap](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#config-first-bootstrap) or a [Discovery First Lookup](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#discovery-first-bootstrap).
With Spring Cloud Vault 3.0 and Spring Boot 2.4, the default bootstrap context initialization (bootstrap.yml, bootstrap.properties) of property sources was deprecated. Hence there are two choices for the Config Client:

  - Use Spring Boot 2.4.0 Config Data API to import configuration from Vault (Preferred)

  - Legacy Processing: Enable the bootstrap context either by setting the configuration property spring.cloud.bootstrap.enabled=true or by including the dependency **spring-cloud-starter-bootstrap**. In order to focus the conversation around Spring Cloud Config we use the second option.

    ```xml
    <dependencyManagement>
    		<dependencies>
    			<dependency>
    				<groupId>org.springframework.cloud</groupId>
    				<artifactId>spring-cloud-dependencies</artifactId>
    				<version>2020.0.4</version>
    				<type>pom</type>
    				<scope>import</scope>
    			</dependency>
    		</dependencies>
    </dependencyManagement>
    ...
    ...
    <dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>
    ```

5. Add a bootstrap.yml (or bootstrap.properties) file in the root of your classpath (src/main/resources recommended). We need to tell todo-server where to look further for it's configuration. That's all. Add the following key/values using the appropriate format:

    ```yml
    spring:
      config:
        activate:
          on-profile:
           - default
      application:
        name: todo-server
      cloud:
        config:
          uri: http://localhost:8888
          fail-fast: true
          password: ${CONFIG_SERVICE_PASSWORD}
          username: user
    ```

6. Add a todo-server.yml file on the config-server project under src/main/resources/shared folder.
Add the following content there:

    ```yml
    spring:
      config:
        activate:
          on-profile:
           - default

    todo-config: TBD

    server:
      port: 8081
      servlet:
        context-path: /todo-server    
    ```

7. Edit the application to read the **todo-config**. For simplicity's sake we add a ```@RestController``` annotation to the SpringBoot class and just returned to the browser.

```
@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class TodoServerApplication {

	@Value("${todo-config}")
	String todoConfig;

	public static void main(String[] args) {
		SpringApplication.run(TodoServerApplication.class, args);
	}

	@GetMapping("/todo-config")
	public String showTodoConfig() {
		return "The todoConfig is: " + todoConfig;
	}
}

```

8. If you use Maven, run the following command in a terminal window (in the complete) directory:
    ```    
    ./mvnw spring-boot:run -Dspring-boot.run.profiles=default \
                           -Dspring-boot.run.jvmArguments="-DCONFIG_SERVICE_PASSWORD=********"
    ```

### Reflection

1. Few Observations:

    Navigate to:

    http://localhost:8888/testConfig-default.properties

    http://localhost:8888/testConfig-default.yml

    http://localhost:8888/testConfig/default

    http://localhost:8081/todo-server/todo-config

2. Notice that the client needed some dependencies for Spring Cloud, and the URI of the Spring Cloud server, but no code.

3. What happens if the Config Server is unavailable when the todo-server application starts? To mitigate this possibility, it is common to run multiple instances of the config server in different racks / zones behind a load balancer.

4. What happens if we change a property after client applications have started? The server picks up the changes immediately, but the client does not. Later we will see how Spring Cloud Bus and refresh scope can be used to dynamically propagate changes.
