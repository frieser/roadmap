---
---

## Summary
Here Strings (`<<<`) are a simplified form of Here Documents used to pass a **single string** (or variable) to the standard input of a command. They eliminate the need for `echo "$VAR" | command` pipes, resulting in cleaner and slightly faster code.

## Detailed Explanation

### Syntax
`command <<< "string"`

### Use Cases
*   **Avoiding Pipes**: Instead of `echo "hello" | grep "h"`, use `grep "h" <<< "hello"`.
*   **Feeding Variables**: `bc <<< "10 + 5"` (Calculator).
*   **Trimming**: `read` takes input from stdin. `read -r var <<< "  trimmed  "` puts the string into var (though standard `read` splits words).

### Comparison
*   **Pipe**: Spawns a subshell for the `echo`.
*   **Here String**: Handled by the current shell (Bash extension).

## Go-Specific Context/Examples

In Go, passing a string to an `io.Reader` interface (which `Stdin` is) is typically done with `strings.NewReader`.

### Example: Simulating Here String in Go Test
```go
func TestReader(t *testing.T) {
	input := "hello world"
	// Behaves like <<< "hello world"
	reader := strings.NewReader(input)
	
	// Use reader...
}
```

## Interview Questions

**Q: Is `<<<` POSIX standard?**
**A:** **No**. Here Strings are a **Bash** (and Zsh/Ksh) extension. They will fail in pure `sh` or `dash` (used as `/bin/sh` on Ubuntu). For portability, use `echo "string" | command`.

**Q: How do you use `<<<` with `read`?**
**A:** `IFS= read -r var <<< "$input"`. This assigns the content of `$input` to `$var`. Useful for processing a string without logic pipes.

**Q: What is the difference between `<<` and `<<<`?**
**A:** `<<` (Here Doc) is for **multi-line** blocks of text defined within the script. `<<<` (Here String) is for passing a **single** pre-existing string or variable.
