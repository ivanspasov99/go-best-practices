We often want to execute Go code at some point in the future, or repeatedly at some interval.

```go
package main

import (
    "fmt"
    "time"
)

func main() {

    timer1 := time.NewTimer(2 * time.Second)
    
	// it blocks for 2 seconds and the event is fired
    <-timer1.C
    fmt.Println("Timer 1 fired")
    
	
    timer2 := time.NewTimer(time.Second)
    go func() {
		// the event won't be fired because the main program will be finished
        <-timer2.C
        fmt.Println("Timer 2 fired")
    }()
    stop2 := timer2.Stop()
    if stop2 {
        fmt.Println("Timer 2 stopped")
    }

    time.Sleep(2 * time.Second)
}
```