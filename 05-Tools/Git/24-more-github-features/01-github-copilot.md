# GitHub Copilot

## Summary
GitHub Copilot is an AI pair programmer that runs inside your IDE (VS Code, JetBrains, Vim). It suggests whole lines or entire functions based on the context of your code.

## Detailed Explanation

### Features
*   **Autocomplete**: Suggests code as you type.
*   **Chat**: Ask questions about your code ("Explain this function", "Fix this bug").
*   **CLI**: `gh copilot suggest "how to list files in s3"`.

### Go-specific Context
Copilot is extremely good at writing boilerplate Go code (e.g., "Write a HTTP handler that accepts JSON and saves to Postgres").
It understands Go idioms like `if err != nil`.

## Interview Questions
**Q: Is Copilot code always correct?**
**A:** No. You must review it. It can hallucinate APIs that don't exist.

**Q: Does Copilot use my code for training?**
**A:** It depends on your settings (Enterprise usually disables this).
