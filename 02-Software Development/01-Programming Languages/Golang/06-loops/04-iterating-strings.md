#Golang
---
---

## Summary

Iterating over strings in Go requires understanding that strings are UTF-8 encoded byte sequences. Using a traditional `for` loop iterates over bytes, while `for range` iterates over runes (Unicode code points). This distinction is crucial when working with non-ASCII characters, as a single character might span multiple bytes.

## Detailed Explanation

### **Two Ways to Iterate Strings**

```go
package main

import "fmt"

func main() {
    s := "Hello, 世界"
    
    // By bytes (traditional for)
    fmt.Println("By bytes:")
    for i := 0; i < len(s); i++ {
        fmt.Printf("  %d: %x (%c)\n", i, s[i], s[i])
    }
    
    // By runes (for range)
    fmt.Println("\nBy runes:")
    for i, r := range s {
        fmt.Printf("  %d: %U (%c)\n", i, r, r)
    }
}

// Output:
// By bytes:
//   0: 48 (H)
//   1: 65 (e)
//   2: 6c (l)
//   ...
//   7: e4 (ä)  <- First byte of 世
//   8: b8 (¸)  <- Second byte of 世
//   9: 96 (?)  <- Third byte of 世
//   ...

// By runes:
//   0: U+0048 (H)
//   1: U+0065 (e)
//   ...
//   7: U+4E16 (世)  <- Index 7, but next is 10!
//   10: U+754C (界)
```

### **Byte Iteration**

```go
func main() {
    s := "café"
    
    // Iterating bytes
    for i := 0; i < len(s); i++ {
        fmt.Printf("byte[%d] = %x (%c)\n", i, s[i], s[i])
    }
    // Output:
    // byte[0] = 63 (c)
    // byte[1] = 61 (a)
    // byte[2] = 66 (f)
    // byte[3] = c3 (Ã)  <- First byte of é
    // byte[4] = a9 (©)  <- Second byte of é
    
    // Note: len("café") is 5 (bytes), not 4 (characters)!
    fmt.Println("Length:", len(s))  // 5
}
```

### **Rune Iteration with Range**

```go
func main() {
    s := "café"
    
    // Range decodes UTF-8 automatically
    for i, r := range s {
        fmt.Printf("rune[%d] = %U (%c)\n", i, r, r)
    }
    // Output:
    // rune[0] = U+0063 (c)
    // rune[1] = U+0061 (a)
    // rune[2] = U+0066 (f)
    // rune[3] = U+00E9 (é)  <- Single rune, index 3
    
    // Index jumps: 0, 1, 2, 3 (not 4!)
    // Because é is at byte index 3, next char would be at 5
}
```

### **Index Behavior with Range**

```go
func main() {
    s := "a中b"
    
    for i, r := range s {
        fmt.Printf("Index %d: %c (size: %d bytes)\n", i, r, len(string(r)))
    }
    // Output:
    // Index 0: a (size: 1 bytes)
    // Index 1: 中 (size: 3 bytes)
    // Index 4: b (size: 1 bytes)
    
    // Notice: Index jumps from 1 to 4 (skipping 2, 3)
    // The index is the BYTE position, not rune position
}
```

### **Getting Rune Position vs Byte Position**

```go
import "unicode/utf8"

func main() {
    s := "Hello, 世界"
    
    // Byte count
    fmt.Println("Byte length:", len(s))  // 13
    
    // Rune count
    fmt.Println("Rune count:", utf8.RuneCountInString(s))  // 9
    
    // Track rune position manually
    runePos := 0
    for bytePos, r := range s {
        fmt.Printf("Rune %d at byte %d: %c\n", runePos, bytePos, r)
        runePos++
    }
}
```

### **Converting to []rune for Random Access**

```go
func main() {
    s := "Hello, 世界"
    
    // ✗ Byte access (wrong for Unicode)
    // fmt.Println(string(s[7]))  // Garbage - partial UTF-8
    
    // ✓ Convert to rune slice
    runes := []rune(s)
    fmt.Println(string(runes[7]))  // 世 (correct!)
    
    // Now you can access by rune index
    for i, r := range runes {
        fmt.Printf("runes[%d] = %c\n", i, r)
    }
    // Consecutive indices: 0, 1, 2, ..., 8
}
```

