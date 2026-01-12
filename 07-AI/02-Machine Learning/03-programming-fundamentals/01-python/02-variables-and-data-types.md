---
tags: ['ai', 'roadmap']
---

## Summary
Python is a **dynamically-typed** language where variables are names that point to objects in memory. It supports several fundamental built-in data types: integers, floating-point numbers, strings, and booleans. Understanding these types and how to convert between them is essential for AI engineering, where data often needs to be parsed, transformed, and cleaned before being fed into models.

## Detailed Explanation

### Dynamic Typing
In Python, the **type is associated with the value**, not the variable name. Unlike statically-typed languages (like C++ or Java), you do not need to declare a variable's type. This allows for high flexibility during data exploration and rapid prototyping.

```python
# Dynamic typing example
data = 100        # data is an int
print(type(data)) # <class 'int'>

data = "100"      # data is now a str
print(type(data)) # <class 'str'>
```

### Basic Data Types

#### 1. Integer (`int`)
Represents whole numbers. Python 3 integers have arbitrary precision, meaning they can be as large as the available memory—critical for handling large IDs or counts in big data.
```python
batch_size = 32
epoch_count = 100
```

#### 2. Floating-Point (`float`)
Represents real numbers with decimal points. They are implemented using `double` in C, following the IEEE 754 standard. Floats are the primary data type for model weights, probabilities, and normalized features.
```python
learning_rate = 0.001
accuracy = 0.985
```

#### 3. String (`str`)
Represents textual data. Strings are immutable sequences of Unicode characters. In AI, strings are the foundation for Natural Language Processing (NLP) tasks.
```python
prompt = "Explain quantum computing in simple terms."
label = 'cat'
```

#### 4. Boolean (`bool`)
Represents truth values: `True` or `False`. Booleans are a subclass of integers (`True` behaves like `1`, `False` like `0`).
```python
is_converged = True
use_gpu = False
```

### Type Casting (Conversion)
Type casting is the process of converting a value from one type to another. This is frequently used when reading data from files (e.g., CSVs where numbers might be read as strings).

*   **Explicit Casting**: Using constructor functions like `int()`, `float()`, and `str()`.
    ```python
    raw_input = "0.75"
    threshold = float(raw_input)  # "0.75" -> 0.75
    
    loss = 0.00456
    loss_str = str(loss)          # 0.00456 -> "0.00456"
    ```
*   **Implicit Casting**: Python automatically promotes types in expressions.
    ```python
    sum_val = 10 + 5.5  # int + float results in float (15.5)
    ```

### Variable Naming (PEP 8)
- Use `snake_case` (e.g., `model_architecture`).
- Names cannot start with a number.
- Avoid using reserved keywords (e.g., `list`, `dict`, `type`) as variable names to prevent shadowing built-ins.

## Interview Questions

### 1. What is the difference between dynamic typing and static typing?
**Answer**: In dynamic typing (Python), types are checked at runtime, and variables can change types. In static typing (Java/C++), types are checked at compile-time, and a variable's type must be declared and remains fixed.

### 2. How do you check the data type of a variable in Python?
**Answer**: Use the built-in `type()` function. For example, `type(3.14)` returns `<class 'float'>`. To check if a variable is of a specific type, `isinstance(var, type)` is often preferred.

### 3. What will `bool(0)`, `bool("")`, and `bool([])` evaluate to?
**Answer**: All will evaluate to `False`. In Python, zero, empty strings, and empty collections are considered "falsy" values.

### 4. Why is type casting important in AI/Data Science pipelines?
**Answer**: Data ingestion often yields mixed types (e.g., numbers read as strings from a text file). Explicitly casting these to `float` or `int` is necessary before performing mathematical operations or feeding data into machine learning frameworks like NumPy or PyTorch.

### 5. Does Python have a maximum value for integers?
**Answer**: No, Python 3 integers have arbitrary precision and are only limited by the amount of available memory on the system.
