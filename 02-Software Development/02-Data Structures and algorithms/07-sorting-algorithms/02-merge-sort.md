---
---

# Merge Sort

## Summary
**Merge Sort** is an efficient, stable, comparison-based sorting algorithm. It uses the **Divide and Conquer** strategy to recursively split the input array into halves until they are single elements, then merges the sorted halves back together. It guarantees $O(n \log n)$ performance but typically requires $O(n)$ auxiliary space.

## Detailed Explanation

### Mechanism
1.  **Divide**: Find the middle point to divide the array into two halves.
2.  **Conquer**: Recursively call Merge Sort for the first half and the second half.
3.  **Combine**: Merge the two sorted halves into a single sorted array.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (All Cases)** | $O(n \log n)$ | Consistent performance. |
| **Space** | $O(n)$ | Requires extra space for temporary arrays. |
| **Stable?** | Yes | Order of duplicates is preserved during merge. |

## Code Examples (Go)

```go
package main

import "fmt"

// MergeSort entry point
func MergeSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }
    
    mid := len(arr) / 2
    left := MergeSort(arr[:mid])
    right := MergeSort(arr[mid:])
    
    return merge(left, right)
}

// merge combines two sorted slices
func merge(left, right []int) []int {
    result := make([]int, 0, len(left)+len(right))
    i, j := 0, 0
    
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }
    
    // Append remaining elements
    result = append(result, left[i:]...)
    result = append(result, right[j:]...)
    
    return result
}

func main() {
    arr := []int{12, 11, 13, 5, 6, 7}
    sorted := MergeSort(arr)
    fmt.Println(sorted)
}
```

## Go Application
*   **Linked Lists**: Merge Sort is the preferred algorithm for sorting Linked Lists because it can be implemented without extra space (modifying pointers).
*   **Standard Lib**: Go's `sort.Stable` uses a form of Merge Sort (often SymMerge) when stability is required.

## Interview Questions

**Q: Why is Merge Sort preferred over Quick Sort for Linked Lists?**
**A:** Quick Sort requires random access (indexing) for efficient partitioning, which is slow ($O(n)$) in Linked Lists. Merge Sort only requires sequential access, making it much faster for lists. Also, Merge Sort on lists can be done in $O(1)$ auxiliary space by changing pointers.

**Q: What is the main disadvantage of Merge Sort?**
**A:** Space Complexity. It requires $O(n)$ extra memory to store the sub-arrays during merging. This makes it less desirable for systems with limited RAM compared to in-place sorts like Quick Sort or Heap Sort.

**Q: Is Merge Sort stable?**
**A:** Yes, provided the merge step handles equality correctly (e.g., `if left[i] <= right[j]`, pick left). This ensures the element that appeared earlier in the original array remains earlier.
