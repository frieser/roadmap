# Table Driven Tests

## Summary
Table-Driven Testing is the idiom in Go for writing clearer and more maintainable tests. Instead of writing separate test functions or repeated assertions for different scenarios, you define a "table" (slice of structs) containing inputs and expected outputs. You then iterate over this table, running the test logic for each entry, often using `t.Run()` to create subtests.

## Detailed Explanation

### Why Table-Driven?
*   **DRY (Don't Repeat Yourself)**: The test logic (setup, execution, assertion) is written once.
*   **Extensibility**: Adding a new test case is as simple as adding a line to the struct slice.
*   **Clarity**: It separates the *test data* from the *test logic*, making it easy to see all edge cases covered.

### Structure
1.  Define a struct that holds your input arguments and expected result.
2.  Create a slice of these structs (the "table").
3.  Iterate over the slice using `range`.
4.  Inside the loop, run the test logic.
5.  (Optional but recommended) Use `t.Run(name, func(t *testing.T))` to execute each case as a distinct subtest.

### Code Example

Let's test a function `Split` that splits a string by a separator.

```go
package strings_test

import (
	"reflect"
	"strings"
	"testing"
)

func TestSplit(t *testing.T) {
	// 1. Define the table
	tests := []struct {
		name     string // Description of the test case
		input    string // Input argument 1
		sep      string // Input argument 2
		expected []string
	}{
		{
			name:     "basic comma",
			input:    "a,b,c",
			sep:      ",",
			expected: []string{"a", "b", "c"},
		},
		{
			name:     "no separator",
			input:    "abc",
			sep:      ",",
			expected: []string{"abc"},
		},
		{
			name:     "trailing separator",
			input:    "a,b,",
			sep:      ",",
			expected: []string{"a", "b", ""},
		},
	}

	// 2. Iterate
	for _, tt := range tests {
		// 3. Subtest
		t.Run(tt.name, func(t *testing.T) {
			got := strings.Split(tt.input, tt.sep)
			
			// DeepEqual is useful for comparing slices
			if !reflect.DeepEqual(got, tt.expected) {
				t.Errorf("Split(%q, %q) = %v; want %v", tt.input, tt.sep, got, tt.expected)
			}
		})
	}
}
```

### Subtests (`t.Run`)
Using `t.Run` allows you to:
*   Run specific cases from the CLI: `go test -run TestSplit/trailing_separator`
*   Parallelize subtests using `t.Parallel()`.
*   Get clear output; if one case fails, it is identified by its name.

## Interview Questions

**Q: What is the main advantage of table-driven tests?**
**A:** It decouples the test data from the test logic. This makes it trivial to add new test cases (especially edge cases) without duplicating code, ensuring high test coverage with minimal effort.

**Q: How do you run a specific subtest in a table-driven test?**
**A:** You use the `-run` flag with a slash pattern. If the main test is `TestFoo` and the subtest name is "Basic Case", you can run `go test -run TestFoo/Basic_Case`. Spaces in names are often replaced by underscores in the matcher.

**Q: How would you handle a test case that is expected to panic in a table-driven test?**
**A:** You can add a field to your struct (e.g., `expectPanic bool`). Inside the loop, defer a recovery function that checks if a panic occurred. If `expectPanic` is true, assert that `recover()` is not nil; otherwise, assert that it is nil.
