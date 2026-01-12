# Commit-Msg Hook

## Summary
The `commit-msg` hook runs after the user enters a commit message but before the commit is created. It receives the path to the temporary file containing the message.

## Detailed Explanation

### Usage
It can exit with non-zero status to abort the commit.
Commonly used to enforce:
*   **Pattern**: "Subject must start with [JIRA-123]".
*   **Length**: "Subject must be > 10 chars".
*   **Format**: Conventional Commits (feat, fix, etc.).

### Go Example
A bash script to check regex:
```bash
#!/bin/sh
INPUT_FILE=$1
START_LINE=$(head -n1 $INPUT_FILE)
PATTERN="^(feat|fix|docs|style|refactor|perf|test|chore)(\(.+\))?: .+$"
if ! [[ "$START_LINE" =~ $PATTERN ]]; then
  echo "Bad commit message, see CONTRIBUTING.md"
  exit 1
fi
```

## Interview Questions
**Q: Can the hook modify the message?**
**A:** Yes, it can edit the file in place (e.g., automatically appending a sign-off trailer).
