# Iterators

## Summary
An **Iterator** is an object that contains a countable number of values and can be iterated upon (e.g., in a `for` loop). Technically, it implements `__iter__()` and `__next__()`.

## Detailed Explanation

### Iterable vs Iterator
*   **Iterable**: An object capable of returning its members one at a time. It implements `__iter__()` which returns an *Iterator*. Examples: list, tuple, string.
*   **Iterator**: An object representing a stream of data. It implements `__next__()` which returns the next item or raises `StopIteration`.

### Creating a Custom Iterator
To make an object iterable, you must implement the iterator protocol.

```python
class MyNumbers:
    def __init__(self, limit):
        self.limit = limit
        self.a = 1

    def __iter__(self):
        return self

    def __next__(self):
        if self.a <= self.limit:
            x = self.a
            self.a += 1
            return x
        else:
            raise StopIteration

myclass = MyNumbers(3)
for x in myclass:
    print(x) # 1, 2, 3
```

### Built-in `iter()` and `next()`
*   `iter(obj)`: Calls `obj.__iter__()`.
*   `next(iterator)`: Calls `iterator.__next__()`.

```python
s = "ABC"
it = iter(s) # String is iterable, returns iterator
print(next(it)) # 'A'
print(next(it)) # 'B'
```

### Sentinel Value
`iter()` can also accept a callable and a sentinel. It calls the function until it returns the sentinel.
```python
# Read fixed-size chunks until empty bytes
# iter(f.read(1024), b'')
```

## Interview Questions

**Q: What is the difference between an Iterable and an Iterator?**
**A:** An Iterable is a container (like a list) that *has* an `__iter__` method. An Iterator is the object returned by that method, which tracks the current state and *has* a `__next__` method.

**Q: What happens when an iterator runs out of items?**
**A:** It raises a `StopIteration` exception. The `for` loop catches this exception internally to stop looping.

**Q: Can you iterate over an iterator twice?**
**A:** Generally no. Once an iterator is exhausted (consumed), it cannot be reset. You must create a new iterator object from the iterable.
