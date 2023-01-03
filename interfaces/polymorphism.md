# Polymorphism

## Static type vs dynamic type
Variables have type known at compilation phase.
It’s specified while declaration, never changes and is known as static type (or just type).
Variables of **interface type** also have static type which is an **interface itself**.
They additionally have **dynamic type so the type of assigned value**:

```go
type I interface {
    M()
}
type T1 struct {}
func (T1) M() {}

type T2 struct {}
func (T2) M() {}

func main() {
    var i I = T1{}
    i = T2{}
    _ = i
}
```
Static type of variable `i` is `I`. 
Dynamic type at first is `T1` then `T2`. 
When value of interface type value is nil (which is zero value for interfaces) then dynamic type is not set.


## Dynamic value
Start with an example:
```go
type I interface {
    M()
}
type T struct {}
func (T) M() {}
func main() {
    var t *T
    if t == nil {
        fmt.Println("t is nil")
    } else {
        fmt.Println("t is not nil")
    }
    var i I = t
    if i == nil {
        fmt.Println("i is nil")
    } else {
        fmt.Println("i is not nil")
    }
}
```

Output:
```
t is nil
i is not nil
```

Dynamic value is the actual value assigned. In discussed snippet after assignment `var i I = t`, dynamic value of `i` is nil but dynamic type is `T`.
**Interface type value is nil if both dynamic value and dynamic type are `nil`.**
**The effect is that even if interface type value holds a nil pointer then such interface value is not `nil`.**