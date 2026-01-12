#Golang
---
---

## Summary

The `continue` statement skips the remaining code in the current iteration and jumps to the next iteration of the loop. Like `break`, Go supports labeled `continue` to affect outer loops in nested structures. This is useful for filtering, skipping invalid data, or avoiding deep nesting in loop bodies.

## Detailed Explanation

### **Basic Continue**

```go
package main

import "fmt"

func main() {
    // Skip even numbers
    for i := 0; i < 10; i++ {
        if i%2 == 0 {
            continue  // Skip to next iteration
        }
        fmt.Println(i)
    }
    // Output: 1, 3, 5, 7, 9
}
```

### **Continue Skips Remaining Code**

```go
func main() {
    for i := 0; i < 5; i++ {
        fmt.Println("Before continue:", i)
        
        if i == 2 {
            continue
        }
        
        fmt.Println("After continue:", i)  // Skipped when i == 2
    }
    
    // Output:
    // Before continue: 0
    // After continue: 0
    // Before continue: 1
    // After continue: 1
    // Before continue: 2
    // Before continue: 3    <- "After continue: 2" was skipped
    // After continue: 3
    // Before continue: 4
    // After continue: 4
}
```

### **Continue in Nested Loops**

```go
func main() {
    // Continue only affects innermost loop
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            if j == 1 {
                continue  // Skip j=1, continue inner loop
            }
            fmt.Printf("i=%d, j=%d\n", i, j)
        }
    }
    
    // Output:
    // i=0, j=0
    // i=0, j=2
    // i=1, j=0
    // i=1, j=2
    // i=2, j=0
    // i=2, j=2
}
```

### **Labeled Continue**

```go
func main() {
OuterLoop:
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            if j == 1 {
                continue OuterLoop  // Skip to next i
            }
            fmt.Printf("i=%d, j=%d\n", i, j)
        }
    }
    
    // Output:
    // i=0, j=0
    // i=1, j=0
    // i=2, j=0
}
```

### **Filtering Pattern**

```go
func main() {
    numbers := []int{-2, -1, 0, 1, 2, 3, 4, 5}
    
    // Process only positive numbers
    for _, n := range numbers {
        if n <= 0 {
            continue  // Skip non-positive
        }
        fmt.Printf("Processing %d\n", n)
    }
}
```

### **Skip Invalid Data**

```go
type User struct {
    Name  string
    Email string
}

func processUsers(users []User) {
    for _, user := range users {
        // Skip invalid entries
        if user.Name == "" {
            continue
        }
        if user.Email == "" {
            continue
        }
        
        // Process valid user
        sendEmail(user)
    }
}

// Alternative: Combine conditions
func processUsersAlt(users []User) {
    for _, user := range users {
        if user.Name == "" || user.Email == "" {
            continue
        }
        sendEmail(user)
    }
}
```

### **Continue vs Early Return**

```go
// Using continue (when processing a collection)
func validateAll(items []Item) []error {
    var errors []error
    
    for _, item := range items {
        if item.IsValid() {
            continue
        }
        errors = append(errors, item.ValidationError())
    }
    
    return errors
}

// Using helper with early return (cleaner for complex validation)
func validateAllBetter(items []Item) []error {
    var errors []error
    
    for _, item := range items {
        if err := validateItem(item); err != nil {
            errors = append(errors, err)
        }
    }
    
    return errors
}

func validateItem(item Item) error {
    if item.Name == "" {
        return errors.New("name required")
    }
    if item.Price < 0 {
        return errors.New("price must be positive")
    }
    return nil  // Valid
}
```

### **Avoid Deep Nesting with Continue**

```go
// ✗ Deep nesting
func processItemsBad(items []Item) {
    for _, item := range items {
        if item.Active {
            if item.InStock {
                if item.Price > 0 {
                    // actual processing here
                    ship(item)
                }
            }
        }
    }
}

// ✓ Flat with continue (guard clauses)
func processItemsGood(items []Item) {
    for _, item := range items {
        if !item.Active {
            continue
        }
        if !item.InStock {
            continue
        }
        if item.Price <= 0 {
            continue
        }
        
        // Actual processing at normal indentation
        ship(item)
    }
}
```

### **Continue with Range**

```go
func main() {
    // Continue works with all for-range forms
    
    // Slice
    nums := []int{1, 2, 3, 4, 5}
    for _, n := range nums {
        if n%2 == 0 {
            continue
        }
        fmt.Println(n)
    }
    
    // Map
    ages := map[string]int{"Alice": 25, "Bob": 17, "Carol": 30}
    for name, age := range ages {
        if age < 18 {
            continue
        }
        fmt.Printf("%s is an adult\n", name)
    }
    
    // String
    s := "Hello, 世界"
    for _, r := range s {
        if r == ',' || r == ' ' {
            continue
        }
        fmt.Printf("%c", r)
    }
    fmt.Println()  // "Hello世界"
}
```

### **Continue in Condition-Only Loops**

```go
func main() {
    i := 0
    for i < 10 {
        i++
        
        if i%3 != 0 {
            continue
        }
        
        fmt.Println(i)
    }
    // Output: 3, 6, 9
}
```

### **Continue in Infinite Loops**

```go
func worker(jobs <-chan Job) {
    for {
        job := <-jobs
        
        if job.Skip {
            continue
        }
        
        if err := process(job); err != nil {
            log.Println("Error:", err)
            continue
        }
        
        job.MarkComplete()
    }
}
```

### **Common Patterns**

```go
// Skip empty lines
func processLines(lines []string) {
    for _, line := range lines {
        line = strings.TrimSpace(line)
        if line == "" {
            continue
        }
        process(line)
    }
}

// Skip comments
func parseConfig(lines []string) {
    for _, line := range lines {
        if strings.HasPrefix(line, "#") {
            continue
        }
        parseLine(line)
    }
}

// Retry loop with continue
func fetchWithRetry(url string, maxRetries int) (*Response, error) {
    var lastErr error
    
    for attempt := 0; attempt < maxRetries; attempt++ {
        resp, err := fetch(url)
        if err != nil {
            lastErr = err
            time.Sleep(time.Second * time.Duration(attempt+1))
            continue  // Retry
        }
        return resp, nil
    }
    
    return nil, lastErr
}
```

## Interview Questions

**Q: What does `continue` do in Go?**
**A:** `continue` skips the remaining code in the current loop iteration and immediately starts the next iteration. The post statement (i++ in traditional for) still executes. It only affects the innermost loop unless a label is used.

**Q: What is the difference between `continue` and `break`?**
**A:** `break` exits the loop entirely; execution continues after the loop. `continue` skips only the current iteration; the loop continues with the next iteration. Both can use labels to affect outer loops in nested structures.

**Q: How do you use `continue` to skip to an outer loop's next iteration?**
**A:** Use a labeled continue: place a label before the outer loop (`OuterLoop:`), then use `continue OuterLoop` inside the inner loop. This skips to the next iteration of the outer loop, abandoning the current iteration of both loops.

**Q: When should you use `continue` vs restructuring with guard clauses?**
**A:** Use `continue` for filtering in loops to avoid deep nesting (guard clause pattern). Check for skip conditions early with `if condition { continue }`, then write the main logic at normal indentation. This is preferred over deeply nested if-else structures.
