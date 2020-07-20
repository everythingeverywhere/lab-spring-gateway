Lets test this new route start the application:
```execute-1
mvn compile && \
mvn package && \
java -jar target/gs-gateway-0.1.0.jar 
```

This time you are going to make a request to /`delay/3`. You must include a `Host` header that has the a host of `hystrix.com` or else the request won’t be routed. In cURL this would look like

```execute-2
curl --dump-header - --header 'Host: www.hystrix.com' http://localhost:8080/delay/3
```

> We are using `--dump-header` to see the response headers, the `-` after `--dump-header` is telling cURL to print the headers to stdout.

After executing this command you should see the following in your terminal:
```
HTTP/1.1 504 Gateway Timeout
content-length: 0
```

As you can see Hystrix timed out waiting for the response from HTTPBin. When Hystrix times out we can optionaly provide a fallback so that clients do not just received a `504` but something more meaninful. In a production scenario you may return some data from a cache for example, but in our simple example we will just return a response with the body `fallback` instead.

To do this, lets modify our Hystrix filter to provide a URL to call in the case of a timeout.

`src/main/java/gateway/Application.java`

```copy
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p
            .path("/get")
            .filters(f -> f.addRequestHeader("Hello", "World"))
            .uri("http://httpbin.org:80"))
        .route(p -> p
            .host("*.hystrix.com")
            .filters(f -> f.hystrix(config -> config
                .setName("mycmd")
                .setFallbackUri("forward:/fallback")))
            .uri("http://httpbin.org:80"))
        .build();
}
```

Now when the Hystrix wrapped route times out it will call `/fallback` in the Gateway app. Lets add the `/fallback` endpoint to our application.

In `Application.java` import the `Mono` reactive stream,  a `@RestController` and `@RequestMapping` to the class under the other import statements.

```copy
import reactor.core.publisher.Mono;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
```

Place the annotation `@RestController` in the line above `public class Application {` such as:

``` 
@SpringBootApplication
@RestController
public class Application {
```

```copy
@RestController
```

Next add the following below the `myRoutes` function in 
`src/main/java/gateway/Application.java`

```copy
@RequestMapping("/fallback")
public Mono<String> fallback() {
  return Mono.just("fallback");
}
```

To test this new fallback functionality, restart the application and again issue the following cURL command

```execute-2
curl --dump-header - --header 'Host: www.hystrix.com' http://localhost:8080/delay/3
```

With the fallback in place, we now see that we get a `200` back from the Gateway with the response body of `fallback`.

```
HTTP/1.1 200 OK
transfer-encoding: chunked
Content-Type: text/plain;charset=UTF-8

fallback
```