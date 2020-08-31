To test our very simple Gateway, just run `Application.java`, it should be run on port `8080`. 

```execute-1
mvn spring-boot:run
```

**Once the application is running**, make a request to `http://localhost:8080/get`. You can do this using cURL by issuing the following command in your terminal.

```execute-2
 curl http://localhost:8080/get
``` 
You should receive a response that looks like the following:

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