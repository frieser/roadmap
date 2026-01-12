---
---

## Summary
Positional parameters are special variables (`$1`, `$2`, `$3`...) that hold the arguments passed to a script or function. They are read-only and are reset when a function is called.

## Detailed Explanation

### Variables
*   **`$0`**: Name of the script itself.
*   **`$1` - `$9`**: The first 9 arguments.
*   **`${10}`**: Arguments 10 and above (curly braces required).
*   **`$#`**: The **count** of arguments.
*   **`$@`**: All arguments as a list of separate strings (`"$1" "$2"...`). **Preferred**.
*   **`$*`**: All arguments as a single string (`"$1 $2..."`).

### Usage
```bash
if [ $# -lt 2 ]; then
    echo "Usage: $0 <source> <dest>"
    exit 1
fi
src=$1
dest=$2
```

## Go-Specific Context/Examples

In Go, arguments are available via the `os.Args` slice. `os.Args[0]` is the program name.

### Analogy
**Bash**:
```bash
echo "First arg: $1"
echo "Count: $#"
```
**Go**:
```go
if len(os.Args) > 1 {
    fmt.Println("First arg:", os.Args[1])
}
fmt.Println("Count:", len(os.Args)-1)
```

## Interview Questions

**Q: Why use `"$@"` instead of `"$*"`?**
**A:** `"$@"` preserves the integrity of arguments that contain spaces. If you pass `"My File.txt"` as an argument, `"$@"` keeps it as one unit. `"$*"` merges all arguments into one long string separated by spaces (IFS), losing the distinction between separate arguments.

**Q: How do you access the last argument?**
**A:** `${@: -1}` (Bash substring expansion) or using indirect reference `${!#}`.

**Q: What happens to `$1` inside a function?**
**A:** Inside a function, `$1` refers to the first argument passed **to that function**, not the script. The script's global `$1` is shadowed.
