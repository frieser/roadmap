#Golang
---
---

## Summary

Interpreted string literals in Go are enclosed in double quotes (`"`) and process escape sequences like `\n`, `\t`, `\\`, and Unicode escapes. They are the standard string type for most use cases, especially when you need to embed special characters, use escape sequences, or work with single-line strings. Unlike raw strings, interpreted strings cannot span multiple lines directly.

## Detailed Explanation

### **Basic Syntax**

```go
package main

import "fmt"

func main() {
    // Basic interpreted string
    greeting := "Hello, World!"
    
    // With escape sequences
    withNewline := "Line 1\nLine 2"
    withTab := "Column1\tColumn2"
    
    fmt.Println(greeting)
    fmt.Println(withNewline)
    fmt.Println(withTab)
}
```

### **Escape Sequences**

| Escape | Description | Example |
| --- | --- | --- |
| `\n` | Newline | `"Line1\nLine2"` |
| `\r` | Carriage return | `"Hello\rWorld"` |
| `\t` | Horizontal tab | `"Col1\tCol2"` |
| `\\` | Backslash | `"C:\\Users"` |
| `\"` | Double quote | `"He said \"Hi\""` |
| `\'` | Single quote | `"It\'s"` (not needed in strings) |
| `\a` | Alert (bell) | `"\a"` |
| `\b` | Backspace | `"Back\bspace"` |
| `\f` | Form feed | `"\f"` |
| `\v` | Vertical tab | `"\v"` |

### **Unicode Escape Sequences**

```go
func main() {
    // \xNN - hexadecimal byte value (00-FF)
    hex := "\x48\x65\x6c\x6c\x6f"  // "Hello"
    fmt.Println(hex)
    
    // \uNNNN - Unicode code point (4 hex digits)
    unicode4 := "\u0048\u0065\u006c\u006c\u006f"  // "Hello"
    fmt.Println(unicode4)
    
    // \UNNNNNNNN - Unicode code point (8 hex digits)
    unicode8 := "\U00000048\U00000065\U0000006c\U0000006c\U0000006f"  // "Hello"
    fmt.Println(unicode8)
    
    // Practical examples
    omega := "\u03A9"        // Ω
    heart := "\u2764"        // ❤
    rocket := "\U0001F680"   // 🚀 (requires 8-digit form)
    
    fmt.Printf("%s %s %s\n", omega, heart, rocket)
    
    // \NNN - octal byte value (000-377)
    octal := "\110\145\154\154\157"  // "Hello"
    fmt.Println(octal)
}
```

### **Common Escape Patterns**

```go
func main() {
    // Quotes inside strings
    quote1 := "She said, \"Hello!\""
    quote2 := "It's a beautiful day"  // Single quote doesn't need escaping
    
    fmt.Println(quote1)  // She said, "Hello!"
    fmt.Println(quote2)  // It's a beautiful day
    
    // File paths (Windows)
    path := "C:\\Users\\name\\Documents\\file.txt"
    fmt.Println(path)  // C:\Users\name\Documents\file.txt
    
    // JSON with quotes
    json := "{\"name\": \"John\", \"age\": 30}"
    fmt.Println(json)  // {"name": "John", "age": 30}
    
    // Combining escapes
    table := "Name\tAge\tCity\nAlice\t30\tNYC\nBob\t25\tLA"
    fmt.Println(table)
    // Name    Age     City
    // Alice   30      NYC
    // Bob     25      LA
}
```

### **Multi-Line Strings**

```go
func main() {
    // Cannot have literal newlines in interpreted strings
    // invalid := "Line 1
    // Line 2"  // Syntax error!
    
    // Option 1: Use \n
    multiline := "Line 1\nLine 2\nLine 3"
    
    // Option 2: Concatenate strings
    multiline2 := "Line 1\n" +
                  "Line 2\n" +
                  "Line 3"
    
    // Option 3: Use raw strings for multi-line (see other note)
    
    fmt.Println(multiline)
    fmt.Println(multiline2)
}
```

### **String Concatenation**

```go
func main() {
    // Using + operator
    first := "Hello"
    second := "World"
    combined := first + ", " + second + "!"
    fmt.Println(combined)  // Hello, World!
    
    // Using += operator
    var result string
    result += "Hello"
    result += " "
    result += "World"
    fmt.Println(result)  // Hello World
    
    // Using fmt.Sprintf
    name := "Alice"
    age := 30
    message := fmt.Sprintf("Name: %s, Age: %d", name, age)
    fmt.Println(message)  // Name: Alice, Age: 30
    
    // Using strings.Builder (efficient for many concatenations)
    import "strings"
    var builder strings.Builder
    builder.WriteString("Hello")
    builder.WriteString(" ")
    builder.WriteString("World")
    fmt.Println(builder.String())  // Hello World
}
```

