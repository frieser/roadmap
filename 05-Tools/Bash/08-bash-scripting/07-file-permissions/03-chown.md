---
---

## Summary
`chown` (Change Owner) is used to change the file owner and group. Only the root user can change the owner of a file (in most modern systems). This is critical for security and service management (e.g., ensuring Nginx owns its webroot).

## Detailed Explanation

### Syntax
`chown [options] user[:group] file`

### Examples
*   **Change Owner**: `chown alice file.txt`.
*   **Change Owner and Group**: `chown alice:developers file.txt`.
*   **Change Group Only**: `chown :developers file.txt` (Similar to `chgrp`).
*   **Recursive**: `chown -R www-data:www-data /var/www/html`.

### Preservation of Root
Running `chown` usually strips the SUID/SGID bits for security reasons, unless run by root.

## Go-Specific Context/Examples

In Go, `os.Chown` is used to change ownership. It requires the UID (User ID) and GID (Group ID), not usernames. You must look up IDs first.

### Example: Changing Owner in Go
```go
package main

import (
	"log"
	"os"
	"os/user"
	"strconv"
)

func main() {
	// Lookup User ID
	u, err := user.Lookup("www-data")
	if err != nil {
		log.Fatal(err)
	}
	uid, _ := strconv.Atoi(u.Uid)
	gid, _ := strconv.Atoi(u.Gid)

	// Change Owner
	if err := os.Chown("/var/www/html/index.html", uid, gid); err != nil {
		log.Fatal(err)
	}
}
```

## Interview Questions

**Q: Who can change the owner of a file?**
**A:** Only **root** (Superuser). Even if you own the file, you cannot "give it away" to another user on most Linux systems (this is to prevent users from bypassing disk quotas or hiding malicious files in others' directories).

**Q: What does `chown user:` (with a trailing colon) do?**
**A:** It changes the owner to `user` AND changes the group to that user's login group (usually the same name). It saves typing `chown user:user`.

**Q: Why use `-R` carefully?**
**A:** `chown -R` creates a lot of I/O operations (metadata updates) and can accidentally break system permissions if run on `/` or `/usr` by mistake.
