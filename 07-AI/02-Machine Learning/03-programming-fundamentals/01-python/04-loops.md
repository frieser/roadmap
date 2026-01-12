---
tags: ['ai', 'roadmap']
---

## Summary
Loops are fundamental control flow structures in Python used to execute a block of code repeatedly. Python primarily supports `for` loops, which iterate over a sequence or iterable object, and `while` loops, which run as long as a specified condition remains true. Advanced iteration tools like `enumerate()`, `zip()`, and `range()` enhance loop functionality, while `break` and `continue` provide granular control over the execution flow.

## Detailed Explanation

### The `for` Loop
The `for` loop in Python is used for **definite iteration**. It iterates over the items of any sequence (such as a list, string, or tuple) in the order they appear.

```python
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(f"I like {fruit}")
```

### The `while` Loop
The `while` loop is used for **indefinite iteration**. It continues to execute the block of code as long as the test expression is `True`.

```python
count = 0
while count < 5:
    print(f"Count is {count}")
    count += 1
```

### Essential Iteration Tools

#### 1. `range()`
Generates a sequence of numbers, often used to control the number of iterations.
- `range(stop)`
- `range(start, stop)`
- `range(start, stop, step)`

```python
for i in range(2, 10, 2):
    print(i)  # Output: 2, 4, 6, 8
```

#### 2. `enumerate()`
Adds a counter to an iterable and returns it as an enumerate object. This is useful for getting both the index and the value during iteration.

```python
colors = ["red", "green", "blue"]
for index, value in enumerate(colors):
    print(f"Index {index}: {value}")
```

#### 3. `zip()`
Aggregates elements from two or more iterables and returns an iterator of tuples. It stops when the shortest input iterable is exhausted.

```python
names = ["Alice", "Bob"]
scores = [85, 92]
for name, score in zip(names, scores):
    print(f"{name} scored {score}")
```

### Loop Control Statements

- **`break`**: Terminates the current loop and resumes execution at the next statement after the loop.
- **`continue`**: Skips the rest of the code inside the current loop iteration and moves to the next iteration.

```python
for n in range(1, 10):
    if n == 5:
        break  # Exit loop when n is 5
    if n % 2 == 0:
        continue  # Skip even numbers
    print(n)  # Output: 1, 3
```

### Loop `else` Clause
Python allows an optional `else` clause at the end of a loop. The `else` block runs **only if the loop completes naturally** (i.e., it was NOT terminated by a `break` statement).

```python
for i in range(3):
    print(i)
else:
    print("Loop finished successfully!")
```

## Interview Questions

**Q: What is the difference between `range()` and `enumerate()`?**
**A:** `range()` generates a sequence of integers (usually for counting), while `enumerate()` takes an existing iterable and returns pairs of `(index, item)`. Use `range()` when you need to repeat an action N times, and `enumerate()` when you need both the value and its position in a collection.

**Q: How does the `zip()` function handle iterables of different lengths?**
**A:** By default, `zip()` stops iterating as soon as the shortest iterable is exhausted. If you need to iterate until the longest one is finished, you should use `itertools.zip_longest()`.

**Q: In what scenario would you use a `while` loop instead of a `for` loop?**
**A:** Use a `while` loop when the number of iterations is not known in advance and depends on a dynamic condition (e.g., waiting for user input, reading until EOF, or a mathematical convergence). Use `for` loops when iterating over a known collection or sequence.

**Q: What happens if you use `break` inside a loop that has an `else` block?**
**A:** If a `break` statement is executed, the `else` block is skipped entirely. The `else` block only runs if the loop reaches its natural end (exhausts the iterable or the `while` condition becomes false).

**Q: How can you skip the rest of the current iteration and move to the next one?**
**A:** Use the `continue` statement. It immediately stops the current iteration and jumps back to the top of the loop to evaluate the next item or condition.
