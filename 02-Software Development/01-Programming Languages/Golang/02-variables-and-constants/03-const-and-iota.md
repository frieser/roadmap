#Golang
---
---

## Summary

Go constants are immutable values known at compile time, declared with the `const` keyword. They can be typed or untyped, with untyped constants having higher precision and flexibility. The `iota` identifier is a powerful enumerator that auto-increments within constant blocks, enabling elegant enum patterns without repetitive code.

## Detailed Explanation

### **Declaring Constants**

```go
// Single constant
const Pi = 3.14159

// Multiple constants
const (
    StatusOK    = 200
    StatusError = 500
)

// Typed constant
const MaxSize int = 1024

// Untyped constant (more flexible)
const MaxValue = 1000
```

### **Typed vs Untyped Constants**

**Untyped constants** have no fixed type until used:

```go
const Big = 1000000000000  // Untyped, high precision

func main() {
    var i int = Big       // Works
    var f float64 = Big   // Works
    var i32 int32 = Big   // Works (if value fits)
    
    fmt.Printf("%T\n", Big * 2)  // int (type determined by context)
}
```

**Typed constants** have a fixed type:

```go
const TypedBig int64 = 1000000000000

func main() {
    var i int = TypedBig     // Error: cannot use int64 as int
    var f float64 = TypedBig // Error: cannot use int64 as float64
    var i64 int64 = TypedBig // Works
}
```

### **Constant Expressions**

Constants can be computed at compile time:

```go
const (
    KB = 1024
    MB = KB * 1024      // 1048576
    GB = MB * 1024      // 1073741824
    TB = GB * 1024      // 1099511627776
)

const (
    Pi      = 3.14159265358979323846
    TwoPi   = Pi * 2
    HalfPi  = Pi / 2
    DegToRad = Pi / 180
)

// String concatenation
const (
    Prefix = "api"
    Version = "v1"
    Endpoint = Prefix + "/" + Version  // "api/v1"
)
```

### **The iota Identifier**

`iota` is a compile-time counter that starts at 0 and increments by 1 for each constant in a block:

```go
const (
    Zero  = iota  // 0
    One   = iota  // 1
    Two   = iota  // 2
    Three = iota  // 3
)

// Simplified (iota reused implicitly)
const (
    Zero  = iota  // 0
    One           // 1 (iota continues)
    Two           // 2
    Three         // 3
)
```

### **iota Patterns**

#### Basic Enumeration

```go
type Status int

const (
    StatusPending Status = iota  // 0
    StatusActive                 // 1
    StatusComplete               // 2
    StatusFailed                 // 3
)

func (s Status) String() string {
    return [...]string{"Pending", "Active", "Complete", "Failed"}[s]
}
```

#### Skip Values with Blank Identifier

```go
const (
    _  = iota  // Skip 0
    KB = 1 << (10 * iota)  // 1 << 10 = 1024
    MB                      // 1 << 20 = 1048576
    GB                      // 1 << 30 = 1073741824
    TB                      // 1 << 40 = 1099511627776
)
```

#### Bit Flags (Powers of 2)

```go
type Permission uint

const (
    Read    Permission = 1 << iota  // 1 (001)
    Write                           // 2 (010)
    Execute                         // 4 (100)
)

func main() {
    // Combine with bitwise OR
    rwx := Read | Write | Execute  // 7 (111)
    rw := Read | Write             // 3 (011)
    
    // Check with bitwise AND
    if rwx&Execute != 0 {
        fmt.Println("Has execute permission")
    }
}
```

#### Starting from Non-Zero

```go
const (
    Sunday = iota + 1  // 1
    Monday             // 2
    Tuesday            // 3
    Wednesday          // 4
    Thursday           // 5
    Friday             // 6
    Saturday           // 7
)
```

#### Complex Expressions

```go
const (
    _  = iota             // 0 (skipped)
    KB = 1 << (10 * iota) // 1 << 10
    MB                    // 1 << 20
    GB                    // 1 << 30
)

// Multiple constants per line
const (
    bit0, mask0 = 1 << iota, 1<<iota - 1  // 1, 0
    bit1, mask1                            // 2, 1
    bit2, mask2                            // 4, 3
    bit3, mask3                            // 8, 7
)
```

