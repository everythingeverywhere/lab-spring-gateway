#### Writing Tests

As a good developer, we should write some tests to make sure our Gateway is doing what we expect it should. In most cases we want to limit out dependencies on outside resources, especially in unit tests, so we should not depend on HTTPBin. One solution to this problem is to make the URI in our routes configurable, so we can easily change the URI if we need to.

In `Application.java` create a new class called `UriConfiguration`.

```copy
@ConfigurationProperties
class UriConfiguration {

  private String httpbin = "http://httpbin.org:80";

  public String getHttpbin() {
    return httpbin;
  }

  public void setHttpbin(String httpbin) {
    this.httpbin = httpbin;
  }
}
```

To enable this `ConfigurationProperties` we need to also add a class-level annotation to `Application.java`.

```copy
@EnableConfigurationProperties(UriConfiguration.class)
```

With our new configuration class in place lets use it in the `myRoutes` method.

`src/main/java/gateway/Application.java`

```copy
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder, UriConfiguration uriConfiguration) {
  String httpUri = uriConfiguration.getHttpbin();
  return builder.routes()
    .route(p -> p
      .path("/get")
      .filters(f -> f.addRequestHeader("Hello", "World"))
      .uri(httpUri))
    .route(p -> p
      .host("*.hystrix.com")
      .filters(f -> f
        .hystrix(config -> config
          .setName("mycmd")
          .setFallbackUri("forward:/fallback")))
      .uri(httpUri))
    .build();
}
```

As you can see, instead of hardcoding the URL to HTTPBin we are getting the URL from our new configuration class instead.

Below is the complete contents of the class `Application` below import statements.

`src/main/java/gateway/Application.java`

```copy
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;


@SpringBootApplication
@EnableConfigurationProperties(UriConfiguration.class)
@RestController
public class Application {

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

  @Bean
  public RouteLocator myRoutes(RouteLocatorBuilder builder, UriConfiguration uriConfiguration) {
    String httpUri = uriConfiguration.getHttpbin();
    return builder.routes()
      .route(p -> p
        .path("/get")
        .filters(f -> f.addRequestHeader("Hello", "World"))
        .uri(httpUri))
      .route(p -> p
        .host("*.hystrix.com")
        .filters(f -> f
          .hystrix(config -> config
            .setName("mycmd")
            .setFallbackUri("forward:/fallback")))
        .uri(httpUri))
      .build();
  }

  @RequestMapping("/fallback")
  public Mono<String> fallback() {
    return Mono.just("fallback\n");
  }
}

@ConfigurationProperties
class UriConfiguration {

  private String httpbin = "http://httpbin.org:80";

  public String getHttpbin() {
    return httpbin;
  }

  public void setHttpbin(String httpbin) {
    this.httpbin = httpbin;
  }
}
```

Create a new class called `ApplicationTest` in `src/main/test/java/gateway`. 

```execute-2
mkdir -p ./src/main/test/java/gateway &&  
touch ./src/main/test/java/gateway/ApplicationTest.java
```
>
> Note: **You may need to reload the editor to see these files,  use the button next to the lab timer to refresh.**

In the new class add the following content.

```copy
package gateway;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
    properties = {"httpbin=http://localhost:${wiremock.server.port}"})
@AutoConfigureWireMock(port = 0)
public class ApplicationTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  public void contextLoads() throws Exception {
    //Stubs
    stubFor(get(urlEqualTo("/get"))
        .willReturn(aResponse()
          .withBody("{\"headers\":{\"Hello\":\"World\"}}")
          .withHeader("Content-Type", "application/json")));
    stubFor(get(urlEqualTo("/delay/3"))
      .willReturn(aResponse()
        .withBody("no fallback")
        .withFixedDelay(3000)));

    webClient
      .get().uri("/get")
      .exchange()
      .expectStatus().isOk()
      .expectBody()
      .jsonPath("$.headers.Hello").isEqualTo("World");

    webClient
      .get().uri("/delay/3")
      .header("Host", "www.hystrix.com")
      .exchange()
      .expectStatus().isOk()
      .expectBody()
      .consumeWith(
        response -> assertThat(response.getResponseBody()).isEqualTo("fallback".getBytes()));
  }
}
```

Our test is actually taking advantage of WireMock from Spring Cloud Contract in order stand up a server that can mock the APIs from HTTPBin. The first thing to notice is the use of `@AutoConfigureWireMock(port = 0)`. This annotation will start WireMock on a random port for us.

Next notice that we are taking advantage of our `UriConfiguration` class and setting the `httpbin` property in the `@SpringBootTest` annotation to the WireMock server running locally. Within the test we then setup "stubs" for the HTTPBin APIs we call via the Gateway and mock the behavior we expect. Finally we use `WebTestClient` to actually make requests to the Gateway and validate the responses.

Start your Spring Gateway application:

```execute-1
mvn spring-boot:run
```

Start your test server to execute your tests:

```execute-2
mvn test
```