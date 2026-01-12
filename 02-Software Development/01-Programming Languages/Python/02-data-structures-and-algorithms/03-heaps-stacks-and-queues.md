# Heaps, Stacks, and Queues

## Summary
Python provides built-in modules for these structures: `list` for Stacks, `collections.deque` for Queues, and `heapq` for Heaps (Priority Queues).

## Detailed Explanation

### 1. Stacks (LIFO)
Python's built-in `list` is optimized for stack operations.
*   **Push**: `list.append(item)` - O(1)
*   **Pop**: `list.pop()` - O(1) (pops from the end)

```python
stack = []
stack.append(1) # Push
val = stack.pop() # Pop
```

### 2. Queues (FIFO)
Do **NOT** use `list` for queues. Popping from the front (`pop(0)`) is O(n).
Use `collections.deque` (Doubly Ended Queue).
*   **Enqueue**: `deque.append(item)` - O(1)
*   **Dequeue**: `deque.popleft()` - O(1)

```python
from collections import deque
q = deque()
q.append(1)      # Enqueue
val = q.popleft() # Dequeue
```

### 3. Heaps (Priority Queue)
Python's `heapq` module implements a **Min-Heap** on top of a standard list.
*   **Push**: `heapq.heappush(heap, item)` - O(log n)
*   **Pop**: `heapq.heappop(heap)` - O(log n) (returns smallest)
*   **Peek**: `heap[0]` - O(1)

To use as a **Max-Heap**, typically numbers are inverted (multiplied by -1) before pushing.

```python
import heapq

min_heap = []
heapq.heappush(min_heap, 10)
heapq.heappush(min_heap, 1)
heapq.heappush(min_heap, 5)

print(heapq.heappop(min_heap)) # 1 (smallest)
```

## Interview Questions

**Q: Why shouldn't you use a Python list as a Queue?**
**A:** Because removing the first element (`pop(0)`) forces all subsequent elements to shift left in memory, making it an O(n) operation. `deque` is implemented as a doubly-linked list of blocks, allowing O(1) pops from both ends.

**Q: Does Python have a Max-Heap?**
**A:** The `heapq` module only implements a Min-Heap. To simulate a Max-Heap with numbers, developers usually negate the values (store `-x`). For objects, you typically implement a custom `__lt__` method or use a wrapper class.

**Q: What is the time complexity of `heapq.heapify(list)`?**
**A:** O(n). It rearranges the list in-place to satisfy the heap property using a bottom-up approach, which is more efficient than pushing elements one by one (O(n log n)).
