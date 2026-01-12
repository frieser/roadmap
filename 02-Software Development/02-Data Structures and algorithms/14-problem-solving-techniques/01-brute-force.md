---
---

# Brute Force

## Summary
**Brute Force** is the most straightforward problem-solving technique. It involves trying **every possible solution** to see which one works. It is rarely the most efficient approach but guarantees finding a solution if one exists.

## Detailed Explanation

### Mechanism
*   **Enumeration**: List all candidate solutions.
*   **Validation**: Check each candidate against the problem constraints.
*   **Selection**: Pick the best valid candidate (or the first one found).

### Pros & Cons
*   **Pros**: Simple to implement, guarantees correctness, works for small inputs.
*   **Cons**: Extremely slow ($O(n!)$, $O(2^n)$, $O(n^2)$), impractical for large inputs.

## Code Examples (Go)

### Two Sum (Brute Force)
Find two numbers that add up to a target.
*   **Complexity**: $O(n^2)$

```go
func TwoSum(nums []int, target int) []int {
    for i := 0; i < len(nums); i++ {
        for j := i + 1; j < len(nums); j++ {
            if nums[i]+nums[j] == target {
                return []int{i, j}
            }
        }
    }
    return nil
}
```

## Interview Context
Always start with the Brute Force solution in an interview to prove you understand the problem, then optimize it.
