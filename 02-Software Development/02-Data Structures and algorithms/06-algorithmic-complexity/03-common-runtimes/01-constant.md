---
---

# O(1) Constant Time

## Summary
**$O(1)$ Constant Time** represents the ideal algorithmic efficiency. It means the execution time (or space) remains the **same** regardless of how large the input data grows. Whether you have 10 items or 10 billion, the operation takes one unit of time.

## Characteristics
*   **Scalability**: Perfect. The gold standard for system design.
*   **Mechanism**: Usually involves direct access via index, key, or math, avoiding iteration.
*   **Graph**: A flat horizontal line.

## Common Operations
1.  **Array Indexing**: Accessing `arr[5]` is instant because it's a simple memory offset calculation (`base_address + 5 * item_size`).
2.  **Hash Map Lookup**: Finding a key in a map (on average).
3.  **Push/Pop Stack**: Adding/removing from the top of a stack.
4.  **Math Formula**: Calculating `(n * (n+1)) / 2`.

## Go Code Examples

### 1. Slice Access
```go
// GetFirst is O(1)
// It doesn't matter if 'nums' has 5 or 5,000,000 items.
func GetFirst(nums []int) int {
    if len(nums) == 0 {
        return -1
    }
    return nums[0] // Direct memory access
}
```

### 2. Map Lookup
```go
// GetScore is O(1) on average
func GetScore(scores map[string]int, name string) int {
    return scores[name] // Hashing allows direct bucket access
}
```

### 3. Length Calculation
In Go, `len()` is $O(1)$ because slices and strings store their length in a struct header. It does **not** count the characters.
```go
str := "Hello World"
l := len(str) // O(1) - reads the header
```
