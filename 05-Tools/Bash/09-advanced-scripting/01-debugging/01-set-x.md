---
---

## Summary
`set -x` (Xtrace) enables "Print Debugging" mode. It prints every command to stderr *after* variable expansion but *before* execution. It is the most common way to debug Bash scripts.

## Detailed Explanation

### Usage
*   **Whole Script**: Add `set -x` at the top or run `bash -x script.sh`.
*   **Partial**:
    ```bash
    set -x  # Start debugging
    ... buggy code ...
    set +x  # Stop debugging
    ```

### Output Format
Debug lines are prefixed with `+`.
```bash
name="Alice"
echo "Hello $name"
# Output:
# + name=Alice
# + echo 'Hello Alice'
# Hello Alice
```

### Customization (`PS4`)
You can change the `+` prefix by setting the `PS4` variable.
`export PS4='+ $BASH_SOURCE:$LINENO: '` prints filename and line number!

## Go-Specific Context/Examples

Go doesn't have a global "trace" flag like Bash. You typically use loggers.

### Analogy
*   **Bash**: `set -x`
*   **Go**: Adding `log.Printf("Executing: %v", args)` before every function call manually, or using a debugger like **Delve**.

## Interview Questions

**Q: Where does the debug output go?**
**A:** It goes to **Standard Error (stderr)**. This means it doesn't corrupt your script's actual output (stdout) if you are piping it to another tool.

**Q: How do you debug a specific section of a script without touching the rest?**
**A:** Wrap the section in `set -x` and `set +x`.

**Q: Why is `set -x` showing passwords in logs?**
**A:** Because it expands variables *before* printing. `echo "$PASSWORD"` becomes `+ echo 'secret123'`. To avoid this, either disable `-x` around sensitive sections or avoid passing secrets as CLI arguments.
