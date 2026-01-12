#Rust
---
---

## Summary
**HashSet** is a collection of unique values, implemented as a HashMap where the value is void. **LinkedList** is a doubly-linked list. While standard in CS theory, **LinkedList is rarely used in Rust** due to poor cache locality and ergonomic difficulties with the borrow checker; `Vec` is almost always preferred.

## Detailed Explanation

### 1. HashSet (`HashSet<T>`)
- **Use case**: Storing unique items, fast membership testing (`O(1)`), set operations (union, intersection).
- **Requirements**: Type `T` must implement `Eq` and `Hash`.
- **Implementation**: It is literally a `HashMap<T, ()>`.

```rust
use std::collections::HashSet;

let mut books = HashSet::new();
books.insert("The Hobbit");
books.insert("The Hobbit"); // Ignored, duplicate

if books.contains("The Hobbit") {
    println!("We have the book!");
}

// Set Operations
let union: HashSet<_> = s1.union(&s2).collect();
```

### 2. LinkedList (`LinkedList<T>`)
- **Use case**: When you specifically need `O(1)` splitting or merging of lists at arbitrary points, or pushing/popping from both ends (though `VecDeque` is better for ends).
- **Drawbacks**:
  - High memory overhead (pointer per node).
  - Poor CPU cache locality (nodes scattered in heap).
  - Indexing is `O(N)`.

```rust
use std::collections::LinkedList;

let mut list = LinkedList::new();
list.push_back('a');
list.push_front('b');
```

**Note**: The Rust documentation explicitly warns: *"It is almost always better to use `Vec` or `VecDeque` because array-based containers are faster, have better memory locality, and better compiler optimization support."*

## Interview Questions

1. **When should you use a `HashSet` over a `Vec`?**
   - Use `HashSet` when you need to ensure all elements are unique and require fast (`O(1)`) lookups/membership checks. Use `Vec` if order matters, duplicates are allowed, or the collection is very small (where linear scan might beat hashing overhead).

2. **Why is `LinkedList` discouraged in Rust?**
   - `LinkedList` has poor cache locality compared to contiguous memory buffers like `Vec`. Additionally, implementing standard operations on linked lists in Rust is notoriously difficult due to ownership rules (self-referential structures), so the standard library implementation is provided but rarely the most performant choice.

3. **How do you perform a union of two HashSets?**
   - Using the `.union()` iterator method. Note that this returns references to the values; you usually need to `.cloned()` and `.collect()` to create a new Set.
