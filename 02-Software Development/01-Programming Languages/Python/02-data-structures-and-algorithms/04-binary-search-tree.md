# Binary Search Tree (BST)

## Summary
A Binary Search Tree is a tree where for every node, the left child is smaller and the right child is larger. Python does not provide a built-in BST class (unlike Java's `TreeMap`), so it is commonly implemented manually for interviews.

## Detailed Explanation

### Class Structure
A simple recursive implementation.

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class BST:
    def insert(self, root, val):
        if not root:
            return TreeNode(val)
        if val < root.val:
            root.left = self.insert(root.left, val)
        else:
            root.right = self.insert(root.right, val)
        return root

    def search(self, root, val):
        if not root or root.val == val:
            return root
        if val < root.val:
            return self.search(root.left, val)
        return self.search(root.right, val)
```

### Traversal (DFS)
*   **In-order**: Left -> Root -> Right (Returns sorted values)
*   **Pre-order**: Root -> Left -> Right
*   **Post-order**: Left -> Right -> Root

### Complexity
*   **Search/Insert/Delete**: O(h), where h is height.
*   **Balanced**: O(log n)
*   **Skewed (Worst case)**: O(n) (acts like a linked list)

## Interview Questions

**Q: Does Python have a built-in balanced tree data structure?**
**A:** No. The standard library does not include Red-Black Trees or AVL trees. `dict` and `set` use Hash Tables. If you need an ordered dictionary, `collections.OrderedDict` preserves insertion order, but it's not a BST.

**Q: What is the worst-case complexity of a BST search?**
**A:** O(n), which happens if the tree is completely skewed (e.g., inserting sorted numbers 1, 2, 3, 4, 5 results in a linked list structure).

**Q: How do you validate if a tree is a valid BST?**
**A:** Perform an In-order traversal and check if the resulting list is strictly increasing. Alternatively, use recursion passing `min` and `max` constraints down to each node.
