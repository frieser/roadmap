---
---

# Constant Time - O(1)

## Abstract
**Constant Time**, denoted as **O(1)**, represents an operation whose runtime does not change regardless of the size of the input data ($n$). It is the ideal complexity for any algorithm because it is instant and predictable. In Go, many built-in operations are O(1) by design.

## Development

### Core Concept
If you have a function that takes an array of 10 items or 10 billion items, an O(1) operation will take roughly the same amount of time (CPU cycles) to execute. It does not iterate or scan the input.

### Key Characteristics
- **No loops** dependent on input size.
- **No recursion**.
- **Direct memory access**.

## Code Examples (Go)

### 1. Map Access (Average Case)
Go maps are hash tables. Accessing a value by key is O(1) on average.
```go
func getValue(m map[string]int, key string) int {
    return m[key] // O(1)
}
```

### 2. Array/Slice Indexing
Accessing an element by index is instantaneous because it uses pointer arithmetic: `base_address + (index * element_size)`.
```go
func getFirst(nums []int) int {
    if len(nums) > 0 {
        return nums[0] // O(1)
    }
    return 0
}
```

### 3. Length & Capacity
Functions `len()` and `cap()` in Go are O(1). The values are stored in the slice/string header struct, so Go doesn't need to count elements.
```go
func checkSize(s string) int {
    return len(s) // O(1), not O(n)
}
```

### 4. Stack Operations (Push/Pop)
Pushing to the end of a slice (amortized) or popping from the end is O(1).
```go
func pop(stack []int) (int, []int) {
    n := len(stack)
    return stack[n-1], stack[:n-1] // O(1)
}
```

## Go Application & Ecosystem
- **Slice Append**: `append()` is **Amortized O(1)**. Most calls are O(1), but occasionally it triggers a resize (allocation + copy), which is O(n). Averaged out over many calls, it's considered O(1).
- **Channels**: Sending or receiving from a buffered channel is O(1) (assuming no blocking contention).

## Interview Preparation
1.  **Is `append` always O(1)?**
    -   *Answer*: No, it is **amortized** O(1). If the capacity is exceeded, it becomes O(n) due to reallocation and copying.
2.  **Is `len(string)` O(1) or O(n)?**
    -   *Answer*: O(1). A Go string is a struct `| data_ptr | length |`. `len()` simply returns the length field.
3.  **Map access worst case?**
    -   *Answer*: Technically O(n) if there are many hash collisions, but Go's runtime uses efficient hashing (AES-based) and bucket structures to keep it close to O(1) in practice.
