# Channels
Like maps, channels are allocated with make, and the resulting value acts as a reference to an underlying data structure.
**Channels should be closed always from the sender**


## Unbuffered

```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

**Receivers always block** until there is data to receive.
If the channel is **unbuffered**, the sender blocks until the receiver has received the value.


## Buffered
If the channel has a buffer, the sender blocks only until the value has been copied to the buffer;
```go
    // there no problem because we write into the buffer 1 string and it does not block
    c := make(chan string, 1)  // Allocate a buffered channel.
	c <- "message"
	fmt.Println(<-c)
```

if the buffer is full, this means waiting until some receiver has retrieved a value.
```go
   c := make(chan string, 1)  // Allocate a buffered channel with 1 capacity.
   	go func(c chan string) {
   		fmt.Println("Start writing")
   		c <- "message"
   		fmt.Println("When the first message is read, writing second in buffer")
   		c <- "message2"
   		fmt.Println("This code here is not executed as no one read the first message and the buffer is full and the go routine is in waiting state")
   	}(c)

   	time.Sleep(10000)
   	fmt.Println("End of main")
```

## Channel Directions
Always define if you sent or receive from channel in a function `func ping(receiveFrom <-chan string, sendTo chan<- string)`
This `ping` func tion accepts a channel for receiving and sending values. It would be a compile-time error to try to call ping with ping(sendChannel, receiveChannel)

## Non Blocking Channel Operations
Use with caution
```go
package main

import "fmt"

func main() {
    messages := make(chan string)
    signals := make(chan bool)

	// non-blocking receive
    select {
		// If a value is available on messages then select will take the <-messages case with that value.
    case msg := <-messages:
        fmt.Println("received message", msg)
		// If not it will immediately take the default case.
    default:
        fmt.Println("no message received")
    }

	//  non-blocking send works similarly
    msg := "hi"
    select {
    case messages <- msg:
        fmt.Println("sent message", msg)
    default:
        fmt.Println("no message sent")
    }

	// We can use multiple cases above the default clause to implement a multi-way non-blocking select.
    select {
    case msg := <-messages:
        fmt.Println("received message", msg)
    case sig := <-signals:
        fmt.Println("received signal", sig)
    default:
        fmt.Println("no activity")
    }
}
```
Result:
```go
$ go run non-blocking-channel-operations.go
no message received
no message sent
no activity
```

## Range Over Channel
You can use `for range` for iterating channels. It is **blocking**. If the **channels is closed it leaves the for range**.
```go
func main() {
    queue := make(chan string, 2)
    queue <- "one"
    queue <- "two"
    close(queue)

    for elem := range queue {
        fmt.Println(elem)
    }
}
```
