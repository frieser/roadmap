---
---

## Summary
**Combinatorics** is the study of counting, arrangement, and combination of objects. It is the mathematical backbone of **algorithm complexity analysis** (how many ways can this loop run?), **cryptography** (key space size), and **optimization problems**.

## Detailed Explanation

### Core Concepts

1.  **Factorial ($n!$)**: The product of all positive integers less than or equal to $n$.
    *   $5! = 5 \times 4 \times 3 \times 2 \times 1 = 120$
    *   $0! = 1$

2.  **Permutations ($nPr$)**: Arrangement of $r$ items from a set of $n$ distinct items where **order matters**.
    *   Formula: $P(n, r) = \frac{n!}{(n-r)!}$
    *   Example: Arranging 3 distinct books on a shelf ($3! = 6$).

3.  **Combinations ($nCr$)**: Selection of $r$ items from a set of $n$ distinct items where **order does not matter**.
    *   Formula: $C(n, r) = \frac{n!}{r!(n-r)!}$
    *   Example: Lottery outcomes.

### Pigeonhole Principle
If $n$ items are put into $m$ containers, with $n > m$, then at least one container must contain more than one item.
*   **CS Application**: Proving hash collisions are inevitable if the input space is larger than the hash space.

## Go Example

Combinatorics often involves large numbers that exceed standard `int64`. Go's `math/big` package is essential here.

```go
package main

import (
	"fmt"
	"math/big"
)

func main() {
	n := int64(10)
	r := int64(3)

	fmt.Printf("Permutations P(%d, %d): %s\n", n, r, Permutation(n, r).String())
	fmt.Printf("Combinations C(%d, %d): %s\n", n, r, Combination(n, r).String())
}

// Factorial calculates n! using big.Int to prevent overflow
func Factorial(n int64) *big.Int {
	result := big.NewInt(1)
	for i := int64(2); i <= n; i++ {
		result.Mul(result, big.NewInt(i))
	}
	return result
}

// Permutation P(n, r) = n! / (n-r)!
func Permutation(n, r int64) *big.Int {
	num := Factorial(n)
	denom := Factorial(n - r)
	return new(big.Int).Div(num, denom)
}

// Combination C(n, r) = n! / (r! * (n-r)!)
func Combination(n, r int64) *big.Int {
	if r < 0 || r > n {
		return big.NewInt(0)
	}
	num := Factorial(n)
	denom := new(big.Int).Mul(Factorial(r), Factorial(n-r))
	return new(big.Int).Div(num, denom)
}

// Generating Permutations (Backtracking) - O(N!)
func GeneratePermutations(arr []int) [][]int {
	var res [][]int
	var backtrack func(int)
	
	backtrack = func(first int) {
		if first == len(arr) {
			temp := make([]int, len(arr))
			copy(temp, arr)
			res = append(res, temp)
			return
		}
		for i := first; i < len(arr); i++ {
			arr[first], arr[i] = arr[i], arr[first] // Swap
			backtrack(first + 1)
			arr[first], arr[i] = arr[i], arr[first] // Backtrack
		}
	}
	backtrack(0)
	return res
}
```

## Interview Questions

### Q: How many ways can you arrange the letters of the word "MISSISSIPPI"?
**A:** This is a permutation with repetition. Total letters $n=11$. M=1, I=4, S=4, P=2.
$$ \frac{11!}{1! \cdot 4! \cdot 4! \cdot 2!} = 34,650 $$

### Q: What is the complexity of generating all subsets of a set (Power Set)?
**A:** $O(2^n)$. For a set with $n$ elements, each element can either be in the subset or not (2 choices).

### Q: Explain the Birthday Paradox.
**A:** In a group of just 23 people, there is a greater than 50% chance that two people share the same birthday. This relates to **hash collisions**—collisions are much more likely than intuition suggests.
