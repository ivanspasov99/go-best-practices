## Guidelines
- Log only actionable information, which will be read by a human or a machine
- Avoid fine-grained log levels — info and debug are probably enough, as well as error
- Use structured logging - JSON format as most of the tools take use of it
- Try to log the information as high as possible in the stack

## Examples
Go project layout
```
|── main.go
|── pkg/
|   |── handlers
|   |   |── handler.go
|   |── api # used in handlers
|   |   |── ...
|   |──parser # used in handlers
|   |   |── ...
```

You should try to propagate the logging into the `handler.go` or handlers if this is the second in the stack after `main.go`.
If you do not do it, you will result logging bottleneck.

For example you will have a lot of repeated logs as one team cannot know where and what is logged all the time.
The loggers are dependencies, they should be initialized as well and it should happen on program start or at least as high as possible in the stack


