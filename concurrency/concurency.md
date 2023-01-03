Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables.
Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design.
Do not communicate by sharing memory; instead, share memory by communicating.

The Goroutines are multiplexed to a fewer number of OS threads. There might be only one thread in a program with thousands of Goroutines.
If any Goroutine in that thread blocks say waiting for user input, then another OS thread is created and the remaining Goroutines are moved to the new OS thread.
All these are taken care of by the runtime and we as programmers are abstracted from these intricate details and are given a clean API to work with concurrency.

