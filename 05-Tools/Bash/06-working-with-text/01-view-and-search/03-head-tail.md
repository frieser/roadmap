---
---

## Summary
`head` and `tail` are essential utilities for peeking at the beginning or end of files. They are commonly used to inspect log files, headers of CSVs, or check the output of a long-running process without opening the full file.

## Detailed Explanation

### `head`
Outputs the *first* part of files.
*   `head file.txt`: First 10 lines (default).
*   `head -n 5`: First 5 lines.
*   `head -c 100`: First 100 bytes.

### `tail`
Outputs the *last* part of files.
*   `tail file.txt`: Last 10 lines.
*   `tail -n 20`: Last 20 lines.
*   `tail -f file.log`: **Follow** mode. Keeps the stream open and prints new lines as they are appended. Critical for monitoring logs.

### Combining (The "Middle" Trick)
To get lines 10-15:
`head -n 15 file.txt | tail -n 5`
(Take first 15, then take the last 5 of *that* result).

## Go-Specific Context/Examples

Implementing a `tail -f` in Go requires reading to EOF, then waiting/sleeping and trying to read again. The `github.com/hpcloud/tail` library is a robust implementation.

### Example: Simple Tail Follower in Go

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"time"
)

func main() {
	file, _ := os.Open("app.log")
	defer file.Close()

	// Seek to end
	file.Seek(0, io.SeekEnd)
	reader := bufio.NewReader(file)

	for {
		line, err := reader.ReadString('\n')
		if err != nil {
			if err == io.EOF {
				time.Sleep(500 * time.Millisecond) // Wait for data
				continue
			}
			break
		}
		fmt.Print(line)
	}
}
```

## Interview Questions

**Q: How does `tail -f` detect new lines?**
**A:** It monitors the file descriptor. When it reaches EOF, it sleeps for a short interval (e.g., 1s) or uses `inotify` (file system events) to wake up when the file size changes, then reads the new bytes.

**Q: What happens if you `tail -f` a file and it gets rotated (deleted and recreated)?**
**A:** Standard `tail -f` follows the *file descriptor* (inode). If the file is renamed/rotated, it continues reading the old file (which might stop receiving data). Use `tail -F` (capital F) to follow by *name*, which handles rotation by retrying to open the filename if the descriptor becomes stale.

**Q: How do you extract the header of a CSV file?**
**A:** `head -n 1 data.csv`.
