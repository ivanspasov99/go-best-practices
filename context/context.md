# Context Cancellation & Propagation and Values
You should be familiar with goroutines and channels before checking this guideline
When you are requesting sandwich to a person and then you said actually never-mind. He should stop whatever it have done and clean the mess

1. [Simple Example](#simple-example)
2. [HTTP Handler](#http-server)
3. [Client Side](#client-context-handler)
4. [Context Values - Middlewares - Logging](#context-values)

*Usages*
- When request is canceled
- When you want to be finished something within timeline (eventually stucked resoure)
- When you use goroutines, and you want to notify them that they should finish what they do (in select statement as below)


## Simple Example

### Building Blocks 
- `context` - could have values in it, or request scoped data (often is used for logging)
- `cancel()` function which triggers cancellation
- you should have mechanism for receiving and handling cancellation event

Let's introduce a sleepAndTalk function
```go
func sleepAndTalk(ctx context.Context, d time.Duration, s string) {
	// channel select
	select {
	// unblock after period of time and execute logic
	case <-time.After(d):
		fmt.Println(s)
	// unblock if context is canceled	
	case <-ctx.Done():
		log.Println(ctx.Err())
	}
}
```

#### WithTimeout
```go
func main() {
	// WithTimeout
	ctx := context.Background()
	// read WithTimeoutDocs 
	// Use it if you want to cancel something within timeout
	ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
	// 
	defer cancel()

	// should print hello after 10s if context is not canceled 
	sleepAndTalk(ctx, 10*time.Second, "hello")
```

#### WithCancel
```go
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	
	// if we type something we are going to cancel the sleepAndTalk using the cancel()
	go func() {
		s := bufio.NewScanner(os.Stdin)
		s.Scan()
		cancel()
	}()
	
	// call cancel after time
	// time.AfterFunc(5*time.Second, cancel)
	
	// The initialised context with the cancel function should be passed
	// should finish on 10 second
	sleepAndTalk(ctx, 10*time.Second, "hello")
```

## HTTP Server
What do you do when the client cancel the request?

### Server 
```go

func main() {
	// server
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe("localhost:8080", nil))
}
```

### Handler
```go
func handler(writer http.ResponseWriter, request *http.Request) {
	// request context
	ctx := request.Context()

	log.Println("handler started")
	// just to check when the handler finished its job
	defer log.Println("handler ended")

	select {
	// wait for 5 seconds before sending response
	case <-time.After(5 * time.Second):
		fmt.Fprintln(writer, "hello i am message from server to client")
	// context cancel - you can try to call it with curl to have the option for canceling
	// when user cancel the request you will execute logic
	case <-ctx.Done():
		err := ctx.Err()
		log.Println(err)
		http.Error(writer, err.Error(), http.StatusInternalServerError)
	}
}
```

## Client Context Handler
What if the Server is taking long to response? You can handle manage the request and cancel that inside the client code

You can not send values from client to server through context as they are not propagated in the request.

You can add context on the client side too
```go
func clientCall() {
	ctx := context.Background()

	req, err := http.NewRequest(http.MethodGet, "http://localhost:8080", nil)
	if err != nil {
		log.Fatal(err)
	}
    
	// if the request is taking more than 2 second we are going to cancel it
	ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
	defer cancel()

	// define request with timeout context 
	req = req.WithContext(ctx)
	
	// context cancellation result will be returned as error
	res, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer res.Body.Close()
	if res.StatusCode != http.StatusOK {
		log.Fatal(res.StatusCode)
	}
	io.Copy(os.Stdout, res.Body)
}
```

## Context Values
#### Rules:
- Whatever in the context should be request specific
- Whatever in the context should be useful for your program but should not be impact the flow

#### What we can use
- RequestID
- Monitoring Details

#### How to do it
Using middleware pattern

##### Logging Package

```go

// with custom type definition we have limit the option someone to override the key type with another value
// that is called collision
// The `key` will be available only in the log package, so no one will have access to such type 
type key int
const requestIDKey = key(40)

// Println prints message with id
// should change the standard logging function 
func Println(ctx context.Context, msg string) {
    id, ok := ctx.Value(requestIDKey).(int)
	if !ok {
		log.Println("could not find request id in context")
	}
	log.Printf("[%d] %s", id, msg)
}

// Decorate is used as middleware for logging id of the request
func Decorate(f http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		ctx := r.Context()
		id := rand.Int()
		ctx = context.WithValue(ctx, requestIDKey, id)
		f(w, r.WithContext(ctx))
	}
}
```

##### Handler Middleware
```go
func main() {
	// server
	// we are going to decorate(do a logging middleware) for loggin
	// Applying Decorator and Middleware (chain of responsibility) pattern
	// The handler by itself does not know what should be logged, but it is not important
	// as the handler is not created to manage logging
	http.HandleFunc("/", Decorate(handler))
	log.Fatal(http.ListenAndServe("localhost:8080", nil))
}

func handler(writer http.ResponseWriter, request *http.Request) {
	// request context
	ctx := request.Context()
        
	// used instead of log.Println but you can do log.Println just the logging should be in log package
	Println(ctx, "handler started")
	// just to check when the handler finished its job
	defer Println(ctx, "handler ended")

	select {
	// wait for 5 seconds before sending response
	case <-time.After(5 * time.Second):
		fmt.Fprintln(writer, "hello i am message from server to client")
	// context cancel - you can try to call it with curl to have the option for canceling
	// when user cancel the request you will execute logic
	case <-ctx.Done():
		err := ctx.Err()
		Println(ctx, err.Error())
		http.Error(writer, err.Error(), http.StatusInternalServerError)
	}
}
```