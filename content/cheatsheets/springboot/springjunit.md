---
title: SpringBoot Junit
linktitle: SpringBoot Junit
type: book
date: "2019-05-05T00:00:00+01:00"
tags:
  - SpringBoot

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 10
---

Automated Testing with SpringBoot
<!--more-->

### Overview

Testing is a critical component of the software development cycle. A good set of tests as part of your build pipeline enables your team to:

* Validate that the software meets its goals.
* Search for defects that can be fixed to improve software quality.
* Facilitate refactoring and upgrades by validating that everything is still working after the changes are applied.

This cheatsheet provides an overview of the role and structure of testing and the **Spring** :yum: of it. It does not attempt to teach you how to create a complete and excellent set of tests for your project; if you want to learn more about good testing practices, the end of this page I have some suggested resources to checkout.

Spring Boot offers great support to test different slices (web, database, etc.) of your application. This allows you to write tests for specific parts of your application in isolation without bootstrapping the whole Spring Context. Technically this is achieved by creating a Spring Context with only a subset of beans by applying only specific auto-configurations.

Before we dive into the SpringBoot testing features here's a bit of TLDR :smile:

### Automated testing
Testing should be automated as much as possible, based on the following principles and practices:
* Tests can be run frequently and always in the same order.
* Running tests frequently means that problems are found early and you usually know which small piece of code caused the problem.
* Automated tests consume machine resources but require little human time beyond what is required to review the test results.
* Tests should be independent from each other as much as possible.
* Many tests can be run in parallel—​especially tests that validate your code for different operating systems or JDK versions.
* Define different tests to run at different stages of the build chain.

### Categories of testing
The testing field has identified different categories of test types; you can find long discussions about the proper definitions of all these types of tests.

Test types can be categorized by how quickly they run. Faster automated test types include:

* **Unit tests** test a small piece of code (a function, method, or command). They run the fastest and are often written by the person who writes the code.
* **Integration tests** validate integration between multiple subsystems, including external subsystems such as a database.
* **Smoke tests** (also known as sanity checking) validate basic functions of the system.

Slower automated test types include:

* **Functional tests** validate the normal software behaviors against the expectations and requirements.
* **Non-regression tests** validate that the system still produces the same result.
* **Acceptance tests** test the full product from the perspective of the end user use cases and sentiment. These tests usually include manual testing.

Manual testing should be performed rarely, and only on software that has passed all automated tests. It is appropriate when the test result is subjective, such as user experience testing, and when the cost of automation is excessive.

### Testing Pyramid

Jenkins enables you to run large numbers of tests frequently and at appropriate stages in the build cycle.
Your testing portfolio should have more low-level tests than high-level tests.
* Unit tests usually run every time you compile the code.
* You can define whether functional and non-regression tests run if the unit tests fail.
* Large, broad tests can be set up to run periodically (for example, during non-work hours) rather than being run each time new code is committed.

The Testing Pyramid is a visual representation of these principles:

![](/images/uploads/springboot-junit-testing-pyramid.PNG)

Therefore, the following principles should be reflected in your testing portfolio:

* The low-level tests at the bottom of the pyramid run quickly and inexpensively and should be run very frequently.
* The higher-level tests at the top of the pyramid take more time to run and are expensive; they should be run less frequently and only on software that has passed all the tests that are lower on the pyramid.
* When low-level tests fail, it seldom makes sense to run higher-level tests before fixing the problems detected by the low-level tests.
* When a higher-level test fails, consider that it may have detected a defect in the lower-level tests as well as a defect in the code.

### JUnit5

JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage
• JUnit Platform: Foundation to launch testing frameworks & it defines the TestEngine API for developing testing frameworks
• JUnit Jupiter: New programming model and extension model and provides the JupiterTestEngine that implements the TestEngine interface to
run tests on the JUnit Platform
• JUnit Vintage: Provides a TestEngine to run both JUnit 3 and JUnit 4 tests

{{% callout note %}}
As of Spring Boot 2.4.0, the spring-boot-starter-test no longer includes the JUnit Vintage Engine but does include the JUnit Jupiter
{{% /callout %}}

So basically you got your Swiss Army Knife with the below testing dependency for all your testing needs.

### Unit Testing

We should be able to write unit tests for UserService WITHOUT using any Spring features.
We are going to create a mock repository using Mockito.mock() and create Service instance using the mock repository instance.

### Component Testing

#### Web Layer

Using @WebMvcTest annotation, you'll get a Spring Context that includes components required for testing Spring MVC parts of your application.

