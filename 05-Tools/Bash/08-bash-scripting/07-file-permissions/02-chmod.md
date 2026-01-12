---
---

## Summary
`chmod` (Change Mode) is the command used to change the file mode bits (permissions) of a file or directory. It supports both **Symbolic** mode (letters) and **Octal** mode (numbers).

## Detailed Explanation

### Octal Mode (Absolute)
Sets the permissions exactly.
*   `chmod 755 script.sh`: User=rwx, Group=rx, Other=rx.
*   `chmod 644 file.txt`: User=rw, Group=r, Other=r.
*   `chmod 600 key.pem`: User=rw, Group=none, Other=none (Private key).

### Symbolic Mode (Relative)
Modifies existing permissions without recalculating the whole set.
*   `chmod +x script.sh`: Add execute for everyone.
*   `chmod u+x script.sh`: Add execute for User only.
*   `chmod go-w file.txt`: Remove write for Group and Others.
*   `chmod u=rwx,go=r dir`: Set User to rwx, Group/Others to read-only.

### Recursive
*   `chmod -R 755 /var/www`: Apply to all files and subdirectories.

## Go-Specific Context/Examples

Go's `os.Chmod` uses the octal representation.

### Example: Changing Mode in Go
```go
package main

import (
	"log"
	"os"
)

func main() {
	// Equivalent to chmod 644 file.txt
	err := os.Chmod("file.txt", 0644)
	if err != nil {
		log.Fatal(err)
	}
}
```

## Interview Questions

**Q: Why use `chmod +x` instead of `chmod 755`?**
**A:** `chmod +x` is safer if you want to make a file executable but preserve its existing read/write flags. `chmod 755` forces the exact permission set, potentially granting read access to others when you didn't intend to.

**Q: What is the SUID bit?**
**A:** Set User ID. When a file with SUID (`chmod u+s file`) is executed, it runs with the permissions of the file **owner**, not the user running it. (e.g., `passwd` runs as root so you can update `/etc/shadow`).

**Q: How do you ensure newly created files have specific permissions?**
**A:** Use **umask**. The default permissions (usually 666 for files) are subtracted by the umask value (e.g., 022) to get the final permission (644).
