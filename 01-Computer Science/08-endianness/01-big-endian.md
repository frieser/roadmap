---
---

## Summary
**Big-Endian** is a byte ordering format where the **Most Significant Byte (MSB)** is stored at the **lowest memory address**. It is analogous to how we read numbers in English (left to right). It is the standard for network protocols (TCP/IP), often referred to as **Network Byte Order**.

## Detailed Explanation

In computing, memory is a linear array of bytes. When storing a multi-byte data type (like a 32-bit integer) across consecutive memory addresses, we must decide which byte goes first.

### How it works
Consider the 32-bit hexadecimal number: `0x12345678`

*   **MSB (Most Significant Byte)**: `0x12` (The "biggest" part of the number)
*   **LSB (Least Significant Byte)**: `0x78` (The "smallest" part of the number)

In **Big-Endian** storage, it is laid out as:

| Address | Byte Value | Description |
| :--- | :--- | :--- |
| `0x00` | **`12`** | MSB |
| `0x01` | `34` | |
| `0x02` | `56` | |
| `0x03` | **`78`** | LSB |

### Pros and Cons
*   **Pros**:
    *   **Human Readable**: Dumps of memory read naturally (left-to-right matches the written number).
    *   **Sign Bit Access**: The sign bit (in the MSB) is at a fixed offset (0), allowing quick positivity checks without knowing the number's length.
*   **Cons**:
    *   **Arithmetic**: Addition/subtraction starts at the LSB, which is at the *highest* address, requiring calculation to start from the "end" of the number.

### Usage
*   **Networking**: The Internet Protocol (IP) defines Big-Endian as the standard **Network Byte Order**. All multi-byte fields in IP packets headers are big-endian.
*   **Legacy Architectures**: Motorola 68k, SPARC, IBM Mainframes (System z).
*   **File Formats**: JPEG, Java .class files.

## Go Example

Go's `encoding/binary` package provides explicit support for Big-Endian encoding.

```go
package main

import (
	"encoding/binary"
	"fmt"
)

func main() {
	// Our value: 305419896 (Decimal) -> 0x12345678 (Hex)
	value := uint32(0x12345678)

	// Create a byte slice buffer
	buf := make([]byte, 4)

	// Write using Big-Endian
	binary.BigEndian.PutUint32(buf, value)

	// Print the byte layout
	fmt.Printf("Original Hex: 0x%X\n", value)
	fmt.Printf("Memory Layout (Big-Endian): %X\n", buf)
	// Output: [12 34 56 78]

	// Reading it back
	decoded := binary.BigEndian.Uint32(buf)
	fmt.Printf("Decoded: 0x%X\n", decoded)
}
```

## Interview Questions

### Q: Why is Big-Endian called "Network Byte Order"?
**A:** Early network protocols (IP, TCP) chose Big-Endian as the standard format for consistency across different machine architectures. Any machine sending data onto the network must convert its native format to Big-Endian, and vice-versa when receiving.

### Q: How do you determine if a system is Big-Endian or Little-Endian at runtime?
**A:** You can store a known multi-byte integer (like `1`) and inspect its first byte. If the first byte is `0`, it's Big-Endian (stored as `00 00 00 01`). If the first byte is `1`, it's Little-Endian (stored as `01 00 00 00`).

### Q: Does Big-Endian affect string storage?
**A:** Generally, no. Strings are arrays of single bytes (characters/UTF-8 bytes), so order is preserved naturally (Address 0 = char 0). Endianness only applies to multi-byte numeric types (int16, int32, float64, etc.).
