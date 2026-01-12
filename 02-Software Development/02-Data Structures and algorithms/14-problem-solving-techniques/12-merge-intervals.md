---
---

# Merge Intervals

## Summary
**Merge Intervals** is a pattern for dealing with problems involving overlapping time periods or ranges. It typically requires **sorting** the intervals by start time first, then iterating through to merge overlaps.

## Detailed Explanation

### Mechanism
1.  Sort intervals by `Start` time.
2.  Create a `merged` list. Add the first interval.
3.  Iterate through the rest:
    *   If current interval's `Start` $\le$ previous interval's `End`: **Overlap found**. Merge them by updating previous `End` to `max(prev.End, curr.End)`.
    *   Else: No overlap. Add current to `merged`.

## Code Examples (Go)

```go
import "sort"

func Merge(intervals [][]int) [][]int {
    if len(intervals) <= 1 { return intervals }
    
    // Sort by start time
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    
    result := [][]int{intervals[0]}
    
    for _, curr := range intervals[1:] {
        last := result[len(result)-1]
        
        if curr[0] <= last[1] {
            // Overlap: update end time
            if curr[1] > last[1] {
                last[1] = curr[1]
                result[len(result)-1] = last // Update in slice
            }
        } else {
            // No overlap
            result = append(result, curr)
        }
    }
    return result
}
```
