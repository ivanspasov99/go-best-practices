**Don't communicate by sharing memory instead share memory by communicating.**

## Worker Pool
The worker pool implementation is inspired from the **producer-consumer-reducer** problem
- Producer is sending subtasks to the workers
- Workers are processing the subtasks
- Reducer is doing final work

**Combines all techniques said above in the example below:**
```go
func GetFriends(ctx context.Context, user int64) (map[string]*User, error) {
  g, ctx := errgroup.WithContext(ctx)
  friendIds := make(chan int64)

  // Produce
  g.Go(func() error {
     defer close(friendIds)
     for it := GetFriendIds(user); ; {
        if id, err := it.Next(ctx); err != nil {
           if err == io.EOF {
              return nil
           }
           return fmt.Errorf("GetFriendIds %d: %s", user, err)
        } else {
           select {
           case <-ctx.Done():
              return ctx.Err()
           case friendIds <- id:
           }
        }
     }
  })

  friends := make(chan *User)

  // Consume
  workers := int32(nWorkers)
  for i := 0; i < nWorkers; i++ {
     g.Go(func() error {
        defer func() {
           // Last one out closes does -1 to workers for every finished go routine
           if atomic.AddInt32(&workers, -1) == 0 {
              close(friends)
           }
        }()

        for id := range friendIds {
           if friend, err := GetUserProfile(ctx, id); err != nil {
              return fmt.Errorf("GetUserProfile %d: %s", user, err)
           } else {
              select {
              case <-ctx.Done():
                 return ctx.Err()
              case friends <- friend:
              }
           }
        }
        return nil
     })
  }

  // Reduce
  ret := map[string]*User{}
  g.Go(func() error {
     for friend := range friends {
        ret[friend.Name] = friend
     }
     return nil
  })

  return ret, g.Wait()
}
```

**Select Statement**
```go
select {
case <-ctx.Done():
  return ctx.Err()
case friendIds <- id:
}
```
In Go, a `select` statement blocks until any one of the possible channel operations can be performed.
So this statement blocks until either we can write id into the friendIds channel or the `ctx.Done()` channel becomes readable. (The latter happens when the context is canceled: the `ctx.Done()` channel closes, making it readable.)
When `error` is returned by the `errgroup` the `Context` is canceled and `ctx.Done()` is a Done channel for cancellation.

If any subtask returns an error, the shared group context is canceled, `ctx.Done()` becomes readable, and any writers who were blocked trying to write into full channels exit immediately.
Similarly, if the parent context is canceled (the request terminates early), the same thing will happen. And since writers close our data channels on exit, all the reads will unblock as well.

**What if we are not using select?**
Consider
```go
for id := range friendIds {
           if friend, err := GetUserProfile(ctx, id); err != nil {
              return fmt.Errorf("GetUserProfile %d: %s", user, err)
           } else {
              friends <- friend:
           }
        }
...
// Reduce
ret := map[string]*User{}
g.Go(func() error {
    for friend := range friends {
    ret[friend.Name] = friend
	// return an error for example
    }
    return nil
})
```
In our code, if the reducer loop exits with an error and stops pulling data from the friends channel, the **workers can deadlock trying to write** into a full channel.
You can think that `errgroup will stop all goroutines, but it would not happen as the goroutine is blocked.
**That's why you use select with context cancellation.**

**Improvements**
- You can start the workers before the producer, so they could be ready to read tasks  

## The or-channel 
Allows us to wait on "N" number of channels at the same time. Care about the first to return
```go
package main

import (
	"fmt"
	"time"
)

func startGoroutine(num int, channel chan interface{}) {
	go func() {
		time.Sleep(time.Second * 3)
		fmt.Printf("goroutine %d done\n", num)
		channel <- nil
	}()
}

