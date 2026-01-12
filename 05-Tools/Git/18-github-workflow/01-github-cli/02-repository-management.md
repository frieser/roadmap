# Repo Management with CLI

## Summary
`gh repo` commands allow you to create, clone, and fork repositories without leaving the terminal.

## Detailed Explanation

### Creating
```bash
# Create a public repo from current directory
gh repo create --public --source=. --push
```

### Cloning
```bash
gh repo clone user/repo
```
Faster than `git clone` because you don't need to copy-paste the URL.

### Forking
```bash
gh repo fork user/repo --clone
```
This forks the repo and immediately clones your fork to your machine.

### Go-specific Context
When starting a new Go project:
```bash
mkdir my-go-lib && cd my-go-lib
go mod init github.com/me/my-go-lib
gh repo create --public --source=.
```

## Interview Questions
**Q: How do you open the current repo in the browser?**
**A:** `gh repo view --web`.
