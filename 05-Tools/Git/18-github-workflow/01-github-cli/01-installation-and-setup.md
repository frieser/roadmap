# GitHub CLI Installation

## Summary
The GitHub CLI (`gh`) is a command-line tool that brings GitHub features (Issues, PRs, Releases) into your terminal, sitting alongside `git`.

## Detailed Explanation

### Installation
*   **macOS**: `brew install gh`
*   **Windows**: `winget install --id GitHub.cli`
*   **Linux**: `sudo apt install gh` (requires adding repo).

### Setup
Run:
```bash
gh auth login
```
Follow the prompts to authenticate via browser or token. Choose "HTTPS" or "SSH" as your preferred git protocol.

## Interview Questions
**Q: Can `gh` replace `git`?**
**A:** No, they complement each other. `git` manages the local history and syncing. `gh` manages the GitHub-specific layer (PRs, Issues, Releases).
