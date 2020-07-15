### Creating A Simple Route
The Spring Cloud Gateway uses routes in order to process requests to downstream services. In this guide we will route all of our requests to HTTPBin. Routes can be configured a number of ways but for this guide we will use the Java API provided by the Gateway.

To get started, create a new `Bean` of type `RouteLocator` in `Application.java`.

`src/main/java/gateway/Application.java`

```copy
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes().build();
}
```

The above `myRoutes method takes in a `RouteLocatorBuilder` which can easily be used to create routes. In addition to just creating routes, `RouteLocatorBuilder` allows you to add predicates and filters to your routes so you can route handle based on certain conditions as well as alter the request/response as you see fit.

Let’s create a route that routes a request to `https://httpbin.org/get` when a request is made to the Gateway at `/get`. In our configuration of this route we will add a filter that will add the request header `Hello` with the value `World` to the request before it is routed.

`src/main/java/gateway/Application.java`

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

```execute
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