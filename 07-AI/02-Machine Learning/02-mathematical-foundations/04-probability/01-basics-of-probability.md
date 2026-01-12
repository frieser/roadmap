---
tags: ['ai', 'roadmap']
---

## Summary

Probability theory provides the formal framework for quantifying uncertainty and reasoning about random events. In the context of AI and Machine Learning, it is the bedrock upon which algorithms handle noisy data, make predictions, and quantify confidence levels. Basics include understanding sample spaces, events, axioms, and the fundamental rules of conditional probability and independence.

## Detailed Explanation

Probability is the measure of the likelihood that an event will occur, expressed as a number between 0 and 1.

### Core Concepts

1.  **Experiment**: Any procedure that can be infinitely repeated and has a well-defined set of possible outcomes (e.g., rolling a die).
2.  **Sample Space ($S$)**: The set of all possible outcomes of an experiment. For a 6-sided die, $S = \{1, 2, 3, 4, 5, 6\}$.
3.  **Event ($A$)**: A subset of the sample space. For example, "rolling an even number" is $A = \{2, 4, 6\}$.
4.  **Probability $P(A)$**: A value such that $0 \le P(A) \le 1$.

### The Three Axioms (Kolmogorov)

Any function $P$ that maps events to real numbers is a probability measure if it satisfies:
*   **Non-negativity**: $P(A) \ge 0$ for any event $A$.
*   **Normalization**: $P(S) = 1$ (the probability of *something* happening is 1).
*   **Additivity**: For any sequence of mutually exclusive events $A_1, A_2, \dots$, the probability of their union is the sum of their individual probabilities.

### Key Rules and Formulas

*   **Complement Rule**: $P(A^c) = 1 - P(A)$, where $A^c$ is the event that $A$ does not occur.
*   **Addition Rule**: $P(A \cup B) = P(A) + P(B) - P(A \cap B)$.
*   **Conditional Probability**: The probability of $A$ occurring given that $B$ has already occurred:
    $$P(A|B) = \frac{P(A \cap B)}{P(B)}$$
*   **Independence**: Two events $A$ and $B$ are independent if the occurrence of one does not affect the probability of the other:
    $$P(A \cap B) = P(A) \cdot P(B) \iff P(A|B) = P(A)$$

### Python Example: Simulating Dice Rolls

Using `numpy` to demonstrate the Law of Large Numbers and basic probability rules.

```python
import numpy as np

# Set seed for reproducibility
np.random.seed(42)

# Simulate 100,000 rolls of a fair 6-sided die
n_rolls = 100000
rolls = np.random.randint(1, 7, size=n_rolls)

# Event A: Rolling an even number (2, 4, 6)
# Event B: Rolling a number greater than 4 (5, 6)

prob_A = np.mean((rolls % 2 == 0))
prob_B = np.mean((rolls > 4))
prob_A_and_B = np.mean((rolls % 2 == 0) & (rolls > 4)) # Only 6 matches

print(f"P(Even): {prob_A:.4f} (Theoretical: 0.5000)")
print(f"P(>4): {prob_B:.4f} (Theoretical: 0.3333)")
print(f"P(Even and >4): {prob_A_and_B:.4f} (Theoretical: 0.1667)")

# Demonstrate Conditional Probability: P(Even | >4)
# Formula: P(A|B) = P(A and B) / P(B)
prob_A_given_B = prob_A_and_B / prob_B
print(f"P(Even | >4): {prob_A_given_B:.4f} (Theoretical: 1/2 = 0.5000)")

# Check for Independence: Does P(A and B) == P(A) * P(B)?
is_independent = np.isclose(prob_A_and_B, prob_A * prob_B, atol=1e-3)
print(f"Are A and B independent? {is_independent}")
# They are NOT independent because P(Even | >4) = 0.5 and P(Even) = 0.5? 
# Wait, 6 is the only number that is both even and > 4.
# P(Even) = 3/6 = 0.5.
# P(>4) = 2/6 = 1/3.
# P(Even | >4) = P({6}) / P({5, 6}) = (1/6) / (2/6) = 0.5.
# Since P(A|B) == P(A), they ARE independent in this specific case!
```

## Interview Questions

**Q: What is the difference between Mutually Exclusive and Independent events?**
**A:** Mutually exclusive events cannot happen at the same time ($P(A \cap B) = 0$). Independent events are those where the occurrence of one does not change the probability of the other ($P(A|B) = P(A)$). If two events have non-zero probability and are mutually exclusive, they *cannot* be independent.

**Q: Explain the Law of Total Probability.**
**A:** It expresses the total probability of an outcome which can be realized via several distinct events. If $\{B_1, B_2, \dots, B_n\}$ is a partition of the sample space, then $P(A) = \sum_i P(A|B_i)P(B_i)$.

**Q: What is a Probability Mass Function (PMF) vs. a Probability Density Function (PDF)?**
**A:** A PMF is used for discrete random variables (like a die roll) and gives the probability that the variable exactly equals a value. A PDF is used for continuous random variables; the probability of a specific point is 0, but the integral over an interval gives the probability of the variable falling within that range.

**Q: If you flip a fair coin 10 times and get 10 heads, what is the probability that the next flip is heads?**
**A:** It is still 0.5. Since the coin flips are independent events, previous outcomes do not influence future ones (assuming the coin is truly fair). Thinking otherwise is known as the "Gambler's Fallacy."
