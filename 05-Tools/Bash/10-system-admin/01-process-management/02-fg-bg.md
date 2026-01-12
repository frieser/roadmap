---
---

## Summary
`fg` (Foreground) and `bg` (Background) are commands to manage job execution states in the shell. They allow you to switch a process between running interactively, running silently in the background, or being paused.

## Detailed Explanation

### Usage Flow
1.  **Start Background**: `long_script.sh &`.
2.  **Suspend Foreground**: Press `Ctrl+Z`. The process stops (Paused).
3.  **Resume in Background**: Type `bg`. The paused process starts running again, but in the background.
4.  **Bring to Foreground**: Type `fg`. A background process takes over the terminal input again.

### Syntax
*   `fg %1`: Bring job 1 to front.
*   `bg %1`: Resume job 1 in background.

## Go-Specific Context/Examples

When writing Go CLI tools, you can handle signals like `SIGTSTP` (Ctrl+Z), but usually, this is handled by the OS/Shell.

### Example: Go Process
If you run `go run main.go` and hit Ctrl+Z, the OS sends SIGTSTP. The Go runtime pauses. `fg` sends SIGCONT (Continue).

## Interview Questions

**Q: Why use `bg`?**
**A:** If you forgot to add `&` when running a long command (like a backup), you don't have to kill it. You can pause it (`Ctrl+Z`) and then resume it in the background (`bg`), freeing up your terminal immediately.

**Q: Can a background process read from Stdin?**
**A:** No. If a background job tries to read from the terminal, it gets suspended (SIGTTIN signal) until you bring it to the foreground (`fg`).

**Q: How do you start a command so it survives logout?**
**A:** `nohup command &` or use a terminal multiplexer like `tmux` / `screen`.