#### Reset iota in New Block

```go
const (
    a = iota  // 0
    b         // 1
    c         // 2
)

const (
    x = iota  // 0 (reset!)
    y         // 1
    z         // 2
)
```

### **Practical Enum Pattern**

```go
type LogLevel int

const (
    LogDebug LogLevel = iota
    LogInfo
    LogWarning
    LogError
    LogFatal
)

// Stringer interface for nice printing
func (l LogLevel) String() string {
    switch l {
    case LogDebug:
        return "DEBUG"
    case LogInfo:
        return "INFO"
    case LogWarning:
        return "WARNING"
    case LogError:
        return "ERROR"
    case LogFatal:
        return "FATAL"
    default:
        return fmt.Sprintf("LogLevel(%d)", l)
    }
}

// Usage
func main() {
    level := LogWarning
    fmt.Println(level)  // WARNING
    
    if level >= LogError {
        fmt.Println("High severity!")
    }
}
```

### **Typed Enums with Validation**

```go
type Color int

const (
    ColorRed Color = iota
    ColorGreen
    ColorBlue
    colorEnd  // Unexported sentinel
)

func (c Color) Valid() bool {
    return c >= ColorRed && c < colorEnd
}

func ParseColor(s string) (Color, error) {
    switch s {
    case "red":
        return ColorRed, nil
    case "green":
        return ColorGreen, nil
    case "blue":
        return ColorBlue, nil
    default:
        return 0, fmt.Errorf("unknown color: %s", s)
    }
}
```

### **Constants vs Variables**

| Aspect | Constants | Variables |
| --- | --- | --- |
| **Mutability** | Immutable | Mutable |
| **Evaluation** | Compile time | Runtime |
| **Types** | Basic types only | Any type |
| **Memory** | No runtime allocation | Allocated at runtime |
| **Use in expressions** | Substituted inline | Referenced by address |

```go
// Constants: compile-time
const MaxItems = 100
const Greeting = "Hello"

// Variables: runtime (even if never changed)
var config = loadConfig()
var now = time.Now()

// What CAN'T be a constant
// const arr = [3]int{1,2,3}  // Error: array literal
// const slice = []int{1,2,3} // Error: slice literal
// const m = map[string]int{} // Error: map literal
// const p = &value           // Error: pointer
```

### **Best Practices**

```go
// ✓ Good: Group related constants
const (
    StatusPending = iota
    StatusActive
    StatusDone
)

// ✓ Good: Use typed constants for type safety
type Direction int

const (
    North Direction = iota
    East
    South
    West
)

// ✓ Good: Document the enum
// HTTPStatus represents standard HTTP response codes.
const (
    HTTPStatusOK       = 200
    HTTPStatusCreated  = 201
    HTTPStatusNotFound = 404
)

// ✗ Avoid: Magic numbers
// if status == 2 {  // What does 2 mean?
// ✓ Better: Named constant
// if status == StatusDone {
```

## Interview Questions

**Q: What is the difference between typed and untyped constants in Go?**
**A:** Untyped constants have no fixed type and high precision, allowing them to be used in contexts requiring different types (e.g., `const Big = 1000` can be assigned to int, float64, or int32). Typed constants (e.g., `const Max int = 100`) have a fixed type and require explicit conversion for other types.

**Q: What is `iota` and how does it work?**
**A:** `iota` is a compile-time counter that starts at 0 and increments by 1 for each constant specification within a `const` block. It resets to 0 in each new `const` block. It's commonly used to create enumerations and bit flags without manually specifying values.

**Q: How do you create bit flags using `iota`?**
**A:** Use bit shifting with iota: `const (Read = 1 << iota; Write; Execute)`. This creates Read=1, Write=2, Execute=4. Combine flags with bitwise OR (`Read | Write`) and check with AND (`flags & Read != 0`).

**Q: Why can't slices, maps, or arrays be constants?**
**A:** Constants must be evaluable at compile time with a fixed value. Slices, maps, and arrays are reference types that require runtime memory allocation and can be modified. Only basic types (numbers, strings, booleans) and expressions combining them can be constants.
