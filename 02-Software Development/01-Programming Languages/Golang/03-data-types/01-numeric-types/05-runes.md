#Golang
---
---

## Summary

A `rune` in Go is an alias for `int32` and represents a Unicode code point. While strings in Go are UTF-8 encoded byte sequences, runes allow you to work with individual Unicode characters correctly, including multi-byte characters like emojis and non-Latin scripts. Understanding the difference between bytes and runes is essential for proper string manipulation in Go.

## Detailed Explanation

### **What is a Rune?**

```go
package main

import "fmt"

func main() {
    // rune is an alias for int32
    var r rune = 'A'
    
    fmt.Printf("Rune: %c\n", r)        // A
    fmt.Printf("Unicode: U+%04X\n", r) // U+0041
    fmt.Printf("Decimal: %d\n", r)     // 65
    fmt.Printf("Type: %T\n", r)        // int32
    
    // Multi-byte characters
    heart := '❤'
    fmt.Printf("Heart: %c, Code: U+%04X\n", heart, heart)  // ❤, U+2764
    
    emoji := '🚀'
    fmt.Printf("Emoji: %c, Code: U+%04X\n", emoji, emoji)  // 🚀, U+1F680
}
```

### **Bytes vs Runes**

```go
func main() {
    s := "Hello, 世界"
    
    // len() returns byte count, not character count
    fmt.Println("Byte length:", len(s))  // 13 (not 9!)
    
    // Break down:
    // "Hello, " = 7 bytes (ASCII, 1 byte each)
    // "世" = 3 bytes (UTF-8)
    // "界" = 3 bytes (UTF-8)
    // Total: 7 + 3 + 3 = 13 bytes
    
    // To get character count, convert to []rune
    runes := []rune(s)
    fmt.Println("Rune count:", len(runes))  // 9
    
    // Or use utf8.RuneCountInString
    import "unicode/utf8"
    fmt.Println("Rune count:", utf8.RuneCountInString(s))  // 9
}
```

### **Iterating Over Strings**

```go
func main() {
    s := "Go语言"
    
    // By bytes (WRONG for Unicode)
    fmt.Println("By bytes:")
    for i := 0; i < len(s); i++ {
        fmt.Printf("%d: %x\n", i, s[i])
    }
    // 0: 47 (G)
    // 1: 6f (o)
    // 2: e8 (first byte of 语)
    // 3: af (second byte of 语)
    // 4: ad (third byte of 语)
    // 5: e8 (first byte of 言)
    // ...
    
    // By runes (CORRECT)
    fmt.Println("\nBy runes:")
    for i, r := range s {
        fmt.Printf("Index %d: %c (U+%04X)\n", i, r, r)
    }
    // Index 0: G (U+0047)
    // Index 1: o (U+006F)
    // Index 2: 语 (U+8BED)
    // Index 5: 言 (U+8A00)  // Note: index jumps!
}
```

### **Rune Literals**

```go
func main() {
    // Single quotes for rune literals
    a := 'A'       // 65
    newline := '\n' // 10
    tab := '\t'     // 9
    
    // Unicode escape sequences
    omega := '\u03A9'      // Ω (U+03A9)
    snowman := '\U0001F3C2' // 🏂 (U+1F3C2)
    
    // Octal (limited to \000-\377)
    bell := '\007'
    
    // Hexadecimal (limited to \x00-\xFF)
    hexA := '\x41'  // 'A'
    
    fmt.Printf("%c %c %c\n", omega, snowman, hexA)
}
```

### **String and Rune Conversion**

```go
func main() {
    // String to []rune
    s := "café"
    runes := []rune(s)
    fmt.Println(runes)  // [99 97 102 233]
    
    // []rune to string
    newStr := string(runes)
    fmt.Println(newStr)  // café
    
    // Single rune to string
    r := '世'
    str := string(r)
    fmt.Println(str)  // 世
    
    // Integer to rune/string
    n := 65
    fmt.Println(string(rune(n)))  // A
    
    // String to []byte
    bytes := []byte(s)
    fmt.Println(bytes)  // [99 97 102 195 169] - UTF-8 bytes
}
```

### **Working with Runes**

```go
import (
    "fmt"
    "unicode"
    "unicode/utf8"
)

func main() {
    // Check rune properties
    r := 'A'
    
    fmt.Println(unicode.IsLetter(r))  // true
    fmt.Println(unicode.IsDigit(r))   // false
    fmt.Println(unicode.IsUpper(r))   // true
    fmt.Println(unicode.IsLower(r))   // false
    fmt.Println(unicode.IsSpace(' ')) // true
    fmt.Println(unicode.IsPunct('.')) // true
    
    // Case conversion
    fmt.Printf("%c\n", unicode.ToLower('A'))  // a
    fmt.Printf("%c\n", unicode.ToUpper('a'))  // A
    fmt.Printf("%c\n", unicode.ToTitle('a'))  // A
    
    // Check if valid rune
    fmt.Println(utf8.ValidRune(r))        // true
    fmt.Println(utf8.ValidRune(-1))       // false
    fmt.Println(utf8.ValidRune(0x110000)) // false (above Unicode range)
}
```

