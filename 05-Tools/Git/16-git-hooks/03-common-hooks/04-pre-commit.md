# Pre-Commit Hook

## Summary
The `pre-commit` hook is the most popular client-side hook. It runs before you even type a commit message. It is used to inspect the snapshot that is about to be committed.

## Detailed Explanation

### Usage
It checks the **Staged** files.
If the script exits with non-zero, the commit is aborted.

### Best Practices
*   **Fast**: It should run in < 2 seconds. Don't run the full test suite here.
*   **Focused**: Only check staged files (`git diff --cached --name-only`).

### Go-specific Context
A `pre-commit` hook for Go:
```bash
#!/bin/sh
# Fail if go fmt makes changes
gofmt -l -w .
if [ -n "$(git diff --name-only)" ]; then
    echo "Go files were formatted. Please stage them again."
    exit 1
fi
```

## Interview Questions
**Q: Does pre-commit run on `git merge`?**
**A:** Yes, if the merge requires a commit (not fast-forward), `pre-commit` runs.
