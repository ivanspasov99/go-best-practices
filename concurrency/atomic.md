
## Atomic

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

func main() {
	// We’ll use an unsigned integer to represent our (always-positive) counter.
	var ops uint64
	// A WaitGroup will help us wait for all goroutines to finish their work.
	var wg sync.WaitGroup
	
	// We’ll start 50 goroutines that each increment the counter exactly 1000 times.
	for i := 0; i < 50; i++ {
		wg.Add(1)

		go func() {
			for c := 0; c < 1000; c++ {
				// To atomically increment the counter we use AddUint64,
				// giving it the memory address of our ops counter with the & syntax.
				atomic.AddUint64(&ops, 1)
			}
			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Println("ops:", ops)
}
```