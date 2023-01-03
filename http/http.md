## Close HTTP Response Body
The most common why to close the response body is by using a defer call after the http response error check.

```go
package main

import (  
    "fmt"
    "net/http"
    "io/ioutil"
)

func main() {  
    resp, err := http.Get("https://api.ipify.org?format=json")
	// DO NOT CLOSE HERE as response could be bil - defer resp.Body.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    defer resp.Body.Close()// OK
	
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println(err)
        return
    }

    fmt.Println(string(body))
}
```

Most of the time when your http request fails the resp variable will be nil and the err variable will be non-nil. 

However, **when you get a redirection failure both variables will be non-nil. This means you can still end up with a leak.**


You can fix this leak by adding a call to close non-nil response bodies in the http response error handling block. Another option is to use one defer call to close response bodies for all failed and successful requests.
```go
func main() {  
    resp, err := http.Get("https://api.ipify.org?format=json")
    if resp != nil {
        defer resp.Body.Close()
    }

    if err != nil {
        fmt.Println(err)
        return
    }

    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

## Closing HTTP Connection
Some HTTP servers keep network connections open for a while (based on the HTTP 1.1 spec and the server "keep-alive" configurations). By default, the standard http library will close the network connections only when the target HTTP server asks for it. 
This means your app may run out of sockets/file descriptors under certain conditions.

You can ask the http library to close the connection after your request is created by setting the Close field in the request variable to true.
Another option is to add a Connection request header and set it to close. The target HTTP server should respond with a Connection: close header too. When the http library sees this response header it will also close the connection.
```go
req, err := http.NewRequest("GET","http://golang.org",nil)
    if err != nil {
        fmt.Println(err)
        return
    }

    req.Close = true
    //or do this:
    //req.Header.Add("Connection", "close")

    resp, err := http.DefaultClient.Do(req)
    if resp != nil {
        defer resp.Body.Close()
    }
```

You can also disable http connection reuse globally. You'll need to create a custom http transport configuration for it.
```go
 tr := &http.Transport{DisableKeepAlives: true}
    client := &http.Client{Transport: tr}

    resp, err := client.Get("http://golang.org")
    if resp != nil {
        defer resp.Body.Close()
    }
```
If you send a lot of requests to the same HTTP server it's ok to keep the network connection open. 
However, if your app sends one or two requests to many different HTTP servers in a short period of time it's a good idea to close the network connections right after your app receives the responses.

