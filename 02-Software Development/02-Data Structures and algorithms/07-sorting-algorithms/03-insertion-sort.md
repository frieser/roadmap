---
---

# Insertion Sort

## Summary
**Insertion Sort** is a simple comparison algorithm that builds the final sorted array one item at a time. It works similarly to the way you sort playing cards in your hand. While inefficient for large lists ($O(n^2)$), it is extremely fast for **small** or **nearly sorted** datasets, often outperforming $O(n \log n)$ algorithms in those specific cases.

## Detailed Explanation

### Mechanism
1.  Assume the first element is already sorted.
2.  Take the next element (the "key").
3.  Compare the key with elements in the sorted sub-list (to its left).
4.  Shift all elements greater than the key one position to the right.
5.  Insert the key into its correct position.
6.  Repeat until the array is sorted.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (Worst)** | $O(n^2)$ | Reverse sorted array. |
| **Time (Best)** | $O(n)$ | Already sorted array (adaptive). |
| **Space** | $O(1)$ | In-place. |
| **Stable?** | Yes | |

## Code Examples (Go)

```go
package main

import "fmt"

func InsertionSort(arr []int) {
    n := len(arr)
    for i := 1; i < n; i++ {
        key := arr[i]
        j := i - 1
        
        // Move elements of arr[0..i-1], that are greater than key,
        // to one position ahead of their current position
        for j >= 0 && arr[j] > key {
            arr[j+1] = arr[j]
            j = j - 1
        }
        arr[j+1] = key
    }
}

func main() {
    data := []int{12, 11, 13, 5, 6}
    InsertionSort(data)
    fmt.Println(data)
}
```

## Go Application
**Optimization Trick**:
Advanced sorting implementations (like Go's `pdqsort` used in `slices.Sort`) switch to **Insertion Sort** when the partition size becomes small (e.g., < 12-24 elements).
*   Why? For very small $N$, the overhead of recursion and complex partitioning in Quick Sort outweighs the simplicity of Insertion Sort.
*   Insertion Sort has excellent cache locality and low overhead.

## Interview Questions

**Q: When is Insertion Sort a good choice?**
**A:**
1.  When $N$ is small (e.g., < 20).
2.  When the array is **almost sorted** (it runs in near $O(n)$ time).
3.  When writing a base case for a Divide and Conquer sort (like Quick Sort).

**Q: Why is Insertion Sort stable?**
**A:** We only shift elements that are strictly greater than the key. If we find an element equal to the key, we stop shifting and insert the key *after* it, preserving the original order.
