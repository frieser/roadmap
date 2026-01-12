---
---

# Linear Time - O(n)

## Abstract
**Linear Time**, denoted as **O(n)**, means the runtime grows **directly proportionally** to the input size ($n$). If the input doubles, the execution time doubles. This is the complexity of "brute force" searches or iterating through a list once.

## Development

### Core Concept
Any algorithm that must examine every single element of the input exactly once (or a constant number of times) is O(n).
- $n = 10 \to 10$ steps.
- $n = 100 \to 100$ steps.

### Common Sources
- **Iteration**: For-loops over a slice/map.
- **Copying**: Copying a slice or string.
- **Linear Search**: Finding an item in an unsorted list.
- **String Traversal**: Checking palindromes, counting characters.

## Code Examples (Go)

### 1. Simple Iteration (Sum)
```go
func sum(nums []int) int {
    total := 0
    // We visit every element once -> O(n)
    for _, n := range nums {
        total += n
    }
    return total
}
```

### 2. Linear Search
Finding an item in an unsorted slice requires checking every element in the worst case (item at the end or not present).
```go
func contains(nums []int, target int) bool {
    for _, n := range nums {
        if n == target {
            return true
        }
    }
    return false
}
```

### 3. String & Byte Operations
Go's `bytes.Contains` or simple string loops are O(n).
```go
func countVowels(s string) int {
    count := 0
    for _, char := range s { // Iterates n runes
        switch char {
        case 'a', 'e', 'i', 'o', 'u':
            count++
        }
    }
    return count
}
```

## Go Application & Ecosystem
- **`copy(dst, src)`**: This built-in function is O(n), bounded by the length of the smaller slice.
- **`slices.Clone`**: Creates a copy, O(n).
- **`strings.Repeat`**: O(n).
- **Defer overhead**: While small, deferring N function calls in a loop is O(n) work for the runtime.

## Interview Preparation
1.  **Is O(2n) different from O(n)?**
    -   *Answer*: No. In Big O, we drop constants. $O(2n) \to O(n)$.
2.  **Can we sort in O(n)?**
    -   *Answer*: Generally no (comparison sorts are O(n log n)). But strict O(n) sorts exist for specific constraints (Counting Sort, Radix Sort).
3.  **Space Complexity**: Iterating a slice is O(1) space, but creating a copy of it is O(n) space.
