---
---

## Probability
- **E[X]**: `Σ x·P(X=x)` → expected-case complexity analysis.
- **Var(X)**: `E[(X-μ)²]` → algorithm performance variance.
- **Independence**: `P(A∩B) = P(A)·P(B)` → naive Bayes classifiers.
- **Uniform**: `P(x) = 1/n` → hash-function analysis, RNG.
- **Normal**: `N(μ, σ²)` → ML weight init, noise modeling.
- **Binomial**: `B(n, p)` → A/B testing, error-rate estimation.
- **Poisson**: `Pois(λ)` → server request modeling, queue theory.
- **Bayes' Theorem**: `P(A|B) = P(B|A)·P(A) / P(B)` → spam filters, classification.
- **Law of Large Numbers**: `x̄ₙ → μ` as `n → ∞` → Monte Carlo convergence.
- **Bloom Filter**: `P(fp) ≈ (1 − e⁻ᵏⁿ/ᵐ)ᵏ` → space-efficient set membership.

## Combinatorics
- **Factorial**: `n! = ∏ᵢ₌₁ⁿ i` → `O(n!)` brute-force enumeration.
- **Permutations**: `P(n, r) = n! / (n−r)!` → ordered selections.
- **Combinations**: `C(n, r) = n! / (r!(n−r)!)` → subset count, binomial coefficients.
- **Perm w/ Repetition**: `n! / ∏cᵢ!` → anagram counting, arrangement with dupes.
- **Pigeonhole Principle**: `n > m` ⇒ at least one container has >1 item → hash collisions.
- **Power Set**: `|P(S)| = 2ⁿ` → `O(2ⁿ)` subset enumeration.
- **Birthday Paradox**: `≈ √(2m·ln(1/(1-p)))` for collision → hash collision probability intution.
