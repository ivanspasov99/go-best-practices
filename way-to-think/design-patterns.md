*The use of the design patterns is an art, no rules when to use what*

1. [Creational Patterns](#creational-patterns)
   - [Singleton](#singleton)
   - [Builder](#builder)
   - [Factory Method](#factory-method)
   - [Abstract Factory](#abstract-factory)
2. [Structural Patterns](#structural-patterns)
3. [Behavioral Patterns](#behavioral-patterns)


## Creational Patterns

### Singleton
Singleton is a creational design pattern that lets you ensure that a class has only one instance, while providing a global access point to this instance
You should be careful when using multiple goroutines

Example using `once sync.Once` which is thread safe
```go
package main

import (
	"fmt"
	"sync"
)

var once sync.Once

type single struct{}

var singleInstance *single

func getInstance() *single {
	if singleInstance == nil {
		once.Do(
			func() {
				fmt.Println("Creating single instance now.")
				singleInstance = &single{}
			})
	} else {
		fmt.Println("Single instance already created.")
	}
	return singleInstance
}

func main() {

	for i := 0; i < 30; i++ {
		go getInstance()
	}

	// Scanln is similar to Scan, but stops scanning at a newline and
	// after the final item there must be a newline or EOF.
	fmt.Scanln()
}
```

Example using standard approach library for multiple goroutines 
```go
package main

import (
    "fmt"
    "sync"
)

var lock = &sync.Mutex{}

type single struct {
}

var singleInstance *single

func getInstance() *single {
	if singleInstance == nil {
		lock.Lock()
		defer lock.Unlock()
		if singleInstance == nil {
			fmt.Println("Creating single instance now.")
			singleInstance = &single{}
		} else {
			fmt.Println("Single instance already created.")
		}
	} else {
		fmt.Println("Single instance already created.")
	}
	return singleInstance
}
```

### Builder
The Builder pattern is used when the desired product is complex and requires multiple steps to complete. The pattern lets you construct complex objects step by step.

#### Smell
- Constructor with a lot of arguments

#### Usage
- Build objects step by step (*Builder*)
- Construction of various representations of the product (*Director*) involves similar steps that differ only in the details.

#### Examples
- Building seeds (k8s clusters) on different infrastructures (AWS, Alibaba, GCP)

#### Composition
>You can use enum types instead of string, so you can limit your errors
- *Product*, is the object that is being built
- *Concrete Builder*, inherits from abstract builder and implements interfaces
- *Abstract Builder*, class provides all interfaces required
- *Director*, charge of building the product using Builder object

#### Factory Method Comparison
Technically it is okay to say that the Builder pattern is using Factory like function but the difference is in the ideology
- *Builder DP is used when an object cannot be produced in one single step.*
- *Factory Method DP is used to build one entire object in a single method call.*

#### Decorator Comparison
There's no need to add toppings to a Pizza after it has been fully constructed. You don't eat half a pizza and then add another topping to it.
In other words, the Builder Pattern makes it easy to construct an object which is extensible in independent directions at construction time, 
while the Decorator Pattern lets you add extensions to functionality to an object after construction time.
Using the decorator pattern to construct objects is bad because it leaves the object in an inconsistent (or at least incorrect) 
state until all the required decorators are in place - similar to the JavaBean problem of using setters to specify optional constructor arguments.

#### Product
The object which should be modeled with different values
```go
type House struct {
	windowType string
	doorType   string
}
```

#### Concrete Builder - Normal
```go
type NormalBuilder struct {
    windowType string
    doorType   string
}

func newNormalBuilder() *NormalBuilder {
    return &NormalBuilder{}
}

func (b *NormalBuilder) setWindowType() {
    b.windowType = "Wooden Window"
}

func (b *NormalBuilder) setDoorType() {
    b.doorType = "Wooden Door"
}

func (b *NormalBuilder) getHouse() House {
    return House{
        doorType:   b.doorType,
        windowType: b.windowType,
        floor:      b.floor,
    }
}
```

#### Concrete Builder - Igloo
```go
type IglooBuilder struct {
    windowType string
    doorType   string
}

func newIglooBuilder() *IglooBuilder {
    return &IglooBuilder{}
}

func (b *IglooBuilder) setWindowType() {
    b.windowType = "Snow Window"
}

func (b *IglooBuilder) setDoorType() {
    b.doorType = "Snow Door"
}

func (b *IglooBuilder) getHouse() House {
    return House{
        doorType:   b.doorType,
        windowType: b.windowType,
        floor:      b.floor,
    }
}
```

#### Builder Interface
```go
type Builder interface {
	setWindowType()
	setDoorType()
	getHouse() House
}

func getBuilder(builderType string) Builder {
	if builderType == "normal" {
		return newNormalBuilder()
	}

	if builderType == "igloo" {
		return newIglooBuilder()
	}
	return nil
}
```

The director is only responsible for executing the building
steps in a particular sequence. It's helpful when producing
products according to a specific order or configuration.
Strictly speaking, the director class is optional, since the
client can control builders directly.
#### Director
```go
type Director struct {
    builder Builder
}

func newDirector(b Builder) *Director {
    return &Director{
        builder: b,
    }
}

func (d *Director) setBuilder(b Builder) {
    d.builder = b
}

func (d *Director) buildHouse() House {
    d.builder.setDoorType()
    d.builder.setWindowType()
    d.builder.setNumFloor()
    return d.builder.getHouse()
}
```

#### Client Code
```go
import "fmt"

func main() {
    normalBuilder := getBuilder("normal")
    iglooBuilder := getBuilder("igloo")

    // if you want to build the object in specific way using only part it
    // house without windows
    customIglooBuilder.setDoorType()
    customIglooBuilder.getHouse()
	
    director := newDirector(normalBuilder)
    normalHouse := director.buildHouse()

    fmt.Printf("Normal House Door Type: %s\n", normalHouse.doorType)
    fmt.Printf("Normal House Window Type: %s\n", normalHouse.windowType)
    fmt.Printf("Normal House Num Floor: %d\n", normalHouse.floor)

    director.setBuilder(iglooBuilder)
    iglooHouse := director.buildHouse()

    fmt.Printf("\nIgloo House Door Type: %s\n", iglooHouse.doorType)
    fmt.Printf("Igloo House Window Type: %s\n", iglooHouse.windowType)
    fmt.Printf("Igloo House Num Floor: %d\n", iglooHouse.floor)

}
```

### Factory Method
Factory Method is a creational design pattern used to create concrete implementations of a common interface.

#### Problem
A framework needs to standardize the architectural model for a range of applications, 
but allow for individual applications to define their own domain objects and provide for their instantiation.

#### Simpler Problem Example

#### Truck
You have a transportation company which have started to deliver products with trucks
```go
type truck struct{}

func newCar() *truck {
	return &truck{}
}

func (c *truck) Deliver() {
	fmt.Println("Car Deliver")
}

func (c *truck) Tax() {
	fmt.Println("Car Tax")
}
```

#### Transport Interface
After some time you have expanded your company, and you think how to make your architecture extendable for different kind of transport (sea, air, planets).
That leads you to defining *Factory Method* which will create the different transportation object for you depending on the input
```go
type Transport interface {
	Deliver()
	Tax()
}

// IdentifyTransport transport identifier is used to identify which transport to
// be used could be starting and ending endpoint
func IdentifyTransport(transportIdentifier string) Transport {
   switch transportIdentifier {
   case "sea":
      fmt.Println("Sea Transport")
      return newBoat()
   case "land":
      fmt.Println("Land Transport")
      return newCar()
   default:
      // should return error also instead only nil
      return nil
    }
}
```

#### Boat
Now you can extend with the boat struct
```go
type boat struct{}

func newBoat() *boat {
	return &boat{}
}

func (c *boat) Deliver() {
	fmt.Println("Car Deliver")
}

func (c *boat) Tax() {
	fmt.Println("Car Tax")
}
```

#### Client Code
```go
func main() {
   // some input
   transport := IdentifyTransport("sea")
   // transport could be passed to a processing function of transport
   transport.Deliver()
   transport.Tax()
}
```

#### Builder Pattern Comparison
Technically it is okay to say that the Builder pattern is using Factory like function but the difference is in the ideology
- *Builder DP is used when an object cannot be produced in one single step.*
- *Factory Method DP is used to build one entire object in a single method call.*

### Abstract Factory  
Abstract Factory is a creational design pattern, which solves the problem of creating entire product families without specifying their concrete classes.

The abstract factory will help us create sets of products so that they would always match each other.

#### Usage
The pattern is best utilised when your system has to create multiple families of products 
or you want to provide a library of products without exposing the implementation details.

// TODO

### Prototype
// TODO


## Structural Patterns

### Adapter
### Decorator
### Facade


### Behavioral Patterns

### Chain Of Responsibility
*Separation of Concerns - Design Principle*
### Command
### Observer