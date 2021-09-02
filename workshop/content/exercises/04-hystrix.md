Now lets do something a little more interesting. Since the services behind the Gateway could potentially behave poorly effecting our clients we might want to wrap the routes we create in Circuit Breakers. You can do this in the Spring Cloud Gateway using Hystrix. This is implemented via a simple filter that you can add to your requests. Lets create another route to demonstrate this.

In this example we will leverage HTTPBin’s delay API that waits a certain number of seconds before sending a response. Since this API could potentially take a long time to send its response we can wrap the route that uses this API in a HystrixCommand. Add a new route to our RouteLocator object that looks like the following:

`src/main/java/gateway/Application.java`

```editor:insert-lines-before-line
file: ~/gs-gateway/initial/src/main/java/gateway/Application.java
line: 24
text: |
        // Step 5. Hystrix Circuit Breaker
        .route(p -> p
            .host("*.hystrix.com")
            .filters(f -> f.circuitBreaker(config -> config
                .setName("mycmd")
                )) 
            .uri("http://httpbin.org:80"))
```

There are some differences between this new route configuration and the previous one we created. For one, we are using the `host` predicate instead of the `path` predicate. This means that as long as the host is `hystrix.com` we will route the request to HTTPBin and wrap that request in a `HystrixCommand`. We do this by applying a filter to the route. The Hystrix filter can be configured using a configuration object. In this example you gave the `HystrixCommand` the name mycmd.

---

At this point your `Application.java` has the following:

```copy
package gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;

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
                )) 
            .uri("http://httpbin.org:80"))

// ****- Other routes -****
        .build();
}
}
```