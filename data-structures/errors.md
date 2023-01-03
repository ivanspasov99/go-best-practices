# Errors

A library writer is free to implement this interface with a richer model under the covers, making it possible not only to see the error but also to provide some context.
As mentioned, alongside the usual *os.File return value, os.Open also returns an error value.
If the file is opened successfully, the error will be nil, but when there is a problem, it will hold an os.PathError:

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

Callers that care about the precise error details can use a type switch or a type assertion to look for specific errors and extract details.
For PathErrors this might include examining the internal Err field for recoverable failures.

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

## Custom Errors
Error strings should identify their origin, such as by having a prefix naming the operation or package that generated the error.
For example, in package image, the string representation for a decoding error due to an unknown format is "image: unknown format".

For every more complex package you should implement as above your custom error struct and 
try to use the `error` abstraction, so you can be consistent.

## Error Propagation and Handling
You should always try to propagate your error as high as possible in the stack, so they can be processed in one place following one pattern.

The following is code that I come across frequently.
```go
func WriteAll(w io.Writer, buf []byte) error {
	_, err := w.Write(buf)
	if err != nil {
		log.Println("unable to write:", err) // annotated error goes to log file
		return err                           // unannotated error returned to caller
	}
	return nil
}
```
The error is logged and also returned to the caller, who possibly will log it, and return it, all the way back up to the top of the program. Which result in twice logging.
You can prevent that if you propagate your errors to the as high as possible in the stack.

There should be consistency in the error handling through the whole application They should implement `error`. There should be one point where you can have for example:

```go
type ErrorHandler interface {
	HandlerError(err error)
}
``` 
In this way you can resolve different kind of errors in different way if needed. Always using interfaces as they are easy to mock.

You can have multiple kind of errors structs which represents errors for the different packages. Assume that you have:
```
|── main.go
|── pkg/
|   |── handlers # propagate here all errors as the handler is using api and renderer packages
|   |   |── handler.go
|   |── api # ApiErrors struct
|   |   |── ...
|   |──renderer # RendererErrors struct
|   |   |── ...
```

You can result in using interface in `handler.go` which can have:
```go
func HandleError(err error) {
    switch v := err.(type) {
	case ApiError: // complex specific logic, call the police 
	case RendererError: // complex specific login, call the ambulance
    }
}
```
You can see how easier it would be to manage errors in this way. No errors bottleneck


Guard clause is a good idea because it clearly indicates that current method is not interested in certain cases.
When you clear up at the very beginning of the method that it doesn't deal with some cases (e.g. when some value is less than zero), then the rest of the method is pure implementation of its responsibility.

Compare it to:
```go
func (b *Buffer) UnreadRune() error {
	if b.lastRead > opInvalid {
		if b.off >= int(b.lastRead) {
			b.off -= int(b.lastRead)
		}
		b.lastRead = opInvalid
		return nil
	}
	return errors.New("bytes.Buffer: UnreadRune: previous operation was not a successful ReadRune")
}
```

The body of the successful case, the most common, is nested inside the first if condition and the successful exit condition, return nil, has to be discovered by careful matching of closing braces.
The final line of the function now returns an error, and the called must trace the execution of the function back to the matching opening brace to know when control will reach this point.
This is more error-prone for the reader, and the maintenance programmer, hence why Go prefer to use guard clauses and returning early on errors.

## Choosing what type of error to use

There are few options for declaring errors.
Consider the following before picking the option best suited for your use case.

- Does the caller need to match the error so that they can handle it?
  If yes, we must support the [`errors.Is`] or [`errors.As`] functions
  by declaring a top-level error variable or a custom type.
- Is the error message a static string,
  or is it a dynamic string that requires contextual information?
  For the former, we can use [`errors.New`], but for the latter we must
  use [`fmt.Errorf`] or a custom error type.
- Are we propagating a new error returned by a downstream function?
  If so, see the [section on error wrapping](#error-wrapping).

[`errors.Is`]: https://golang.org/pkg/errors/#Is
[`errors.As`]: https://golang.org/pkg/errors/#As

| Error matching? | Error Message | Guidance                            |
|-----------------|---------------|-------------------------------------|
| No              | static        | [`errors.New`]                      |
| No              | dynamic       | [`fmt.Errorf`]                      |
| Yes             | static        | top-level `var` with [`errors.New`] |
| Yes             | dynamic       | custom `error` type                 |

[`errors.New`]: https://golang.org/pkg/errors/#New
[`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf

For example,
use [`errors.New`] for an error with a static string.
Export this error as a variable to support matching it with `errors.Is`
if the caller needs to match and handle this error.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

if err := foo.Open(); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

For an error with a dynamic string,
use [`fmt.Errorf`] if the caller does not need to match it,
and a custom `error` if the caller does need to match it.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

// package bar

if err := foo.Open("testfile.txt"); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

func Open(file string) error {
  return &NotFoundError{File: file}
}


// package bar

if err := foo.Open("testfile.txt"); err != nil {
  var notFound *NotFoundError
  if errors.As(err, &notFound) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Note that if you export error variables or types from a package,
they will become part of the public API of the package.

### Error Wrapping

There are three main options for propagating errors if a call fails:

- return the original error as-is
- add context with `fmt.Errorf` and the `%w` verb
- add context with `fmt.Errorf` and the `%v` verb

Return the original error as-is if there is no additional context to add.
This maintains the original error type and message.
This is well suited for cases when the underlying error message
has sufficient information to track down where it came from.

Otherwise, add context to the error message where possible
so that instead of a vague error such as "connection refused",
you get more useful errors such as "call service foo: connection refused".

Use `fmt.Errorf` to add context to your errors,
picking between the `%w` or `%v` verbs
based on whether the caller should be able to
match and extract the underlying cause.

- Use `%w` if the caller should have access to the underlying error.
  This is a good default for most wrapped errors,
  but be aware that callers may begin to rely on this behavior.
  So for cases where the wrapped error is a known `var` or type,
  document and test it as part of your function's contract.
- Use `%v` to obfuscate the underlying error.
  Callers will be unable to match it,
  but you can switch to `%w` in the future if needed.

When adding context to returned errors, keep the context succinct by avoiding
phrases like "failed to", which state the obvious and pile up as the error
percolates up through the stack:

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %w", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %w", err)
}
```

</td></tr><tr><td>

```
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```
x: y: new store: the error
```

</td></tr>
</tbody></table>

However once the error is sent to another system, it should be clear the
message is an error (e.g. an `err` tag or "Failed" prefix in logs).

[`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause

### Error Naming

For error values stored as global variables,
use the prefix `Err` or `err` depending on whether they're exported.
This guidance supersedes the [Prefix Unexported Globals with _](#prefix-unexported-globals-with-_).

```go
var (
  // The following two errors are exported
  // so that users of this package can match them
  // with errors.Is.

  ErrBrokenLink = errors.New("link is broken")
  ErrCouldNotOpen = errors.New("could not open")

  // This error is not exported because
  // we don't want to make it part of our public API.
  // We may still use it inside the package
  // with errors.Is.

  errNotFound = errors.New("not found")
)
```

For custom error types, use the suffix `Error` instead.

```go
// Similarly, this error is exported
// so that users of this package can match it
// with errors.As.

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

// And this error is not exported because
// we don't want to make it part of the public API.
// We can still use it inside the package
// with errors.As.

type resolveError struct {
  Path string
}

func (e *resolveError) Error() string {
  return fmt.Sprintf("resolve %q", e.Path)
}
```