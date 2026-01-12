---
---

## Summary
Test-Driven Development (TDD) is a software development process where you write tests before writing the actual code. It follows a strict "Red-Green-Refactor" cycle. In Go, TDD is highly encouraged by the built-in `testing` package and conventions like table-driven tests.

## Detailed Explanation

The TDD cycle consists of three repeatable steps:
1.  **Red**: Write a small test for a new function or feature that doesn't exist yet. Run it and watch it fail.
2.  **Green**: Write the minimum amount of code required to make the test pass.
3.  **Refactor**: Clean up the code while ensuring the tests continue to pass. Improve names, remove duplication, and optimize.

### Benefits of TDD
*   **Design Guidance**: Writing tests first forces you to think about the API and usability of your code before implementation.
*   **Confidence**: Having a comprehensive test suite allows you to refactor and add features without fear of breaking existing functionality.
*   **Documentation**: Tests serve as executable documentation that shows how the code is intended to be used.

## Go-specific Context and Examples

Go has a very strong testing culture. The standard library provides everything needed for TDD without requiring third-party frameworks.

### Table-Driven Tests
This is the idiomatic way to write tests in Go. It allows you to test many cases using a single test function.

```go
package calculator

import "testing"

// The function we are developing
func Add(a, b int) int {
	return a + b
}

// TestAdd uses the table-driven pattern
func TestAdd(t *testing.T) {
	tests := []struct {
		name     string
		a, b     int
		expected int
	}{
		{"positive numbers", 2, 3, 5},
		{"negative numbers", -1, -1, -2},
		{"zero", 0, 5, 5},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			result := Add(tt.a, tt.b)
			if result != tt.expected {
				t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, result, tt.expected)
			}
		})
	}
}
```

### Mocking in Go
Go uses interfaces for mocking. If a function accepts an interface, you can pass a mock implementation in your tests.

```go
package service

import "testing"

type DataFetcher interface {
	Fetch() string
}

type MockFetcher struct{}
func (m *MockFetcher) Fetch() string { return "mocked data" }

func Process(f DataFetcher) string {
	return "Processed: " + f.Fetch()
}

func TestProcess(t *testing.T) {
	mock := &MockFetcher{}
	result := Process(mock)
	expected := "Processed: mocked data"
	if result != expected {
		t.Errorf("got %s, want %s", result, expected)
	}
}
```

## Interview Questions

**Q: What is the Red-Green-Refactor cycle in TDD?**
**A:** Red: Write a failing test. Green: Write enough code to pass the test. Refactor: Clean the code while keeping it passing.

**Q: What are table-driven tests in Go and why are they used?**
**A:** Table-driven tests involve defining a slice of structs (the "table") containing input values and expected outputs, then iterating over them. They are used to reduce code duplication and make it easy to add new test cases.

**Q: How does TDD improve the design of your code?**
**A:** Since you write the test first, you are forced to use your own API as a client. This often leads to better-defined interfaces, smaller functions, and less coupling, as highly coupled code is difficult to test.
