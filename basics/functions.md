One of Go's unusual features is that functions and methods can return multiple values.
The return or result "parameters" of a Go function can be given names and used as regular variables, just like the incoming parameters.
When named, they are initialized to the zero values for their types when the function begins; 
if the function executes a return statement with no arguments, the current values of the result parameters are used as the returned values.
**The names are not mandatory but they can make code shorter and clearer: they're documentation.**
```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

## Function Grouping and Ordering

- Functions should be sorted in rough call order.
- Functions in a file should be grouped by receiver.

Therefore, exported functions should appear first in a file, after
`struct`, `const`, `var` definitions.

A `newXYZ()`/`NewXYZ()` may appear after the type is defined, but before the
rest of the methods on the receiver.

Since functions are grouped by receiver, plain utility functions should appear
towards the end of the file.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>


## Defer
Go's defer statement schedules a function call (the deferred function) to be run immediately before the function executing the defer returns.
It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return.
Deferred functions are executed in LIFO order

Examples are:

**Closing a file:**

```go
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

**Unlocking mutex:**

```go
func Add(i int) {
    mu.Lock()
    defer mu.Unlock()
    val += i
}
```

Deferring a call to a function such as Close has two advantages. 
First, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path. 
Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.


## Init
**Init functions cause an import to have a side effects, and side effects are hard to test, reduce readability and increase the complexity of code.**

`init()` function is called after all the variable declarations in the package have evaluated their initializers, and those are evaluated only after all the imported packages have been initialized.
Keep in mind that `init()` is always called, regardless if there's main or not, so if you import a package that has an init function, it will be executed.

Read this guide if you consider using: [Global State](../way-to-think/model-go.md#global-state-magic) 

Example:

```go
var WhatIsThe = AnswerToLife()

func AnswerToLife() int { // 1
    return 42
}

func init() { // 2
    WhatIsThe = 0
}

func main() { // 3
    if WhatIsThe == 0 {
        fmt.Println("It's all a lie.")
    }
}
```

`AnswerToLife()` is guaranteed to run before `init()` is called, and `init()` is guaranteed to run before `main()` is called.

## Avoid `init()`

Avoid `init()` where possible. When `init()` is unavoidable or desirable, code
should attempt to:

1. Be completely deterministic, regardless of program environment or invocation.
2. Avoid depending on the ordering or side-effects of other `init()` functions.
   While `init()` ordering is well-known, code can change, and thus
   relationships between `init()` functions can make code brittle and
   error-prone.
3. Avoid accessing or manipulating global or environment state, such as machine
   information, environment variables, working directory, program
   arguments/inputs, etc.
4. Avoid I/O, including both filesystem, network, and system calls.

Code that cannot satisfy these requirements likely belongs as a helper to be
called as part of `main()` (or elsewhere in a program's lifecycle), or be
written as part of `main()` itself. In particular, libraries that are intended
to be used by other programs should take special care to be completely
deterministic and not perform "init magic".

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Foo struct {
    // ...
}

var _defaultFoo Foo

func init() {
    _defaultFoo = Foo{
        // ...
    }
}
```

</td><td>

```go
var _defaultFoo = Foo{
    // ...
}

// or, better, for testability:

var _defaultFoo = defaultFoo()

func defaultFoo() Foo {
    return Foo{
        // ...
    }
}
```

</td></tr>
<tr><td>

```go
type Config struct {
    // ...
}

var _config Config

func init() {
    // Bad: based on current directory
    cwd, _ := os.Getwd()

    // Bad: I/O
    raw, _ := ioutil.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )

    yaml.Unmarshal(raw, &_config)
}
```

</td><td>

```go
type Config struct {
    // ...
}

func loadConfig() Config {
    cwd, err := os.Getwd()
    // handle err

    raw, err := ioutil.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )
    // handle err

    var config Config
    yaml.Unmarshal(raw, &config)

    return config
}
```

</td></tr>
</tbody></table>

Considering the above, some situations in which `init()` may be preferable or
necessary might include:

- Complex expressions that cannot be represented as single assignments.
- Pluggable hooks, such as `database/sql` dialects, encoding type registries, etc.
- Optimizations to [Google Cloud Functions] and other forms of deterministic
  precomputation.

  [Google Cloud Functions]: https://cloud.google.com/functions/docs/bestpractices/tips#use_global_variables_to_reuse_objects_in_future_invocations