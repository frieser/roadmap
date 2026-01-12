# Loops

## Summary
Python has two primitive loop commands: `while` loops and `for` loops. The `for` loop is primarily used for iterating over sequences (like a list, tuple, dictionary, set, or string).

## Detailed Explanation

### While Loop
Executes as long as a condition is true.

```python
i = 1
while i < 6:
    print(i)
    i += 1
```

### For Loop
Iterates over a sequence.

```python
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)
```

### Range Function
To loop a specific number of times, use `range()`.

```python
# 0 to 4
for x in range(5):
    print(x)

# 2 to 5
for x in range(2, 6):
    print(x)
```

### Loop Control Statements
*   **`break`**: Exits the loop completely.
*   **`continue`**: Skips the current iteration and moves to the next.
*   **`else` block**: Runs when the loop completes normally (did NOT hit a `break`).

```python
for x in range(6):
    if x == 3:
        break
    print(x)
else:
    print("Done!") # Will NOT print because break was hit
```

### Enumerate
Access the index and value simultaneously.

```python
for index, value in enumerate(fruits):
    print(f"{index}: {value}")
```

## Interview Questions

**Q: What does the `else` clause do in a loop?**
**A:** It executes only if the loop finishes successfully without encountering a `break` statement.

**Q: How does `range(10)` work? Does it create a list?**
**A:** In Python 3, `range()` returns an immutable sequence object (generator-like), not a list. It generates numbers on demand, saving memory. In Python 2, it created a list.

**Q: How do you iterate over a dictionary?**
**A:** By default, iterating a dictionary yields its keys. Use `.items()` to get key-value pairs, or `.values()` for just values.