### **Handling Invalid UTF-8**

```go
import "unicode/utf8"

func main() {
    // Invalid UTF-8 bytes
    invalid := []byte{0xff, 0xfe, 0x65}
    s := string(invalid)
    
    // Range produces U+FFFD for invalid sequences
    for i, r := range s {
        fmt.Printf("%d: %U\n", i, r)
    }
    // 0: U+FFFD (replacement character)
    // 1: U+FFFD
    // 2: U+0065 (e)
    
    // Check validity
    if !utf8.ValidString(s) {
        fmt.Println("Invalid UTF-8!")
    }
}
```

### **Common String Iteration Patterns**

#### Count Specific Characters

```go
func countRune(s string, target rune) int {
    count := 0
    for _, r := range s {
        if r == target {
            count++
        }
    }
    return count
}

func main() {
    s := "hello"
    fmt.Println(countRune(s, 'l'))  // 2
}
```

#### Reverse a String (Correctly)

```go
func reverseString(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}

func main() {
    fmt.Println(reverseString("Hello, 世界"))
    // Output: 界世 ,olleH
}
```

#### Check for Substring (Manual)

```go
func containsRune(s string, target rune) bool {
    for _, r := range s {
        if r == target {
            return true
        }
    }
    return false
}
```

#### Transform Characters

```go
import "unicode"

func toUpperRunes(s string) string {
    runes := []rune(s)
    for i, r := range runes {
        runes[i] = unicode.ToUpper(r)
    }
    return string(runes)
}
```

### **Performance Considerations**

```go
func main() {
    s := "Hello, 世界"
    
    // Fast: Byte iteration (but wrong for Unicode)
    for i := 0; i < len(s); i++ {
        _ = s[i]
    }
    
    // Normal: Range iteration (decodes UTF-8)
    for _, r := range s {
        _ = r
    }
    
    // Slow: Converting to []rune first
    runes := []rune(s)  // Allocates new slice
    for _, r := range runes {
        _ = r
    }
    
    // Recommendation:
    // - Use range for most cases
    // - Convert to []rune only when you need random access
    // - Use byte iteration only for ASCII-only strings
}
```

### **ASCII-Only Optimization**

```go
import "unicode/utf8"

func processString(s string) {
    // Fast path for ASCII-only strings
    if utf8.RuneCountInString(s) == len(s) {
        // All single-byte characters, use byte iteration
        for i := 0; i < len(s); i++ {
            process(rune(s[i]))
        }
        return
    }
    
    // Slow path for Unicode
    for _, r := range s {
        process(r)
    }
}
```

### **Comparison Table**

| Method | Iterates | Index Is | Use When |
| --- | --- | --- | --- |
| `for i := 0; i < len(s); i++` | Bytes | Byte position | ASCII-only or raw bytes |
| `for i, r := range s` | Runes | Byte position | Unicode text |
| `for i, r := range []rune(s)` | Runes | Rune position | Need random access |

## Interview Questions

**Q: What is the difference between iterating a string with a traditional for loop vs for range?**
**A:** Traditional `for` iterates over bytes (indices 0 to len(s)-1), while `for range` iterates over runes (Unicode code points), automatically decoding UTF-8. For ASCII text they're equivalent, but for Unicode characters, `for range` correctly handles multi-byte characters.

**Q: Why does the index in `for i, r := range str` sometimes skip values?**
**A:** The index represents the byte position, not the rune position. Multi-byte characters (like Chinese or emoji) cause the index to jump by 2, 3, or 4 bytes. For consecutive rune indices, convert to `[]rune` first.

**Q: How do you correctly count characters in a Unicode string?**
**A:** Use `utf8.RuneCountInString(s)` which counts Unicode code points. Don't use `len(s)` which counts bytes. For "Hello, 世界", `len()` returns 13 but `RuneCountInString()` returns 9.

**Q: How do you reverse a string that contains Unicode characters?**
**A:** Convert to `[]rune`, reverse the slice, then convert back to string. Direct byte reversal would corrupt multi-byte UTF-8 sequences.
