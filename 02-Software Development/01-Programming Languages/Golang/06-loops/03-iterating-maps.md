#Golang
---
---

## Summary

Iterating over maps in Go uses `for range` to access key-value pairs. Unlike slices, map iteration order is intentionally randomized by the Go runtime for security reasons—you cannot rely on any specific order. Understanding this behavior and knowing how to iterate in a sorted order when needed is crucial for predictable Go code.

## Detailed Explanation

### **Basic Map Iteration**

```go
package main

import "fmt"

func main() {
    ages := map[string]int{
        "Alice": 25,
        "Bob":   30,
        "Carol": 28,
    }
    
    // Key and value
    for name, age := range ages {
        fmt.Printf("%s is %d years old\n", name, age)
    }
    
    // Key only
    for name := range ages {
        fmt.Println("Name:", name)
    }
    
    // Value only (rare for maps)
    for _, age := range ages {
        fmt.Println("Age:", age)
    }
}
```

### **Iteration Order is Random**

```go
func main() {
    m := map[int]string{
        1: "one",
        2: "two",
        3: "three",
        4: "four",
        5: "five",
    }
    
    // Run this multiple times - order changes!
    fmt.Println("First run:")
    for k, v := range m {
        fmt.Printf("%d: %s\n", k, v)
    }
    
    fmt.Println("\nSecond run:")
    for k, v := range m {
        fmt.Printf("%d: %s\n", k, v)
    }
    
    // Different order each time!
}
```

### **Why Random Order?**

Go intentionally randomizes map iteration to:
1. **Prevent security vulnerabilities**: Predictable iteration could be exploited for hash collision attacks
2. **Discourage order dependency**: Forces developers to not rely on iteration order
3. **Allow optimization**: Runtime can optimize without preserving order

### **Sorted Iteration**

When you need deterministic order, sort the keys first:

```go
import (
    "fmt"
    "sort"
)

func main() {
    ages := map[string]int{
        "Charlie": 35,
        "Alice":   25,
        "Bob":     30,
    }
    
    // Get keys
    keys := make([]string, 0, len(ages))
    for k := range ages {
        keys = append(keys, k)
    }
    
    // Sort keys
    sort.Strings(keys)
    
    // Iterate in sorted order
    for _, k := range keys {
        fmt.Printf("%s: %d\n", k, ages[k])
    }
    // Output (always):
    // Alice: 25
    // Bob: 30
    // Charlie: 35
}
```

### **Generic Sorted Map Iteration (Go 1.21+)**

```go
import (
    "cmp"
    "fmt"
    "slices"
)

func sortedKeys[K cmp.Ordered, V any](m map[K]V) []K {
    keys := make([]K, 0, len(m))
    for k := range m {
        keys = append(keys, k)
    }
    slices.Sort(keys)
    return keys
}

func main() {
    m := map[int]string{3: "c", 1: "a", 2: "b"}
    
    for _, k := range sortedKeys(m) {
        fmt.Printf("%d: %s\n", k, m[k])
    }
}
```

### **Modifying Map During Iteration**

```go
func main() {
    m := map[string]int{"a": 1, "b": 2, "c": 3}
    
    // ✓ Safe: Modify existing values
    for k := range m {
        m[k] *= 2
    }
    fmt.Println(m)  // map[a:2 b:4 c:6]
    
    // ✓ Safe: Delete current or any key
    for k, v := range m {
        if v > 3 {
            delete(m, k)
        }
    }
    fmt.Println(m)  // map[a:2]
    
    // ⚠ Unpredictable: Adding keys during iteration
    // New keys may or may not be visited
    m = map[string]int{"a": 1}
    for k := range m {
        if k == "a" {
            m["b"] = 2  // May or may not be iterated
        }
        fmt.Println(k)
    }
}
```

### **Safe Deletion During Iteration**

```go
func main() {
    m := map[string]int{
        "keep1":   1,
        "delete1": 2,
        "keep2":   3,
        "delete2": 4,
    }
    
    // Method 1: Delete directly (safe in Go)
    for k := range m {
        if strings.HasPrefix(k, "delete") {
            delete(m, k)
        }
    }
    
    // Method 2: Collect keys first, then delete
    var toDelete []string
    for k, v := range m {
        if v > 2 {
            toDelete = append(toDelete, k)
        }
    }
    for _, k := range toDelete {
        delete(m, k)
    }
}
```

