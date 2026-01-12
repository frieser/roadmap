---
---

# Trie (Prefix Tree)

## Summary
A **Trie** (derived from re**trie**val) is a tree-based data structure used for storing strings efficiently. It excels at **prefix-based searches** (like autocomplete) where looking up a key takes time proportional to the key's length, independent of the total number of keys.

## Detailed Explanation

### Structure
*   **Root**: Represents an empty string.
*   **Edges**: Labeled with characters (e.g., 'a', 'b', 'c').
*   **Nodes**: Represent prefixes. A node might have a boolean flag `isEndOfWord` to mark that the path from root to this node forms a valid key.

### Key Advantage
To search for a string of length $L$ in a set of $N$ strings:
*   **Hash Table**: $O(L)$ to compute hash, but no prefix support.
*   **BST**: $O(L \cdot \log N)$ (each comparison takes $L$).
*   **Trie**: **$O(L)$**. The complexity is completely independent of $N$.

## Complexity
| Operation | Time | Space |
| :--- | :--- | :--- |
| **Insert** | $O(L)$ | $O(L \cdot \Sigma)$ (Worst case) |
| **Search** | $O(L)$ | $O(1)$ |
| **Prefix Search** | $O(L)$ | $O(1)$ |

*Where $L$ is key length and $\Sigma$ is alphabet size (e.g., 26).*

## Code Examples (Go)

```go
package main

type TrieNode struct {
    children map[rune]*TrieNode
    isEnd    bool
}

type Trie struct {
    root *TrieNode
}

func NewTrie() *Trie {
    return &Trie{root: &TrieNode{children: make(map[rune]*TrieNode)}}
}

func (t *Trie) Insert(word string) {
    curr := t.root
    for _, ch := range word {
        if _, exists := curr.children[ch]; !exists {
            curr.children[ch] = &TrieNode{children: make(map[rune]*TrieNode)}
        }
        curr = curr.children[ch]
    }
    curr.isEnd = true
}

func (t *Trie) Search(word string) bool {
    curr := t.root
    for _, ch := range word {
        if next, exists := curr.children[ch]; exists {
            curr = next
        } else {
            return false
        }
    }
    return curr.isEnd
}
```

## Go Application
*   **Autocomplete**: Used in search bars.
*   **IP Routing**: Longest Prefix Match in network routers (stored as bits 0/1).
*   **Spell Checkers**: Storing dictionaries.

## Interview Questions

**Q: What is the main disadvantage of a Trie?**
**A:** **Memory usage**. Since each node stores pointers to its children (e.g., array of 26 pointers or a map), a Trie can use significantly more memory than a simple array of strings, especially if the strings don't share many common prefixes.

**Q: How do you optimize Trie space?**
**A:** Use a **Compressed Trie** (Radix Tree). Instead of each edge being one character, an edge can represent a string of characters (e.g., "root" -> "to" -> "ast" for "toast").
