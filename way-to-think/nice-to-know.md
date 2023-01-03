
## Increments and Decrements
Many languages have increment and decrement operators.
Unlike other languages, Go doesn't support the prefix version of the operations.
You also can't use these two operators in expressions.

```go
package main

import "fmt"

func main() {  
    data := []int{1,2,3}
    i := 0
    ++i //error
    fmt.Println(data[i++]) //error
}
```

## Unexported Structure Fields Are Not Encoded
The struct fields starting with lowercase letters will not be (json, xml, gob, etc.) encoded, so when you decode the structure you'll end up with zero values in those unexported fields.

```go
package main

import (  
    "fmt"
    "encoding/json"
)

type MyData struct {  
    One int
    two string
}

func main() {  
    in := MyData{1,"two"}
    fmt.Printf("%#v\n",in) //prints main.MyData{One:1, two:"two"}

    encoded,_ := json.Marshal(in)
    fmt.Println(string(encoded)) //prints {"One":1}

    var out MyData
    json.Unmarshal(encoded,&out)

    fmt.Printf("%#v\n",out) //prints main.MyData{One:1, two:""}
}
```

## JSON Encoder Adds a Newline Character
If you are using the JSON Encoder object then you'll get an extra newline character at the end of your encoded JSON object.
The JSON Encoder object is designed for streaming. Streaming with JSON usually means newline delimited JSON objects and this is why the Encode method adds a newline character. This is a documented behavior, but it's commonly overlooked or forgotten.

```go
package main

import (
  "fmt"
  "encoding/json"
  "bytes"
)

func main() {
  data := map[string]int{"key": 1}
  
  var b bytes.Buffer
  json.NewEncoder(&b).Encode(data)

  raw,_ := json.Marshal(data)
  
  if b.String() == string(raw) {
    fmt.Println("same encoded data")
  } else {
    fmt.Printf("'%s' != '%s'\n",raw,b.String())
    //prints:
    //'{"key":1}' != '{"key":1}\n'
  }
}
```

## JSON Package Escapes Special HTML Characters in Keys and String Values
This is a documented behavior, but you have to be careful reading all of the JSON package documentation to learn about it.
The `SetEscapeHTML` method description talks about the default encoding behavior for the and, less than and greater than characters.
**You can't disable this behavior for the json.Marshal**

```go
package main

import (
  "fmt"
  "encoding/json"
  "bytes"
)

func main() {
  data := "x < y"
  
  raw,_ := json.Marshal(data)
  fmt.Println(string(raw))
  //prints: "x \u003c y" <- probably not what you expected
  
  var b1 bytes.Buffer
  json.NewEncoder(&b1).Encode(data)
  fmt.Println(b1.String())
  //prints: "x \u003c y" <- probably not what you expected
  
  var b2 bytes.Buffer
  enc := json.NewEncoder(&b2)
  enc.SetEscapeHTML(false)
  enc.Encode(data)
  fmt.Println(b2.String())
  //prints: "x < y" <- looks better
}
```

## Comparing Struct, Arrays, Slice, Maps
:exclamation: **You can use the equality operator, ==, to compare struct variables if each structure field can be compared with the equality operator.**

The most generic solution is to use the `DeepEqual()` function in the reflect package. **Should be very cautious when using it**
- `DeepEqual()` doesn't consider an empty slice to be equal to a `nil` slice. This behavior is different from the behavior you get using the `bytes.Equal()` function. `bytes.Equal()` considers `nil` and empty slices to be equal.

`DeepEqual()` **isn't always perfect comparing slices.**
```go
package main

import (  
    "fmt"
    "reflect"
    "encoding/json"
)

func main() {  
    var str string = "one"
    var in interface{} = "one"
    fmt.Println("str == in:",str == in,reflect.DeepEqual(str, in)) 
    //prints: str == in: true true

    v1 := []string{"one","two"}
    v2 := []interface{}{"one","two"}
    fmt.Println("v1 == v2:",reflect.DeepEqual(v1, v2)) 
    //prints: v1 == v2: false (not ok)

    data := map[string]interface{}{
        "code": 200,
        "value": []string{"one","two"},
    }
    encoded, _ := json.Marshal(data)
    var decoded map[string]interface{}
    json.Unmarshal(encoded, &decoded)
    fmt.Println("data == decoded:",reflect.DeepEqual(data, decoded)) 
    //prints: data == decoded: false (not ok)
}
```