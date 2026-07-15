# Data Structures — Ultra-Compact Schematic
## Structure Overview
| # | DS | Folder | Impl in Go |
|---|----|--------|------------|
| 1 | Array/Slice | `01-array.md` | `[n]T` (value), `[]T` (slice header) |
| 2 | Linked List | `02-linked-list.md` | `container/list` (dbl), custom `*Node[T]` |
| 3 | Stack | `03-stack.md` | Slice `s[:len(s)-1]` |
| 4 | Queue | `04-queue.md` | Slice / chan / `container/list` |
| 5 | Hash Table | `05-hash-table.md` | `map[K]V`, `sync.Map` |
| 6 | Binary Tree | `06-tree/01-binary-tree.md` | `type TreeNode struct` |
| 7 | BST | `06-tree/02-binary-search-tree.md` | Custom struct |
| 8 | Full Binary Tree | `06-tree/03-full-binary-tree.md` | Leaf prop check |
| 9 | Complete Binary Tree | `06-tree/04-complete-binary-tree.md` | Array idx `2i+1` |
| 10 | Balanced Tree | `06-tree/05-balanced-tree.md` | AVL/RB libs only |
| 11 | Unbalanced Tree | `06-tree/06-unbalanced-tree.md` | Naive BST → O(n) |
| 12 | Heap | `08-heap.md` | `container/heap` + `Interface` |
| 13 | Directed Graph | `07-graph/01-directed-graph.md` | `map[int][]int` |
| 14 | Undirected Graph | `07-graph/02-undirected-graph.md` | Symmetric AddEdge |
| 15 | MST/Spanning Tree | `07-graph/03-spanning-tree.md` | Kruskal + UnionFind |
| 16 | Adjacency List | `07-graph/04-representation/01-adjacency-list.md` | `[][]T` / `map[K][]V` |
| 17 | Adjacency Matrix | `07-graph/04-representation/02-adjacency-matrix.md` | `[][]int` V×V |
## Complexity Matrix (avg)
| DS | Insert | Delete | Search | Access |
|----|--------|--------|--------|--------|
| Array (unsorted) | O(n) | O(n) | O(n) | O(1) |
| Array (sorted) | O(n) | O(n) | O(log n) | O(1) |
| Linked List | O(1)* | O(1)* | O(n) | O(n) |
| Stack | O(1) | O(1) | O(n) | O(1)† |
| Queue | O(1) | O(1) | O(n) | O(1)† |
| Hash Table | O(1) | O(1) | O(1) | O(1) |
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(log n) |
| BST (skewed) | O(n) | O(n) | O(n) | O(n) |
| Heap | O(log n) | O(log n) | O(n) | O(1)‡ |
| Adj Matrix | O(1) | O(1) | O(1) | O(1) |
| Adj List | O(1) | O(degree) | O(degree) | O(degree) |
*if pointer known, †top/front only, ‡root peek only
## DS Selection (use case → DS)
| Need | Pick | Why |
|------|------|-----|
| Fast index access | Slice | O(1) via pointer math |
| Fast key→value | map / sync.Map | O(1) avg |
| FIFO processing | Queue (chan/slice) | Order preserved |
| LIFO / backtrack | Stack (slice) | DFS, undo, parens |
| Max/min always | Heap | O(1) peek, O(log n) push |
| Ordered + search | BST / B-Tree | O(log n) sorted ops |
| Hierarchical data | Tree (struct ptr) | Org chart, DOM |
| Relationships | Graph (adj map) | Social, routes, deps |
| Fixed-size fast FIFO | Ring Buffer | No alloc, wraps |
| LRU cache | map + dbl linked list | O(1) get/put |
## Go Patterns & Pitfalls
| Pattern | Detail |
|---------|--------|
| Slice vs array | `[3]int` = value, `[]int` = descriptor (ptr/len/cap) |
| Slice append growth | <256 → 2×, >256 → ~1.25× |
| Slice memory leak | Small sub-slice of big array keeps entire array alive |
| Stack mem leak ptr | Pop pointers: set `s[idx] = nil` before truncating |
| Queue memory drift | `q = q[1:]` leaks front; compact or use ring buffer |
| map zero value | Missing key returns zero-value; use `v, ok` idiom |
| map concurrency | NOT safe; use `sync.RWMutex` or `sync.Map` |
| sync.Map niche | Read-heavy / disjoint keys only; mutex+map faster |
| heap.Interface | Must impl Len/Less/Swap + Push(x any)/Pop() any |
| graph default | `map[int][]int` — sparse, idiomatic, JSON-friendly |
| no std DS | Only `container/list`, `container/heap`, `container/ring` |
| GC pressure | Pointer-heavy DS (linked list, tree nodes) increases STW |
## Interview Hits (Top 5)
- Cycle detect: Floyd Tortoise+Hare (list) / DFS 3-color (graph) / Union-Find (undirected)
- Valid BST: Track min/max bounds recursively — checking immediate children is wrong
- Min-Stack: Two stacks — main + monotonic min stack, O(1) getMin
- Kth largest stream: Min-heap of size K, root = Kth largest
- Topological sort: Kahn (indegree BFS) or DFS post-order on DAG
## Key Big Ideas
- Cache locality > Big O: Slice beats linked list at same O(n) — contiguous CPU cache hits
- Contiguous = fast: Array/slice → prefetcher friendly; pointers → cache misses
- Hash table trade-off: O(1) avg, O(n) worst, no ordering, cannot `&m[k]`
- Heap ≠ BST: Heap finds min/max O(1); BST finds any element O(log n)
- Complete tree = array: Heap uses `2i+1` parent/child math, no pointer overhead
