---
---

# Selection Sort

## Summary
**Selection Sort** is a simple, comparison-based algorithm that divides the input list into two parts: the sublist of items already sorted and the sublist of items remaining to be sorted. It works by repeatedly finding the **minimum element** from the unsorted sublist and moving it to the beginning.

## Detailed Explanation

### Mechanism
1.  Set `min_index` to the first element `i`.
2.  Iterate through the rest of the array (`j = i+1` to `n`).
3.  If `arr[j] < arr[min_index]`, update `min_index = j`.
4.  Swap `arr[i]` with `arr[min_index]`.
5.  Increment `i` and repeat.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (All)** | $O(n^2)$ | Always scans the remaining array. |
| **Space** | $O(1)$ | In-place. |
| **Stable?** | No | Swapping long-distance can jump over equal elements. |

## Code Examples (Go)

```go
package main

import "fmt"

func SelectionSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        minIndex := i
        for j := i + 1; j < n; j++ {
            if arr[j] < arr[minIndex] {
                minIndex = j
            }
        }
        // Swap only if a smaller element was found
        arr[i], arr[minIndex] = arr[minIndex], arr[i]
    }
}

func main() {
    data := []int{64, 25, 12, 22, 11}
    SelectionSort(data)
    fmt.Println(data)
}
```

## Go Application
Selection Sort is rarely used in Go applications because Insertion Sort is generally faster for small arrays, and Quick Sort is faster for large ones.
*   **Niche Use Case**: Selection Sort makes the **minimum number of swaps** ($N$ swaps total). If "writing" to memory is extremely expensive (e.g., sorting data on EEPROM or Flash memory where write cycles are limited), Selection Sort might be theoretically preferred over Insertion Sort (which does $O(n^2)$ writes).

## Interview Questions

**Q: What is the one advantage of Selection Sort over Bubble Sort?**
**A:** Selection Sort performs fewer swaps. In the worst case, Bubble Sort performs $O(n^2)$ swaps, whereas Selection Sort performs exactly $O(n)$ swaps. This matters if the cost of swapping (writing to memory) is high.

**Q: Is Selection Sort stable?**
**A:** No. Consider `[5a, 5b, 1]`.
1.  First pass: Finds `1` (min). Swaps `5a` with `1`.
2.  Result: `[1, 5b, 5a]`.
3.  The order of 5s is reversed.
