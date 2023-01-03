# Type assertion and Type switch

## Type assertion
Type assertion is a way to retrieve dynamic value from interface type value.

Using `v.(T)` as `v` stands for current variable and `(T)` is the type you want to cast to

```go
type I interface {
    M()
}
type T struct{}
func (T) M() {}

func main() {
    var v1 I = T{}
    v2 := v1.(T)
    fmt.Printf("%T\n", v2) // main.T
}
```

What happens if `v1` is not type `T`. It would panic.
Type assertion can be used in **multi-valued form where the additional, second value is a boolean indicating if assertion holds or not**.

```go
type I interface {
    M()
}
type T1 struct{}
func (T1) M() {}

type T2 struct{}
func (T2) M() {}

func main() {
    var v1 I = T1{}
    v2, ok := v1.(T2)
    if !ok {
        fmt.Printf("ok: %v\n", ok) // ok: false
        fmt.Printf("%v,  %T\n", v2, v2) // {},  main.T2
    }
}
```

## Type switch
A type switch is a construct that permits several type assertions in series.
A type switch is like a regular switch statement, but the cases in a type switch specify types (not values), 
and those values are compared against the type of the value held by the given interface value.

```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

You can:
- Single switch case can specify more than one type, separated by comma. 
- Default case
- Optional simple statement

```go
var v I1 = T1{}
// optional aux var
switch aux := 1; v.(type) {
case nil:
    fmt.Println("nil")
case T1:
    fmt.Println("T1", aux)
case T2:
    fmt.Println("T2", aux)
}
```