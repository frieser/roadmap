# Dictionaries

## Summary
Dictionaries store data in key-value pairs. They are ordered (since Python 3.7), mutable, and do not allow duplicate keys. Defined with `{key: value}`.

## Detailed Explanation

### Basic Usage
```python
car = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}

# Access
print(car["model"])
print(car.get("model")) # Safer, returns None if missing
```

### Modifying
```python
car["year"] = 2020       # Update
car["color"] = "red"     # Add
car.pop("model")         # Remove
```

### Merging (Python 3.9+)
Use the `|` operator to merge dictionaries.

```python
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}
merged = dict1 | dict2 
# {'a': 1, 'b': 3, 'c': 4} (values from dict2 overwrite)
```

### Dictionary Comprehension
```python
squares = {x: x*x for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

## Interview Questions

**Q: Are dictionaries ordered?**
**A:** As of Python 3.7+, yes, insertion order is preserved. Before that, they were unordered.

**Q: What can be a dictionary key?**
**A:** Any immutable (hashable) type: strings, numbers, tuples. Lists and other dictionaries cannot be keys.

**Q: How do you safely get a value without risking a KeyError?**
**A:** Use the `.get()` method. `d.get("key", default_value)`. If the key doesn't exist, it returns `None` (or the default value) instead of raising an error.
