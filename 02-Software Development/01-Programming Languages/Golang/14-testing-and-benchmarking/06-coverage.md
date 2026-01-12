# Test Coverage

## Summary
Test coverage measures the percentage of your code that is executed during tests. Go has built-in support for coverage analysis via the `-cover` flag. It provides both a summary percentage and detailed HTML reports showing exactly which lines were hit (green) and missed (red), helping developers identify untested logic paths.

## Detailed Explanation

### Basics
To check coverage, simply add the `-cover` flag:
```bash
go test -cover ./...
```
Output:
```
ok      github.com/myproject/math    0.123s    coverage: 85.4% of statements
```

### Advanced Profiling
For detailed analysis, you need to generate a "coverage profile" (a text file recording execution counts) and then view it.

1.  **Generate Profile**:
    ```bash
    go test -coverprofile=coverage.out ./...
    ```

2.  **View HTML Report** (Interactive):
    ```bash
    go tool cover -html=coverage.out
    ```
    This opens your default browser with a source code view.
    *   **Green**: Covered lines.
    *   **Red**: Uncovered lines.
    *   **Grey**: Non-executable lines (comments, types).

3.  **View CLI Summary**:
    ```bash
    go tool cover -func=coverage.out
    ```
    Shows coverage percentage per function.

### Coverage Modes (`-covermode`)
*   `set` (default): Did this statement run? (Bool)
*   `count`: How many times did this statement run? (Int) - Useful for finding hotspots.
*   `atomic`: Same as `count` but thread-safe (more expensive). Required if testing parallel code.

### 100% Coverage Myth
While high coverage is good, 100% is rarely a practical or necessary goal. It guarantees code *execution*, not code *correctness*. It is better to focus on covering critical business logic and error handling paths than chasing 100% on getters/setters or boilerplate.

### Code Example (How it works internally)
Go "rewrites" your source code before compiling the test binary.
**Original**:
```go
func Max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```
**Rewritten (conceptual)**:
```go
func Max(a, b int) int {
    _cover[0]++
    if a > b {
        _cover[1]++
        return a
    }
    _cover[2]++
    return b
}
```

## Interview Questions

**Q: Does 100% test coverage mean the code is bug-free?**
**A:** No. Coverage only proves that the code was *executed*, not that it produces the *correct results* for all possible inputs. You could have 100% coverage with a test that asserts nothing (`expect(true).toBe(true)` equivalent). It also doesn't account for missing logic (code that *should* be there but isn't).

**Q: How do you exclude a file from test coverage in Go?**
**A:** There is no native configuration file (like `.gitignore`) to exclude files from coverage. However, you can filter the output.
1.  Use build tags: `// +build !test` (rare).
2.  Use `grep -v` on the generated `coverage.out` file before running the report tool.
3.  Structure your project so that generated code (mocks, protobufs) lives in separate packages that you can exclude from the `go test` arguments.

**Q: What is the difference between `-covermode=set` and `-covermode=count`?**
**A:** `set` (the default) only records *if* a statement was executed (boolean). `count` records *how many times* it was executed. `count` is useful for spotting "hot" code paths (heatmaps) in the HTML view, showing which parts of the code are heavily exercised by tests.
