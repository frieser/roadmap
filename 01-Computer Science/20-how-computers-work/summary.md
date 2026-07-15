---
---

## CPU Cycle — Fetch → Decode → Execute
- **Fetch**: PC → address to RAM → instruction loaded into IR. PC auto-incremented.
- **Decode**: Control Unit reads opcode. Determines operation + operands.
- **Execute**: ALU runs math/logic, or data moves between registers. Result optionally stored.

## Memory Hierarchy
| Level | Size | Latency | Managed By |
|-------|------|---------|------------|
| Registers | ~bytes | <1 ns | Compiler (register allocation) |
| L1 Cache | ~64 KB | ~1 ns | CPU hardware |
| L2 Cache | ~256 KB | ~4 ns | CPU hardware |
| L3 Cache | ~8-32 MB | ~10+ ns | CPU hardware (shared across cores) |
| RAM | ~GB | ~100 ns | OS (virtual memory) |
| Disk/SSD | ~TB | ~ms | OS (filesystem) |

## Von Neumann vs Harvard
- **Von Neumann**: Code + data share same memory bus. Simpler. Bottleneck: fetch instruction AND data compete for bandwidth.
- **Harvard**: Separate buses for instruction memory and data memory. Faster (parallel fetch). Used in microcontrollers, DSPs, L1 (split I-cache/D-cache).

## Key CPU Components
- **PC (Program Counter)**: Address of next instruction. JUMP overwrites it for loops/branches.
- **SP (Stack Pointer)**: Top of stack frame. Decremented on function call; restored on return.
- **ALU**: Adder circuits (XOR+AND gates). Two's complement for negatives. Flags: Zero, Overflow.
- **FPU**: IEEE 754 floats. Sign | Exponent | Mantissa. `0.1 + 0.2 != 0.3` — repeating binary fraction.

## Instruction Set Architecture
- **CISC (x86/x64)**: Complex multi-step instructions. Variable-length encoding.
- **RISC (ARM)**: Simple fixed-length instructions. More lines, less power. Load-store model.
- **Instruction format**: `Opcode + Operands`. Assembly = human-readable machine code (1:1 mapping).

## Locality & Cache Lines
- **Temporal**: Recently accessed data likely reused (loop counter).
- **Spatial**: Adjacent addresses likely accessed together (array traversal). Cache lines = 64 bytes.
- **Cache miss**: Data not in cache → stall while fetching from next level.
- **False sharing**: 2 goroutines write different vars on same cache line → cache coherency thrashing.

## Go Relevance
- Escape analysis: pointer returned → heap. Stays local → stack.
- Slices cache-friendly (contiguous). Linked lists cache-unfriendly (scattered).
- Go compiler compiles directly to machine code (no C transpilation).
- Register allocation by compiler for hot variables.

## Data Flow
```
Source Code → Compiler → Assembly → Assembler → Object File → Linker → Binary → Loader → RAM → CPU (Fetch-Decode-Execute)
```
