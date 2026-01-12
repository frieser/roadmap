#Golang
---
---

## Summary

The `for range` construct provides a clean way to iterate over arrays, slices, maps, strings, and channels. It automatically handles indexing and bounds checking, returning both the index (or key) and the value for each element. Understanding `for range` behavior with different types and the loop variable gotcha is essential for writing correct Go code.

## Detailed Explanation

### **Basic Range Syntax**

```go
for index, value := range collection {
    // use index and value
}

// Or just the index/key
for index := range collection {
    // use index only
}

// Or just the value (discard index with _)
for _, value := range collection {
    // use value only
}
```

### **Range Over Slices and Arrays**

```go
package main

import "fmt"

func main() {
    nums := []int{10, 20, 30, 40, 50}
    
    // Index and value
    for i, v := range nums {
        fmt.Printf("Index %d: %d\n", i, v)
    }
    
    // Index only
    for i := range nums {
        fmt.Printf("Index: %d\n", i)
    }
    
    // Value only
    for _, v := range nums {
        fmt.Printf("Value: %d\n", v)
    }
    
    // Arrays work the same way
    arr := [3]string{"a", "b", "c"}
    for i, v := range arr {
        fmt.Printf("%d: %s\n", i, v)
    }
}
```

### **Range Over Channels**

```go
func main() {
    ch := make(chan int, 3)
    ch <- 1
    ch <- 2
    ch <- 3
    close(ch)  // Must close for range to terminate
    
    // Range reads until channel is closed
    for v := range ch {
        fmt.Println(v)
    }
    // Output: 1, 2, 3
}

// Common pattern: worker reading from channel
func worker(jobs <-chan Job) {
    for job := range jobs {
        process(job)
    }
    // Loop exits when channel is closed
}
```

### **Range Returns Copies**

```go
func main() {
    nums := []int{1, 2, 3}
    
    // ✗ This doesn't modify the slice!
    for _, v := range nums {
        v *= 2  // v is a copy, original unchanged
    }
    fmt.Println(nums)  // [1 2 3] (not modified)
    
    // ✓ Use index to modify
    for i := range nums {
        nums[i] *= 2
    }
    fmt.Println(nums)  // [2 4 6]
    
    // With structs (copies the struct)
    type Point struct{ X, Y int }
    points := []Point{{1, 2}, {3, 4}}
    
    for _, p := range points {
        p.X = 100  // Modifies copy only
    }
    fmt.Println(points)  // [{1 2} {3 4}] (unchanged)
    
    // ✓ Use index for structs too
    for i := range points {
        points[i].X = 100
    }
    fmt.Println(points)  // [{100 2} {100 4}]
}
```

### **Range Over Pointers**

```go
type User struct {
    Name string
    Age  int
}

func main() {
    // Slice of pointers allows modification through range
    users := []*User{
        {Name: "Alice", Age: 25},
        {Name: "Bob", Age: 30},
    }
    
    for _, u := range users {
        u.Age++  // Works! u is a pointer
    }
    
    for _, u := range users {
        fmt.Printf("%s: %d\n", u.Name, u.Age)
    }
    // Alice: 26
    // Bob: 31
}
```

### **The Loop Variable Gotcha (Pre-Go 1.22)**

```go
// ✗ Classic bug (Go < 1.22)
func main() {
    nums := []int{1, 2, 3}
    var funcs []func()
    
    for _, v := range nums {
        funcs = append(funcs, func() {
            fmt.Println(v)  // Captures v, not its value!
        })
    }
    
    for _, f := range funcs {
        f()
    }
    // Output: 3, 3, 3 (all same value!)
}

// ✓ Fix 1: Copy the variable (works in all Go versions)
for _, v := range nums {
    v := v  // Shadow with local copy
    funcs = append(funcs, func() {
        fmt.Println(v)
    })
}

// ✓ Fix 2: Pass as parameter
for _, v := range nums {
    funcs = append(funcs, func(x int) func() {
        return func() { fmt.Println(x) }
    }(v))
}

// ✓ Fix 3: Use Go 1.22+ (fixed by default)
// In Go 1.22+, loop variables are per-iteration, not per-loop
```

### **Range Evaluation**

```go
func main() {
    // The range expression is evaluated once
    nums := []int{1, 2, 3, 4, 5}
    
    for i, v := range nums {
        if i == 0 {
            nums = append(nums, 100)  // Modify slice
        }
        fmt.Println(v)
    }
    // Output: 1, 2, 3, 4, 5 (100 not printed)
    // The range was captured at loop start
    
    fmt.Println(nums)  // [1 2 3 4 5 100] (but 100 was added)
}
```

### **Range Over Integer (Go 1.22+)**

```go
// Go 1.22 introduced ranging over integers
for i := range 5 {
    fmt.Println(i)
}
// Output: 0, 1, 2, 3, 4

// Useful for simple counting
for range 3 {
    fmt.Println("Hello")
}
// Output: Hello (3 times)
```

### **Empty Range**

```go
func main() {
    // Ranging over nil or empty collections is safe
    var nilSlice []int
    for _, v := range nilSlice {
        fmt.Println(v)  // Never executes
    }
    
    emptySlice := []int{}
    for _, v := range emptySlice {
        fmt.Println(v)  // Never executes
    }
    
    var nilMap map[string]int
    for k, v := range nilMap {
        fmt.Println(k, v)  // Never executes
    }
}
```

### **Range Patterns**

#### Collecting Results

```go
nums := []int{1, 2, 3, 4, 5}
doubled := make([]int, len(nums))

for i, v := range nums {
    doubled[i] = v * 2
}
```

#### Filtering

```go
nums := []int{1, 2, 3, 4, 5, 6}
var evens []int

for _, v := range nums {
    if v%2 == 0 {
        evens = append(evens, v)
    }
}
```

#### Finding

```go
func find(nums []int, target int) (int, bool) {
    for i, v := range nums {
        if v == target {
            return i, true
        }
    }
    return -1, false
}
```

#### Summing

```go
nums := []int{1, 2, 3, 4, 5}
sum := 0
for _, v := range nums {
    sum += v
}
```

### **When to Use Range vs Traditional For**

| Use `for range` | Use traditional `for` |
| --- | --- |
| Iterating all elements | Need custom step/increment |
| Don't need to modify iteration | Need to skip or repeat iterations |
| Maps and channels | Need to access adjacent elements |
| Cleaner, more idiomatic | Need fine-grained control |

## Interview Questions

**Q: What does `for range` return for different collection types?**
**A:** For slices/arrays: index (int) and value (copy). For maps: key and value (copies). For strings: byte index (int) and rune (int32). For channels: just the value (no index). Use blank identifier `_` to discard unwanted values.

**Q: Why doesn't modifying the range value affect the original collection?**
**A:** Range returns a copy of each element, not a reference. Modifying the value variable only changes the local copy. To modify the original, use the index: `slice[i] = newValue` or use a slice of pointers.

**Q: What is the loop variable gotcha in Go, and how was it fixed?**
**A:** Before Go 1.22, loop variables were reused across iterations, causing closures to capture the same variable (ending up with the last value). The fix was to create a local copy (`v := v`) or pass as a parameter. Go 1.22+ creates new variables per iteration, fixing this by default.

**Q: Is it safe to range over a nil slice or map?**
**A:** Yes, ranging over nil collections is safe—the loop body simply never executes. This is useful because you don't need nil checks before iterating.
