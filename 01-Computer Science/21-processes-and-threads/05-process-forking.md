---
---

## Summary
**Forking** is the primary method of process creation in Unix-like systems. The `fork()` system call creates a **new process** (Child) by duplicating the calling process (Parent). The child is an almost exact copy of the parent.

## Detailed Explanation
### The `fork()` System Call
*   **Clone**: Creates a new process with a new PID.
*   **Memory**: Historically copied all memory. Modern OS use **Copy-On-Write (COW)**.
    *   *COW*: Parent and Child share physical memory pages (Read-Only). If either writes to a page, the OS pauses, copies that specific page, and lets them write to their own private copies.
*   **Return Value**: `fork()` returns `0` to the Child, and the `ChildPID` to the Parent.

### The `exec()` System Call
Usually, after `fork()`, the child calls `exec()`.
*   **Replace**: `exec()` replaces the current process's memory space (Code, Data, Stack) with a *new* program from disk.
*   **Flow**: Fork (Duplicate) -> Exec (Replace).

### Go Context
Go's `os/exec` package wraps this logic. It forks, sets up pipes/files, and then execs the command.

```go
package main

import (
	"fmt"
	"os/exec"
)

func main() {
	// This performs Fork + Exec internally
	cmd := exec.Command("ls", "-la")
	output, _ := cmd.Output()
	fmt.Println(string(output))
}
```

## Interview Questions
**Q: Why do we have `fork()` AND `exec()`? Why not just one `spawn()`?**
A: Separation of concerns. `fork()` allows the parent to manipulate the child's environment (File Descriptors, Environment Variables, User ID) *before* the new program starts running via `exec()`. This is how Shell redirection (`ls > file.txt`) works.

**Q: What is Copy-On-Write (COW)?**
A: An optimization where the OS delays copying memory pages until a process actually modifies them. It makes `fork()` extremely fast because most of the time the child calls `exec()` immediately, discarding the memory anyway.

## Diagram
```mermaid
graph TD
    Parent[Parent Process] -->|fork()| Child[Child Process]
    
    note[Child is a clone]
    
    Child -->|exec('ls')| NewProg[New Program 'ls']
    
    note2[Memory replaced]
```
