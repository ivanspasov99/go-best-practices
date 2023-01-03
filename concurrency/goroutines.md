# Goroutines

## Select Statement

Using select allows you to listen to multiple channels
**Syntax**
```go
package main

func main() {
	done := make(chan struct{})
	result := make(chan struct{})
	for {
		select {
		// you can receive
		case <-done:
			return
		// you can send 	
		case result<-struct{}{}:
		// if both operations above are blocked you enter in default case
		// be cautious as it consumes CPU because it is in infinite loop
		default:
			// do something
		}
	}
}
```

Example Program 
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	elapsed := time.After(time.Second * 4)
	done := make(chan struct{})
	go func() {
		for {
			select {
			case <-elapsed:
				fmt.Println("Time passed, time for end")
				done <- struct{}{}
				return
			default:
			}
			fmt.Println("hello from goroutine")
			time.Sleep(time.Second * 1)
		}
	}()
	<-done
}
```
**Default** select case is allowing you to do some job in case other cases are blocked

The program enters in infinite loop and is executing the default case as the other ones are blocked.
After 4 seconds we are sending a ready signal from `elapsed channel` and the program enters 
the case where the program should exit.

**Be caution using default statement as it consumes CPU in infinite loop, use with purpose**

## Channel Ownership
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	owner := func() <-chan struct{} {
		done := make(chan struct{})
		go func() {
			time.Sleep(time.Second * 2)
			done <- struct{}{}
		}()
		return done
	}

	consumer := func(done <-chan struct{}) {
		<-done
		fmt.Println("I am told to stop now")
	}

	done := owner()
	consumer(done)
}
```
Channel Owners
- Instantiate the channel
- Perform writes, or pass ownership to another goroutine.
- Close the channel.
- Encapsulate the previous three things in this list and expose them via a reader channel.



## WaitGroup
To wait for multiple goroutines to finish, we can use a wait group.
```go
func main() {
    var wg sync.WaitGroup
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        i := i
        go func() {
            defer wg.Done()
            worker(i)
        }()
    }
    wg.Wait()
}
```

## ErrorGroup
Package `errgroup` provides synchronization, error propagation, and Context cancellation for groups of goroutines working on subtasks of a common task.
`errgroup.WithContext()` - WithContext returns a new Group and an associated Context derived from ctx. The derived Context is canceled the first time a function passed to Go returns a non-nil error or the first time Wait returns, whichever occurs first.

**How it works**
1. Create new group: `g, ctx := errgroup.WithContext(ctx)`
2. Start all the group workers `g.Go(func() error {...})`
3. Wait for the result `err := g.Wait()`

An `errgroup.Group` unifies error propagation and context cancellation. It performs several critical functions for us:
1. The calling routines `Wait()` for subtasks to complete; if any subtask returns an error, `Wait()` returns that error back to the caller.
2. If any subtask returns an error, the group `Context` is canceled, which early-terminates all subtasks.
3. If the parent Context (say, an http request context) is canceled, the group Context is also canceled. This helps avoid unnecessary work. For example, if the user navigates away from the page and cancels the http request, we can stop immediately.