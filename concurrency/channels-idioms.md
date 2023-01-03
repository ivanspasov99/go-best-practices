## Rules

- Channels should be closed always from the sender
- Technically it is safe not to close a channel. The garbage collector will take care of it but **IT IS NOT GOOD PRACTICE DOING IT SO CLOSE THE CHANNELS**
- Channels are concurrent safe, you can write from multiple goroutines in one channel
- Send to a closed channel panics
- Receive from a closed channel returns the zero value immediately

## A closed channel never blocks
- The first property I want to talk about is a closed channel. Once a channel has been closed, you cannot send a value on this channel, but you can still receive from the channel.

```go
package main

import "fmt"

func main() {
	ch := make(chan bool, 2)
	ch <- true
	ch <- true
	close(ch)

	for i := 0; i < cap(ch)+1; i++ {
		v, ok := <-ch
		fmt.Println(v, ok)
	}
}
```

## A nil channel always blocks
:exclamation: **If you close the channel it won't block**
A nil channel; a channel value that has not been initialised, or has been set to nil will always block. For example:

```go
package main

func main() {
        var ch chan bool
        ch <- true // blocks forever
}
```

The same is true for receiving
```go
package main

func main() {
        var ch chan bool
        <- ch // blocks forever
}
```
