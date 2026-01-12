---
---

## Summary
The **Null Object Pattern** uses an object with defined "do nothing" (neutral) behavior to replace `null` (or `nil` in Go) values. The intent is to eliminate the need for repetitive `if obj != nil` checks throughout the code, avoiding `nil` pointer panics and making the code cleaner.

## Detailed Explanation

In many object-oriented languages, calling a method on a null reference causes a crash. The Null Object pattern provides a concrete class that implements the same interface as the real object but its methods do nothing (or return default values).

### Problem
Without Null Object:
```go
if logger != nil {
    logger.Log("Some message")
}
```
If you forget the check, the program panics.

### Solution
With Null Object:
```go
logger.Log("Some message") // Safe, even if it's the NullLogger
```

### Usage Scenarios
*   Logging (Disabling logs without changing code).
*   Optional dependencies (Strategy pattern where "no strategy" is valid).
*   Testing (Stubbing out complex dependencies).

## Go Example

```go
package main

import "fmt"

// 1. The Interface
type Logger interface {
	Log(message string)
}

// 2. Real Implementation
type ConsoleLogger struct{}

func (c *ConsoleLogger) Log(message string) {
	fmt.Println(message)
}

// 3. Null Object Implementation (The "No-Op")
type NullLogger struct{}

func (n *NullLogger) Log(message string) {
	// Do nothing!
}

// 4. Client
type Service struct {
	logger Logger
}

func NewService(l Logger) *Service {
	// Safety check: if nil is passed, swap it for Null Object
	if l == nil {
		l = &NullLogger{}
	}
	return &Service{logger: l}
}

func (s *Service) DoWork() {
	// No need to check for nil here
	s.logger.Log("Work started") 
}

func main() {
	// Usage with real logger
	realSvc := NewService(&ConsoleLogger{})
	realSvc.DoWork()

	// Usage with nil (automatically becomes NullLogger)
	// This would panic without the pattern/check
	silentSvc := NewService(nil)
	silentSvc.DoWork() // Safe, prints nothing
}
```

## Interview Questions

### Q: What is the main disadvantage of the Null Object Pattern?
**A:** It can hide configuration errors. If a dependency is missing (passed as nil) and silently converted to a Null Object, the system might fail to perform critical actions (like logging errors or sending emails) without crashing or alerting the developer, making debugging difficult.

### Q: Does Go's `nil` interface behavior affect this pattern?
**A:** Yes. In Go, a nil pointer to a concrete type (e.g., `*ConsoleLogger(nil)`) can still satisfy an interface, and methods can be called on it. However, if the method implementation tries to access fields of the receiver, it will panic. The Null Object Pattern explicitly creates a specific struct (`NullLogger`) to handle "do nothing" logic safely.
