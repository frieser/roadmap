#Golang
---
---

## Summary

In Go, every variable is automatically initialized to its "zero value" when declared without an explicit value. This eliminates undefined behavior found in languages like C/C++. Zero values are type-specific: `0` for numbers, `false` for booleans, `""` for strings, and `nil` for pointers, slices, maps, channels, interfaces, and functions.

## Detailed Explanation

### **Zero Values by Type**

| Type Category | Types | Zero Value |
| --- | --- | --- |
| **Integers** | int, int8, int16, int32, int64, uint, etc. | `0` |
| **Floats** | float32, float64 | `0.0` |
| **Complex** | complex64, complex128 | `(0+0i)` |
| **Boolean** | bool | `false` |
| **String** | string | `""` (empty string) |
| **Pointer** | *T | `nil` |
| **Slice** | []T | `nil` |
| **Map** | map[K]V | `nil` |
| **Channel** | chan T | `nil` |
| **Function** | func() | `nil` |
| **Interface** | interface{}, any | `nil` |
| **Struct** | struct{} | Each field set to its zero value |
| **Array** | [N]T | Each element set to its zero value |

### **Numeric Zero Values**

```go
package main

import "fmt"

func main() {
    var i int       // 0
    var i8 int8     // 0
    var i64 int64   // 0
    var u uint      // 0
    var f32 float32 // 0
    var f64 float64 // 0
    var c64 complex64   // (0+0i)
    var c128 complex128 // (0+0i)
    
    fmt.Printf("int: %d, float64: %f, complex: %v\n", i, f64, c128)
    // Output: int: 0, float64: 0.000000, complex: (0+0i)
}
```

### **String and Boolean Zero Values**

```go
func main() {
    var s string  // "" (empty string, not nil)
    var b bool    // false
    
    fmt.Printf("string: %q, len: %d\n", s, len(s))  // string: "", len: 0
    fmt.Printf("bool: %t\n", b)                      // bool: false
    
    // Empty string is usable immediately
    s += "Hello"  // Works fine
}
```

### **Reference Type Zero Values (nil)**

```go
func main() {
    var p *int         // nil pointer
    var sl []int       // nil slice
    var m map[string]int // nil map
    var ch chan int    // nil channel
    var fn func()      // nil function
    var i interface{}  // nil interface
    
    // All are nil
    fmt.Println(p == nil)  // true
    fmt.Println(sl == nil) // true
    fmt.Println(m == nil)  // true
    fmt.Println(ch == nil) // true
    fmt.Println(fn == nil) // true
    fmt.Println(i == nil)  // true
}
```

### **Nil Slice vs Nil Map Behavior**

**Nil slices are safe to use in many operations:**

```go
func main() {
    var sl []int  // nil slice
    
    // Safe operations on nil slice
    fmt.Println(len(sl))    // 0
    fmt.Println(cap(sl))    // 0
    
    for _, v := range sl {  // Works (no iterations)
        fmt.Println(v)
    }
    
    sl = append(sl, 1, 2, 3)  // Works! Creates new slice
    fmt.Println(sl)           // [1 2 3]
}
```

**Nil maps will panic on write:**

```go
func main() {
    var m map[string]int  // nil map
    
    // Safe operations
    fmt.Println(len(m))      // 0
    val := m["key"]          // Returns zero value (0), no panic
    val, ok := m["key"]      // ok is false
    
    for k, v := range m {    // Works (no iterations)
        fmt.Println(k, v)
    }
    
    // PANIC! Cannot write to nil map
    // m["key"] = 1  // panic: assignment to entry in nil map
    
    // Must initialize first
    m = make(map[string]int)
    m["key"] = 1  // Now works
}
```

### **Struct Zero Values**

Each field gets its type's zero value:

```go
type User struct {
    Name    string
    Age     int
    Active  bool
    Manager *User
    Tags    []string
}

func main() {
    var u User
    
    fmt.Printf("Name: %q\n", u.Name)       // Name: ""
    fmt.Printf("Age: %d\n", u.Age)         // Age: 0
    fmt.Printf("Active: %t\n", u.Active)   // Active: false
    fmt.Printf("Manager: %v\n", u.Manager) // Manager: <nil>
    fmt.Printf("Tags: %v\n", u.Tags)       // Tags: []
    
    // Struct itself is not nil (it's a value type)
    // But pointer fields are nil
}
```

