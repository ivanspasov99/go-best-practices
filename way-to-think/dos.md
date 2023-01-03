## Make the zero value useful
Every variable declaration, assuming no explicit initializer is provided, will be automatically initialised to a value that matches the contents of zeroed memory.
This is the values zero value. The type of the value determines the value’s zero value; for numeric types it is zero, for pointer types nil, the same for slices, maps, and channels.

Example of a type with a useful zero value is bytes.Buffer. You can declare a bytes.Buffer and start writing to it without explicit initialisation.
```go
func main() {
	var b bytes.Buffer
	b.WriteString("Hello, world!\n")
	io.Copy(os.Stdout, &b)
}
```

A useful property of slices is their zero value is nil. This makes sense if we look at the runtime’s definition of a slice header.
```go
type slice struct {
        array *[...]T // pointer to the underlying array
        len   int
        cap   int
}
````
The zero value of this struct would imply len and cap have the value 0, and array, the pointer to memory holding the contents of the slice’s backing array, would be nil. This means you don’t need to explicitly make a slice, you can just declare it.
```go
func main() {
	// s := make([]string, 0)
	// s := []string{}
	var s []string

	s = append(s, "Hello")
	s = append(s, "world")
	fmt.Println(strings.Join(s, " "))
}
```
`var s []string` is similar to the two commented lines above it, but not identical. It is possible to detect the difference between a slice value that is `nil` and a slice value that has zero length. The following code will output false.
```go
func main() {
	var s1 = []string{}
	var s2 []string
	fmt.Println(reflect.DeepEqual(s1, s2))
}
```

**Wherever you can set default values**

## Same types parameters
You can do
```go
func Max(a, b int) int
```

Be cautious when defining function like `func CopyFile(to, from string) error`
as when you write `CopyFile("/tmp/backup", "presentation.md")` you are not sure which is `to` and which `from` if you do not see the function definition

One simple solution will be to define helper type:
```go
type Source string

func (src Source) CopyTo(dest string) error {
	return CopyFile(dest, string(src))
}

func main() {
	var from Source = "presentation.md"
	from.CopyTo("/tmp/backup")
}
```

## Receiving Slice and Maps
Keep in mind that users can modify a map or slice you received as an argument if you store a reference to it.

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// We can now modify trips[0] without affecting d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

## Returning Slices and Maps

Similarly, be wary of user modifications to maps or slices exposing internal
state.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot returns the current stats.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot is no longer protected by the mutex, so any
// access to the snapshot is subject to data races.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

## Use Enums for non-readable parameters
```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady Status = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

## Use Raw String Literals to Avoid Escaping

Go supports [raw string literals](https://golang.org/ref/spec#raw_string_lit),
which can span multiple lines and include quotes. Use these to avoid
hand-escaped strings which are much harder to read.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>