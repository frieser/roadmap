---
---

# LRU Cache (Least Recently Used)

## Abstract
**LRU (Least Recently Used)** is a cache eviction policy that discards the **least recently used** items first. It assumes that items used recently are likely to be used again soon (temporal locality).

## Development

### Core Concept
To achieve $O(1)$ for both `Get` and `Put`, we combine two data structures:
1.  **Doubly Linked List**: Maintains order of usage.
    -   Front: Most Recently Used (MRU).
    -   Back: Least Recently Used (LRU).
2.  **Hash Map**: Maps `Key` $\rightarrow$ `ListNode*`. Allows $O(1)$ access to list nodes.

### Operations
-   **Get(key)**: Check map. If exists, move node to Front (MRU). Return value.
-   **Put(key, value)**:
    -   If key exists: Update value, move to Front.
    -   If new: Create node, add to Front, add to map.
    -   If capacity exceeded: Remove node from Back (LRU), delete from map.

## Code Examples (Go)

Using `container/list` standard library.

```go
package main

import (
    "container/list"
    "fmt"
)

type LRUCache struct {
    capacity int
    cache    map[int]*list.Element
    list     *list.List
}

type Pair struct {
    Key   int
    Value int
}

func Constructor(capacity int) LRUCache {
    return LRUCache{
        capacity: capacity,
        cache:    make(map[int]*list.Element),
        list:     list.New(),
    }
}

func (this *LRUCache) Get(key int) int {
    if elem, ok := this.cache[key]; ok {
        this.list.MoveToFront(elem)
        return elem.Value.(Pair).Value
    }
    return -1
}

func (this *LRUCache) Put(key int, value int) {
    if elem, ok := this.cache[key]; ok {
        // Update existing
        this.list.MoveToFront(elem)
        elem.Value = Pair{Key: key, Value: value}
        return
    }

    if this.list.Len() >= this.capacity {
        // Evict LRU (Back)
        oldest := this.list.Back()
        if oldest != nil {
            this.list.Remove(oldest)
            delete(this.cache, oldest.Value.(Pair).Key)
        }
    }

    // Add new
    newElem := this.list.PushFront(Pair{Key: key, Value: value})
    this.cache[key] = newElem
}
```

## Go Application
-   **Database Caches**: Redis LRU eviction.
-   **Web Servers**: Caching user sessions or expensive API responses.
-   **`groupcache`**: Google's caching library (uses LRU).

## Interview Preparation
1.  **Why Doubly Linked List?**: To remove a node from the middle/tail in $O(1)$ (given a pointer) and move it to head.
2.  **Thread Safety**: This implementation is NOT thread-safe. Use `sync.RWMutex` (`Lock` on Put, `Lock` on Get because Get mutates order).
