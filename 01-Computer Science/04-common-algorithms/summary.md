---
---

# Common Algorithms — Compact Reference

## Graph Traversal

| Algo | Data | Time | Space | Purpose |
|------|------|------|-------|---------|
| BFS | Queue | O(V+E) | O(V) | Shortest path (unweighted), level-order |
| DFS | Stack/Recursion | O(V+E) | O(V) | Topo sort, cycle detect, maze |

- BFS: Level-by-level via queue. Shortest path in unweighted graphs.
- DFS: Deep-first via stack. Backtracking + cycle detection.

## Shortest Path

| Algo | Time | Space | Neg. Weights | Heuristic |
|------|------|-------|-------------|-----------|
| Dijkstra | O(E log V) | O(V) | No | None |
| Bellman-Ford | O(V·E) | O(V) | Yes (detects cycles) | None |
| A* | O(E log V) | O(V) | No | f(n)=g(n)+h(n) |

- Dijkstra: Greedy pick closest unvisited. Non-negative weights only.
- Bellman-Ford: Relax all edges V-1x. Handles negatives, slower.
- A*: Dijkstra + admissible heuristic. h(n)=0 → Dijkstra.

## Tree Traversal

| Order | Pattern | Use |
|-------|---------|-----|
| Pre-order | Root→Left→Right | Serialize, copy, prefix notation |
| In-order | Left→Root→Right | BST → sorted output |
| Post-order | Left→Right→Root | Delete, RPN evaluation, LCA |
| Level-order | Queue, level by level | BFS on tree |

- DFS (tree): Pre/In/Post. Stack O(h). BFS (tree): Queue O(w). No visited set.

## Sorting

| Name | Best | Avg | Worst | Space | Stable |
|------|------|-----|-------|-------|--------|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |

- Bubble: Educational. Selection: Min swaps O(n), unstable.
- Insertion: Fast small/almost-sorted. Go stdlib hybrid sort uses it.
- Heap: In-place guarantee. Poor cache. Quick: Fastest. Pivot avoids O(n²).
- Merge: Guaranteed stable O(n log n). Best for linked lists.

## Searching

| Algo | Time | Space | Sorted? | Notes |
|------|------|-------|---------|-------|
| Linear | O(n) | O(1) | No | Faster for n<50. Sentinel opt. |
| Binary | O(log n) | O(1) | Yes | `sort.Search`. `low+(high-low)/2`. |

## Recursion

| Type | Last Action | Stack (TCO) | Go |
|------|------------|-------------|----|
| Tail | Recursive call | O(1) w/ TCO | No TCO |
| Non-tail | Post-call work | Always O(n) | Prefer loops |

- Tail: Accumulator. Go lacks TCO — use loops. Non-tail: Work on unwind.

## Greedy

- Dijkstra: Closest unvisited → global optimum (non-neg weights).
- Huffman: Merge two smallest. Optimal prefix-free code. DEFLATE.
- Kruskal: Sort edges, pick non-cyclic. Union-Find. Best sparse.
- Prim: Grow MST via PQ. Near-identical to Dijkstra (conn cost).
- Ford-Fulkerson: Augmenting paths. Edmonds-Karp O(V·E²). Max flow = min cut.

## Backtracking

| Problem | Complexity | Key |
|---------|-----------|-----|
| Hamiltonian | O(N!) | Held-Karp DP: O(N²·2^N) |
| N-Queens | O(N!) | Bitmasks for O(1) validity. 92 solutions N=8 |
| Maze | O(4^(N²)) | BFS=shortest. DFS=simpler, any path |
| Knight's Tour | O(8^64) brute | Warnsdorff: fewest onward moves first |

## Caches

| Policy | Evicts | Ops | For |
|--------|--------|-----|-----|
| LRU | Least recently used | O(1) | Recency bias, sessions |
| LFU | Least frequently used | O(1) | Stable popularity, CDN |
| MFU | Most frequently used | O(1) | Rare. Cyclic scans |

- LRU: DLL+map. `container/list`. LFU: Freq→list+minFreq. Aging prevents pollution.

## String Algorithms

| Algo | Time (Avg) | Space | Strength |
|------|-----------|-------|----------|
| Naive | O(N·M) | O(1) | Simple |
| KMP | O(N+M) | O(M) | Text pointer never backtracks |
| Rabin-Karp | O(N+M) | O(1) | Rolling hash. Multi-pattern |
| Suffix Array | O(M log N) lookup | O(N) | Static text. LCP for repeats |

- Default: `strings.Index`. KMP: LPS array. j resets on mismatch.
- Rabin-Karp: O(1) hash slide. Char verify on match. `index/suffixarray` for SA.

## Go Rules

1. `sort.Slice`/`slices.Sort` (pdqsort). Never hand-roll sorting.
2. `strings.Index` for pattern search. KMP/RK only for multi-pattern/streaming.
3. Go lacks TCO. Tail recursion → loop.
4. Binary search via `sort.Search`. Mid: `low+(high-low)/2`.
5. Cache: `container/list` for LRU. `sync.RWMutex` for concurrency.
