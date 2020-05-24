---
title: Spring Boot testing
---
Recently I looked into the different options to write tests for Spring Boot applications.
As a summary, the following patterns can be used:

- Bare bones Unit Test: No Spring involved. Mocking with Mockito
- WebMvcTest: Only WebMvc components are active. @Component, @Service or @Repository beans can not be used, these must be mocked.
- Integration Test without server: All beans available. MockMvc used to make HTTP calls.
- Integration Test with server: All beans available. Make HTTP calls using TestRestTemplate or similar.

To get a better understanding of the Spring Context initialisation during test execution, the [JUnit insights extension](https://github.com/adessoAG/junit-insights) was very helpful.

For details and examples of all four options, see [https://github.com/adrianbaertschi/spring-boot-testing-insights](https://github.com/adrianbaertschi/spring-boot-testing-insights)
