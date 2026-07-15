---
---

## Class Hierarchy

| Class | Definition | Example Problem |
|-------|-----------|----------------|
| **P** | Solvable by deterministic TM in O(n^k) | Sorting, Dijkstra, Primality (AKS) |
| **NP** | "Yes" instance verifiable in polynomial time | SAT, Sudoku, Subset Sum |
| **co-NP** | "No" instance verifiable in polynomial time | TAUT (always true?), UNSAT |
| **NP-Complete** | In NP + NP-Hard (every NP reduces to it) | 3-SAT, TSP Decision, 0/1 Knapsack Decision, Clique |
| **NP-Hard** | At least as hard as NP; ∀L ∈ NP, L ≤_p H | Halting Problem, TSP Optimization, Bin Packing |

## Class Relations

- P ⊆ NP  (proven)
- P ⊆ co-NP  (proven)
- NP-Complete = NP ∩ NP-Hard
- If NP ≠ co-NP → P ≠ NP
- Unknown: P = NP? NP = co-NP?

## P vs NP

- Status: Millennium Prize ($1M), unsolved. Consensus: P ≠ NP.
- If P = NP → RSA/ECC broken, perfect optimization, theorem proving trivial.
- If P ≠ NP → crypto secure, heuristics/approximations necessary (current reality).

## Problem Details

| Problem | Type | Exact | Trick |
|---------|------|-------|-------|
| TSP | NP-Hard (optimization) / NP-Complete (decision) | Held-Karp DP: O(n²·2^n) | Metric: MST 2-approx, Christofides 1.5-approx |
| 0/1 Knapsack | NP-Complete (decision) / NP-Hard (optimization) | DP: O(n·W) pseudo-poly | Fractional variant in P (greedy) |
| Longest Path | NP-Hard (general) | Backtracking O(V!) | DAG: O(V+E) via topological sort |

## Engineer Strategy

| Problem Type | Strategy |
|--------------|----------|
| P | Find best polynomial algorithm |
| NP-Complete (small n) | Exact DP/backtracking |
| NP-Complete (large n) | Heuristics, approximations, branch-and-bound |
| NP-Hard optimization | Approximation with proven bounds |
| NP-Hard undecidable | Restrict input, incomplete methods |

<!-- Space: P ⊆ NP ⊆ PSPACE ⊆ EXPTIME ⊆ EXPSPACE. TSP Held-Karp: O(n·2^n) space. -->
