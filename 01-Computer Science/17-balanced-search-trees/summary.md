---
---

## Rotations
- **Left**: Right child becomes pivot. Parent becomes pivot's left child. Used: right-heavy.
- **Right**: Left child becomes pivot. Parent becomes pivot's right child. Used: left-heavy.

## Tree Comparison
| Tree | Balance Method | Search | Insert | Delete | Use Case |
|------|---------------|--------|--------|--------|----------|
| AVL | BF ∈ {-1,0,1} | O(log n) | O(log n) | O(log n) | Read-heavy, strict lookup |
| Red-Black | Color invariants, equal black height | O(log n) | O(log n) | O(log n) | Write-heavy, std libs (C++ `map`, Java `TreeMap`) |
| 2-3 Tree | Perfect balance, splits grow upward | O(log n) | O(log n) | O(log n) | Theoretical, basis for RB/B-Trees |
| 2-3-4 Tree | Top-down split, isomorphic to RB | O(log n) | O(log n) | O(log n) | Theoretical, 1:1 RB mapping |
| B-Tree | min ⌈m/2⌉ children, all leaves same depth | O(log_m n) | O(log_m n) | O(log_m n) | DB/file systems, BoltDB |
| B+ Tree | B-Tree variant, data only in leaves, leaves linked | O(log_m n) | O(log_m n) | O(log_m n) | Range scans, MySQL, Postgres |
| K-D Tree | Axis-cycling median split | O(log n)* | O(log n) | O(log n) | Spatial NN, range queries |
| K-ary Tree | Fixed k children | O(log_k n) | O(log_k n) | O(log_k n) | Tries (k=26), Quadtrees (k=4), Octrees (k=8) |
| Skip List | Probabilistic levels (coin flip) | O(log n) avg | O(log n) avg | O(log n) avg | Concurrent/lock-free, Redis Sorted Sets |
> *K-D Tree: O(n) in high dimensions (curse of dimensionality).

## Rotations per Op
| Structure | Insert (max rotations) | Delete (max rotations) |
|-----------|----------------------|----------------------|
| AVL | 2 | O(log n) |
| Red-Black | 2 | 3 |
| B-Tree | 0 (splits) | 0 (merges) |
| Skip List | 0 | 0 |

## Key Relationships
- 2-3 Tree = B-Tree of order 3
- 2-3-4 Tree ↔ Red-Black (isomorphic: 4-node = black+2 red children)
- B+ Tree: internal nodes = routing only, data = leaves, leaves = linked list
- K-ary Tree generalizes Binary (k=2), Trie (k=26), Quadtree (k=4), Octree (k=8)
