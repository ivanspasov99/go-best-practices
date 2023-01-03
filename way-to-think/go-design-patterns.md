
1. [Chain of Responsibility](#chain-of-responsibility---separation-of-concerns)
   - [HTTP Decorator & Error Handler](#decorator)
   - [Decorator & Adapter With Params](#decorator---adapter-with-params)
     - [Web Example](#web-example)
     - [CLI Example](#cli-example)


# Chain of Responsibility - Separation of Concerns
The great in the example below is that you accept interfaces everywhere 
and the testing will be much easier - check http

## Decorator

**Go middlewares**
- Error Handler Middleware
- Decorator Middleware

### Handler & Error Handler
Define a handler function

```go
func DocsHandler(writer http.ResponseWriter, request *http.Request) error {
	// logic with errors
	return nil
}
```

Define input type for the error handler which is custom function type

You can mock the function in the tests easily

```go
// Use this special type for your error handler
// you can mock easily
type HTTPTypeHandler func(w http.ResponseWriter, r *http.Request) error

// Pattern for endpoint on middleware chain, not takes a diff signature.
func DocsErrorHandler(h HTTPTypeHandler) http.HandlerFunc {
	// return handler function so it can be used in http.HandleFunc
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Execute the final handler, and deal with errors
		err := h(w, r)
		if err != nil {
			// Deal with error here, show user error template, log etc
		}
	})
}
```

How it should look like in the `main.go`
```go
func main() {
	// separation of concerns - done with individual error handler
	// you now can have cleaner logic in the handler without error handling which on
	// top level handler would pollute the code
	// you have independent handler and error handler
	http.HandleFunc("/git", DocsErrorHandler(DocsHandler))))
}
```

### Decorator
You have realised that you need logging middleware and alerting middleware because of different requirements.

So it would be better to separate them in different components (middlewares/decorators) and to be independent loosely coupled

**Logging and Alerting**
```go
// The `key` will be available only in the log package, so no one will have access to such type
// therefore they can not change it
type key int

const requestIDKey = key(40)

// define logging decorator which will accept handler function
// so you can accept the ErrorHandler from the example above and implement implicitly logging functionality in both
// using context 
// for more information check context best practices
func LoggingDecorate(f http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		ctx := r.Context()
		id := rand.Int()
		ctx = context.WithValue(ctx, requestIDKey, id)
		f(w, r.WithContext(ctx))
	}
}

// same idea but with defer functionality, so when all internal functions are called 
// to defer will be called
func AlertingDecorate(f http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("Some Alert rule happening")
		// alert before execution

		// this will be executed when the f has been executed
		defer fmt.Println("everything is okay, alert for end")

		fmt.Println("Now continue to the next middleware")

		// if the alert can not be completed for example
		// we would not continue with the next handler
		// we write to the user what's wrong
		f(w, r)
	}
}
```
How `main.go` will look like
```go
func main() {
	http.HandleFunc("/git", AlertingDecorate(LoggingDecorate(errorHandler(DocsHandler))))
}
```

### Conclusion
You can use the pattern not only for handlers but for different business, presentation, processing logic. 
The only thing you need to do is to define the correct "concerns".

## Decorator - Adapter with params

### Web Example
When you need to have different handlers with different (specific) input parameters
If the input parameters are not required and can be resolved (encapsulated) in the middleware
stick to the simpler [decorator](#decorator) approach


General idea of the code below is that you decorate different handlers (adapters)
with custom input parameters. 
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

// Adapter custom adapter
type Adapter func(http.Handler) http.Handler

// Adapt function is used to decorate all adapters at once
func Adapt(h http.Handler, adapters ...Adapter) http.Handler {
	// decorating the handlers so this results in 
	// handler(handler(handler()))
	// they will be in reverse order because of the decorating 
	for _, adapter := range adapters {
		h = adapter(h)
	}
	return h
}

func Notify(logger *log.Logger) Adapter {
	// return handler as the Adapter type requires
	return func(h http.Handler) http.Handler {
		// handlerFunc implements handler with implementing its ServeHTTP func
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			logger.Println("before notify")
			defer logger.Println("after notify")
			// must call h.ServeHTTP as this would call the inner decorated adapter 
			h.ServeHTTP(w, r)
		})
	}
}

func Auth(token string) Adapter {
	// return handler as the Adapter type requires
	return func(h http.Handler) http.Handler {
		// handlerFunc implements handler with implementing its ServeHTTP func
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			fmt.Println("before auth", token)
			defer fmt.Println("after auth", token)
			// must call h.ServeHTTP as this would call the inner decorated adapter
			h.ServeHTTP(w, r)
		})
	}
}

func main() {
	logger := log.New(os.Stdout, "server: ", log.Lshortfile)
	customHandler := Handler{}
	
	// with such adaptation (and adapt function) you will result in
	// calling Auth
	// calling Notify
	// calling customHandler.ServeHTTP function
	h := Adapt(customHandler, Notify(logger), Auth("token"))
	
	
	http.Handle("/", h)
	http.ListenAndServe("localhost:8080", nil)
}

// Handler could be changed to func
type Handler struct{}

func (h Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Println("handle something here")
}

```

### CLI Example
This is CLI which have business logic for processing HTML 
We have two layers of abstraction
1. The main component which generates the HTML and make some http calls `ProcessHTML`
2. Lower level functions which are responsible for processing different part of the HTML

They are decoupled by using interface like function `HTMLRenderer` and all small processing units should
implement it in order to be passed to the higher level logic in `ProcessHTML`

They all are passed directly in the function `ProcessHTML` or in global variable depending on what
state you have in the package and what you like most. Only the global variable should not be exposed. Not good practice as the state could be changed
somewhere else

```go
package main

// HTMLRenderer is type of html renderer
// html input string is just for example
// you can use different types of interfaces or whatever your similar renderers you have
type HTMLRenderer func(html *string) (*string, error)

// could be done with global package variable too
// not good practice to use exported global variables
var renderers = []HTMLRenderer{parseEmojis, parseLinks}

// ProcessHTML could be assigned to a struct if it needs a state
func ProcessHTML(html *string, renderers ...HTMLRenderer) error {
	// process
	
	// iterate over all of your renderers
	for _, renderer := range renderers {
		// render each of them
		// you can extend that logic with error function if you need that
		var err error
		html, err = renderer(html)
		if err != nil {
			return err
		}
	}
	
	
	// make some http call
	
	return nil
}

func parseEmojis(html *string) (*string, error) {
	// parse emojis on html input
	newHTML := "testing"
	return &newHTML, nil
}

func parseLinks(html *string) (*string, error) {
	// parse emojis on html input
	newHTML := "testing"
	return &newHTML, nil
}

func main() {
	// you can do that in special HTML render package

	html := "I am coming from request"
	// they will be executed in the order you have passed them
	ProcessHTML(&html, parseEmojis, parseLinks)

	// using global variable
	ProcessHTML(&html, renderers...)
}
```