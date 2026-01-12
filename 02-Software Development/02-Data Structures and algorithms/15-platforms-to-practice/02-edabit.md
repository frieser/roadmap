---
---

# Edabit

## Summary
**Edabit** is a coding platform that focuses on **educational** progression rather than competitive programming. It bridges the gap between learning syntax and solving complex LeetCode algorithm problems by providing bite-sized challenges.

## Why Edabit?
*   **Syntax Mastery**: The "Very Easy" and "Easy" problems are excellent for drilling language syntax (loops, string manipulation, type conversion) until it becomes muscle memory.
*   **Gamification**: XP points and levels make the grind feel rewarding.
*   **Incremental Difficulty**: The jump in difficulty is much smoother than LeetCode.

## Strategy
1.  **Warm-up**: Use Edabit when learning a *new* language (like Go) to get comfortable with the standard library (`strings`, `sort`, `math`).
2.  **Transition**: Once you are consistently solving "Medium/Hard" problems on Edabit, move to LeetCode "Easy/Medium".

## Go Specifics on Edabit
Edabit challenges are great for learning Go's quirks:
*   **Rune vs Byte**: Handling UTF-8 strings.
*   **Type Conversions**: `strconv.Atoi`, `fmt.Sprintf`.
*   **Formatting**: Using `strings.Split`, `strings.Join`, `strings.ToUpper`.

### Example Challenge (Easy)
*Task: Return the sum of two numbers.*
```go
func Sum(a, b int) int {
	return a + b
}
```
*(Edabit focuses on the basics like function signatures and return types).*
