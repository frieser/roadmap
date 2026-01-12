#Golang
---
---

## Summary

Go has only one looping construct: the `for` loop. However, it's versatile enough to replace `while` and `do-while` loops from other languages. The `for` loop comes in three forms: the traditional three-component loop, the condition-only loop (like `while`), and the infinite loop. Understanding these variations is essential for writing idiomatic Go code.

## Detailed Explanation

### **The Three Forms of For**

```go
// Form 1: Traditional (init; condition; post)
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// Form 2: Condition only (like while)
for condition {
    // body
}

// Form 3: Infinite loop
for {
    // runs forever until break
}
```

### **Traditional For Loop**

```go
package main

import "fmt"

func main() {
    // Basic loop: init; condition; post
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
    // Output: 0, 1, 2, 3, 4
    
    // Counting down
    for i := 5; i > 0; i-- {
        fmt.Println(i)
    }
    // Output: 5, 4, 3, 2, 1
    
    // Custom step
    for i := 0; i < 10; i += 2 {
        fmt.Println(i)
    }
    // Output: 0, 2, 4, 6, 8
    
    // Multiple variables
    for i, j := 0, 10; i < j; i, j = i+1, j-1 {
        fmt.Printf("i=%d, j=%d\n", i, j)
    }
}
```

### **Components Are Optional**

```go
func main() {
    // Skip init (variable declared outside)
    i := 0
    for ; i < 5; i++ {
        fmt.Println(i)
    }
    
    // Skip post (increment inside loop)
    for i := 0; i < 5; {
        fmt.Println(i)
        i++
    }
    
    // Skip both (condition-only)
    i = 0
    for i < 5 {
        fmt.Println(i)
        i++
    }
}
```

### **Condition-Only Loop (While)**

Go doesn't have a `while` keyword, but `for` with only a condition serves the same purpose:

```go
func main() {
    // "While" loop
    count := 0
    for count < 5 {
        fmt.Println(count)
        count++
    }
    
    // Reading until condition
    reader := bufio.NewReader(os.Stdin)
    for {
        line, err := reader.ReadString('\n')
        if err != nil {
            break
        }
        fmt.Print(line)
    }
    
    // Processing a queue
    queue := []int{1, 2, 3, 4, 5}
    for len(queue) > 0 {
        item := queue[0]
        queue = queue[1:]
        fmt.Println("Processing:", item)
    }
}
```

### **Infinite Loop**

```go
func main() {
    // Infinite loop (must break or return)
    for {
        fmt.Println("Running...")
        // Use break, return, or os.Exit to stop
    }
}

// Common patterns with infinite loops
func server() {
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Println(err)
            continue
        }
        go handleConnection(conn)
    }
}

func worker(jobs <-chan Job) {
    for {
        select {
        case job := <-jobs:
            process(job)
        case <-quit:
            return
        }
    }
}
```

### **Loop Variable Scope**

```go
func main() {
    // i is scoped to the loop
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
    // fmt.Println(i)  // Error: i is undefined
    
    // Declare outside for wider scope
    var j int
    for j = 0; j < 5; j++ {
        fmt.Println(j)
    }
    fmt.Println("Final j:", j)  // 5
}
```

### **Nested Loops**

```go
func main() {
    // Multiplication table
    for i := 1; i <= 5; i++ {
        for j := 1; j <= 5; j++ {
            fmt.Printf("%d×%d=%2d  ", i, j, i*j)
        }
        fmt.Println()
    }
    
    // Matrix traversal
    matrix := [][]int{
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
    }
    
    for row := 0; row < len(matrix); row++ {
        for col := 0; col < len(matrix[row]); col++ {
            fmt.Printf("%d ", matrix[row][col])
        }
        fmt.Println()
    }
}
```

### **Common Patterns**

#### Iterating with Index

```go
nums := []int{10, 20, 30, 40, 50}

for i := 0; i < len(nums); i++ {
    fmt.Printf("Index %d: %d\n", i, nums[i])
}
```

#### Reverse Iteration

```go
nums := []int{1, 2, 3, 4, 5}

for i := len(nums) - 1; i >= 0; i-- {
    fmt.Println(nums[i])
}
// Output: 5, 4, 3, 2, 1
```

#### Sliding Window

```go
data := []int{1, 2, 3, 4, 5, 6, 7}
windowSize := 3

for i := 0; i <= len(data)-windowSize; i++ {
    window := data[i : i+windowSize]
    fmt.Println(window)
}
// Output: [1 2 3], [2 3 4], [3 4 5], [4 5 6], [5 6 7]
```

#### Two Pointers

```go
func isPalindrome(s string) bool {
    runes := []rune(s)
    for left, right := 0, len(runes)-1; left < right; left, right = left+1, right-1 {
        if runes[left] != runes[right] {
            return false
        }
    }
    return true
}
```

### **Range Over Integers (Go 1.22+)**

```go
// Go 1.22+ allows ranging over integers
for i := range 5 {
    fmt.Println(i)
}
// Output: 0, 1, 2, 3, 4

// Equivalent to:
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

### **Loop Performance Tips**

```go
// ✗ Avoid: Calling len() on each iteration
for i := 0; i < len(slice); i++ {  // len() called each time
    // ...
}

// ✓ Better: Cache the length
n := len(slice)
for i := 0; i < n; i++ {
    // ...
}

// Note: For slices, the compiler usually optimizes this,
// but caching is still good practice for complex expressions

// ✗ Avoid: Growing slice in loop
var result []int
for i := 0; i < 1000; i++ {
    result = append(result, i)  // Multiple allocations
}

// ✓ Better: Pre-allocate capacity
result := make([]int, 0, 1000)
for i := 0; i < 1000; i++ {
    result = append(result, i)  // No reallocations
}
```

### **Loop Control Flow**

```go
func main() {
    // break: exit the loop immediately
    for i := 0; i < 10; i++ {
        if i == 5 {
            break
        }
        fmt.Println(i)
    }
    // Output: 0, 1, 2, 3, 4
    
    // continue: skip to next iteration
    for i := 0; i < 10; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Println(i)
    }
    // Output: 1, 3, 5, 7, 9
    
    // return: exit the function
    // goto: jump to label (see 07-goto.md)
}
```

## Interview Questions

**Q: How many loop constructs does Go have?**
**A:** Go has only one: the `for` loop. However, it can be used in three forms: traditional three-component (`for i := 0; i < n; i++`), condition-only like `while` (`for condition`), and infinite (`for { }`). There is no `while` or `do-while` in Go.

**Q: How do you create a "while" loop in Go?**
**A:** Use `for` with only a condition: `for condition { body }`. For example, `for x < 10 { x++ }` is equivalent to `while (x < 10)` in other languages.

**Q: What is the scope of a loop variable declared in the init statement?**
**A:** Variables declared in the init statement (`for i := 0; ...`) are scoped to the loop body and the for statement itself. They're not accessible after the loop ends. To access the final value outside the loop, declare the variable before the for statement.

**Q: How do you iterate backwards over a slice?**
**A:** Use a decrementing loop: `for i := len(slice) - 1; i >= 0; i-- { }`. Start at the last index and decrement until reaching 0. There's no built-in reverse range in Go.
