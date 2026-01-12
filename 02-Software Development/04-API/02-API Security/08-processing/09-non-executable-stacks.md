#API #Security #System
---
---

# Non-Executable Stacks (NX Bit / DEP)

## Summary
**Non-Executable Stacks** are a system-level security defense (hardware and OS) designed to prevent **Buffer Overflow** attacks. By marking the memory stack as "Non-Executable" (NX), the system ensures that even if an attacker injects malicious shellcode into a buffer on the stack, the CPU will refuse to execute it, crashing the program instead of yielding control.

## Detailed Explanation

### 1. The Vulnerability: Stack Smashing
In languages like C/C++, local variables are stored on the **Stack**.
*   **The Attack**: An attacker sends more data than a buffer can hold (Buffer Overflow).
*   **The Payload**: The excess data overwrites the **Return Address** on the stack and points it to the attacker's injected code (Shellcode), which is also stored in the stack buffer.
*   **Execution**: When the function returns, the CPU jumps to the shellcode and executes it.

### 2. The Defense: DEP / NX Bit
*   **DEP (Data Execution Prevention)**: The operating system policy that separates memory into areas for *code* (executable) and *data* (non-executable).
*   **NX Bit (No-Execute)**: The hardware feature (Bit 63 in Page Table Entries) that enforces DEP.
*   **Mechanism**: The Stack and Heap are marked as Data (Read/Write only). If the Instruction Pointer (IP) tries to execute code in these regions, the CPU raises a hardware exception (`Segmentation Fault`).

### 3. Relevance to High-Level Languages (Go)
*   **Memory Safety**: Go is memory-safe. It has bounds checking, which prevents Buffer Overflows in the first place.
*   **Stack Management**: Go manages its own stacks (goroutine stacks). While Go binaries typically have non-executable stacks enabled by the linker, the primary defense in Go is the language design itself.
*   **CGO**: If your Go app uses **CGO** (C libraries), you lose Go's memory safety guarantees for that part of the code, making NX/DEP critical again.

---

## Configuration & Verification

### Compiler Flags (GCC/Clang)
When compiling C code (or CGO):
*   `-z noexecstack`: Tells the linker to mark the stack as non-executable.
*   `-z execstack`: (Bad) Enables executable stack (often needed for nested functions or trampolines, but insecure).

### Verification
You can check if a binary has an executable stack using `readelf` or `execstack`.

```bash
# Check headers for 'GNU_STACK'
readelf -l my_binary | grep GNU_STACK

# Output:
# GNU_STACK      0x000000 0x000000 0x000000 0x00000 0x00000 RW  0x10
# 'RW' means Read/Write (Safe). If it says 'RWE', it is Executable (Unsafe).
```

### Go Implementation Detail
Go binaries default to non-executable stacks. You can verify this on a standard Go build.

```bash
go build main.go
readelf -l main
# Look for GNU_STACK -> RW (Secure)
```

---

## Interview Questions

**Q1: What is the "NX Bit" and what does it prevent?**
**A:** The NX (No-Execute) Bit is a hardware feature that allows the OS to mark specific areas of memory (like the Stack and Heap) as non-executable. It prevents **Code Execution** exploits resulting from Buffer Overflows. If an attacker injects shellcode into the stack, the CPU refuses to run it.

**Q2: Does enabling NX/DEP completely stop Buffer Overflow attacks?**
**A:** No. It stops the execution of *injected* code (Shellcode). However, attackers adapted by using **Return-Oriented Programming (ROP)**. Instead of injecting new code, ROP chains together existing valid code snippets (gadgets) already present in the executable memory to perform malicious actions, bypassing NX.

**Q3: How does Go handle stack execution?**
**A:** Go is a memory-safe language, so buffer overflows are generally impossible in pure Go code (due to bounds checking). Additionally, the Go linker marks stacks as non-executable by default. However, if using `unsafe` or `CGO`, memory safety can be violated, making system-level protections like NX relevant.

**Q4: What is the output of `readelf` that indicates a binary is secure against stack execution?**
**A:** You look for the `GNU_STACK` program header. The flags should be `RW` (Read/Write). If the flags are `RWE` (Read/Write/Execute), the stack is executable and the binary is vulnerable.
