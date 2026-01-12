---
---

# Array

## Abstract
An **Array** is a fundamental data structure consisting of a collection of elements, each identified by at least one array index or key. It stores elements of the same type in **contiguous memory locations**, which allows for extremely efficient access to any element given its index. In modern software development, specifically in Go, the concept splits into **Arrays** (fixed-size, value types) and **Slices** (dynamic, reference-like views).

## Development

### Core Concept
The defining characteristic of an array is **contiguous memory allocation**. If you have an array of 5 integers, and each integer takes 4 bytes, the array occupies exactly 20 consecutive bytes in memory.

**Visual Memory Layout**:
```text
Index:  0    1    2    3    4
Addr:  100  104  108  112  116
Value: [10] [20] [30] [40] [50]
```
Because the size of each element is known, the address of any element `i` can be calculated instantly:
`Address(i) = Base_Address + (i * Element_Size)`

### Time Complexity

| Operation | Complexity | Description |
|-----------|------------|-------------|
| **Access**| **O(1)**   | Constant time via pointer arithmetic. |
| **Search**| O(n)       | Linear scan (unless sorted, then O(log n)). |
| **Insert**| O(n)       | Requires shifting elements to make space. |
| **Delete**| O(n)       | Requires shifting elements to fill the gap. |

### Static vs. Dynamic Arrays
- **Static Array**: Size is fixed at compile time. Cannot grow or shrink. (e.g., C arrays, Go Arrays).
- **Dynamic Array**: Can resize itself. Usually implemented by allocating a larger static array and copying elements over when full. (e.g., Python List, Java ArrayList, Go Slice).

## Code Examples (Go)

In Go, the distinction between **Arrays** and **Slices** is strict and important.

### 1. Go Arrays (Fixed Size)
Arrays are **value types**. Assigning one array to another **copies** all elements.

```go
package main

import "fmt"

func main() {
    // Declaration: Size is part of the type [5]int
    var arr [5]int 
    arr[0] = 100
    arr[4] = 500

    // Array literal
    primes := [4]int{2, 3, 5, 7}

    // Iterating
    for i, v := range primes {
        fmt.Printf("Index: %d, Value: %d\n", i, v)
    }

    // WARNING: Arrays are copied by value!
    changeArray(primes)
    fmt.Println(primes) // Prints [2 3 5 7] - UNCHANGED
}

func changeArray(a [4]int) {
    a[0] = 999 // Modifies only the local copy
}
```

### 2. Go Slices (Dynamic)
Slices are **descriptors** (pointers) to an underlying array. They are what you use 99% of the time.

```go
package main

import "fmt"

func main() {
    // Creating a slice using make(type, len, cap)
    // len: number of accessible elements
    // cap: number of elements in underlying array before resize needed
    s := make([]int, 3, 5) 
    
    s[0] = 10
    s[1] = 20
    s[2] = 30
    
    fmt.Printf("Len: %d, Cap: %d\n", len(s), cap(s)) // Len: 3, Cap: 5

    // Appending handles resizing automatically
    s = append(s, 40, 50, 60)
    
    // Capacity likely doubled to accommodate new elements
    fmt.Printf("Len: %d, Cap: %d\n", len(s), cap(s))
}
```

## Go Application & Ecosystem

### Arrays vs. Slices
In Go, an array's size is part of its type. `[4]int` and `[5]int` are completely different types and cannot be assigned to each other. This rigidity makes arrays rare in application code, but they have unique properties:
- **Comparable**: Unlike slices, arrays (if their elements are comparable) can be used as **Map Keys**.
- **Allocation Control**: Functions like `sha256.Sum256` return `[32]byte` to keep the data on the stack, avoiding Garbage Collection overhead.

**Slices** are the idiomatic choice for collections. A slice is a lightweight 24-byte struct (on 64-bit systems) containing:
1. **Pointer**: To the start of the segment in the array.
2. **Length**: Number of elements in the slice.
3. **Capacity**: Total space in the underlying array from the pointer.

### The "Memory Leak" Gotcha
Since a slice references an underlying array, keeping a small slice of a massive array keeps the **entire** massive array in memory.

```go
// BAD: Keeps the whole file in memory
func getHeader(filename string) []byte {
    data, _ := os.ReadFile(filename) // returns huge []byte
    return data[:16] // references the huge array
}

// GOOD: Copies only what is needed
func getHeaderFixed(filename string) []byte {
    data, _ := os.ReadFile(filename)
    header := make([]byte, 16)
    copy(header, data[:16]) // New allocation, original 'data' can be GC'd
    return header
}
```

## Interview Preparation

### Common Questions

1. **What is the difference between an Array and a Slice in Go?**
   - **Answer**: An Array has a fixed size which is part of its type (`[3]int`) and is a **value type** (copied when passed). A Slice is a dynamic view (`[]int`) backed by an array; it is a reference-like struct (ptr, len, cap) and is cheap to pass.

2. **How does `append()` work internally?**
   - **Answer**: If the backing array has capacity, `append` simply writes the value and increases `len`. If capacity is exceeded, it allocates a new, larger array, **copies** all existing data, and returns a new slice.
   - **Growth Strategy**: For small slices (<256 elements), capacity typically **doubles**. For larger slices, it grows by a factor of ~1.25x to balance memory usage.

3. **Why is array access O(1)?**
   - **Answer**: Because of contiguous memory. The CPU can calculate the exact memory address of any element using `Base + (Index * Size)` without iterating.

4. **Can you use a Slice as a Map key?**
   - **Answer**: No, slices are not comparable because they are references. However, **Arrays** are comparable (if their elements are) and can be used as map keys (e.g., `map[[16]byte]string`).

5. **When would you use an Array instead of a Slice in Go?**
   - **Answer**: When you need absolute control over memory layout, are working with fixed-length protocol headers (like an SHA-256 hash), or need to avoid heap allocation (arrays can often be stack-allocated if small and not escaping).
