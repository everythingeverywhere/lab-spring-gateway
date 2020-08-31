To return a response with the body `fallback` instead of `504` modify our Hystrix filter to provide a URL to call in the case of a timeout.

`src/main/java/gateway/Application.java`

```editor:append-lines-after-match
file: ~/gs-gateway/initial/src/main/java/gateway/Application.java
match: .setName("mycmd")
text: |
        // Step 7. uri=/fallback
        .setFallbackUri("forward:/fallback")
```

Now when the Hystrix wrapped route times out it will call `/fallback` in the Gateway app. Lets add the `/fallback` endpoint to our application.

In `Application.java` import the `Mono` reactive stream,  a `@RestController` and `@RequestMapping` to the class under the other import statements.

```editor:insert-lines-before-line
file: ~/gs-gateway/initial/src/main/java/gateway/Application.java
line: 9
text: |
        import reactor.core.publisher.Mono;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RestController;


```

Place the annotation `@RestController` in the line above `public class Application {` such as:

```editor:insert-lines-before-line
file: ~/gs-gateway/initial/src/main/java/gateway/Application.java
line: 12
text: |
        @RestController
```

Next add the following below your `myRoutes` function in 
`src/main/java/gateway/Application.java`

> Note: adding \n for readability in response

```editor:insert-lines-before-line
file: ~/gs-gateway/initial/src/main/java/gateway/Application.java
line: 41
text: |
        @RequestMapping("/fallback")
        public Mono<String> fallback() {
        return Mono.just("fallback\n");
        }
```

The entire contents of `Application.java` should be:

```copy
package gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;

import reactor.core.publisher.Mono;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

// Step 3. myRoutes a simple route method
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p
            .path("/get")
            .filters(f -> f.addRequestHeader("Hello", "World"))
            .uri("http://httpbin.org:80"))
// Step 5. Hystrix Circuit Breaker
.route(p -> p
            .host("*.hystrix.com")
            .filters(f -> f.hystrix(config -> config
                .setName("mycmd")
// Step 7. uri=/fallback
.setFallbackUri("forward:/fallback")
                )) 
            .uri("http://httpbin.org:80"))

// ****- Other routes -****
        .build();
}
@RequestMapping("/fallback")
public Mono<String> fallback() {
return Mono.just("fallback\n");
}
}

```