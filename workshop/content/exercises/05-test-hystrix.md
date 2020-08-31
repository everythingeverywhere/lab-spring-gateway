Lets test this new route, stop the running terminal.

```terminal:interrupt-all
```

Restart your application

```execute-1
mvn spring-boot:run
```

This time you are going to make a request to /`delay/3`. You must include a `Host` header that has the a host of `hystrix.com` or else the request won’t be routed. In cURL this would look like:

Execute the following when your application starts:

```execute-2
curl --dump-header - --header 'Host: www.hystrix.com' http://localhost:8080/delay/3
```

> We are using `--dump-header` to see the response headers, the `-` after `--dump-header` is telling cURL to print the headers to stdout.

After executing this command you should see something similar to the following in your terminal:
```
HTTP/1.1 504 Gateway Timeout
```

As you can see Hystrix timed out waiting for the response from HTTPBin. When Hystrix times out we can provide a fallback so that clients do not just receive a `504` but rather something more meaninful. In a production scenario you may return some data from a cache for example, but in our simple example we will return a response with the body `fallback` instead.
