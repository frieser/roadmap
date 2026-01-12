---
---

## Summary
`gzip` is the standard file compression utility in Unix/Linux. It uses the **DEFLATE** algorithm (LZ77 + Huffman coding) to reduce file size. Files compressed with gzip typically have the extension `.gz`. It compresses files *in place*, replacing the original.

## Detailed Explanation

### Commands
*   **Compress**: `gzip file.txt` -> Creates `file.txt.gz`, deletes `file.txt`.
*   **Decompress**: `gunzip file.txt.gz` (or `gzip -d`) -> Restores `file.txt`.
*   **View**: `zcat file.txt.gz` (Print content without decompressing).
*   **Keep Original**: `gzip -k file.txt`.

### Compression Levels
*   `-1` (Fastest): Less compression, faster.
*   `-9` (Best): Max compression, slower.
*   `-6`: Default compromise.

## Go-Specific Context/Examples

Go's standard library `compress/gzip` allows reading/writing gzip streams. This is heavily used in web servers (gzipping HTTP responses).

### Example: Gzip Writer in Go
```go
package main

import (
	"compress/gzip"
	"os"
)

func main() {
	output, _ := os.Create("data.txt.gz")
	defer output.Close()

	w := gzip.NewWriter(output)
	w.Write([]byte("Hello Compressed World"))
	w.Close() // Important to flush footer
}
```

## Interview Questions

**Q: Can `gzip` compress a directory?**
**A:** **No**. `gzip` only compresses a *single file*. To compress a directory, you must first archive it into a single file using `tar` (creating a `.tar`), and then gzip that file (creating a `.tar.gz`).

**Q: What is the benefit of `zcat` or `zless`?**
**A:** They allow you to inspect compressed logs (e.g., `access.log.2.gz`) instantly without wasting time and disk space decompressing them to a temporary file.

**Q: Is gzip lossy or lossless?**
**A:** Lossless. The decompressed data is bit-for-bit identical to the original.
