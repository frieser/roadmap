#Golang
---
---

## Summary

Go provides built-in tools for exploring type documentation directly from the command line. The `go doc` command displays documentation for packages, types, functions, and methods. Combined with package documentation on pkg.go.dev, these tools help developers understand Go's type system and standard library without leaving the terminal.

## Detailed Explanation

### **go doc Command**

```bash
# Basic usage
go doc <package>
go doc <package>.<Symbol>
go doc <package>.<Type>.<Method>
```

### **Viewing Type Documentation**

#### Built-in Types

```bash
# View builtin package (predeclared types)
go doc builtin

# Specific built-in type
go doc builtin.int
go doc builtin.string
go doc builtin.error
go doc builtin.bool

# Built-in functions
go doc builtin.len
go doc builtin.make
go doc builtin.append
go doc builtin.copy
```

#### Standard Library Types

```bash
# fmt package
go doc fmt
go doc fmt.Println
go doc fmt.Sprintf

# strings package
go doc strings
go doc strings.Builder
go doc strings.Contains

# strconv package (type conversions)
go doc strconv
go doc strconv.Atoi
go doc strconv.ParseInt
go doc strconv.FormatFloat

# time package
go doc time.Time
go doc time.Duration
go doc time.Time.Format
```

### **go doc Flags**

```bash
# Show all documentation (not just exported)
go doc -all fmt

# Show source code
go doc -src fmt.Println

# Show unexported symbols too
go doc -u strings.Builder

# Show method documentation
go doc strings.Builder.WriteString

# Short output (one line per symbol)
go doc -short fmt
```

### **Numeric Types Documentation**

```bash
# Integer types
go doc builtin.int
go doc builtin.int8
go doc builtin.int64
go doc builtin.uint

# Float types
go doc builtin.float32
go doc builtin.float64

# Complex types
go doc builtin.complex64
go doc builtin.complex128

# Byte and rune
go doc builtin.byte   # Alias for uint8
go doc builtin.rune   # Alias for int32

# Math package for limits
go doc math.MaxInt64
go doc math.MaxFloat64
```

### **String-Related Documentation**

```bash
# String type
go doc builtin.string

# strings package
go doc strings
go doc strings.Contains
go doc strings.Split
go doc strings.Join
go doc strings.Builder
go doc strings.Reader

# strconv for conversions
go doc strconv
go doc strconv.Atoi
go doc strconv.Itoa
go doc strconv.ParseFloat

# unicode packages
go doc unicode
go doc unicode.IsLetter
go doc unicode.ToUpper
go doc unicode/utf8
go doc unicode/utf8.RuneCountInString
```

### **Exploring Package Contents**

```bash
# List all exported symbols in a package
go doc -all strconv | head -50

# Example output for strconv:
# package strconv // import "strconv"
#
# func Atoi(s string) (int, error)
# func FormatBool(b bool) string
# func FormatFloat(f float64, fmt byte, prec, bitSize int) string
# func FormatInt(i int64, base int) string
# ...
```

### **Practical Examples**

#### Understanding a Function

```bash
$ go doc strconv.ParseInt

func ParseInt(s string, base int, bitSize int) (i int64, err error)
    ParseInt interprets a string s in the given base (0, 2 to 36) and
    bit size (0 to 64) and returns the corresponding value i.

    The string may begin with a leading sign: "+" or "-".

    If the base argument is 0, the true base is implied by the string's
    prefix following the sign (if present): 2 for "0b", 8 for "0" or "0o",
    16 for "0x", and 10 otherwise.
```

#### Understanding a Type

```bash
$ go doc time.Duration

type Duration int64
    A Duration represents the elapsed time between two instants as an
    int64 nanosecond count. The representation limits the largest
    representable duration to approximately 290 years.

const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

#### Understanding a Method

```bash
$ go doc strings.Builder.WriteString

func (b *Builder) WriteString(s string) (int, error)
    WriteString appends the contents of s to b's buffer.
    It returns the length of s and a nil error.
```

### **Online Documentation**

```bash
# Open local documentation server
go doc -http=:6060
# Then visit http://localhost:6060 in browser

# Or use pkg.go.dev
# https://pkg.go.dev/std           # Standard library
# https://pkg.go.dev/fmt           # Specific package
# https://pkg.go.dev/strconv       # strconv package
```

### **Common Type Exploration Commands**

```bash
# Data types
go doc builtin.bool
go doc builtin.int
go doc builtin.float64
go doc builtin.string
go doc builtin.byte
go doc builtin.rune
go doc builtin.error

# Built-in functions for types
go doc builtin.len      # Length of arrays, slices, strings, maps
go doc builtin.cap      # Capacity of slices
go doc builtin.make     # Create slices, maps, channels
go doc builtin.new      # Allocate memory
go doc builtin.append   # Append to slices
go doc builtin.copy     # Copy slices
go doc builtin.delete   # Delete from maps
go doc builtin.close    # Close channels

# Type conversion packages
go doc strconv          # String conversions
go doc reflect          # Runtime type information
go doc unsafe           # Unsafe pointer operations

# Numeric operations
go doc math             # Math functions
go doc math/big         # Arbitrary-precision arithmetic
go doc math/cmplx       # Complex number functions
```

### **IDE Integration**

Most Go IDEs and editors provide hover documentation using the same source as `go doc`:

```go
// In VSCode with Go extension:
// Hover over any type/function to see documentation
// Press F12 to go to definition
// Press Ctrl+Shift+F12 to peek definition

import "strings"

func main() {
    // Hover over "Contains" to see:
    // func Contains(s, substr string) bool
    // Contains reports whether substr is within s.
    strings.Contains("hello", "ell")
}
```

### **Quick Reference Commands**

```bash
# What does this function do?
go doc fmt.Printf

# What methods does this type have?
go doc -all strings.Builder

# Show me the source code
go doc -src strings.Contains

# What's in this package?
go doc strings | head -30

# Find all Parse functions in strconv
go doc strconv | grep Parse

# Full documentation for a type
go doc -all time.Time
```

## Interview Questions

**Q: How do you view documentation for a Go type from the command line?**
**A:** Use `go doc <package>.<Type>`. For example, `go doc time.Time` shows the Time type documentation, and `go doc time.Time.Format` shows a specific method. Use `go doc -all <type>` for complete documentation including all methods.

**Q: Where can you find documentation for built-in types like int and string?**
**A:** Use `go doc builtin.<type>`. For example, `go doc builtin.int` or `go doc builtin.string`. The `builtin` package contains documentation for all predeclared identifiers including types, constants, and functions.

**Q: How do you explore what functions a package provides?**
**A:** Run `go doc <package>` for an overview, or `go doc -all <package>` for complete documentation. You can also pipe to grep: `go doc strconv | grep Parse` to find specific functions.

**Q: How do you view the source code of a standard library function?**
**A:** Use `go doc -src <package>.<function>`. For example, `go doc -src strings.Contains` shows the actual implementation. This is useful for understanding how functions work internally.
