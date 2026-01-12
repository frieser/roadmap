---
---

## Summary
**co-NP** is the complexity class containing problems where the "No" answer can be verified efficiently (in polynomial time). It is the complement of the NP class. If a problem $L$ is in NP, its complement $\bar{L}$ is in co-NP. A classic example is checking if a boolean formula is a Tautology (always true).

## Detailed Explanation
### Definition
*   **NP**: "Yes" instances have a short certificate (proof) that can be verified in polynomial time.
*   **co-NP**: "No" instances have a short counter-example that can be verified in polynomial time. (Equivalently, "Yes" instances for the complement problem are verifiable).

### Relation to P and NP
*   $P \subseteq co\text{-}NP$.
*   It is unknown if $NP = co\text{-}NP$.
*   Ideally, $P = NP \cap co\text{-}NP$.
*   If $NP \neq co\text{-}NP$, then $P \neq NP$.

### Example: Tautology vs SAT
*   **SAT (Satisfiability)** is in **NP**: "Is there *some* assignment that makes the formula TRUE?" (Certificate: The assignment).
*   **TAUT (Tautology)** is in **co-NP**: "Is the formula TRUE for *all* assignments?"
    *   To show it's NOT a tautology (answer "No"), you just need **one** assignment that makes it FALSE. This counter-example is easy to verify.

### Go Context
In verification logic:
*   NP: "Prove to me this code HAS a bug." (Show me the input that crashes it).
*   co-NP: "Prove to me this code is SAFE." (Harder, requires exhaustive proof, but disproof is easy if you find one bug).

## Interview Questions
**Q: What is the difference between NP and co-NP?**
A: NP problems have efficient verification for "Yes" answers. co-NP problems have efficient verification for "No" answers (counter-examples).

**Q: Is Primality Testing in co-NP?**
A: Yes. Historically it was in both NP and co-NP, and later proved to be in P (AKS algorithm). To prove a number is Composite (not Prime), you just provide a factor (NP).

**Q: What implies if NP != co-NP?**
A: It strongly implies that P != NP.

## Diagram
```mermaid
venn
    NP [NP]
    coNP [co-NP]
    P [P]
    
    P subset NP
    P subset coNP
    Intersection [P]
```
