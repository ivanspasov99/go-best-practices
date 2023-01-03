Tickers are for when you want to do something repeatedly at regular intervals.

```go
func main() {
    ticker := time.NewTicker(500 * time.Millisecond)
    done := make(chan bool)
    
    go func() {
        for {
            // Here we’ll use the select builtin on the channel to await the values as they arrive every 500ms	
            select {
            case <-done:
                return
            case t := <-ticker.C:
                fmt.Println("Tick at", t)
            }
        }
    }()
    //  Once a ticker is stopped it won’t receive any more values on its channel. We’ll stop ours after 1600ms.
    time.Sleep(1600 * time.Millisecond)
	// You should stop the ticker, so it stops sending values to the channel
	// It's OK to leave a Go channel open forever and never close it. When the channel is no longer used, it will be garbage collected. Note that it is only necessary to close a channel if the receiver is looking for a close.
    ticker.Stop()
	// if we do not close the channel done there will be goroutine leak as the t:= <-ticker.C will block and wait to receive value
    done <- true
    fmt.Println("Ticker stopped")
}
```

Tickers use a similar mechanism to [timers](./timers.md): a channel that sent values