### **Array Zero Values**

Arrays are value types; each element is zeroed:

```go
func main() {
    var arr [5]int      // [0 0 0 0 0]
    var strs [3]string  // ["" "" ""]
    var bools [2]bool   // [false false]
    
    fmt.Println(arr)    // [0 0 0 0 0]
    fmt.Println(strs)   // [  ]
    fmt.Println(bools)  // [false false]
}
```

### **Practical Applications**

#### Check for Zero Values

```go
func main() {
    var name string
    var count int
    var prices []float64
    
    // Check if string is empty
    if name == "" {
        name = "default"
    }
    
    // Check if number is zero
    if count == 0 {
        count = 1
    }
    
    // Check if slice is nil or empty
    if len(prices) == 0 {
        prices = []float64{9.99}
    }
}
```

#### Using Zero Values as Defaults

```go
type Config struct {
    Host    string
    Port    int
    Timeout int  // Zero means "use default"
    Debug   bool
}

func NewServer(cfg Config) *Server {
    // Use zero values as signals for defaults
    if cfg.Host == "" {
        cfg.Host = "localhost"
    }
    if cfg.Port == 0 {
        cfg.Port = 8080
    }
    if cfg.Timeout == 0 {
        cfg.Timeout = 30
    }
    // Debug false by default is often desired
    
    return &Server{config: cfg}
}

// Usage: only specify what you need
server := NewServer(Config{Port: 3000})
// Host defaults to "localhost", Timeout to 30
```

#### Comma-ok Idiom

```go
func main() {
    m := map[string]int{"a": 1, "b": 0}
    
    // Problem: can't distinguish "key not found" from "value is 0"
    val := m["c"]  // Returns 0 (zero value)
    val2 := m["b"] // Also returns 0 (actual value)
    
    // Solution: comma-ok idiom
    val, ok := m["c"]
    if !ok {
        fmt.Println("key not found")
    }
    
    val2, ok := m["b"]
    if ok {
        fmt.Println("b exists with value:", val2)  // 0
    }
}
```

### **Zero Value Gotchas**

```go
// Gotcha 1: Nil map write panic
var m map[string]int
// m["key"] = 1  // PANIC

// Gotcha 2: Nil channel blocks forever
var ch chan int
// <-ch  // Blocks forever (deadlock)
// ch <- 1  // Blocks forever

// Gotcha 3: Nil function call panic
var fn func()
// fn()  // PANIC: nil pointer dereference

// Gotcha 4: Nil pointer dereference
var p *int
// *p = 10  // PANIC

// Gotcha 5: Interface nil comparison
var w io.Writer
var buf *bytes.Buffer
w = buf  // w holds (*bytes.Buffer, nil)
fmt.Println(w == nil)  // false! (interface has type info)
```

## Interview Questions

**Q: What is a zero value in Go?**
**A:** A zero value is the default value automatically assigned to a variable when declared without explicit initialization. Go guarantees every variable is initialized, eliminating undefined behavior. Zero values are type-specific: `0` for numbers, `false` for booleans, `""` for strings, and `nil` for reference types (pointers, slices, maps, channels, functions, interfaces).

**Q: What is the difference between a nil slice and an empty slice?**
**A:** A nil slice has no underlying array (`nil`), while an empty slice has an array but with zero length. Both have `len() == 0` and behave identically in most operations (range, append). The difference matters when serializing to JSON: nil becomes `null`, empty becomes `[]`. Create empty slice with `make([]int, 0)` or `[]int{}`.

**Q: Why does writing to a nil map panic but writing to a nil slice works?**
**A:** `append` on a nil slice allocates a new underlying array and returns a new slice, so the nil slice is replaced. Maps, however, need an initialized hash table for write operations. Reading from a nil map returns zero values safely, but writing requires `make(map[K]V)` first.

**Q: How do you distinguish between "key not found" and "value is zero" in a map?**
**A:** Use the comma-ok idiom: `value, ok := m[key]`. If `ok` is false, the key doesn't exist. If `ok` is true, the key exists even if `value` is the zero value. This is essential when zero is a valid value in your map.
