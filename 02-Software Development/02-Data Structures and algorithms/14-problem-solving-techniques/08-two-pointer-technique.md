---
---

# Two Pointer Technique

## Summary
The **Two Pointer Technique** uses two pointers (indexes) to traverse a data structure (usually an array or linked list) in different directions or speeds to solve a problem efficiently (often reducing $O(n^2)$ to $O(n)$).

## Detailed Explanation

### Patterns
1.  **Collision**: One pointer at start, one at end. They move towards each other. (e.g., Two Sum Sorted, Reverse Array).
2.  **Parallel**: Both pointers start at 0 but move independently. (e.g., Remove Duplicates, Merging Sorted Arrays).

## Code Examples (Go)

### Two Sum (Sorted Array)
Find two numbers that add to target in a sorted array.
*   **Time**: $O(n)$

```go
func TwoSumSorted(arr []int, target int) []int {
    left, right := 0, len(arr)-1
    
    for left < right {
        sum := arr[left] + arr[right]
        if sum == target {
            return []int{left, right}
        } else if sum < target {
            left++ // Need bigger sum
        } else {
            right-- // Need smaller sum
        }
    }
    return nil
}
```