### **String Immutability**

```go
func main() {
    s := "Hello"
    
    // Strings are immutable
    // s[0] = 'h'  // Error: cannot assign to s[0]
    
    // To modify, convert to []byte or []rune
    bytes := []byte(s)
    bytes[0] = 'h'
    s = string(bytes)
    fmt.Println(s)  // hello
    
    // Or create a new string
    s = "h" + s[1:]
    fmt.Println(s)  // hello
}
```

### **String Comparison**

```go
func main() {
    a := "apple"
    b := "banana"
    c := "apple"
    
    // Equality
    fmt.Println(a == c)  // true
    fmt.Println(a == b)  // false
    
    // Lexicographic comparison
    fmt.Println(a < b)   // true (a comes before b)
    fmt.Println(a > b)   // false
    
    // Case-insensitive comparison
    import "strings"
    s1 := "Hello"
    s2 := "hello"
    fmt.Println(strings.EqualFold(s1, s2))  // true
}
```

### **Empty and Zero Strings**

```go
func main() {
    // Empty string literal
    empty := ""
    
    // Zero value is also empty string
    var zeroValue string
    
    fmt.Println(empty == zeroValue)  // true
    fmt.Println(len(empty))          // 0
    
    // Check for empty string
    if s := ""; s == "" {
        fmt.Println("Empty string")
    }
    
    // Or check length
    if len(s) == 0 {
        fmt.Println("Empty string")
    }
}
```

### **Formatting Strings**

```go
func main() {
    s := "Hello, World!"
    
    // Default format
    fmt.Printf("%s\n", s)     // Hello, World!
    
    // Quoted string
    fmt.Printf("%q\n", s)     // "Hello, World!"
    
    // Width and alignment
    fmt.Printf("|%20s|\n", s)   // |       Hello, World!| (right-aligned)
    fmt.Printf("|%-20s|\n", s)  // |Hello, World!       | (left-aligned)
    
    // With escape sequences shown
    newline := "Line1\nLine2"
    fmt.Printf("%q\n", newline)  // "Line1\nLine2"
    
    // Type
    fmt.Printf("%T\n", s)  // string
}
```

### **Comparison: Interpreted vs Raw**

```go
func main() {
    // Same content, different syntax
    interpreted := "Line 1\nLine 2\tTabbed\\Backslash"
    
    raw := `Line 1
Line 2	Tabbed\Backslash`
    
    fmt.Println("Interpreted:")
    fmt.Println(interpreted)
    
    fmt.Println("\nRaw:")
    fmt.Println(raw)
    
    // Both produce identical output
}
```

### **Best Practices**

```go
// ✓ Good: Use interpreted strings for simple strings
message := "Hello, World!"

// ✓ Good: Use escape sequences for special characters
withQuote := "She said \"Hi\""
path := "C:\\Users\\name"

// ✓ Good: Use fmt.Sprintf for complex formatting
result := fmt.Sprintf("User %s has %d items", name, count)

// ✓ Good: Use strings.Builder for many concatenations
var b strings.Builder
for _, item := range items {
    b.WriteString(item)
}

// ✗ Avoid: Using interpreted strings for multi-line (hard to read)
bad := "Line 1\nLine 2\nLine 3\nLine 4"

// ✓ Better: Use raw strings for multi-line
good := `Line 1
Line 2
Line 3
Line 4`

// ✗ Avoid: Using interpreted strings for regex
badRegex := "\\d{3}-\\d{3}-\\d{4}"

// ✓ Better: Use raw strings for regex
goodRegex := `\d{3}-\d{3}-\d{4}`
```

## Interview Questions

**Q: What is an interpreted string literal in Go?**
**A:** An interpreted string literal is enclosed in double quotes (`"`) and processes escape sequences like `\n` (newline), `\t` (tab), `\\` (backslash), and Unicode escapes. It's the standard string type for most use cases, especially for single-line strings with special characters.

**Q: How do you include a double quote inside an interpreted string?**
**A:** Use the escape sequence `\"`. For example: `"She said \"Hello!\""` produces `She said "Hello!"`. Backslashes must also be escaped with `\\`.

**Q: What Unicode escape sequences does Go support?**
**A:** Go supports three forms: `\xNN` for byte values (2 hex digits), `\uNNNN` for Unicode code points up to U+FFFF (4 hex digits), and `\UNNNNNNNN` for any Unicode code point including emoji (8 hex digits).

**Q: When should you use interpreted strings vs raw strings?**
**A:** Use interpreted strings for: single-line strings, strings needing escape sequences (\n, \t), and simple text. Use raw strings for: multi-line content, regex patterns, SQL queries, file paths, and content with many backslashes or special characters.
