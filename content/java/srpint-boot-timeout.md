---
title: "How to setup Request timeout in Spring Boot REST API"
date: 2021-06-27T18:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
#### Timeout using @Transactional annotation

To set request timeout on database queries or calls by using Spring's @Transactional annotation. We may set the timeout attribute that it has. It has a default value of -1, which is the same as having no timeout at all. 

Let's say we have set this timeout to 50 seconds. An exception will be raised if the annotated method takes longer than this amount of time to execute. This approach is quite handy for time-consuming database searches or long-running SQL queries.
<!--more-->
Let’s see it in action, we will create a simple rest endpoint that returns the student's details as shown below.

```
@RequestMapping(value = "/students", method = RequestMethod.GET)
@Transactional(timeout = 10) 
public ResponseEntity<List<Student>> getAllStudents(){ 

    return new ResponseEntity<>(studentRepoCustom.getStudents(), HttpStatus.OK); 
}

```
In thie example, endpoint /students are annotated with @Transactional with a custom timeout of 10 seconds. This means the endpoint has to respond within 10 seconds before it throws an error. 
Note: It is obvious, the endpoint /students will return a 500 error message due to the timeout limit as it requires a response within 10 seconds where as the student service will take at least 15 seconds to respond.

![image](https://blogger.googleusercontent.com/img/a/AVvXsEgu8vDx8Kl4DkJm3buujNWdoE-4-wy8Pdv2qXnuqCqTND-t3wcWZGK1VRss5O3xSr9BJlmOoyf56gqaDR_SHI1g4KuwnWGx2b76gikO72Eg4k7MRYrtiTPY_JufM_DXZ8tU0k_WN1esX3z7wTQgLDu4fbXVSnUQnW2gNCX6c-boWR17pD-vA5TSegh6=w557-h344)

**Error Response**
org.springframework.transaction.TransactionTimedOutException: `Transaction timed out: deadline was Sat Mar 04 19:26:04 GMT 2021`

#### Using WebClient timeout
Spring introduces the WebFlux framework, which supports reactive programming. To make HTTP requests we use the WebClient interface. It is a reactive client that doesn't block when making HTTP requests. Its default underlying HTTP client library is Reactor Netty. 

It allows us to set a timeout on a single external call rather than setting it on an entire endpoint. Although the majority of developers now use WebClient over RestTemplate, the older RestTemplate object from Spring may still have timeouts configured.

Let’s see WebClient in action with the help of an example.

Add WebFlux dependency in your project
```
<dependency>

    <groupId>org.springframework.boot</groupId>

    <artifactId>spring-boot-starter-webflux</artifactId>

    <version>3.0.4</version>

</dependency>
```
The example below shows how a WebClient request to a reactive downstream endpoint would typically appear.

```
webClient.get()
         .uri(uri)
         .retrieve()
         .onStatus(HttpStatus::isError, clientResponse -> { 
            LOGGER.error("Error while calling endpoint {} with status code {}",
            uri.toString(), clientResponse.statusCode()); 
            throw new RuntimeException("Error while calling accounts endpoint"); 
        }).bodyToMono(JsonNode.class) 
        .timeout(Duration.ofMillis(timeout));
```
**Timeout**

The time we wait after requesting a response is known as the response timeout. The responseTimeout() function may be used to set it up for the client.

Let's create a WebClient with a base URL that allows us to call oneself using localhost and a response timeout of 10 seconds

```
@Bean public WebClient webClient() { 
    return WebClient.builder() 
    .baseUrl("http://localhost:8080/api/v1") 
    .clientConnector(new ReactorClientHttpConnector( 
        HttpClient.create().responseTimeout(Duration.ofSeconds(10)) 
    )) 
    .build(); 
}

```
We can see that after contacting this endpoint, we do indeed obtain a 500 HTTP error response indicating that the WebClient has timed out. The downstream @Transactional timeout can also be seen in the logs, but if we requested an external service rather than localhost, its timeout would be reported remotely.

With this method, it is possible to configure different request timeouts for various backend services, which may be required. Also, there are other techniques for managing errors in the Mono or Flux response that publishers get from WebClient in response to a general timeout issue. 

#### Spring MVC timeout
There is another way to set a timeout in Spring Boot is by setting up the spring mvc property as mentioned below.

```
spring.mvc.async.request-timeout=milliseconds-precision
```
This allows us to define request timeout in milliseconds precision. It is to note here that, one can set this property externally, the property applies to the endpoints that return a Callable. 

For demonstration, we have defined an endpoint that further calls the student’s service class and fetches the record.

```
@GetMapping("/mvc-timeout") public Callable<List<Student>> getWithMvcRequestTimeout() { return () -> { return studentRepoCustom.getStudents(); }; }

```
When we declare the application property, Spring immediately implements the configuration. When the timeout is reached, the response is promptly returned, and an explicit 503 HTTP error is given in place of a more general 500 error. This timeout option will be automatically inherited by all of the endpoints in our app.
