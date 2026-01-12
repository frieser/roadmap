---
---

## Summary
File permissions in Linux/Unix determine who can read, write, or execute a file. They are defined for three categories: **User** (Owner), **Group**, and **Others** (World). Permissions are represented by characters (`rwx`) or octal numbers (`755`).

## Detailed Explanation

### The Bits
*   **Read (`r`)**: Value **4**. View file contents or list directory.
*   **Write (`w`)**: Value **2**. Modify file content or create/delete files in directory.
*   **Execute (`x`)**: Value **1**. Run file as a program or enter (`cd`) a directory.

### Categories
*   **u**: User (Owner).
*   **g**: Group.
*   **o**: Others.

### Examples
*   `rwx`: 4+2+1 = 7 (Full access).
*   `r-x`: 4+0+1 = 5 (Read and Execute).
*   `rw-`: 4+2+0 = 6 (Read and Write).

### Directory Permissions
*   **Read**: Can `ls` the directory.
*   **Write**: Can `touch`, `mkdir`, `rm` files inside.
*   **Execute**: Can `cd` into the directory.

## Go-Specific Context/Examples

In Go, `os.FileMode` represents these permissions.

### Analogy
*   **Bash**: `ls -l` shows `-rw-r--r--`.
*   **Go**: `0644`.
    ```go
    // Create file with rw-r--r--
    os.WriteFile("test.txt", data, 0644)
    ```

## Interview Questions

**Q: What does `777` mean?**
**A:** `rwxrwxrwx`. User, Group, and Others all have full Read, Write, and Execute permissions. This is generally a security risk.

**Q: What is the difference between execute on a file vs a directory?**
**A:**
*   **File**: Allows the OS to run it as a process.
*   **Directory**: Allows you to **enter** (`cd`) it and access metadata of files inside. Without `x` on a dir, you cannot access `dir/file` even if you have read permissions on `file`.

**Q: How do you calculate `755`?**
**A:**
*   User: 7 (4+2+1) -> `rwx`
*   Group: 5 (4+0+1) -> `r-x`
*   Other: 5 (4+0+1) -> `r-x`
