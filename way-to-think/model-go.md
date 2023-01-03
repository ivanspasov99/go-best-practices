## Global State Magic
- **No package level vars**
- **No func init**

Package-global objects can encode state and/or behavior that is hidden from external callers.
Code that calls on those globals can have surprising side effects, which subverts the reader’s ability to understand and mentally model the program.

Consider the following function definition:
`func NewObject(n int) (*Object, error)`

By convention, we expect that functions of the form NewXxx are type constructors.
That expectation is validated when we see that the function returns a pointer to an Object, and an error.
From this we can deduce that the constructor function may or may not succeed, and if it fails, that we will receive an error telling us why.
We observe that the function takes a single int parameter, which we assume controls some aspect or capability of the returned Object.
Presumably, there is some constraint on n, which, if not met, will result in an error. But because the function takes no other parameter, we expect it should have no other effect, beyond (hopefully) allocating some memory.

By reading the function signature alone, we are able to make all of these deductions, and build a mental model of this function.
This process, applied repeatedly and recursively from the first line of func main, is how we read and understand programs.


Now, consider if this were the body of the function.
```go
func NewObject(n int) (*Object, error) {
	row := dbconn.QueryRow("SELECT ... FROM ... WHERE ...")
	var id string
	if err := row.Scan(&id); err != nil {
		logger.Log("during row scan: %v", err)
		id = "default"
	}
	resource, err := pool.Request(n)
	if err != nil {
		return nil, err
	}
	return &Object{
		id:  id,
		res: resource,
	}, nil
}
```

The function invokes a package global database/sql.Conn, to make a query against some unspecified database; a package global logger, to output a string of arbitrary format to some unknown location;
and a package global pool object of some kind, to request a resource of some kind.
All of these operations have side effects that are completely invisible from an inspection of the function signature.

There is no way for a caller to predict any of these things will happen, except by reading the function and diving to the definition of all of the globals.

Consider this alternative signature:
`func NewObject(db *sql.DB, pool *resource.Pool, n int, logger log.Logger) (*Object, error)`

By lifting each of the dependencies into the signature as parameters, we allow readers to accurately model the scope and potential behaviors of the function.
The caller knows exactly what the function needs to do its work, and can provide them accordingly.

## Modeling Public API

If we’re designing the public API for this package, we can even take it one helpful step further.
```go
// RowQueryer models part of a database/sql.DB.
type RowQueryer interface {
	QueryRow(string, ...interface{}) *sql.Row
}

// Requestor models the requesting side of a resource.Pool.
type Requestor interface {
	Request(n int) (*resource.Value, error)
}

func NewObject(q RowQueryer, r Requestor, n int, logger log.Logger) (*Object, error) {
	// ...
}
```

By modeling each concrete object as an interface, capturing only the methods we use, we allow callers to swap in alternative implementations.
This reduces source-level coupling between packages, and enables us to mock out the concrete dependencies in tests.
Testing the original version of the code, with concrete package-level globals, involves tedious and error-prone swapping-out of components.

If all of our constructors and functions take their dependencies explicitly, then we no longer have any use for globals. Instead, we can construct all of our database connections, our loggers, our resource pools, in our func main,
so that future readers can very clearly map out a component graph.
And we can very explicitly pass those dependencies to the components that use them, so that we eliminate the comprehension-subverting magic of globals.
Also, observe that if we have no global variables, we have no more use for func init, whose only purpose is to instantiate or mutate package-global state.

## Guard clause
Go code is written in a style where the success path continues down the screen as the function progresses. The practice is called 'line of sight' coding.

This is achieved by using **guard clauses**; conditional blocks with assert preconditions upon entering a function. Here is an example from the `bytes` package,

Guard clause: A guard clause is simply a check that immediately exits the function, either with a return statement or an exception.
```go
func (b *Buffer) UnreadRune() error {
	if b.lastRead <= opInvalid {
		return errors.New("bytes.Buffer: UnreadRune: previous operation was not a successful ReadRune")
	}
	if b.off >= int(b.lastRead) {
		b.off -= int(b.lastRead)
	}
	b.lastRead = opInvalid
	return nil
}
```

## Avoid Mutable Globals

Avoid mutating global variables, instead opting for dependency injection.
This applies to function pointers as well as other kinds of values.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// sign.go

var _timeNow = time.Now

func sign(msg string) string {
  now := _timeNow()
  return signWithTime(msg, now)
}
```

</td><td>

```go
// sign.go

type signer struct {
  now func() time.Time
}

func newSigner() *signer {
  return &signer{
    now: time.Now,
  }
}

func (s *signer) Sign(msg string) string {
  now := s.now()
  return signWithTime(msg, now)
}
```
</td></tr>
<tr><td>

```go
// sign_test.go

func TestSign(t *testing.T) {
  oldTimeNow := _timeNow
  _timeNow = func() time.Time {
    return someFixedTime
  }
  defer func() { _timeNow = oldTimeNow }()

  assert.Equal(t, want, sign(give))
}
```

</td><td>

```go
// sign_test.go

func TestSigner(t *testing.T) {
  s := newSigner()
  s.now = func() time.Time {
    return someFixedTime
  }

  assert.Equal(t, want, s.Sign(give))
}
```

</td></tr>
</tbody></table>
