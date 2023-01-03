# Interfaces
A type need not declare explicitly that it implements an interface (implemented implicitly). 
Instead, a type implements the interface just by implementing the interface's methods.
Programmer doesn't have to specify that type T implements interface I. That work is done by the Go compiler (never send a human to do a machine’s job).

Interfaces with only one or two methods are common in Go code, and are usually given a name derived from the method, 
such as `io.Writer` for something that implements Write.


## Type switch
```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

If you want to convert to specific type you can use "comma, ok" syntax checking if it is possible.
```go
str, ok := value.(string)
```
If the type assertion fails, str will still exist and be of type string, but it will have the zero value, an empty string.

Because if you do not use "comma, ok" `str := value.(string)` and it turns out that the value does not contain a string (for example), 
the program will crash with a run-time error.

## Generality
If a type exists only to implement an interface and will never have exported methods beyond that interface, there is no need to export the type itself.
Exporting just the interface makes it clear the value has no interesting behavior beyond what is described in the interface.
It also avoids the need to repeat the documentation on every instance of a common method.

Example:

```go
// do not export the struct
type someImplementationStruct struct {
    field1 string
    field2 string
}

// make constructor method
func InitSomeImplementationStruct() someImplementationStruct
```

## Pointer Methods
You should be very careful when using shared resource between concurrent execution.
Example: 
```go
// Simple counter server.
type Counter struct {
    n int
}
// ctr.n++ is not thread safe

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```




