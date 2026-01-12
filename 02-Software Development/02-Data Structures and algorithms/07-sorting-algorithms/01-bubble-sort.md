---
---

# Bubble Sort

## Summary
**Bubble Sort** is one of the simplest sorting algorithms to understand but one of the least efficient to use. It works by repeatedly stepping through the list, comparing adjacent elements and swapping them if they are in the wrong order. The pass through the list is repeated until the list is sorted.

## Detailed Explanation

### Mechanism
1.  Start at the beginning of the array.
2.  Compare elements at `i` and `i+1`.
3.  If `arr[i] > arr[i+1]`, swap them.
4.  Move to the next pair.
5.  Repeat until the end of the array. This "bubbles" the largest element to the end.
6.  Repeat the entire process $N$ times (reducing the range by 1 each time).

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (Worst)** | $O(n^2)$ | Reverse sorted array. |
| **Time (Best)** | $O(n)$ | Already sorted (with optimization flag). |
| **Space** | $O(1)$ | In-place sorting. |
| **Stable?** | Yes | Equal elements are not swapped. |

## Code Examples (Go)

```go
package main

import "fmt"

func BubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        swapped := false
        // Last i elements are already in place
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                // Swap
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = true
            }
        }
        // Optimization: If no two elements were swapped by inner loop, then break
        if !swapped {
            break
        }
    }
}

func main() {
    data := []int{64, 34, 25, 12, 22, 11, 90}
    BubbleSort(data)
    fmt.Println(data)
}
```

## Go Application
**Do not use Bubble Sort in production Go code.**
*   Go's standard library `slices.Sort` (or `sort.Ints`) is far superior ($O(n \log n)$).
*   Bubble Sort is purely educational.

## Interview Questions

**Q: Why is Bubble Sort called "Bubble" Sort?**
**A:** Because with each full pass, the largest (or smallest) element "bubbles up" to its correct position at the end of the array, like an air bubble rising to the surface of water.

**Q: Can Bubble Sort be $O(n)$?**
**A:** Yes, if the array is already sorted. By using a boolean `swapped` flag, we can detect if a pass made no changes and exit early.

**Q: Is Bubble Sort stable?**
**A:** Yes. Since we only swap elements if they are strictly greater (`>`), equal elements maintain their relative order.
