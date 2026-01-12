#Golang
---
---

## Summary

The `break` statement immediately terminates the innermost `for`, `switch`, or `select` statement. For nested loops, Go provides labeled breaks that can exit outer loops directly, avoiding the need for boolean flags or complex control flow. Understanding break behavior with different constructs is essential for writing clean loop control code.

## Detailed Explanation

### **Basic Break**

```go
package main

import "fmt"

func main() {
    // Break exits the loop immediately
    for i := 0; i < 10; i++ {
        if i == 5 {
            break
        }
        fmt.Println(i)
    }
    // Output: 0, 1, 2, 3, 4
    
    fmt.Println("Loop ended")
}
```

### **Break in Nested Loops**

```go
func main() {
    // Break only exits the innermost loop
    for i := 0; i < 3; i++ {
        fmt.Printf("Outer: %d\n", i)
        for j := 0; j < 3; j++ {
            if j == 1 {
                break  // Only exits inner loop
            }
            fmt.Printf("  Inner: %d\n", j)
        }
    }
    // Output:
    // Outer: 0
    //   Inner: 0
    // Outer: 1
    //   Inner: 0
    // Outer: 2
    //   Inner: 0
}
```

### **Labeled Break**

Labels allow breaking out of outer loops:

```go
func main() {
OuterLoop:  // Label (convention: CamelCase)
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            fmt.Printf("i=%d, j=%d\n", i, j)
            if i == 1 && j == 1 {
                break OuterLoop  // Exits outer loop
            }
        }
    }
    fmt.Println("Done")
    
    // Output:
    // i=0, j=0
    // i=0, j=1
    // i=0, j=2
    // i=1, j=0
    // i=1, j=1
    // Done
}
```

### **Break in Switch**

```go
func main() {
    // Break in switch (rarely needed - cases don't fall through)
    x := 2
    switch x {
    case 1:
        fmt.Println("one")
    case 2:
        fmt.Println("two")
        break  // Explicit but redundant
        fmt.Println("never printed")
    case 3:
        fmt.Println("three")
    }
    
    // Break in switch inside loop
    for i := 0; i < 5; i++ {
        switch i {
        case 2:
            break  // Breaks switch, NOT the loop!
        }
        fmt.Println(i)
    }
    // Output: 0, 1, 2, 3, 4 (loop continues!)
}
```

### **Break in Switch Inside Loop (Labeled)**

```go
func main() {
    // To break the loop from inside switch, use a label
Loop:
    for i := 0; i < 5; i++ {
        switch i {
        case 2:
            break Loop  // Now breaks the outer loop
        }
        fmt.Println(i)
    }
    // Output: 0, 1
}
```

### **Break in Select**

```go
func main() {
    ch := make(chan int, 3)
    ch <- 1
    ch <- 2
    ch <- 3
    
    // Break exits the select, not any enclosing loop
Loop:
    for {
        select {
        case v, ok := <-ch:
            if !ok {
                break Loop  // Channel closed, exit loop
            }
            fmt.Println(v)
            if v == 2 {
                break  // Just exits select, loop continues
            }
        default:
            break Loop  // No more data, exit loop
        }
    }
}
```

### **Search Pattern with Break**

```go
func findIndex(slice []int, target int) int {
    index := -1
    for i, v := range slice {
        if v == target {
            index = i
            break  // Found it, stop searching
        }
    }
    return index
}

// Better: Return directly
func findIndexBetter(slice []int, target int) int {
    for i, v := range slice {
        if v == target {
            return i
        }
    }
    return -1
}
```

### **Early Termination Patterns**

```go
// Pattern 1: Found what we're looking for
func containsNegative(nums []int) bool {
    for _, n := range nums {
        if n < 0 {
            return true  // Early exit
        }
    }
    return false
}

// Pattern 2: Error encountered
func processItems(items []Item) error {
    for _, item := range items {
        if err := validate(item); err != nil {
            return err  // Stop on first error
        }
        process(item)
    }
    return nil
}

// Pattern 3: Resource limit
func readUpTo(r io.Reader, maxBytes int) ([]byte, error) {
    var result []byte
    buf := make([]byte, 1024)
    
    for len(result) < maxBytes {
        n, err := r.Read(buf)
        if err != nil {
            break  // EOF or error
        }
        result = append(result, buf[:n]...)
    }
    
    return result, nil
}
```

### **Breaking from Infinite Loops**

```go
func main() {
    count := 0
    
    for {
        count++
        fmt.Println(count)
        
        if count >= 5 {
            break  // Exit infinite loop
        }
    }
    
    // Common pattern: server with shutdown
    done := make(chan struct{})
    
    go func() {
        for {
            select {
            case <-done:
                break  // This only breaks select!
            default:
                // do work
            }
        }
    }()
}

// Correct server pattern
func server(quit <-chan struct{}) {
ServerLoop:
    for {
        select {
        case <-quit:
            break ServerLoop  // Properly exits the loop
        default:
            // handle connections
        }
    }
    fmt.Println("Server stopped")
}
```

### **Anti-Patterns**

```go
// ✗ Avoid: Using flag instead of labeled break
func bad() {
    found := false
    for i := 0; i < 10 && !found; i++ {
        for j := 0; j < 10 && !found; j++ {
            if condition(i, j) {
                found = true  // Messy!
            }
        }
    }
}

// ✓ Better: Use labeled break
func good() {
Search:
    for i := 0; i < 10; i++ {
        for j := 0; j < 10; j++ {
            if condition(i, j) {
                break Search  // Clean!
            }
        }
    }
}

// ✓ Best: Extract to function with return
func best() bool {
    for i := 0; i < 10; i++ {
        for j := 0; j < 10; j++ {
            if condition(i, j) {
                return true  // Cleanest!
            }
        }
    }
    return false
}
```

## Interview Questions

**Q: What does `break` do in Go?**
**A:** `break` terminates the innermost `for`, `switch`, or `select` statement. It jumps to the code immediately following the terminated statement. For nested constructs, use labeled break to exit an outer loop or to break a loop from inside a switch/select.

**Q: How do you break out of nested loops in Go?**
**A:** Use a labeled break: define a label before the outer loop (`OuterLoop:`), then use `break OuterLoop` to exit directly. Alternatively, extract the nested loops into a function and use `return`.

**Q: What happens when you use `break` inside a switch that's inside a loop?**
**A:** `break` exits the switch, NOT the loop. The loop continues with the next iteration. To break the loop from inside the switch, use a labeled break targeting the loop.

**Q: When should you use labeled break vs extracting to a function?**
**A:** Use labeled break for simple nested loop escapes where the logic is straightforward. Extract to a function when the search logic is reusable, complex, or would benefit from a clear return value. Function extraction is generally cleaner and more testable.
