# Generator Expressions

## Summary
Generator Expressions are similar to List Comprehensions but use parentheses `()` instead of brackets `[]`. They return a **generator object** that produces items one by one (lazy evaluation) rather than creating a full list in memory.

## Detailed Explanation

### Syntax
`(expression for item in iterable if condition)`

### Memory Efficiency
Generators are crucial for working with large datasets.

```python
# List Comp: Allocates memory for 1,000,000 integers immediately
my_list = [x * x for x in range(1000000)]

# Gen Expr: Allocates almost zero memory. Computes next value only when asked.
my_gen = (x * x for x in range(1000000))

import sys
print(sys.getsizeof(my_list)) # ~8 MB
print(sys.getsizeof(my_gen))  # ~112 Bytes (constant size)
```

### `yield` Keyword
Functions can become generators using `yield`.

```python
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

counter = count_up_to(5)
print(next(counter)) # 1
```

### `yield from`
Delegates part of its operations to another generator. Useful for flattening nested structures.

```python
def sub_gen():
    yield 1
    yield 2

def main_gen():
    yield 'start'
    yield from sub_gen() # Same as: for x in sub_gen(): yield x
    yield 'end'
```

## Interview Questions

**Q: What is the main advantage of a generator over a list?**
**A:** Memory efficiency. A generator produces items one at a time (lazy evaluation), allowing iteration over infinitely large sequences without crashing RAM.

**Q: Can you slice a generator?**
**A:** No. Generators are not subscriptable. You cannot do `gen[0]` or `gen[:5]`. To slice them, you must use `itertools.islice`.

**Q: What happens if you call `return` inside a generator function?**
**A:** In Python 3, `return value` inside a generator raises `StopIteration(value)`. It effectively stops the generator.
