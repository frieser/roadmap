---
---

# ASCII

## Abstract
**ASCII** (American Standard Code for Information Interchange) is a 7-bit character encoding standard representing text in computers. It includes **128 characters**: English letters, numbers, symbols, and control codes. Although largely superseded by Unicode, ASCII remains the foundation of modern character sets, as **UTF-8 is backwards compatible with it**.

## Development

### Core Concept
Designed in the 1960s, ASCII maps integers **0–127** to specific characters.
- **Range**: 0 to 127 (7 bits).
- **Storage**: Typically stored in an 8-bit byte, with the 8th bit used for parity (historically) or set to 0.

### Structure
1. **Control Characters (0–31 & 127)**: Non-printable codes for controlling devices (teleprinters, terminals).
   - `0` (NULL): Null character
   - `10` (LF): Line Feed (`\n`)
   - `13` (CR): Carriage Return (`\r`)
   - `127` (DEL): Delete
2. **Printable Characters (32–126)**:
   - `32`: Space
   - `48–57`: Digits `0-9`
   - `65–90`: Uppercase `A-Z`
   - `97–122`: Lowercase `a-z`
   - `33-47`, `58-64`, etc.: Punctuation and symbols.

### ASCII vs Extended ASCII
Standard ASCII uses 7 bits. **Extended ASCII** (like ISO-8859-1) uses the 8th bit to add another 128 characters (total 256), such as `ñ`, `ö`, `£`. However, these extensions were incompatible with each other (e.g., ISO-8859-1 Western vs ISO-8859-5 Cyrillic), leading to text corruption. This problem was solved by Unicode.

### Go Implementation Details
- **Byte**: Go's `byte` type is an alias for `uint8`. It is perfect for holding ASCII characters.
- **String Indexing**: `s[i]` returns a `byte`. For ASCII strings, this correctly identifies the i-th character.
- **Raw Strings**: Go supports raw string literals using backticks `` ` ``, which can contain newlines and unescaped characters.

## Code Examples (Go)

### 1. Manipulating ASCII
Since ASCII characters are 1 byte, we can manipulate them directly as bytes without the overhead of `utf8` decoding.

```go
package main

import "fmt"

func main() {
	s := "GoLang" // Pure ASCII
	
	// Direct byte access is safe for ASCII
	fmt.Printf("First char: %c (Byte: %d)\n", s[0], s[0])
	
	// converting case (naive ASCII-only approach)
	// 'a' = 97, 'A' = 65. Difference is 32.
	lower := []byte(s)
	for i := 0; i < len(lower); i++ {
		// Check if it is uppercase A-Z
		if lower[i] >= 'A' && lower[i] <= 'Z' {
			lower[i] += 32 // Convert to lowercase
		}
	}
	fmt.Println(string(lower)) // "golang"
}
```

### 2. Control Characters
Common escape sequences in Go strings map to ASCII control codes.

```go
func main() {
    // \n is Line Feed (ASCII 10)
    // \t is Tab (ASCII 9)
    fmt.Print("Column 1\tColumn 2\nValue 1 \tValue 2\n")
}
```

## Go Application & Ecosystem

### When to use ASCII logic?
If you are parsing specific protocols (like HTTP headers, JSON keys, or legacy data) known to be ASCII, iterating bytes is **O(1)** and faster than decoding runes. The standard library often uses this optimization.

- **Example**: `strings.IndexByte` is implemented in assembly for speed and searches for a single byte.
- **Example**: `strings.ToLower` checks if the string contains non-ASCII characters first; if not, it uses a fast byte loop.

## Interview Preparation

### Common Questions

1. **What is the relationship between ASCII and UTF-8?**
   - **Answer:** UTF-8 is **backwards compatible with ASCII**. The first 128 characters of Unicode (U+0000 to U+007F) correspond exactly to ASCII 0–127. In UTF-8, these characters are encoded as a single byte with the same value. This means any valid ASCII file is also a valid UTF-8 file.

2. **Why is ASCII 7-bit?**
   - **Answer:** It was originally designed to fit in 1 byte (8 bits) while leaving the 8th bit available for a **parity bit** (error checking) during transmission over unreliable lines.

3. **How do you check if a byte is ASCII in Go?**
   - **Answer:** Check if the most significant bit is 0. Since ASCII is 7-bit (0-127), any value `< 128` (or `b & 0x80 == 0`) is valid ASCII.

4. **What is the difference between `byte` and `rune`?**
   - **Answer:** `byte` is `uint8` and represents raw data or ASCII characters. `rune` is `int32` and represents a Unicode code point. You use `byte` for data streams or ASCII text, and `rune` for proper Unicode text handling.
