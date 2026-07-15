---
---

## Concurrency vs Parallelism
- **Concurrency**: structure — multiple tasks interleaved (single core OK).
- **Parallelism**: execution — multiple tasks simultaneous (requires multi-core).

### Amdahl's Law
`S(N) = 1 / ((1-P) + P/N)`. Sequential fraction caps max speedup.

### Race Condition
≥2 threads access shared data, ≥1 is write, no sync → non-deterministic result.

## Process vs Thread

|  | Process | Thread |
|--|---------|--------|
| Memory | Own virtual memory (stack+heap+code) | Own stack+registers; shares heap/code |
| IPC | Pipes, sockets, shared memory (hard) | Direct memory access (easy, risky) |
| Creation overhead | High (clone full process, page tables) | Low (alloc stack only) |
| Isolation | Crash isolated to self | Crash kills entire process |
| Switch cost | Kernel-mode (expensive) | Kernel-mode (thread); user-mode (goroutine) |
| Communication | OS-mediated | Shared memory (need sync primitives) |

## Scheduling Algorithms
- **FCFS**: simple queue; convoy effect (slow blocks all).
- **SJF**: optimal avg wait; impossible to predict burst.
- **Round Robin**: time quantum per process; fair, responsive, higher ctx-switch overhead.
- **Priority**: highest priority runs; starvation risk for low-priority.
- **MLFQ**: multiple queues, dynamic priority based on behavior.

## CPU Interrupts
- **Hardware**: asynchronous (keyboard, disk, network). **Software**: synchronous (traps=syscalls, exceptions=faults).
- **ISR cycle**: save context → IVT lookup → execute ISR (kernel mode) → restore → resume.

## Fork & Exec
- `fork()`: clones calling process. COW optimizes — pages shared until write.
- `exec()`: replaces child memory with new program. Shell redirection relies on fork+exec gap.

## Memory Management
- **Virtual Memory**: MMU translates virtual→physical. Isolation + swap.
- **Paging**: fixed-size pages (4KB). Page table maps to frames. Page fault → load from disk.
- **Stack**: grows down, function frames, fast (pointer move). **Heap**: grows up, dynamic alloc, GC/manual.
- **Thrashing**: spend more time swapping than executing.

## Synchronization Primitives
- **Mutex**: binary lock, ownership required (locker must unlock).
- **Semaphore**: counter N. Binary (N=1, no ownership) or Counting (resource pool).
- **Spinlock**: busy-wait loop. Fast for short holds, wastes CPU for long.
- **RWMutex**: multiple readers OR one writer. Read-many/write-rarely.

## Deadlock
Coffman's 4 conditions (all must hold):
- **Mutual Exclusion**: resource cannot be shared.
- **Hold and Wait**: thread holds resource while waiting for others.
- **No Preemption**: OS cannot forcibly take resource away.
- **Circular Wait**: T1 waits on T2, T2 waits on T3, ..., Tn waits on T1.

## Go-Specific
- **Goroutine**: lightweight (2KB stack, growable), M:N mapped to OS threads.
- **GMP Model**: G=goroutine, M=OS thread, P=logical processor (local queue). Work-stealing balances load.
- **Channels**: preferred over mutexes. "Share memory by communicating."
- **`sync.Mutex`**: protect shared state (maps, counters). `defer mu.Unlock()`.
- **GC**: concurrent mark-sweep. Per-P `mcache` for tiny allocs (lock-free).

## Key Items
- **Context Switch**: save PC+regs to PCB, load next. Expensive (cache flush).
- **Copy-On-Write**: fork optimizes by sharing pages read-only; copy only on write.
- **Zombie Process**: exited but parent hasn't `wait()`ed — entry stays in process table.
