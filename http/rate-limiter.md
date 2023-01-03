
[Rate limiting](https://en.wikipedia.org/wiki/Rate_limiting) is an important mechanism for controlling resource utilization and maintaining quality of service.
Go elegantly supports rate limiting with goroutines, channels, and [tickers](../tips/tickers.md).


```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// First we’ll look at basic rate limiting. 
	// Suppose we want to limit our handling of incoming requests. 
	// We’ll serve these requests off a channel of the same name.
	
	// receive requests from api for example
	requests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		requests <- i
	}
	close(requests)

	// This limiter channel will receive a value every 200 milliseconds. 
	// This is the regulator in our rate limiting scheme.
	limiter := time.Tick(200 * time.Millisecond)
    
	// Limit ourselves to 1 request every 200 milliseconds.
	for req := range requests {
		// block for 200 milliseconds
		<-limiter
		// process req
		fmt.Println("request", req, time.Now())
	}

	// We may want to allow short bursts of requests in our rate limiting scheme while preserving the overall rate limit. 
	// We can accomplish this by buffering our limiter channel. 
	// This burstyLimiter channel will allow bursts of up to 3 events.
	burstyLimiter := make(chan time.Time, 3)

	// Simulate the first three request are already sent
	for i := 0; i < 3; i++ {
		burstyLimiter <- time.Now()
	}

	go func() {
		// the goroutine will be garbage collected after the program ends
		for t := range time.Tick(200 * time.Millisecond) {
			// block as the buffer is full, wait someone to read
			burstyLimiter <- t
		}
	}()

	// Now simulate 5 more incoming requests. 
	// The first 3 of these will benefit from the burst capability of burstyLimiter.
	burstyRequests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		burstyRequests <- i
	}
	close(burstyRequests)

	// first three will be burst, then will have another
	// capped at 5 only
	for req := range burstyRequests {
		<-burstyLimiter
		fmt.Println("request", req, time.Now())
	}
}

```