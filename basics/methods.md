# Methods

## Values
```go
type Car struct {
	model string
	price int
}

// value receiver method will not change the price of the caller object
// the price will be changed only in the scope of the method
func (c Car) ChangePrice(price int) {
	c.price = price
}

func main() {
	car := Car{"audi", 40}
	fmt.Println(car.price) // 40
	car.ChangePrice(50)
	fmt.Println(car.price) // 40
}
```

## Pointers
```go
type Car struct {
	model string
	price int
}

// pointer receiver method will change the price of the caller object
func (c *Car) ChangePrice(price int) {
	c.price = price
}

func main() {
	car := Car{"audi", 40}
	fmt.Println(car.price) // 40
	car.ChangePrice(50)
	fmt.Println(car.price) // 50
}
```


