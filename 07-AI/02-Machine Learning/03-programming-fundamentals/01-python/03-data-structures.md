---
tags: ['ai', 'roadmap']
---

## Summary
Python provides several built-in data structures that serve as the foundation for data manipulation in Machine Learning and AI. The four primary structures—**Lists**, **Tuples**, **Sets**, and **Dictionaries**—each have unique characteristics regarding mutability, ordering, and performance. Selecting the right structure is critical for optimizing data pipelines and managing model configurations.

## Detailed Explanation

### 1. Lists (`list`)
Lists are ordered, mutable collections that can store heterogeneous data types. In AI, they are often used to store sequences of data points, batches of images (as paths), or experimental results.

*   **Characteristics**: Ordered, Mutable, Allows duplicates.
*   **Performance**: $O(1)$ for appending, $O(n)$ for searching/inserting at arbitrary positions.

```python
# List example: Storing a sequence of loss values
losses = [0.5, 0.4, 0.35, 0.2]
losses.append(0.15)  # Mutable
print(f"Current loss: {losses[-1]}")
```

### 2. Tuples (`tuple`)
Tuples are ordered, immutable collections. They are preferred when data should not change throughout the execution, such as the shape of a neural network layer or a fixed coordinate.

*   **Characteristics**: Ordered, Immutable, Allows duplicates.
*   **Performance**: Slightly faster than lists and use less memory.

```python
# Tuple example: Defining tensor dimensions or hyperparameter bounds
input_shape = (224, 224, 3)
learning_rate_range = (0.001, 0.1)
# input_shape[0] = 128  # This would raise a TypeError
```

### 3. Sets (`set`)
Sets are unordered collections of unique elements. They are highly efficient for membership testing and eliminating duplicates from a dataset (e.g., finding all unique labels in a classification task).

*   **Characteristics**: Unordered, Mutable (elements must be hashable), No duplicates.
*   **Performance**: $O(1)$ average for membership testing (`in`).

```python
# Set example: Finding unique classes in a dataset
labels = ["cat", "dog", "cat", "bird", "dog"]
unique_labels = set(labels)
print(f"Classes: {unique_labels}")  # {'cat', 'dog', 'bird'}
```

### 4. Dictionaries (`dict`)
Dictionaries are key-value pairs that provide fast lookup. They are ubiquitous in AI for storing model configurations, hyperparameters, and mapping labels to indices.

*   **Characteristics**: Key-Value pairs, Mutable, Keys must be unique and hashable.
*   **Performance**: $O(1)$ average for lookups and insertions.

```python
# Dictionary example: Model hyperparameters
config = {
    "batch_size": 32,
    "optimizer": "Adam",
    "learning_rate": 1e-4,
    "layers": [64, 128, 256]
}
print(f"Using optimizer: {config['optimizer']}")
```

## Interview Questions

**Q: What is the main difference between a list and a tuple, and why would you use one over the other in ML?**
**A:** The primary difference is **mutability**. Lists are mutable, while tuples are immutable. In ML, you use tuples for fixed configurations (like `input_shape`) to prevent accidental modification. Lists are used for dynamic data, such as accumulating results during training loops.

**Q: How does a Python Dictionary handle collisions internally?**
**A:** Python dictionaries use **hash tables** with **open addressing** (specifically, a pseudo-random probing sequence). When two keys hash to the same index, Python looks for the next available slot based on a deterministic algorithm.

**Q: Why is membership testing faster in a Set than in a List?**
**A:** A Set uses a hash table, making membership testing $O(1)$ on average. A List must iterate through elements one by one, resulting in $O(n)$ time complexity.

**Q: What is a List Comprehension, and why is it preferred?**
**A:** It is a concise way to create lists. It is often faster than a standard `for` loop because it is optimized at the C level in CPython. Example: `squares = [x**2 for x in range(10)]`.
