# Control Structures

## if
Two choices of `if` structures:
```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```
The `err` is `if` scoped.

or
```go
f, err := os.Open(name)
if err != nil {
    return err
}
```

## Switch
Use when you need to evaluate value based on single variable which could have multiple but fixed values.

A switch can also be used to discover the dynamic type of an interface variable. Such a type switch uses the syntax of a type assertion with the keyword type inside the parentheses. 
If the switch declares a variable in the expression, the variable will have the corresponding type in each clause. 
It's also idiomatic to reuse the name in such cases, in effect declaring a new variable with the same name but a different type in each case.

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```
