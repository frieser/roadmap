---
---

## Summary
**Probability** is the branch of mathematics concerning numerical descriptions of how likely an event is to occur. In Computer Science, it is fundamental for **Machine Learning**, **Algorithm Analysis** (randomized algorithms), **Cryptography**, and **System Performance** modeling.

## Detailed Explanation

### Core Concepts

1.  **Random Variable**: A variable whose value is subject to variations due to chance (e.g., the result of a dice roll).
2.  **Expectation (E[X])**: The average value of a random variable over many trials.
    *   Formula: $E[X] = \sum x \cdot P(X=x)$
3.  **Variance (Var(X))**: Measures how far a set of numbers is spread out from their average value.
4.  **Independence**: Two events A and B are independent if the occurrence of one does not affect the probability of the other. $P(A \cap B) = P(A) \cdot P(B)$.

### Key Distributions

*   **Uniform Distribution**: All outcomes are equally likely (e.g., rolling a fair die).
*   **Normal (Gaussian) Distribution**: The "Bell Curve". Many natural phenomena follow this. Defined by Mean ($\mu$) and Standard Deviation ($\sigma$).
*   **Binomial Distribution**: Number of successes in $n$ independent yes/no experiments (Bernoulli trials).
*   **Poisson Distribution**: Probability of a given number of events occurring in a fixed interval of time (e.g., number of requests to a server per minute).

### Bayes' Theorem
Describes the probability of an event, based on prior knowledge of conditions that might be related to the event.
$$P(A|B) = \frac{P(B|A) \cdot P(A)}{P(B)}$$

*   **Application**: Spam filters. Calculate the probability an email is *Spam* (A) given it contains the word *"Buy"* (B).

## Go Example

Go's `math/rand/v2` (Go 1.22+) is the standard for pseudo-random number generation.

```go
package main

import (
	"fmt"
	"math"
	"math/rand/v2"
)

func main() {
	// 1. Uniform Distribution (Float between 0.0 and 1.0)
	fmt.Printf("Uniform: %f\n", rand.Float64())

	// 2. Uniform Range (Int between 0 and 99)
	fmt.Printf("Range [0, 100): %d\n", rand.IntN(100))

	// 3. Normal Distribution (Mean 0, StdDev 1)
	// Useful for simulations or initializing ML weights
	fmt.Printf("Norm: %f\n", rand.NormFloat64())

	// Simulation: Coin Flip (Bernoulli Trial)
	heads := 0
	trials := 1000
	for i := 0; i < trials; i++ {
		// 50% chance
		if rand.Float64() < 0.5 {
			heads++
		}
	}
	fmt.Printf("Heads ratio: %.2f (Expected: 0.50)\n", float64(heads)/float64(trials))
}

// Probability Density Function (PDF) for Normal Distribution
func NormalPDF(x, mu, sigma float64) float64 {
	denom := sigma * math.Sqrt(2*math.Pi)
	exponent := -math.Pow(x-mu, 2) / (2 * math.Pow(sigma, 2))
	return (1.0 / denom) * math.Exp(exponent)
}
```

## Interview Questions

### Q: What is the difference between Independent and Mutually Exclusive events?
**A:** **Independent** events imply that the outcome of one does not affect the other (e.g., rolling two dice). **Mutually Exclusive** events cannot happen at the same time (e.g., rolling a 5 and a 6 on a single die roll).

### Q: Explain the Law of Large Numbers.
**A:** It states that as the number of trials increases, the actual average of the results will get closer and closer to the expected value (theoretical average).

### Q: How does a Bloom Filter work (Probability context)?
**A:** A Bloom Filter is a probabilistic data structure that tests whether an element is a member of a set. It can return "False" (definitely not in set) or "True" (possibly in set). It uses multiple hash functions. The probability of a "False Positive" depends on the size of the bit array and the number of hash functions.
