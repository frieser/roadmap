---
---

# Backtracking

## Summary
**Backtracking** is an algorithmic technique for solving problems recursively by trying to build a solution incrementally, one piece at a time, and removing those solutions that fail to satisfy the constraints of the problem at any point of time. It is essentially **Brute Force with pruning**.

## Detailed Explanation

### Mechanism
1.  **Choose**: Pick an option.
2.  **Constraint Check**: Is it valid so far?
3.  **Recurse**: Move to the next step.
4.  **Undo (Backtrack)**: If the path fails, undo the choice and try the next option.

### Use Cases
*   Sudoku Solver
*   N-Queens Problem
*   Generating Permutations/Combinations

## Code Examples (Go)

### Generating Permutations
```go
func Permute(nums []int) [][]int {
    var res [][]int
    var backtrack func(start int)
    
    backtrack = func(start int) {
        if start == len(nums) {
            // Copy slice to avoid reference issues
            dst := make([]int, len(nums))
            copy(dst, nums)
            res = append(res, dst)
            return
        }
        
        for i := start; i < len(nums); i++ {
            // Swap
            nums[start], nums[i] = nums[i], nums[start]
            // Recurse
            backtrack(start + 1)
            // Backtrack (Swap back)
            nums[start], nums[i] = nums[i], nums[start]
        }
    }
    
    backtrack(0)
    return res
}
```
