# Intro and Git Commands

## Summary
Git provides a suite of commands to manage the lifecycle of changes. The core feedback loop involves checking status, adding files, and committing. Understanding these basic commands is the foundation of Git proficiency.

## Detailed Explanation

### Core Commands
1.  **`git status`**: The most used command. Shows the state of the working directory and staging area. Tells you what is tracked, untracked, modified, or staged.
2.  **`git add <file>`**: Stages content.
3.  **`git commit`**: Records a snapshot.
4.  **`git log`**: View commit history.
5.  **`git help <command>`**: Open manual for a command.

### The Feedback Loop
1.  **Edit**: Modify code in your editor.
2.  **Status**: `git status` (See what changed).
3.  **Diff**: `git diff` (Verify the exact lines changed).
4.  **Add**: `git add .` (Stage changes).
5.  **Status**: `git status` (Verify what is staged).
6.  **Commit**: `git commit -m "..."`.

### Go-specific Context
Integrating Go tools into the loop:
```bash
# 1. Edit code
vim main.go

# 2. Verify
go build ./...
go test ./...

# 3. Git loop
git status
git add main.go
git commit -m "refactor: simplify main loop"
```

## Interview Questions
**Q: Which command shows the history of commits?**
**A:** `git log`. You can use `git log --oneline --graph` for a cleaner view.

**Q: What does `git status` tell you?**
**A:** It tells you which branch you are on, if your branch is ahead/behind remote, and the state of your files (untracked, modified, staged).