### **Rune Manipulation in Strings**

```go
func main() {
    s := "Hello, 世界!"
    
    // Get first rune
    r, size := utf8.DecodeRuneInString(s)
    fmt.Printf("First rune: %c (size: %d bytes)\n", r, size)
    
    // Get last rune
    r, size = utf8.DecodeLastRuneInString(s)
    fmt.Printf("Last rune: %c (size: %d bytes)\n", r, size)
    
    // Count runes
    count := utf8.RuneCountInString(s)
    fmt.Printf("Rune count: %d\n", count)  // 10
    
    // Reverse a string properly
    reversed := reverseString(s)
    fmt.Println(reversed)  // !界世 ,olleH
}

func reverseString(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}
```

### **The utf8 Package**

```go
import "unicode/utf8"

func main() {
    s := "Hello, 世界"
    
    // Count runes
    n := utf8.RuneCountInString(s)
    fmt.Println("Runes:", n)  // 9
    
    // Validate UTF-8
    fmt.Println(utf8.ValidString(s))  // true
    
    // Invalid UTF-8
    invalid := string([]byte{0xff, 0xfe})
    fmt.Println(utf8.ValidString(invalid))  // false
    
    // Encode rune to bytes
    buf := make([]byte, 4)
    n = utf8.EncodeRune(buf, '世')
    fmt.Printf("Encoded %d bytes: %v\n", n, buf[:n])
    
    // Decode bytes to rune
    r, size := utf8.DecodeRune(buf)
    fmt.Printf("Decoded: %c (size: %d)\n", r, size)
    
    // RuneLen: bytes needed for a rune
    fmt.Println(utf8.RuneLen('A'))  // 1
    fmt.Println(utf8.RuneLen('世')) // 3
    fmt.Println(utf8.RuneLen('🚀')) // 4
}
```

### **Common Patterns**

#### Safe String Indexing

```go
func getRuneAt(s string, index int) (rune, bool) {
    for i, r := range s {
        if i == index {
            return r, true
        }
    }
    return 0, false
}

// Or convert to slice (more memory, O(1) access)
func getRuneAtFast(s string, index int) rune {
    runes := []rune(s)
    if index < 0 || index >= len(runes) {
        return utf8.RuneError
    }
    return runes[index]
}
```

#### Truncate String by Runes

```go
func truncateRunes(s string, max int) string {
    if utf8.RuneCountInString(s) <= max {
        return s
    }
    
    runes := []rune(s)
    return string(runes[:max]) + "..."
}

func main() {
    s := "Hello, 世界! How are you?"
    fmt.Println(truncateRunes(s, 10))  // Hello, 世界!...
}
```

### **Rune Error Handling**

```go
import "unicode/utf8"

func main() {
    // Invalid UTF-8 produces RuneError (U+FFFD)
    invalid := []byte{0xff, 0xfe}
    r, _ := utf8.DecodeRune(invalid)
    
    if r == utf8.RuneError {
        fmt.Println("Invalid UTF-8 sequence")
    }
    
    // Range also produces RuneError for invalid sequences
    s := string(invalid)
    for _, r := range s {
        fmt.Printf("%U ", r)  // U+FFFD U+FFFD
    }
}
```

## Interview Questions

**Q: What is a rune in Go?**
**A:** A `rune` is an alias for `int32` that represents a Unicode code point. While strings in Go are UTF-8 encoded byte sequences, runes allow you to work with individual Unicode characters, including multi-byte characters like Chinese characters or emojis.

**Q: Why does `len("世界")` return 6 instead of 2?**
**A:** `len()` returns the byte length, not the character count. In UTF-8, each Chinese character uses 3 bytes, so "世界" is 6 bytes. To get the character count, use `utf8.RuneCountInString()` or `len([]rune(s))`.

**Q: How do you correctly iterate over Unicode characters in a string?**
**A:** Use `for i, r := range s` which decodes UTF-8 and yields runes. Don't use `for i := 0; i < len(s); i++` which iterates over bytes and will produce garbage for multi-byte characters.

**Q: What is the difference between single quotes and double quotes in Go?**
**A:** Single quotes (`'A'`) create a rune literal (int32 value of the Unicode code point). Double quotes (`"A"`) create a string literal. A string is a sequence of bytes, while a rune is a single Unicode code point.
