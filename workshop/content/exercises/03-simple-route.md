The Spring Cloud Gateway uses routes in order to process requests to downstream services. In this guide we will route all of our requests to HTTPBin. Routes can be configured a number of ways but for this guide we will use the Java API provided by the Gateway.

To get started, we will create a new `Bean` of type `RouteLocator` in `Application.java`.


This `Bean` will  live in `src/main/java/gateway/Application.java`. In the example bellow, `myRoutes` method takes in a `RouteLocatorBuilder` which can be used to create routes. 
```
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes().build();
}
```

In addition to just creating routes, the `RouteLocatorBuilder` allows you to add predicates and filters to your routes so you can route handle based on certain conditions as well as alter the request/response as you see fit.


Now, let's create a route that when a request is made to the Gateway at `/get` the request gets sent to `https://httpbin.org/get`. We will include a filter to add the request header `Hello` with the value `World` to the request before it is routed.

In your `Application.java` found in`src/main/java/gateway/Application.java` import `Bean`, `RouteLocator`, and `RouteLocatorBuilder`. Place them below the other import statements.

```copy
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
```

Now, add your `myRoutes` method inside of the `Application` class already in the file bellow `public static void main`.

```copy
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p
            .path("/get")
            .filters(f -> f.addRequestHeader("Hello", "World"))
            .uri("http://httpbin.org:80"))
        .build();
}
```

To test our very simple Gateway, just run `Application.java`, it should be run on port `8080`. Once the application is running, make a request to `http://localhost:8080/get`. You can do this using cURL by issuing the following command in your terminal.

```execute-2
 curl http://localhost:8080/get
``` 
You should receive a response back that looks like this

```
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Connection": "close",
    "Forwarded": "proto=http;host=\"localhost:8080\";for=\"0:0:0:0:0:0:0:1:56207\"",
    "Hello": "World",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.54.0",
    "X-Forwarded-Host": "localhost:8080"
  },
  "origin": "0:0:0:0:0:0:0:1, 73.68.251.70",
  "url": "http://localhost:8080/get"
}
```
Notice that HTTPBin shows that the header `Hello` with the value `World` was sent in the request.