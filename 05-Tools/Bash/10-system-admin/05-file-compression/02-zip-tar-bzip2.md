---
---

## Summary
`tar` (Tape Archiver) is primarily an **archiving** utility, not a compressor. It combines multiple files into a single file (tarball). However, it usually invokes compressors (gzip, bzip2) to reduce the size of the archive. `zip` is both an archiver and compressor, common in Windows. `bzip2` is an alternative compressor (slower but smaller files than gzip).

## Detailed Explanation

### Tar Flags (The classic `cvzf`)
*   **`c`**: Create archive.
*   **`x`**: Extract archive.
*   **`v`**: Verbose (list files).
*   **`f`**: File name (MUST come last before the filename).
*   **`z`**: Use **gzip** compression (`.tar.gz`).
*   **`j`**: Use **bzip2** compression (`.tar.bz2`).

### Examples
*   Create: `tar -cvzf archive.tar.gz /path/to/folder`
*   Extract: `tar -xvzf archive.tar.gz`

### Zip / Unzip
*   `zip -r archive.zip folder/`
*   `unzip archive.zip`

## Go-Specific Context/Examples

Go has `archive/tar` and `archive/zip` in the stdlib. Creating a tarball in Go is more manual than `tar` command (you must iterate files and write headers).

### Example: Reading a Zip in Go
```go
package main

import (
	"archive/zip"
	"fmt"
	"log"
)

func main() {
	r, err := zip.OpenReader("archive.zip")
	if err != nil {
		log.Fatal(err)
	}
	defer r.Close()

	for _, f := range r.File {
		fmt.Printf("File: %s\n", f.Name)
	}
}
```

## Interview Questions

**Q: Why use `.tar.gz` instead of `.zip` on Linux?**
**A:** `tar` preserves Unix file permissions (owner, group, rwx bits) and symbolic links accurately. `zip` (originating from DOS) does not always handle these POSIX attributes correctly or portably.

**Q: What is the difference between `gzip` and `bzip2`?**
**A:** `bzip2` uses the Burrows-Wheeler algorithm, which generally compresses better (smaller files) than `gzip` (DEFLATE), but is significantly slower to compress and decompress. `xz` is the modern successor to bzip2.

**Q: How do you extract to a specific directory?**
**A:** `tar -xvf file.tar.gz -C /target/directory`.
