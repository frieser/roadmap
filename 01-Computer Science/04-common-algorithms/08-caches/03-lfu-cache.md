---
---

# LFU Cache (Least Frequently Used)

## Abstract
**LFU (Least Frequently Used)** is a cache eviction policy that discards the items used **least often**. It tracks the frequency of access for every item. If multiple items have the same lowest frequency, the **Least Recently Used (LRU)** among them is usually evicted.

## Development

### Core Concept
We need to track:
1.  **Frequency**: How many times a key was accessed.
2.  **Order**: For tie-breaking (LRU within frequency bucket).

### Data Structures ($O(1)$ Implementation)
1.  **Key $\to$ Node Map**: `key -> {value, freq, listIterator}`.
2.  **Freq $\to$ List Map**: `freq -> DoublyLinkedList`.
3.  **MinFreq**: Variable to track current minimum frequency to find eviction candidate quickly.

### Operations
-   **Get(key)**: Increment frequency. Move node from `FreqList[f]` to `FreqList[f+1]`. Update `MinFreq` if `FreqList[MinFreq]` empty.
-   **Put(key, value)**:
    -   If new: Insert into `FreqList[1]`. Set `MinFreq = 1`.
    -   If exists: Similar to Get + update value.
    -   If full: Remove tail from `FreqList[MinFreq]`.

## Code Examples (Go)

```go
type LFUCache struct {
    capacity int
    minFreq  int
    nodes    map[int]*list.Element // Key -> Element in List
    lists    map[int]*list.List    // Freq -> List of Nodes
    vals     map[int]int           // Key -> Value
    freqs    map[int]int           // Key -> Frequency
}

// Full implementation is verbose (handling map updates and list moves).
// Key takeaway: You need a map of lists to get O(1).
```

## Go Application
-   **Content Delivery Networks (CDNs)**: LFU is better than LRU for static assets where historical popularity predicts future usage better than just "recency".
-   **DNS Caches**: High frequency domains stay.

## Interview Preparation
1.  **LFU vs LRU**:
    -   LRU: Good for recency bias (bursty traffic).
    -   LFU: Good for stable popularity patterns.
    -   Issue with LFU: "Cache Pollution" - an item accessed 100 times long ago stays forever. Solution: **Aging** (decay frequency over time).
2.  **Complexity**: Naive LFU using a Min-Heap is $O(\log n)$. Optimal is $O(1)$ with Frequency Lists.
