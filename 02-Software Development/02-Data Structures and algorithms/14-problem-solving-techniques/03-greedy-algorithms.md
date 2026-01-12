---
---

# Greedy Algorithms

## Summary
A **Greedy Algorithm** builds up a solution piece by piece, always choosing the next piece that offers the most immediate benefit (**local optimum**). It hopes that by choosing a local optimum at each step, it will end up at a **global optimum**.

## Detailed Explanation

### Core Logic
*   **Greedy Choice Property**: A global optimum can be arrived at by selecting a local optimum.
*   **Optimal Substructure**: An optimal solution to the problem contains an optimal solution to subproblems.

### Warning
Greedy algorithms **do not always work**. For example, in the "Coin Change" problem, greedy works for US coins (1, 5, 10, 25) but fails for arbitrary systems (e.g., coins 1, 3, 4; target 6. Greedy picks 4+1+1 (3 coins), Optimal is 3+3 (2 coins)).

## Code Examples (Go)

### Activity Selection Problem
Pick maximum number of non-overlapping meetings.
*   Sort by **End Time**.
*   Pick the first one. Pick next one that starts after the previous ended.

```go
type Meeting struct { Start, End int }

func MaxMeetings(meetings []Meeting) int {
    // Sort by end time
    sort.Slice(meetings, func(i, j int) bool {
        return meetings[i].End < meetings[j].End
    })
    
    count := 0
    lastEnd := -1
    
    for _, m := range meetings {
        if m.Start >= lastEnd {
            count++
            lastEnd = m.End
        }
    }
    return count
}
```
