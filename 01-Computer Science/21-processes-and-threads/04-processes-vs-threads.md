---
---

## Summary
**Processes** and **Threads** are both units of execution, but they differ in isolation and resource sharing. A Process is an instance of a program with its own memory space. A Thread is a "lightweight process" that lives *inside* a process and shares memory with other threads.

## Detailed Explanation
### Process
*   **Isolation**: Has its own Virtual Memory (Stack, Heap, Code), File Descriptors.
*   **Communication**: IPC (Inter-Process Communication) like Pipes, Sockets, Shared Memory (harder).
*   **Overhead**: High (creation and context switching is slow).
*   **Crash**: If one process crashes, others are safe.

### Thread
*   **Shared Memory**: Shares Heap, Code, Files with other threads in the same process. Has its own **Stack** and **Registers**.
*   **Communication**: Direct memory access (easy but risky - Race Conditions).
*   **Overhead**: Low.
*   **Crash**: If one thread crashes (e.g., segfault), the *entire process* dies.

### Go Context
Go introduces **Goroutines**.
*   **M:N Scheduling**: M Goroutines mapped to N OS Threads.
*   **Size**: Goroutines start at 2KB stack (OS Thread ~1MB).
*   **Switching**: Goroutine switch is cheap (User space). Thread switch is expensive (Kernel space).

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// These run in the SAME process, potentially different OS threads
	go func() {
		fmt.Println("I am a Goroutine")
	}()
	
	time.Sleep(100 * time.Millisecond)
}
```

## Interview Questions
**Q: Which is faster to create: Process or Thread?**
A: Thread. Creating a process requires duplicating the entire memory map (Page Tables) and OS resources. Creating a thread just allocates a small stack and a few kernel structures.

**Q: Do threads share the Stack?**
A: No. Each thread must have its own Stack to maintain its own function call history and local variables. They share the Heap and Code.

**Q: What is a "Zombie Process"?**
A: A process that has completed execution (`exit()`) but still has an entry in the process table because its parent hasn't retrieved its exit status (via `wait()`).

## Diagram
```mermaid
graph TD
    subgraph Process A
    Code[Code Segment]
    Heap[Heap Memory]
    Files[File Descriptors]
    
    subgraph Thread 1
    Stack1[Stack]
    Reg1[Registers]
    end
    
    subgraph Thread 2
    Stack2[Stack]
    Reg2[Registers]
    end
    
    Code --- Thread 1
    Code --- Thread 2
    Heap --- Thread 1
    Heap --- Thread 2
    end
```
