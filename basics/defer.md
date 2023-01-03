Go's defer statement schedules a function call (the deferred function) to be run immediately after the function returns.
The deferred call's arguments are evaluated immediately.
It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return. 
The canonical examples are unlocking a mutex or closing a file.

```go
// Contents returns the file's contents as a string.
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

## Panics
You can use defer with panic if an unexpected panic occurs. Defer is used in the [worker pool](../concurrency/goroutines.md#worker-pool) (producer-consumer pattern)
```go
func ping() {
    defer func() {
        fmt.Println("hello")
    }()

    panic(errors.New("err"))
}
```

## Defer to Clean Up

Use defer to clean up resources such as files and locks.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// easy to miss unlocks due to multiple returns
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
```

</td></tr>
</tbody></table>

