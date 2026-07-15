---
---

## Asymptotic Notation — Summary

- **Asymptotic notation**: growth rate as n→∞, not exact time.
- **Big O = Θ = Ω (one line)**: O(n) = upper (worst), Θ(n) = tight bound, Ω(n) = lower (best).
- **Formal O**: ∃c,n₀: 0 ≤ f(n) ≤ c·g(n) for all n ≥ n₀.
- **Formal Θ**: ∃c₁,c₂,n₀: c₁·g(n) ≤ f(n) ≤ c₂·g(n) for all n ≥ n₀.
- **Formal Ω**: ∃c,n₀: 0 ≤ c·g(n) ≤ f(n) for all n ≥ n₀.
- **Θ iff**: O(g(n)) AND Ω(g(n)) both hold.
- **Industry convention**: "O(n)" often means Θ(n) — imprecise but standard.

## Common Runtimes (fast → slow)

- **O(1) — Constant**: no loops, no recursion, direct access (index, map get, len).
- **O(log n) — Logarithmic**: halves problem each step (binary search, heap push/pop).
- **O(n) — Linear**: visits each element once (for-range, copy, linear search).
- **O(n log n) — Linearithmic**: best comparison sort bound (merge sort, sort.Strings).
- **O(n²) — Quadratic**: nested loops over n (bubble sort, all-pairs).
- **O(n³) — Cubic**: triple nested loops (naive matrix multiply).
- **O(2ⁿ) — Exponential**: branches ×2 per step (naive Fibonacci, power set).
- **O(n!) — Factorial**: all permutations (TSP brute force, n ≤ 12 only).

| n | O(1) | O(log n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) | O(n!) |
|---|------|----------|------|------------|-------|-------|-------|
| 10 | 1 | ~3 | 10 | ~33 | 100 | 1024 | 3.6M |
| 1M | 1 | ~20 | 1M | ~20M | 1T | ∞ | ∞ |

## Time vs Space

- **Time complexity**: operation count growth, not wall-clock seconds.
- **Space complexity**: RAM growth (stack frames + heap allocations).
- **Trade-off**: memoization = O(n) space for O(1) time per query.
- **Go stack**: goroutines start 2KB, grow via copying (no TCO).

## Calculation Rules

- **Drop constants**: O(2n) → O(n), O(500) → O(1).
- **Dominant terms**: O(n² + n) → O(n²).
- **Loops multiply**: nested → multiply depths.
- **Recursion**: O(branches^depth); naive fib = O(2ⁿ).
- **Sequential blocks**: add, then drop non-dominant.

## Go-Specific Rules

- **`append`**: amortized O(1); pre-allocate with `make([]T, 0, n)`.
- **`len` / `cap` / index**: O(1) — struct field, not count.
- **`map` access**: O(1) avg, O(n) worst (hash collisions).
- **`sort.Search`**: O(log n) — standard library binary search.
- **No tail-call optimization**: prefer iteration over deep recursion.