### **Nested Map Iteration**

```go
func main() {
    // Map of maps
    users := map[string]map[string]int{
        "alice": {"age": 25, "score": 90},
        "bob":   {"age": 30, "score": 85},
    }
    
    for name, attrs := range users {
        fmt.Printf("%s:\n", name)
        for attr, value := range attrs {
            fmt.Printf("  %s: %d\n", attr, value)
        }
    }
}
```

### **Map with Struct Values**

```go
type User struct {
    Name  string
    Email string
}

func main() {
    users := map[int]User{
        1: {Name: "Alice", Email: "alice@example.com"},
        2: {Name: "Bob", Email: "bob@example.com"},
    }
    
    // Cannot modify struct fields directly
    // for _, u := range users {
    //     u.Name = "Modified"  // Only modifies copy!
    // }
    
    // Use key to modify
    for id := range users {
        u := users[id]
        u.Name = "Modified " + u.Name
        users[id] = u
    }
    
    // Or use map of pointers
    userPtrs := map[int]*User{
        1: {Name: "Alice"},
        2: {Name: "Bob"},
    }
    
    for _, u := range userPtrs {
        u.Name = "Modified " + u.Name  // Works!
    }
}
```

### **Counting with Maps**

```go
func countWords(text string) map[string]int {
    words := strings.Fields(text)
    counts := make(map[string]int)
    
    for _, word := range words {
        counts[word]++
    }
    
    return counts
}

func main() {
    text := "the quick brown fox jumps over the lazy dog the"
    counts := countWords(text)
    
    for word, count := range counts {
        fmt.Printf("%s: %d\n", word, count)
    }
}
```

### **Grouping with Maps**

```go
type Person struct {
    Name string
    City string
}

func groupByCity(people []Person) map[string][]Person {
    groups := make(map[string][]Person)
    
    for _, p := range people {
        groups[p.City] = append(groups[p.City], p)
    }
    
    return groups
}

func main() {
    people := []Person{
        {"Alice", "NYC"},
        {"Bob", "LA"},
        {"Carol", "NYC"},
        {"Dave", "LA"},
    }
    
    groups := groupByCity(people)
    for city, residents := range groups {
        fmt.Printf("%s: %v\n", city, residents)
    }
}
```

### **Common Patterns**

```go
// Check if all values meet condition
func allPositive(m map[string]int) bool {
    for _, v := range m {
        if v <= 0 {
            return false
        }
    }
    return true
}

// Find key by value
func findKey(m map[string]int, target int) (string, bool) {
    for k, v := range m {
        if v == target {
            return k, true
        }
    }
    return "", false
}

// Invert map (swap keys and values)
func invert(m map[string]int) map[int]string {
    result := make(map[int]string, len(m))
    for k, v := range m {
        result[v] = k
    }
    return result
}
```

## Interview Questions

**Q: Why is map iteration order random in Go?**
**A:** Go intentionally randomizes map iteration order for security (prevents hash collision attacks), to discourage code that depends on order, and to allow runtime optimizations. Never rely on map iteration order—if you need deterministic order, sort the keys first.

**Q: Is it safe to delete map entries while iterating?**
**A:** Yes, deleting entries during iteration is safe in Go. The current key and any other existing keys can be deleted without causing issues. However, adding new keys during iteration may or may not include them in the current iteration—behavior is undefined.

**Q: How do you iterate over a map in sorted order?**
**A:** Extract the keys into a slice, sort the slice using `sort.Strings()` or `slices.Sort()`, then iterate over the sorted keys and access values via `map[key]`. There's no built-in sorted iteration for maps.

**Q: Can you modify struct values in a map during iteration?**
**A:** You cannot modify struct fields directly through the range variable (it's a copy). Either use the key to get/modify/reassign the struct, or use a map of pointers (`map[K]*V`) which allows direct field modification through the pointer.
