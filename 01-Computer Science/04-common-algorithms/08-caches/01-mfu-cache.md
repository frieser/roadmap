---
---

# MFU Cache (Most Frequently Used)

## Abstract
**MFU (Most Frequently Used)** is a cache eviction policy that discards the items used **most often**. It assumes that the item with the smallest frequency is the one that has just been brought in and needs to be kept, while "super-popular" items are done being used. This is a very niche and rarely used policy.

## Development

### Concept
It is the inverse of LFU.
-   **Evict**: The item with the **highest** frequency count.
-   **Logic**: "If I read this block 100 times, I'm probably done with it."

### Use Cases (Rare)
-   **Cyclic Scanning**: If a system repeatedly loops through a large dataset that doesn't fit in cache, LRU fails (flushing everything). MRU/MFU might perform better by keeping the newest data which won't be needed for a full cycle.

## Code Examples (Go)

Implementation is similar to LFU but tracking `MaxFreq` instead of `MinFreq`.

```go
// Conceptual logic
func (c *MFUCache) Evict() {
    // Find bucket with MaxFreq
    list := c.lists[c.maxFreq]
    
    // Remove element (e.g., Head or Tail)
    elem := list.Front()
    list.Remove(elem)
    
    // Cleanup maps
    // Decrease maxFreq if list empty
}
```

## Go Application
-   **None standard**: There are virtually no mainstream production systems using MFU by default. It is mostly an academic counter-example or used in highly specific circular buffer scenarios.

## Interview Preparation
1.  **Why is MFU rare?**: In almost all real-world workloads (Zipfian distribution), past frequency is a positive indicator of future usage. MFU assumes the opposite.
2.  **Contrast**: Compare with LRU/LFU to show depth of understanding.
