---
---

# Randomised Algorithms

## Summary
**Randomised Algorithms** use a source of randomness as part of their logic. They are typically used to reduce time or space complexity in the average case or to solve problems where a deterministic solution is too slow/complex.

## Detailed Explanation

### Types
1.  **Las Vegas**: Always produces the correct result, runtime is random. (e.g., Randomized QuickSort).
2.  **Monte Carlo**: Runtime is fixed, correctness is random (probability of error decreases with more iterations). (e.g., Karger’s Min Cut).

## Code Examples (Go)

### Fisher-Yates Shuffle
Shuffles an array in $O(n)$ time.

```go
import (
    "math/rand"
    "time"
)

func Shuffle(arr []int) {
    rand.Seed(time.Now().UnixNano())
    for i := len(arr) - 1; i > 0; i-- {
        j := rand.Intn(i + 1)
        arr[i], arr[j] = arr[j], arr[i]
    }
}
```
