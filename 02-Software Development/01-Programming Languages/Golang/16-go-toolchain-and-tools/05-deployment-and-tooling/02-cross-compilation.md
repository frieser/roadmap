# Cross Compilation

## Summary
Go has first-class support for cross-compilation. You can build a binary for a different Operating System (`GOOS`) or Architecture (`GOARCH`) directly from your development machine without needing a virtual machine or external toolchain (as long as CGO is disabled).

## Detailed Explanation

### Basic Usage
Set the environment variables before running `go build`.
```bash
# Build for Windows 64-bit on Linux
GOOS=windows GOARCH=amd64 go build -o app.exe

# Build for Mac (Apple Silicon) on Windows
GOOS=darwin GOARCH=arm64 go build -o app-mac
```

### Common Targets
| GOOS | GOARCH | Target |
| :--- | :--- | :--- |
| `linux` | `amd64` | Standard Linux Server |
| `linux` | `arm64` | AWS Graviton, Raspberry Pi 64 |
| `windows` | `amd64` | Windows 10/11 |
| `darwin` | `amd64` | Intel Mac |
| `darwin` | `arm64` | M1/M2 Mac |
| `js` | `wasm` | WebAssembly (Browser) |

### The CGO Problem
If your app uses `CGO_ENABLED=1` (e.g., relies on SQLite or specific C libraries), cross-compilation becomes hard because you need a C cross-compiler (like `gcc-aarch64-linux-gnu`) installed.
*   **Solution 1**: Disable CGO (`CGO_ENABLED=0`) if possible.
*   **Solution 2**: Use `zig cc` as a drop-in C compiler replacement, which handles cross-compilation effortlessly.

## Interview Questions

**Q: How do you check all supported OS/Arch combinations?**
**A:** Run the command `go tool dist list`. It prints all valid pairs like `android/arm`, `linux/mips`, `plan9/386`, etc.

**Q: What is `GOARM`?**
**A:** It is an extra environment variable used only when `GOARCH=arm` (32-bit ARM). It specifies the floating-point hardware version. `GOARM=6` is for Raspberry Pi Zero (ARMv6), while `GOARM=7` is for Raspberry Pi 3 (ARMv7). Using the wrong one causes "Illegal instruction" crashes.

**Q: Can you cross-compile from Windows to Mac?**
**A:** Yes, easily with `GOOS=darwin`. However, you cannot sign or notarize the macOS binary on Windows; Apple requires that step to be done on macOS. But the compilation itself produces a valid Mach-O binary.