What's part of the Spring Test Context: @Controller, @ControllerAdvice, @JsonComponent, @Converter, @Filter, @WebMvcConfigurer.

What's not part of the Spring Test Context: @Service, @Component, @Repository beans

Furthermore, there is also great support if you secure your endpoints with Spring Security. The annotation will auto-configure your security rules, and if you include the Spring Security Test dependency, you can easily mock the authenticated user. As this annotation provides a mocked servlet environment, there is no port to access your application with, e.g., a RestTemplate. Therefore, you rather use the auto-configured MockMvc to access your endpoints:

```java
@WebMvcTest(ShoppingCartController.class)
class ShoppingCartControllerTest {


  @Autowired
  private MockMvc mockMvc;

  @MockBean
  private ShoppingCartRepository shoppingCartRepository;

  @Test
  public void shouldReturnAllShoppingCarts() throws Exception {
    when(shoppingCartRepository.findAll()).thenReturn(
      List.of(new ShoppingCart("42",
        List.of(new ShoppingCartItem(
          new Item("MacBook", 999.9), 2)
        ))));

    this.mockMvc.perform(get("/api/carts"))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$[0].id", Matchers.is("42")))
      .andExpect(jsonPath("$[0].cartItems.length()", Matchers.is(1)))
      .andExpect(jsonPath("$[0].cartItems[0].item.name", Matchers.is("MacBook")))
      .andExpect(jsonPath("$[0].cartItems[0].quantity", Matchers.is(2)));
  }
}
```

#### JPA Components

With @DataJpaTest you can test any JPA-related parts of your application. A good example is to verify that a native query is working as expected.

What's part of the Spring Test Context: @Repository, EntityManager, TestEntityManager, DataSource

What's not part of the Spring Test Context: @Service, @Component, @Controller beans

By default, this annotation tries to auto-configure use an embedded database (e.g., H2) as the DataSource:

```java
@DataJpaTest
class BookRepositoryTest {

  @Autowired
  private DataSource dataSource;

  @Autowired
  private EntityManager entityManager;

  @Autowired
  private BookRepository bookRepository;

  @Test
  public void testCustomNativeQuery() {
    assertEquals(1, bookRepository.findAll().size());

    assertNotNull(dataSource);
    assertNotNull(entityManager);
  }
}
```

While an in-memory database might not be a good choice to verify a native query using proprietary features, you can disable this auto-configuration with:

@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
and use, e.g., Testcontainers to create a PostgreSQL database for testing. This would be more akin to Integration testing and we would discuss that in a following section.

In addition to the auto-configuration, all tests run inside a transaction and get rolled back after their execution.

#### Cross Cutting concerns

### Integration Testing

#### TestContainers

[TestContainers](https://www.testcontainers.org/) is a library for easily using Docker containers directly in your JUnit Test.
Their team has made some pre-built containers for common services like say for eg. a MySQL database.

Here's a snippet from their mission which aptly describes their purpose:

{{< hl >}} Testcontainers is a Java library that supports JUnit tests, providing lightweight, throwaway instances of common databases, Selenium web browsers, or anything else that can run in a Docker container.{{< /hl >}}

#### SpringCloud Contract

### Acceptance Testing

### Further Read

Software testing is a vast subject with many excellent courses, books, and articles available. Here are a few articles to get you started:

* The stackoverflow [What are unit tests, integration tests, smoke tests, and regression tests?](https://stackoverflow.com/questions/520064/what-are-unit-tests-integration-tests-smoke-tests-and-regression-tests) discussion introduces the types of testing you can perform.
* [Wikipedia article about Software Testing](https://en.wikipedia.org/wiki/Software_testing) provides a comprehensive summary and bibliography about types of testing and tools to use.
* Martin Fowler writes extensively about software development, and proper testing figures prominently in his writing. To get started, read the articles on his [tagged by: testing page](https://martinfowler.com/tags/testing.html). You may also enjoy his articles about [Unit testing](https://martinfowler.com/bliki/UnitTest.html), [Test Coverage](https://martinfowler.com/bliki/TestCoverage.html), and [Test Driven Development](https://martinfowler.com/bliki/TestDrivenDevelopment.html)

If you want to further explore the fine points of software testing, Black Box Software Testing (BBST) by Cem Kaner offers a series of four six-week courses about testing. Each course contains video lectures and exams:

* [About the Black Box Software Testing Courses](https://www.associationforsoftwaretesting.org/bbst-black-box-software-testing-courses/)
* [BBST Courses Offering at Altom](https://bbst.courses/#courses)
