---
---
# Non-Executable Stacks & Memory Protection

Memory protection is a critical layer in modern computer security, designed to prevent attackers from executing malicious code even if they find a vulnerability like a buffer overflow.

## 1. The Problem: Buffer Overflows & Code Injection
A **Buffer Overflow** occurs when a program writes more data to a fixed-length block of memory (a buffer) than it can hold. On the **Stack**, this is particularly dangerous because:

- **Stack Layout**: Local variables are stored alongside critical control data, most notably the **Saved Return Address**.
- **The Attack**: An attacker overflows a local buffer to overwrite the return address. Instead of returning to the caller, the function "returns" to a location of the attacker's choosing.
- **Code Injection**: Traditionally, attackers would fill the buffer with **Shellcode** (machine instructions) and point the return address to the start of that buffer. The CPU then begins executing the attacker's injected instructions from the stack.

## 2. The Solution: NX Bit / DEP
**Executable-Space Protection** prevents the execution of code from memory regions designated for data.

- **NX Bit (No-Execute)**: A hardware-level feature (introduced as NX by AMD and XD by Intel). It utilizes **Bit 63** in the Page Table Entry (PTE). When set, the Memory Management Unit (MMU) will refuse to fetch instructions from that page, triggering a fault.
- **DEP (Data Execution Prevention)**: The operating system's implementation of the NX bit.
- **Impact**: By marking the stack (and heap) as **Read/Write (RW)** but **NOT Execute (X)**, the CPU will immediately terminate the program if it tries to run shellcode injected into these regions.

## 3. Related Defenses
Security is "defense in depth." NX bit is rarely used alone:

- **ASLR (Address Space Layout Randomization)**: Randomizes the memory addresses of the stack, heap, and shared libraries (libc) every time the program runs. This makes it difficult for an attacker to know *where* to point the return address.
- **Stack Canaries (Stack Protectors)**: The compiler inserts a "canary" value (a random secret) between local buffers and the return address. Before a function returns, it checks if the canary is still intact. If it has been overwritten (indicating an overflow), the program aborts.
- **Control-flow Enforcement Technology (CET)**: A modern hardware feature (Intel/AMD) that implements a **Shadow Stack**. It keeps a secondary, isolated copy of return addresses that cannot be modified by standard instructions, making return address hijacking nearly impossible.

## 4. Evasion: Return-Oriented Programming (ROP)
When NX/DEP prevents code *injection*, attackers switch to **Code Reuse**.

- **ROP Concepts**: Attackers find small snippets of existing, valid executable code in the program's memory (e.g., in `libc`) that end in a `ret` instruction. These are called **Gadgets**.
- **The Chain**: By carefully crafting a stack of return addresses, the attacker can "chain" these gadgets together to perform complex operations (like calling `system("/bin/sh")`) without ever injecting a single new instruction.
- **Bypass**: ROP bypasses NX because the code being executed is already marked as executable by the OS.

## 5. Go Context: Safety & Risks
Go is designed to be memory-safe, but it still interacts with these low-level protections.

- **Memory Safety**: Go prevents most buffer overflows through **Strict Bounds Checking**. Accessing an index outside a slice or array's range causes a `panic`, not a memory corruption.
- **Go Stacks**: Go uses its own stack management (contiguous stacks). While these are managed by the Go runtime, they are still allocated from memory regions that respect OS-level NX/DEP protections.
- **The `unsafe` Package**: The `unsafe` package allows developers to bypass Go's type safety and perform pointer arithmetic.
    - **Risk**: Using `unsafe.Pointer` or `uintptr` can re-introduce classic buffer overflow vulnerabilities if not handled with extreme care.
    - **CGO**: Calling C code via CGO also bypasses Go's safety nets, making the application susceptible to C-style memory vulnerabilities.

## 6. Interview Questions
1. **What is the NX bit and how does it prevent shellcode execution?**
   - *Answer*: The NX bit is a hardware flag in the page table that marks memory pages (like the stack) as non-executable. If the CPU tries to fetch an instruction from an NX-protected page, it triggers a segmentation fault, stopping injected shellcode from running.
2. **How does ROP (Return-Oriented Programming) bypass NX/DEP?**
   - *Answer*: ROP doesn't inject new code. Instead, it chains together "gadgets"—short sequences of existing executable instructions already present in the binary or libraries. Since this code is already marked as executable, NX/DEP does not block it.
3. **What is the role of a Stack Canary?**
   - *Answer*: A Stack Canary is a random value placed on the stack before the return address. The program verifies this value hasn't changed before returning from a function. If an overflow occurs, the canary is usually corrupted first, allowing the program to detect the attack and shut down.
4. **Why is ASLR important when used alongside NX bit?**
   - *Answer*: NX prevents code injection, but attackers can still use ROP to reuse existing code. ASLR makes ROP much harder by randomizing where that existing code (and the stack) is located, meaning the attacker doesn't know the addresses for their gadgets.
5. **How does Go's "unsafe" package affect memory protection?**
   - *Answer*: `unsafe` allows direct memory manipulation and pointer arithmetic, which bypasses Go's built-in bounds checking. This can lead to buffer overflows and other memory corruption issues that Go's runtime normally prevents, though hardware protections like NX bit will still apply to the memory itself.
