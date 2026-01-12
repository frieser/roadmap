# Recursion

## Summary
Recursion is when a function calls itself. While powerful for algorithms like DFS or Tree Traversal, Python imposes strict limits on recursion depth and **does not** support Tail Call Optimization (TCO).

## Detailed Explanation

### The Recursion Limit
Python limits the stack depth to prevent infinite recursions from causing a C-level stack overflow (segfault).
*   **Default Limit**: Usually 1000.
*   **Check/Set**:
    ```python
    import sys
    print(sys.getrecursionlimit())
    sys.setrecursionlimit(2000)
    ```

### Lack of Tail Call Optimization
In languages like Scheme or optimized C++, tail recursion (returning the result of the recursive call directly) is optimized into a loop. **Python does not do this.** Even a tail-recursive function will add a new stack frame for every call, consuming memory O(n).

### Memoization
To optimize recursion (e.g., Fibonacci), Python provides a built-in decorator.

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)
```

### Code Example: Factorial
```python
def factorial(n):
    # Base case
    if n == 1:
        return 1
    # Recursive step
    return n * factorial(n - 1)
```

## Interview Questions

**Q: Does Python support Tail Call Optimization?**
**A:** No. Guido van Rossum (Python's creator) explicitly decided against it to preserve stack traces for debugging. Every recursive call consumes a stack frame.

**Q: How do you handle `RecursionError: maximum recursion depth exceeded`?**
**A:** You can increase the limit using `sys.setrecursionlimit()`, but the better solution is usually to refactor the recursive algorithm into an iterative one (using a `while` loop and a stack).

**Q: What is `@lru_cache`?**
**A:** It's a decorator from `functools` that implements memoization. It caches the results of function calls based on their arguments, turning O(2^n) recursive algorithms (like naive Fibonacci) into O(n).
