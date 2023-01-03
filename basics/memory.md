# Memory

## Data Allocation

### New - Allocates
It's a built-in function that allocates memory, but unlike its namesakes in some other languages it does not initialize the memory, it only zeros it.
That is, `new(T)` allocates zeroed storage for a new item of type T and returns its address, a value of type *T.
In Go terminology, it returns a pointer to a newly allocated zero value of type T.

Usage with slice, map, chan:

```go
map operation
var mp *map[string]string
mp = new(map[string]string)
//*mp = make(map[string]string) / / if this line is omitted, it will pan "Pan: assignment to entry in nil map"“
(*mp)["name"] = "lc"
fmt.Println((*mp)["name"])
```
As can be seen above, silce, map, channel and other types are reference types.
When the reference type is initialized to nil, nil it can not be assigned directly, nor can new be used to allocate memory, but also need to use make to allocate.


### Make - Initialize
The built-in function `make(T, args)` serves a purpose different from `new(T)`.
It creates **slices**, **maps**, and **channels** only, and it returns an initialized (not zeroed) value of type T (not *T).
The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use.
For slices, maps, and channels, `make` initializes the internal data structure and prepares the value for use.

## Difference between stack and heap
Great video which explains the difference between the **stack** and **heap**.
Recommend watching as it would help to write better performance code.
[**Stack vs Heap**](https://www.youtube.com/watch?v=ZMZpH4yT7M0&ab_channel=SingaporeGophers)
