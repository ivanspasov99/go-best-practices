# Welcome to the GoLang Best Practices learning centre

![golang-logo](./assets/goLogo.png)

## Why Go

1. **Simplicity**
    - Simplicity is prerequisite for reliability
    - There are two ways of constructing a software design: One way is to make it so simple that there are obviously no
      deficiencies, and the other way is to make it so complicated that there are no obvious deficiencies. The first
      method is far more difficult.
2. **Readability**
    - Readability is essential for maintainability
    - Programs must be written for people to read, and only incidentally for machines to execute.
    - The most important skill for a programmer is the ability to effectively communicate ideas.
    
### Basics

| **Link**                                      | **Context**                                                                                                                                                                    |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Control Flow](./basics/control-structure.md) | Introduction                                                                                                                                                                   |
| [Functions](./basics/functions.md)            | Function grouping, Defer, Init, Avoid Init                                                                                                                                     |
| [Methods](./basics/methods.md)                | Value Methods, Pointer Methods                                                                                                                                                 |
| [Names](./basics/names.md)                    | Best Practices, [Identifiers](./basics/names.md#identifiers-guidelines), [Comments](./basics/names.md#comments), [Code Documentation](./basics/names.md#documentation-of-code) | 
| [Memory](./basics/memory.md)                  | New, Make                                                                                                                                                                      |
| [Panic](./basics/panic.md)                    | Best Practices, Recover from Panic                                                                                                                                             |
| [Defer](./basics/defer.md)                    | Introduction, Panic Usage, How to write clean defer                                                                                                                            |
| [Semi Columns](./basics/semicolumns.md)       | How go compiler evaluate semicolons                                                                                                                                            |

### Data Structure

| **Link**                                      | **Context**                                                                                                                          |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| [Arrays](./data-structures/arrays.md)         | Introduction                                                                                                                         |
| [Enums](./data-structures/constants-enums.md) | How to write Enums in Golang                                                                                                         |
| [Structs](./data-structures/struct.md)        | Struct visibility, Avoid embedding types in public structs, [Initializing structs](./data-structures/struct.md#initializing-structs) |
| [Slices](./data-structures/slices.md)         | Introduction, Nil is a valid slice                                                                                                   |
| [Maps](./data-structures/maps.md)             | Existing Value, Map initialization                                                                                                   |
| [Errors](./data-structures/errors.md)         | Introduction, Custom Errors, Error Propagation and Handling, How to choose the error type, Wrapping Errors                           |
| [Interfaces](./data-structures/interfaces.md) | Type Switch, Generality, Pointer Methods                                                                                             |

### Interfaces

| **Link**                                                       | **Context**                                                     |
|----------------------------------------------------------------|-----------------------------------------------------------------|
| [Data Structure Basics](./data-structures/interfaces.md)       | Interface Guidelines                                            | 
| [Usage - Do/Don't](./interfaces/usage.md)                      | Do and Dont's and [Testing](./interfaces/usage.md#mock-testing) | 
| [Polymorphism](./interfaces/polymorphism.md)                   | Static types, Dynamic types & Values                            | 
| [Type Assertion-Switch](./interfaces/type-assertion-switch.md) | Type assertion and switch                                       | 
| [Advance Best Practices and Examples](./interfaces/advance.md) | Preemptive Interfaces                                           | 

### Concurrency

| **Link**                                                      | **Context**                                                                                                                                                                                                                                |
|---------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Concurrency](./concurrency/concurency.md)                    | Introduction                                                                                                                                                                                                                               |
| [Atomic](./concurrency/atomic.md)                             | Atomic                                                                                                                                                                                                                                     |
| [Goroutines](./concurrency/goroutines.md)                     | Select, WaitGroup, ErrGroup                                                                                                                                                                                                                | 
| [Go Scheduler](./concurrency/go-scheduler.md)                 | Deep dive analyse how Go Scheduler works                                                                                                                                                                                                   |
| [Channels](./concurrency/channels.md)                         | Buffered, Unbuffered, Channel Directions, Non Blocking Channel Operations                                                                                                                                                                  |
| [Channels Idioms](./concurrency/channels-idioms.md)           | Channel Idioms, Nil Channel,                                                                                                                                                                                                               |
| [Concurrency Patterns](./concurrency/concurrency-patterns.md) | [Worker Pool Pattern, Context Cancellation](./concurrency/concurrency-patterns.md#worker-pool), [or-channel](./concurrency/concurrency-patterns.md#the-or-channel), [pipeline steam](/concurrency/concurrency-patterns.md#repeat-and-take) |

### Network

| **Link**                                       | **Context**                            |
|------------------------------------------------|----------------------------------------|
| [HTTP Guideline](./http/http.md)               | Closing Body, HTTP Connections         |
| [Request Rate Limiter](./http/rate-limiter.md) | Limit the number of processed requests |


### Layout

| **Link**                                     | **Context**         |
|----------------------------------------------|---------------------|
| [Format Style](./layout/format-style.md)     | Format Code         |
| [Project-layout](./layout/project-layout.md) | Package design      |
| [API structure](./layout/api.md)             | API Internal Design |


### Testing

| **Link**                                      | **Context**                               |
|-----------------------------------------------|-------------------------------------------|
| [Approaches](./testing/approaches.md)         | Testing Approaches (Go Native vs BDT)     |
 | [HTTP testing](./testing/http-server-mock.md) | How to test http handler using `httptest` | 

### Way to think

| **Link**                                                                     | **Context**                                                                                                                                                                                                           | 
|------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Model Go](./way-to-think/model-go.md)                                       | [Global State](./way-to-think/model-go.md#global-state-magic), [Model Public API](./way-to-think/model-go.md#modeling-public-api), Guard clause                                                                       |
| [Do and Dont's](./way-to-think/dos.md)                                       | <ul><li>Go Guidelines</li><li>Make the zero value useful</li><li>Slice and maps pointers to underlying data</li><li>Enums instead non-readable parameters</li><li>Use Raw String Literals to Avoid Escaping</li></ul> |
| [Nice to know - Traps](./way-to-think/nice-to-know.md)                       | Go Specifics, Traps, Hidden Gems                                                                                                                                                                                      |
 | [Design Patterns](./way-to-think/design-patterns.md)                         | Design Pattern in Go terms compared to OOP                                                                                                                                                                            |
| [Clean Architecture](./way-to-think/clean-architecture.md)                   | SOLID, Component Principles, GRASP                                                                                                                                                                                    |
 | [Go Design Pattern Real Life Examples](./way-to-think/go-design-patterns.md) | Gives real life examples of using Decorator/Adapter/Chain-of-Responsibility (Middlewares) patterns and how the abstractions and layers are chosen and designed                                                        | 

### Tips

| **Link**                                   | **Context**                                                                                                                        |
|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Timers](./tips/timers.md)                 | How to set timers (using channels)                                                                                                 |
| [Tickers](./tips/tickers.md)               | How to set tickers (using channels)                                                                                                |
| [Time](./tips/time.md)                     | Time and time duration                                                                                                             |
| [Performance](./tips/performance.md)       | Small Performance improvements                                                                                                     |
| [Style](./tips/style.md)                   | Similar Declaration, Import Aliases, Reduce Nesting, Variables Scope, Unnecessary Else, Local variable declaration, Format Strings |
| [Configurations](./tips/configurations.md) | Configure Environmental                                                                                                            |
| [Context](./context/context.md)            | Context Cancellation, Context Propagation, HTTP Server & Client, Context Values (Middleware Logging)                               |

### External References
Context Package - https://www.youtube.com/watch?v=LSzR0VEraWw&t=312s

#### Go

1. [Effective Go](https://go.dev/doc/effective_go)
2. [A Tour of Go](https://go.dev/tour/welcome/1)

#### Interface Guide
1. [Interfaces in Go - Part 1](https://medium.com/golangspec/interfaces-in-go-part-i-4ae53a97479c)
2. [Interfaces in Go - Part 2](https://medium.com/golangspec/interfaces-in-go-part-ii-d5057ffdb0a6)
3. [Interfaces in Go - Part 3](https://medium.com/golangspec/interfaces-in-go-part-iii-61f5e7c52fb5)
