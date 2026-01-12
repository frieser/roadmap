---
---

# Sliding Window Technique

## Summary
**Sliding Window** is a variation of the Two Pointers technique used to perform operations on a specific window size of a given array or string. It is excellent for finding subarrays with specific properties (e.g., "Longest substring with K distinct characters").

## Detailed Explanation

### Mechanism
1.  Expand the window (`right` pointer) to include elements.
2.  Check if the window violates the condition.
3.  If violated, shrink the window (`left` pointer) until valid again.
4.  Update the result (max length, min sum, etc.).

### Use Cases
*   Max Sum Subarray of size K.
*   Longest Substring Without Repeating Characters.

## Code Examples (Go)

### Max Sum Subarray of Size K
```go
func MaxSumSubarray(arr []int, k int) int {
    maxSum, windowSum := 0, 0
    start := 0
    
    for end := 0; end < len(arr); end++ {
        windowSum += arr[end]
        
        // Window is hit
        if end >= k-1 {
            if windowSum > maxSum {
                maxSum = windowSum
            }
            // Slide window
            windowSum -= arr[start]
            start++
        }
    }
    return maxSum
}
```
