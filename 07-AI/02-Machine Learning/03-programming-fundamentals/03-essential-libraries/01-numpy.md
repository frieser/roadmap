---
tags: ['ai', 'roadmap', 'numpy', 'python']
---

## Summary
**NumPy** (Numerical Python) is the foundational library for scientific computing in Python. It provides a high-performance multidimensional array object (\`ndarray\`) and tools for working with these arrays. It is the backbone of the entire Python AI/ML ecosystem, with libraries like Pandas, Matplotlib, Scikit-learn, and TensorFlow built directly on top of it. Its efficiency comes from "vectorized" operations implemented in C, which eliminate the overhead of Python loops.

## Detailed Explanation

### 1. The ndarray (N-dimensional array)
The core feature of NumPy is the \`ndarray\` object. Unlike Python lists, NumPy arrays are **homogeneous** (all elements must be of the same type) and stored in contiguous memory blocks, allowing for extremely fast access and manipulation.

\`\`\`python
import numpy as np

# Creating an array
arr = np.array([1, 2, 3, 4, 5])
print(f"Shape: {arr.shape}") # (5,)
print(f"Type: {arr.dtype}")   # int64

# Multi-dimensional array (Matrix)
matrix = np.array([[1, 2], [3, 4]])
print(f"Matrix shape: {matrix.shape}") # (2, 2)
\`\`\`

### 2. Vectorization
Vectorization is the ability to perform operations on entire arrays without explicit \`for\` loops. This "batch processing" is what makes NumPy so fast.

\`\`\`python
# Instead of this:
# for i in range(len(arr)):
#     arr[i] = arr[i] * 2

# Do this:
vectorized_result = arr * 2
print(vectorized_result) # [ 2,  4,  6,  8, 10]
\`\`\`

### 3. Broadcasting
Broadcasting allows NumPy to perform arithmetic operations on arrays of different shapes. The smaller array is "broadcast" across the larger one so that they have compatible shapes.

**Rules of Broadcasting:**
1. If the arrays don't have the same number of dimensions, prepend the shape of the lower-dimension array with 1s until both shapes have the same length.
2. If the sizes of each dimension are either equal or one of them is 1, the arrays are compatible.

\`\`\`python
a = np.array([1, 2, 3])
b = 2 # Scalar is broadcast to [2, 2, 2]
print(a + b) # [3, 4, 5]

m = np.ones((3, 3))
print(m + a) # 'a' is broadcast across each row of 'm'
\`\`\`

### 4. Indexing and Slicing
NumPy extends Python's slicing syntax to multiple dimensions and adds powerful "boolean indexing".

\`\`\`python
data = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# Slicing: rows 0 to 1, column 1
print(data[:2, 1]) # [2, 5]

# Boolean Indexing: Select elements greater than 5
print(data[data > 5]) # [6, 7, 8, 9]

# Fancy Indexing: Select specific rows
print(data[[0, 2]]) # Returns row 0 and row 2
\`\`\`

## Interview Questions

**Q: Why is NumPy preferred over Python lists for numerical data?**
**A:** NumPy arrays are more memory-efficient and significantly faster. They use contiguous memory and fixed-type elements, allowing the CPU to optimize operations. Furthermore, NumPy's implementation of vectorized operations in C avoids the overhead of the Python interpreter's loops.

**Q: What happens if you try to add two arrays of different shapes?**
**A:** NumPy will attempt to use **broadcasting**. If the shapes are compatible (according to broadcasting rules), the operation succeeds. If they are not compatible (e.g., adding a (2,3) matrix to a (2,2) matrix), NumPy will raise a \`ValueError\`.

**Q: How do you handle missing values in a NumPy array?**
**A:** NumPy uses \`np.nan\` (Not a Number) to represent missing or undefined data. Note that \`nan\` is a float, so the array must be of a float type. To check for them, use \`np.isnan(arr)\`.

**Q: What is the difference between \`copy\` and \`view\` in NumPy?**
**A:** A **view** is just a different way of looking at the same data (shallow copy). Modifying a view changes the original array. A **copy** is a completely new array with its own data (deep copy). Slicing an array typically returns a view.
