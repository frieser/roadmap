---
---

## Summary
**Scheduling Algorithms** are used by the OS (Process Scheduler) to decide which process/thread runs on the CPU at any given time. The goal is to maximize CPU utilization, throughput, and fairness while minimizing latency.

## Detailed Explanation
### Types of Scheduling
1.  **Preemptive**: The OS can forcibly pause a running process (interrupt) to run another (e.g., Round Robin). Used in modern desktop/server OS.
2.  **Non-Preemptive**: The process keeps the CPU until it finishes or voluntarily yields (e.g., waiting for I/O). Used in batch systems or very old OS.

### Common Algorithms
*   **FCFS (First-Come, First-Served)**: Simple Queue. Suffer from Convoy Effect (one slow process blocks everyone).
*   **SJF (Shortest Job First)**: Optimal for average wait time, but impossible to implement (cannot predict future burst time).
*   **Round Robin (RR)**: Each process gets a "Time Quantum" (e.g., 10ms). Fair, responsive, but high context switching overhead.
*   **Priority Scheduling**: Run highest priority. Suffer from Starvation (low priority never runs).
*   **Multilevel Feedback Queue**: Complex combination. Multiple queues with different priorities and time quantums. Processes move between queues based on behavior.

### Go Context: The Go Scheduler
The Go Runtime has its own scheduler (User-space) that sits on top of the OS scheduler (Kernel-space).
*   **GMP Model**:
    *   **G (Goroutine)**: The code to execute.
    *   **M (Machine)**: The OS Thread.
    *   **P (Processor)**: The logical resource (context) required to execute Go code.
*   **Work Stealing**: If a Processor (P) runs out of Goroutines (Gs), it "steals" half of the Gs from another P's local queue.

## Interview Questions
**Q: What is the "Convoy Effect"?**
A: In FCFS scheduling, if a CPU-bound process arrives first, many I/O-bound processes stack up behind it waiting for the CPU, leaving I/O devices idle and hurting overall throughput.

**Q: Why use Round Robin over FCFS?**
A: Round Robin guarantees responsiveness. In an interactive system (desktop), you want multiple apps to appear to run "at once". FCFS would freeze the UI while a background task crunches numbers.

**Q: Explain Work Stealing in Go.**
A: To balance load across cores, an idle P (Processor) checks other Ps' queues. If it finds one with many runnable Goroutines, it steals half of them to keep itself busy, ensuring efficient CPU utilization.

## Diagram
```mermaid
graph TD
    subgraph Round Robin
    Q[Queue: P1, P2, P3]
    CPU[CPU]
    
    Q -->|P1 (10ms)| CPU
    CPU -->|Time's Up| Q
    end
    
    subgraph Go Scheduler
    P1[P Local Queue] --> M1[M Thread]
    P2[P Local Queue] --> M2[M Thread]
    M1 -.->|Steal| P2
    end
```
