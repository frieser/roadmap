---
---

# Cyclic Sort

## Summary
**Cyclic Sort** is a pattern used to solve array problems where the data involves a range of numbers from $1$ to $N$ (or $0$ to $N-1$) in an array of size $N$. It works by placing each number at its correct index (i.e., placing value `i` at index `i-1`).

## Detailed Explanation

### Mechanism
Iterate through the array. If the current number `nums[i]` is not at its correct index (`nums[i]-1`) and is within range, **swap** it with the number at its correct index. Repeat until the current position holds the correct number or a number out of range.

### Complexity
*   **Time**: $O(n)$ (Each number is swapped at most once).
*   **Space**: $O(1)$.

### Use Cases
*   Find Missing Number.
*   Find Duplicate Numbers.

## Code Examples (Go)

### Find Missing Number
Given array of size $N$ with numbers $0$ to $N$. Find the one missing.

```go
func MissingNumber(nums []int) int {
    i := 0
    for i < len(nums) {
        correctIdx := nums[i]
        // If value < N and not in correct spot, swap
        if nums[i] < len(nums) && nums[i] != nums[correctIdx] {
            nums[i], nums[correctIdx] = nums[correctIdx], nums[i]
        } else {
            i++
        }
    }
    
    // Find the first index that doesn't match
    for i, v := range nums {
        if v != i { return i }
    }
    return len(nums)
}
```
