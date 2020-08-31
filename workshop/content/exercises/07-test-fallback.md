To test this new fallback functionality, stop the running terminal.

```terminal:interrupt-all
```

Restart your application

```execute-1
mvn spring-boot:run
```

Once the terminal states the application is running again, issue the following cURL command.

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