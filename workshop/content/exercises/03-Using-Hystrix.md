Using Hystrix
Now lets do something a little more interesting. Since the services behind the Gateway could potentially behave poorly effecting our clients we might want to wrap the routes we create in circuit breakers. You can do this in the Spring Cloud Gateway using Hystrix. This is implemented via a simple filter that you can add to your requests. Lets create another route to demonstrate this.

In this example we will leverage HTTPBin’s delay API that waits a certain number of seconds before sending a response. Since this API could potentially take a long time to send its response we can wrap the route that uses this API in a HystrixCommand. Add a new route to our RouteLocator object that looks like the following

src/main/java/gateway/Application.java
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
            .filters(f -> f.hystrix(config -> config.setName("mycmd")))
            .uri("http://httpbin.org:80")).
        build();
}
```

There are some differences between this new route configuration and the previous one we created. For one, we are using the host predicate instead of the path predicate. This means that as long as the host is `hystrix.com` we will route the request to HTTPBin and wrap that request in a `HystrixCommand`. We do this by applying a filter to the route. The Hystrix filter can be configured using a configuration object. In this example we are just giving the `HystrixCommand` the name mycmd.

Lets test this new route. Start the application, but this time we are going to make a request to /`delay/3`. It is also important that we include a `Host` header that has the a host of `hystrix.com` or else the request won’t be routed. In cURL this would look like

```copy
curl --dump-header - --header 'Host: www.hystrix.com' http://localhost:8080/delay/3
```

> We are using `--dump-header` to see the response headers, the `-` after `--dump-header` is telling cURL to print the headers to stdout.

After executing this command you should see the following in your terminal

```copy
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

In `Application.java` add the class level annotation `@RestController`, then add the following `@RequestMapping` to the class.

`src/main/java/gateway/Application.java`

```copy
@RequestMapping("/fallback")
public Mono<String> fallback() {
  return Mono.just("fallback");
}
```

To test this new fallback functionality, restart the application and again issue the following cURL command

```copy
$ curl --dump-header - --header 'Host: www.hystrix.com' http://localhost:8080/delay/3
```

With the fallback in place, we now see that we get a `200` back from the Gateway with the response body of `fallback`.

```copy
HTTP/1.1 200 OK
transfer-encoding: chunked
Content-Type: text/plain;charset=UTF-8

fallback
```