---
---

## Summary
The `shift` command is used to process command-line arguments one by one. It shifts the positional parameters to the left: `$2` becomes `$1`, `$3` becomes `$2`, and the original `$1` is discarded.

## Detailed Explanation

### Usage
Typically used in a `while` loop to process arguments until none are left (`$#` becomes 0).

```bash
while [ $# -gt 0 ]; do
    case "$1" in
        -h|--help)
            show_help
            shift
            ;;
        -f|--file)
            FILE="$2"
            shift 2  # Shift twice (past flag and value)
            ;;
        *)
            echo "Unknown: $1"
            shift
            ;;
    case
done
```

### Syntax
*   `shift`: Shift once (default).
*   `shift N`: Shift N places.

## Go-Specific Context/Examples

In Go, you can simulate `shift` by slicing the `args` array.

### Analogy
**Bash**: `shift`
**Go**:
```go
args := os.Args[1:] // Skip program name
for len(args) > 0 {
    flag := args[0]
    args = args[1:] // Shift 1
    
    if flag == "--file" {
        filename := args[0]
        args = args[1:] // Shift 1 again
    }
}
```

## Interview Questions

**Q: What happens if you run `shift` when `$#` is 0?**
**A:** It returns a non-zero exit status (error).

**Q: Can you shift past the last argument?**
**A:** No. `shift` fails if you try to shift more places than there are arguments.

**Q: Why use `shift` instead of looping over `"$@"`?**
**A:** `shift` allows you to consume arguments dynamically (e.g., a flag that takes a value consumes 2 slots). A simple `for arg in "$@"` loop makes it harder to grab the "next" argument as a value for the current flag.
