---
---

# Unicode

## Abstract
**Unicode** is the universal character encoding standard used to represent text in computer processing. It provides a unique number (**Code Point**) for every character, no matter the platform, program, or language, replacing legacy encoding schemes that were limited and incompatible. In Go, Unicode is a first-class citizen, with the `rune` type representing a code point and source code itself being strictly UTF-8.

## Development

### Core Concept
Before Unicode, there were hundreds of encoding systems (ASCII, ISO-8859, Windows-1252), none of which could contain enough characters to cover all world languages. This led to text corruption (mojibake) when exchanging data. Unicode solves this by assigning a unique **Code Point** to every character, usually written as `U+XXXX` (hexadecimal).

- **Code Point**: An integer value assigned to a character (e.g., `U+0041` is 'A', `U+1F600` is 😀).
- **Range**: From `U+0000` to `U+10FFFF` (over 1.1 million possible characters).
- **Planes**: The space is divided into 17 "planes".
    - **BMP (Basic Multilingual Plane)**: Plane 0 (`U+0000`–`U+FFFF`). Contains characters for almost all modern languages.
    - **Supplementary Planes**: Planes 1-16. Used for historic scripts, musical symbols, and emojis.

### Encodings (UTF-8 vs UTF-16)
It is crucial to distinguish between the **Character Set** (Unicode: mapping integers to characters) and the **Encoding** (UTF: mapping integers to bytes).

| Encoding | Type | Description |
| :--- | :--- | :--- |
| **UTF-8** | Variable | Uses **1 to 4 bytes**. **Backwards compatible with ASCII** (ASCII chars are 1 byte). Standard for the web, Linux, and Go. Space efficient for English text. |
| **UTF-16** | Variable | Uses **2 or 4 bytes**. BMP chars are 2 bytes; others use pairs (surrogates). Used by Java, Windows, JavaScript internal representation. |
| **UTF-32** | Fixed | Uses **4 bytes** for every character. Simple handling (O(1) random access) but extremely memory inefficient (wasteful). |

### Go Implementation Details
Go was designed by the creators of UTF-8 (Ken Thompson, Rob Pike), so UTF-8 is deeply integrated.

- **Source Code**: Go source files are always UTF-8.
- **String**: A read-only slice of bytes (`[]byte`). It holds arbitrary bytes, but is conventionally UTF-8 text.
- **Rune**: An alias for `int32`. Represents a **Unicode Code Point**.
- **Iteration**: A `for range` loop on a string iterates over **runes**, not bytes. It automatically decodes the UTF-8 sequence.

## Code Examples (Go)

### 1. Runes vs Bytes
This example demonstrates the difference between the byte length (`len`) and character count (`RuneCount`).

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {
	// "Hello, 世界"
	// H, e, l, l, o (1 byte each)
	// , (1 byte)
	//   (1 byte)
	// 世 (3 bytes: E4 B8 96)
	// 界 (3 bytes: E7 95 8C)
	const s = "Hello, 世界"

	fmt.Printf("String: %s\n", s)
	fmt.Printf("Byte Length (len): %d\n", len(s)) // 13 bytes
	fmt.Printf("Rune Count: %d\n", utf8.RuneCountInString(s)) // 9 characters

	// Accessing by index gives a byte (uint8)
	fmt.Printf("Byte at index 0: %x ('%c')\n", s[0], s[0])
	
	// Accessing the multibyte character part
	// s[7] is just the first byte of '世' (0xe4), not the character
	fmt.Printf("Byte at index 7: %x\n", s[7]) 
}
```

### 2. Iterating Strings
To correctly process text, iterate over runes.

```go
func main() {
	const s = "Hello, 世界"

	fmt.Println("Index | Rune | Hex | Bytes")
	
	// 'range' decodes UTF-8 automatically
	for i, r := range s {
		// i is the starting byte index
		// r is the rune (int32 code point)
		fmt.Printf("%-5d | %-4c | %-3U | %d\n", i, r, r, utf8.RuneLen(r))
	}
}
```

## Go Application & Ecosystem

### The 'rune' Type
When working with individual characters in Go, always use `rune`. Do not use `byte` unless you are certain the data is pure ASCII or you are processing raw binary data.
- `'A'` is a `rune` (default type `int32`, value 65).
- `"A"` is a `string`.

### Validating UTF-8
Since a `string` is just a byte slice, it might contain invalid UTF-8 sequences. Use the `unicode/utf8` package to validate inputs.

```go
import "unicode/utf8"

func main() {
    valid := "Hello"
    invalid := "\xff\xfe\xfd"
    
    fmt.Println(utf8.ValidString(valid))   // true
    fmt.Println(utf8.ValidString(invalid)) // false
}
```

## Interview Preparation

### Common Questions

1. **What is the difference between `len(s)` and `utf8.RuneCountInString(s)` in Go?**
   - **Answer:** `len(s)` returns the number of **bytes** in the string. Since Go strings are UTF-8, multi-byte characters (like emojis or Kanji) take more than 1 byte. `utf8.RuneCountInString(s)` decodes the string and returns the actual number of Unicode code points (characters).

2. **What is a 'rune' in Go?**
   - **Answer:** A `rune` is an alias for `int32`. It represents a single Unicode Code Point. It is distinct from `byte` (`uint8`), which represents raw data or ASCII characters.

3. **Why is UTF-8 preferred over UTF-16 for the web?**
   - **Answer:** UTF-8 is **space-efficient for ASCII** (1 byte vs 2 bytes in UTF-16), which makes up the vast majority of HTML, CSS, and Javascript code. It is also endianness-independent (byte stream) and backwards compatible with legacy ASCII systems.

4.  **How do you iterate over characters in a Go string?**
    -   **Answer:** Use a `for i, r := range str` loop. This automatically decodes the UTF-8 bytes into `rune` types. A standard `for i := 0; i < len(str); i++` loop would iterate over bytes, breaking multi-byte characters.