// you care only about first return
// for example if you want to receive notification when first is done
func main() {
	// you can have different types
	// you can have interface{} so just a return value is needed, no matter what the  return is
	done1 := make(chan interface{})
	done2 := make(chan interface{})
	done3 := make(chan interface{})
	done4 := make(chan interface{})

	startGoroutine(1, done1)
	startGoroutine(2, done2)
	startGoroutine(3, done3)
	startGoroutine(4, done4)

	horseWinner := <-or(done1, done2, done3, done4)
	fmt.Println(horseWinner)
	fmt.Println("main done")
	// waiting to see that all started goroutines have finished their work
	time.Sleep(time.Second * 7)
}

func or(channels ...<-chan interface{}) <-chan interface{} {
	// bottom of the recursion
	switch len(channels) {
	case 0:
		// if you have passed nil
		fmt.Println("Bottom of recursion nil")
		return nil
	case 1:
		fmt.Println("Bottom of recursion, one channel - orDone")
		return channels[0]
	}
	orDone := make(chan interface{})
	go func() {
		// used to notify (when closed) through the recursion that one of the
		// goroutines have received first message
		// after that all the goroutines returns as the orDone is always passed down to the stack
		defer close(orDone)
		switch len(channels) {
		case 2:
			select {
			case <-channels[0]:
			case <-channels[1]:
			}
		default:
			select {
			case <-channels[0]:
			case <-channels[1]:
			case <-channels[2]:
			case <-or(append(channels[3:], orDone)...):
			}
		}
	}()
	return orDone
}
```

## Repeat and Take
Example usage of repeat and take could be found in pipeline services.
You can create multiple repeat and take function and pass to one another

```go
// receive input intStream - coming from stream, could be everything
// multiply by 2
// add 1
// then multiply the rest by 2
pipeline := multiply(done, add(done, multiply(done, intStream, 2), 1), 2)
```

Example realization

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	done := make(chan interface{})
	defer close(done)
	for num := range take(done, repeat(done, "a", "b"), 5) {
		// you can close manually the infinite loop if you want to on first values for example if matches desired state
		//close(done)
		fmt.Printf("%v ", num)
	}
	//time.Sleep(5 * time.Second)
}

// take will only take the first num items off of its incoming valueStream and then exit
// take will not block as the repeat is infinite
func take(done <-chan interface{}, valueStream <-chan interface{}, num int) <-chan interface{} {
	takeStream := make(chan interface{})
	go func() {
		defer close(takeStream)
		defer func() {
			fmt.Println("Take finished")
		}()
		for i := 0; i < num; i++ {
			select {
			case <-done:
				return
			case takeStream <- <-valueStream:
				// you can sleep to check that you can manually stop trough done channel
				// time.Sleep(1 * time.Second)
			}
		}
	}()
	return takeStream
}

// repeat converts a set of discrete values into a stream of values on a channel
// starts goroutine which repeat (send to channel) infinitely the accepted values
// repeat should be stopped with done as it is infinite
func repeat(done <-chan interface{}, values ...interface{}) <-chan interface{} {
	valueStream := make(chan interface{})
	go func() {
		defer close(valueStream)
		defer func() {
			fmt.Println("Repeat finished")
		}()
		for {
			for _, v := range values {
				select {
				case <-done:
					return
				case valueStream <- v:
				}
			}
		}
	}()
	return valueStream
}

// we can extend it with explicitly passing rand function 
func main() {
	repeatFn := func(done <-chan interface{}, fn func() interface{}) <-chan interface{} {
		valueStream := make(chan interface{})
		go func() {
			defer close(valueStream)
			for {
				select {
				case <-done:
					return
				case valueStream <- fn():
				}
			}
		}()
		return valueStream
	}

	done := make(chan interface{})
	defer close(done)

	rand := func() interface{} { return rand.Int()}
    
	// we are passing function to the repeatFn so specific job should be done repeatedly
	for num := range take(done, repeatFn(done, rand), 10) {
		fmt.Println(num)
	}
	
}
```