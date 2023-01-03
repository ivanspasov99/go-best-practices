# Testing HTTP Handlers


## HTTPTEST

Using `httptest` library to satisfy handler interface `handler func(ResponseWriter, *Request)`
```go
// creating new request...

// this setup "mock" recorder writer
// this would not validate the routing of the http handler
// it would just call the handler with function you can test your function
rec := httptest.NewRecorder()

// could init struct with state if required
handleFunc(rec, request)
```

Another way is "running" server
```go
func main() {
	// ignore main - should be testing func
	httptest.NewServer(handler())
}

func handler() http.Handler {
	r := http.NewServeMux()

	// could init custom struct handlerFunc

	r.HandleFunc("/handle", handleFunc)
	return r
}

func handleFunc(w http.ResponseWriter, r *http.Request) {
	fmt.Println("hello, i am handler")
}
```