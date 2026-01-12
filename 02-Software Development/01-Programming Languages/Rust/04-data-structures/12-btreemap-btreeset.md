#Rust
---
---

## Summary
`BTreeMap` and `BTreeSet` are ordered collections based on B-Trees. Unlike `HashMap`/`HashSet` which are unordered and use hashing, B-Tree variants keep keys **sorted** and allow for range queries. They are generally slower for single lookups `O(log N)` vs `O(1)` but are essential when order matters.

## Detailed Explanation

### 1. BTreeMap (`BTreeMap<K, V>`)
- **Key Requirement**: Must implement `Ord` (Total Ordering).
- **Behavior**: Iterating over the map yields keys in sorted order.
- **Cache**: B-Trees are more cache-friendly than binary search trees (BST) because they store multiple keys in a single node, reducing pointer chasing.

```rust
use std::collections::BTreeMap;

let mut map = BTreeMap::new();
map.insert(3, "c");
map.insert(1, "a");
map.insert(2, "b");

for (key, value) in &map {
    println!("{}: {}", key, value);
}
// Output guaranteed:
// 1: a
// 2: b
// 3: c
```

### 2. BTreeSet (`BTreeSet<T>`)
- **Behavior**: A set of unique values that is always sorted.
- **Range Queries**: Very powerful feature.

```rust
use std::collections::BTreeSet;
use std::ops::Bound::Included;

let mut set = BTreeSet::new();
set.insert(1);
set.insert(3);
set.insert(5);
set.insert(7);

// Get numbers between 3 and 6
for &elem in set.range((Included(&3), Included(&6))) {
    println!("{}", elem); // Prints 3, 5
}
```

## Interview Questions

1. **What is the main difference between `HashMap` and `BTreeMap`?**
   - `HashMap` is unordered and uses hashing (O(1) lookups). `BTreeMap` is sorted and uses comparisons (O(log N) lookups). Use `BTreeMap` when you need to iterate over the data in sorted order or perform range queries.

2. **Why does Rust use B-Trees instead of Red-Black Trees (like C++ `std::map`)?**
   - B-Trees have better cache locality. Red-Black trees require following a pointer for every node access, causing cache misses. B-Trees store multiple keys in a node (page), fitting better into CPU cache lines, which often makes them faster in practice despite the theoretical complexity similarity.

3. **Can you use `f64` as a key in `BTreeMap`?**
   - No, because `BTreeMap` keys require the `Ord` trait (total ordering). Floats only implement `PartialOrd` due to NaN. You would need to wrap the float in a type that asserts standard ordering (e.g., using `ordered_float` crate).
