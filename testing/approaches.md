# Testing

## Different approaches evaluation 

**TDD/BDD** packages (Ginkgo, Gomega) bring new, unfamiliar control structures, increasing the cognitive burden on you and your future maintainers.

They just comply with some new theories which is correct and which could be better but in the end they does not bring enough value. 
Only more time to write test, to think more descriptive user scenarios (which sometimes is not necessary and difficult) and more maintaining.

When in Go there is **TDT (Table Driven Tests)**, do as Gophers do: we already have a language for writing simple.
It's simple to write them, simple to maintain and look very structured. No time waisting.

Of course there are cases where technically you would go for Ginko, Gomega as they have greater flexibility and features. 
Could be up to the team's preferences.


## Comparison

**Ginkgo & Gomega** 

```go
var _ = Describe("Shopping cart", func() {
	Context("initially", func() {
		It("has 0 items", func() {})
		It("has 0 units", func() {})
		Specify("the total amount is 0.00", func() {})
	})

	Context("when a new item is added", func() {
		Context("the shopping cart", func() {
			It("has 1 more unique item than it had earlier", func() {})
			It("has 1 more unit than it had earlier", func() {})
			Specify("total amount increases by item price", func() {})
		})
	})

	Context("when an existing item is added", func() {
		Context("the shopping cart", func() {
			It("has the same number of unique items as earlier", func() {})
			It("has 1 more unit than it had earlier", func() {})
			Specify("total amount increases by item price", func() {})
		})
	})

	Context("that has 0 unit of item A", func() {
		Context("removing item A", func() {
			It("should not change the number of items", func() {})
			It("should not change the number of units", func() {})
			It("should not change the amount", func() {})
		})
	})
```

The list could grow bigger and bigger if you have more scenarios and sure of it you would have.

**Go Table Driven Test**

```go
var testFunction = []struct {
	testName     type
	inputData    type
	expectedData type
	hasError     bool
}{
	{"Test name <the scenario which it describe, there are multiple test names conventions>", initInputData(), "outputdata", false}
	{"Test name <the scenario which it describe, there are multiple test names conventions>", "inputdata", "outputdata", true}
}

func TestFunction(t *testing.T) {
	for _, tt := range testParseEmoji {
		t.Run(tt.name, func(t *testing.T) {
			res, err := function(tt.inputData)
			validateError(t, tt.hasError, err)
			validateData(t, res, tt.expectedData)
		})
	}
}
```

As you see if you add more and more tests it would grow with 1, 2, 5, 10 lines depending on the number of tests.
This is not the case with the Ginkgo/Gomega approach. One test case can take up to 5 or 6 lines to start with.
In my point of view it is **much easier to read less and to know more** (saves time and consumes less energy :laughing:). 