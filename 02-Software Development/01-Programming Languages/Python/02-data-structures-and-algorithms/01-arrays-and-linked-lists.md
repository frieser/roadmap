# Arrays and Linked Lists

## Summary
Python does not have a native "Array" type in the traditional sense. It uses **Lists** (dynamic arrays) as the default sequential data structure. Linked Lists must be implemented manually using classes.

## Detailed Explanation

### 1. Python Lists (Dynamic Arrays)
Python's `list` is a dynamic array, meaning it's a contiguous block of memory that holds **references** (pointers) to objects.

*   **Access**: O(1) - indexing is instant because of contiguous memory.
*   **Append**: Amortized O(1) - usually fast, but occasionally triggers a resize.
*   **Insert/Delete**: O(n) - requires shifting elements.

#### Internal Growth Pattern
When a list is full, Python allocates a larger chunk of memory and copies elements over. The growth factor is approximately **1.125 (plus constant)**, defined as: `new_allocated = new_size + (new_size >> 3) + 6`. This prevents frequent reallocations.

### 2. The `array` Module
Python has a built-in `array` module for arrays of basic C types (integers, floats, characters).
*   **Use case**: Memory efficiency when storing millions of homogenous numbers.
*   **Difference**: Stores actual values, not pointers to objects.

```python
import array
# 'i' type code for signed integer
arr = array.array('i', [1, 2, 3]) 
```

### 3. Linked Lists
A linear collection of nodes where each node points to the next. Not built-in, but easy to implement.
*   **Pros**: O(1) insertion/deletion at known position.
*   **Cons**: O(n) access time.

#### Implementation in Python
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def append(self, data):
        new_node = Node(data)
        if not self.head:
            self.head = new_node
            return
        last = self.head
        while last.next:
            last = last.next
        last.next = new_node
```

## Interview Questions

**Q: Why is inserting at the beginning of a Python `list` slow?**
**A:** Because `list` is implemented as an array. Inserting at index 0 requires shifting all existing N elements one position to the right, making it an O(n) operation.

**Q: What is the difference between `list` and `array.array`?**
**A:** `list` can hold objects of mixed types and stores references (pointers). `array.array` is a thin wrapper around C arrays, stores values directly, requires homogenous types, and is more memory efficient for large datasets of numbers.

**Q: How does Python's list resize itself?**
**A:** It over-allocates memory to avoid resizing on every append. The growth pattern is roughly ~12.5% over-allocation (new_size + new_size >> 3 + 6) to achieve amortized O(1) append time.
