---
---

## Summary
In Unix/Linux, `0` represents **Success** (True), and any value from `1` to `255` represents **Failure** (False). This is the opposite of many programming languages (like C/Java boolean) where 0 is often False.

## Detailed Explanation

### Standard Codes
*   **0**: OK.
*   **1**: General error (Catch-all).
*   **2**: Misuse of shell builtins.
*   **126**: Command invoked cannot execute (Permission denied).
*   **127**: Command not found.
*   **128+N**: Fatal error signal "N".
    *   **130**: Script terminated by Control-C (SIGINT = 2, so 128+2).
    *   **137**: Killed by OOM Killer (SIGKILL = 9, so 128+9).

### Custom Codes
Scripts can define their own exit codes (e.g., `exit 10` for config error, `exit 20` for network error) to help callers debug issues.

## Go-Specific Context/Examples

Go programs interact with this system via `os.Exit(code)`.

### Analogy
*   **Success**: `os.Exit(0)` (or simply returning from `main`).
*   **Fatal**: `log.Fatal("error")` calls `os.Exit(1)`.
*   **Custom**: `os.Exit(127)`.

## Interview Questions

**Q: Why is 0 success?**
**A:** Because there is only one way to succeed, but many ways to fail. 0 is unique. Non-zero allows returning specific error codes (1-255) to indicate *why* it failed.

**Q: If I press Ctrl+C, what is the exit code?**
**A:** 130. This is derived from 128 + Signal 2 (SIGINT).

**Q: Is it good practice to `exit -1`?**
**A:** No. Exit codes are unsigned 8-bit integers (0-255). `-1` wraps around to `255`. It works, but it's confusing. Stick to 1 for general errors.
