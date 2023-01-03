# Slices
Slices wrap arrays to give a more general, powerful, and convenient interface to sequences of data.
Except for items with explicit dimension such as transformation matrices, most array programming in Go is done with slices rather than simple arrays.

**Slices hold references to an underlying array, and if you assign one slice to another, both refer to the same array.**
**If a function takes a slice argument, changes it makes to the elements of the slice will be visible to the caller, analogous to passing a pointer to the underlying array.**

Go's arrays and slices are one-dimensional. 
To create the equivalent of a 2D array or slice, it is necessary to define an array-of-arrays or slice-of-slices, like this:

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

## Append

Append values: 

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
// Prints [ 1 2 3 4 5 6 ]
```


Append slices - using `...`: 

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
// Prints [ 1 2 3 4 5 6 ]
```

## nil is a valid slice

`nil` is a valid slice of length 0. This means that,

- You should not return a slice of length zero explicitly. Return `nil`
  instead.

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

```go
  if x == "" {
    return []int{}
  }
```

  </td><td>

```go
  if x == "" {
    return nil
  }
```

  </td></tr>
  </tbody></table>

- To check if a slice is empty, always use `len(s) == 0`. Do not check for
  `nil`.

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

```go
  func isEmpty(s []string) bool {
    return s == nil
  }
```

  </td><td>

```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
```

  </td></tr>
  </tbody></table>

- The zero value (a slice declared with `var`) is usable immediately without
  `make()`.

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
```

  </td><td>

```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
```

  </td></tr>
  </tbody></table>

Remember that, while it is a valid slice, a nil slice is not equivalent to an
allocated slice of length 0 - one is nil and the other is not - and the two may
be treated differently in different situations (such as serialization).