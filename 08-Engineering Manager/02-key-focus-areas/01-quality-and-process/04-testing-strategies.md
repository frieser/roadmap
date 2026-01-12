---
---

## Summary
**Testing Strategies** define *how* an engineering team ensures quality. For an EM, this is an investment decision: "How much effort should we spend on testing to buy down the risk of bugs?" A healthy strategy relies on the **Testing Pyramid**, prioritizing fast, reliable unit tests over slow, brittle end-to-end (E2E) tests. The goal is confidence at speed.

## Detailed Explanation

### 1. The Testing Pyramid (Mike Cohn)
*   **Unit Tests (Base)**: 70% of tests. Fast, isolated, test individual functions. (Cost: Low, Speed: High).
*   **Integration Tests (Middle)**: 20% of tests. Verify that modules work together (e.g., API talks to DB).
*   **E2E / UI Tests (Top)**: 10% of tests. Simulate a real user journey. (Cost: High, Speed: Low, Flakiness: High).

### 2. Shift Left
*   Testing should happen as early as possible in the SDLC. Finding a bug in design costs $1; finding it in Prod costs $100.
*   **TDD (Test Driven Development)**: Writing the test *before* the code.

### 3. Types of Testing
*   **Regression Testing**: Ensuring new code doesn't break old features.
*   **Smoke/Sanity Testing**: Quick checks to see if the system builds and launches.
*   **Performance/Load Testing**: Verifying the system handles scale (e.g., k6, JMeter).
*   **Contract Testing**: Verifying that APIs meet their specs (Consumer-Driven Contracts).

### 4. Metrics
*   **Code Coverage**: Useful to find gaps, but 100% coverage doesn't mean bug-free code.
*   **Flakiness Rate**: The percentage of tests that fail randomly. Flaky tests destroy trust and must be ruthlessly eliminated.

## Go Code Example: Table-Driven Tests
Go has a unique, idiomatic approach to testing: **Table-Driven Tests**. Instead of writing separate functions for every case (`TestAddPositive`, `TestAddNegative`), Go developers define a slice of test cases and iterate over them. This is efficient, readable, and easy to extend.

```go
package math_test

import (
	"testing"
)

// The function we are testing
func Max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

// TestMax demonstrates idiomatic Go table-driven testing
func TestMax(t *testing.T) {
	// 1. Define the table
	tests := []struct {
		name string
		a    int
		b    int
		want int
	}{
		{"A is larger", 10, 5, 10},
		{"B is larger", 2, 8, 8},
		{"Both equal", 5, 5, 5},
		{"Negatives", -1, -5, -1},
	}

	// 2. Iterate over the table
	for _, tt := range tests {
		// t.Run creates a subtest for nice reporting
		t.Run(tt.name, func(t *testing.T) {
			got := Max(tt.a, tt.b)
			if got != tt.want {
				t.Errorf("Max(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
			}
		})
	}
}
```

### Mocking in Go
For Integration tests, Go uses interfaces to mock dependencies.

```go
// Database interface allows us to mock the DB in tests
type Database interface {
	GetUser(id int) string
}

type MockDB struct{}
func (m MockDB) GetUser(id int) string { return "TestUser" }

func TestUserService(t *testing.T) {
	// Inject the mock
	service := UserService{DB: MockDB{}}
	user := service.Find(1)
	if user != "TestUser" {
		t.Fail()
	}
}
```

## Interview Questions

### Q: "We have 100% code coverage but still have bugs. Why?"
**A:** Code coverage measures *execution*, not *correctness*.
*   It checks if a line of code ran, not if it produced the right result.
*   It doesn't cover logic errors, missing requirements, or concurrency race conditions.
*   **Solution**: Focus on meaningful tests and edge cases, not just vanity metrics.

### Q: "Our CI pipeline takes 45 minutes because of E2E tests. What do you do?"
**A:** This is the "Testing Ice Cream Cone" anti-pattern (inverted pyramid).
*   **Audit**: Review the E2E tests. Are they testing logic that could be tested at the unit level?
*   **Push Down**: Move tests down the pyramid. Replace a slow UI login test with a fast API integration test.
*   **Parallelize**: Run tests in parallel shards.

### Q: "How do you handle flaky tests?"
**A:** Flaky tests are toxic.
*   **Quarantine**: Immediately remove them from the blocking gate (put them in a separate non-blocking job).
*   **Fix or Delete**: Assign an owner to fix the root cause (often timing issues or shared state). If it can't be fixed, delete it. No test is better than a lying test.
