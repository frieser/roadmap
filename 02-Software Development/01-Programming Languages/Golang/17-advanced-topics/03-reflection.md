# Reflection

## Summary
Reflection allows a program to inspect and manipulate its own structure (types and values) at runtime. The `reflect` package in Go provides the ability to examine types, kinds, and values of interfaces, enabling the creation of generic libraries like JSON serializers or ORMs.

## Detailed Explanation

### Core Concepts
Reflection is built around two key types in the `reflect` package:
1.  **`reflect.Type`**: Describes the metadata of a variable (name, package, fields, methods). Obtained via `reflect.TypeOf()`.
2.  **`reflect.Value`**: Holds the actual data and allows reading/writing. Obtained via `reflect.ValueOf()`.

### The Three Laws of Reflection
1.  Reflection goes from Interface value to Reflection object.
2.  Reflection goes from Reflection object back to Interface value.
3.  To modify a reflection object, the value must be settable (pointer).

### Pros and Cons
*   **Pros**: Enables dynamic code (serialization, mocks, dependency injection).
*   **Cons**:
    *   **Performance**: Much slower than direct execution.
    *   **Safety**: Errors happen at runtime (panics), not compile time.
    *   **Complexity**: Code is hard to read and maintain.

### Code Example: Inspecting and Modifying

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    ID   int
    Name string
}

func main() {
    u := User{ID: 1, Name: "Alice"}
    
    // Inspect Type and Value
    t := reflect.TypeOf(u)
    v := reflect.ValueOf(u)

    fmt.Printf("Type: %s\n", t.Name())
    fmt.Printf("Kind: %s\n", t.Kind()) // struct

    // Iterate over fields
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("%s: %v (%s)\n", field.Name, value, field.Type)
    }

    // Modifying a value (Must pass pointer!)
    x := 10
    // vPtr := reflect.ValueOf(x) // Wrong: Not settable
    vPtr := reflect.ValueOf(&x).Elem() // Get the element pointed to
    
    if vPtr.CanSet() {
        vPtr.SetInt(20)
    }
    fmt.Println("x is now:", x)
}
```

## Interview Questions

**Q: What is the `reflect` package used for?**
**A:** It allows runtime inspection of types and values, mostly used for implementing libraries like `encoding/json`, `fmt`, or ORMs that need to handle unknown types dynamically.

**Q: Why is reflection discouraged for general application logic?**
**A:** It breaks type safety (compiler can't catch errors), it's slow, and it makes code difficult to read and debug.

**Q: How do you modify a variable using reflection?**
**A:** You must pass a pointer to `reflect.ValueOf()`, and then call `.Elem()` to get the actual settable value.
