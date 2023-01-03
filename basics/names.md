# Names

**Poor naming is symptomatic of poor design**

## Package Names
`package bytes`

 Package name should be good: short, concise, evocative. 
 By convention, packages are given lower case, single-word names, no need for underscores or mixedCaps.
 
 Another convention is that the package name is the base name of its source directory
 the package in `src/encoding/base64` is imported as `"encoding/base64"` but has name `base64`, not `encoding_base64` and not `encodingBase64`.
 
## Getters & Setters
It's neither idiomatic nor necessary to put Get into the getter's name.
If you have a field called owner (lower case, unexported), the getter method should be called `Owner` (upper case, exported), not `GetOwner`.
The use of **upper-case names for export** will make the method useless.
A setter function will likely be called `SetOwner`.
```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

## Interface Names
By convention, one-method interfaces are named by the method name plus an `-er` suffix or similar modification to construct an agent noun: `Reader`, `Writer`, `Formatter`, `CloseNotifier` etc.

## MixedCaps
Convention in Go is to use MixedCaps or mixedCaps rather than underscores to write multiword names.

## Identifiers Guidelines
- Don’t include the name of your type in the name of your variable. 
- Constants should describe the value they hold, not how that value is used. 
- Prefer single letter variables for loops and branches, single words for parameters and return values, multiple words for functions 
- Remember that the name of a package is part of the name the caller uses to refer to it, so make use of that. 
- Short variable names work well when the distance between their declaration and last use is short. 
- You shouldn’t name your variables after their types for the same reason you don’t name your pets "dog" and "cat". 
- Use a consistent naming style. You should use for example only one of `db`, `DB`, `dbase`, `database` in all code. 
- Use consistent declaration style. There are up to six declaration styles in Go.
  - When declaring, but not initialising, a variable, use `var`: `var players int`
  - When declaring and initialising, use `:=`: `players := 5`
  

## Zero Values
- `var players = 0` is redundant because "0" is `players` zero value of int.

## Comments
Comments are very important to the readability of a Go program. Each comments should do one and only one of three things:
- The comment should explain **what** the thing does.
- The comment should explain **how** the thing does what it does.
- The comment should explain **why** the thing is why it is.

The first form (**what**) is ideal for commentary on public symbols:
```go
// Open opens the named file for reading.
// If successful, methods on the returned file can be used for reading.
```

The second form (**what**) is ideal for commentary inside a method:
```go
// queue all dependant actions
var results []chan error
for _, dep := range a.Deps {
        results = append(results, execute(seen, dep))
}
```

The **why** style of commentary exists to explain the external factors that drove the code you read on the page.
Frequently those factors rarely make sense taken out of context, the comment exists to provide that context.
```go
return &v2.Cluster_CommonLbConfig{
	// Disable HealthyPanicThreshold
    HealthyPanicThreshold: &envoy_type.Percent{
    	Value: 0,
    },
}
```
In this example it may not be immediately clear what the effect of setting HealthyPanicThreshold to zero percent will do.
The comment is needed to clarify that the value of 0 will disable the panic threshold behaviour.

**Comments on variables and constants should describe their contents**
`const randomNumber = 6 // determined from an unbiased die`
In this example the comment describes **why** randomNumber is assigned the value six

## Documentation of code
Because godoc is the documentation for your package, you should always add a comment for every public symbol - variable, constant, function, and method - declared in your package.
- Any public function that is not both obvious and short must be commented.
- Any function in a **library** must be commented regardless of length or complexity

There is one exception to this rule; you don’t need to document methods that implement an interface. Specifically don’t do this:
```go
// Read implements the io.Reader interface
func (r *FileReader) Read(buf []byte) (int, error) {}
```

This comment says nothing. It doesn't tell you what the method does, in fact it’s worse, it tells you to go look somewhere else for the documentation. In this situation I suggest removing the comment entirely.

Here is an example from the io package:
```go
// LimitReader returns a Reader that reads from r
// but stops with EOF after n bytes.
// The underlying implementation is a *LimitedReader.
func LimitReader(r Reader, n int64) Reader { return &LimitedReader{r, n} }

// A LimitedReader reads from R but limits the amount of
// data returned to just N bytes. Each call to Read
// updates N to reflect the new amount remaining.
// Read returns EOF when N <= 0 or when the underlying R returns EOF.
type LimitedReader struct {
	R Reader // underlying reader
	N int64  // max bytes remaining
}

func (l *LimitedReader) Read(p []byte) (n int, err error) {
	if l.N <= 0 {
		return 0, EOF
	}
	if int64(len(p)) > l.N {
		p = p[0:l.N]
	}
	n, err = l.R.Read(p)
	l.N -= int64(n)
	return
}
```

Note that the `LimitedReader` declaration is directly preceded by the function that uses it, and the declaration of `LimitedReader.Read` follows the declaration of `LimitedReader` itself.
Even though `LimitedReader.Read` has no documentation itself, it's clear from that it is an implementation of io.Reader.

TIP: Before you write the function, write the comment describing the function. If you find it hard to write the comment, then it’s a sign that the code you’re about to write is going to be hard to understand.


