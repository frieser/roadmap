#API
---
---

# Disabling Directory Listings in Go

## Summary
Directory Listing occurs when a web server displays the file contents of a directory instead of a specific page (like `index.html`). This is a security risk known as **Information Leakage**, as it exposes file structures, backup files, configuration scripts, and other sensitive assets to attackers.

By default, Go's `http.FileServer` enables directory listing if no `index.html` file is found. Disabling this behavior requires creating a custom `http.FileSystem` wrapper.

## Detailed Explanation

### The Problem: `http.FileServer` Default Behavior
When you serve static files in Go:
```go
fs := http.FileServer(http.Dir("./static"))
http.Handle("/", fs)
```
If a user visits `http://localhost:8080/images/` and that folder lacks an `index.html`, Go generates an HTML page listing all files in that folder.

### The Solution: Custom `FileSystem` Wrapper
To disable this, we need to intercept the file opening process. We check if the requested path is a directory and, if so, verify if it contains an index file. If not, we return a `404 Not Found` or `403 Forbidden` instead of the listing.

#### Go Implementation

```go
package main

import (
	"net/http"
	"os"
	"path/filepath"
)

// neutronFileSystem is a wrapper around http.FileSystem to disable directory listings
type neutronFileSystem struct {
	fs http.FileSystem
}

func (nfs neutronFileSystem) Open(path string) (http.File, error) {
	f, err := nfs.fs.Open(path)
	if err != nil {
		return nil, err
	}

	s, err := f.Stat()
	if err != nil {
		return nil, err
	}

	// If it's a directory, check for index.html
	if s.IsDir() {
		index := filepath.Join(path, "index.html")
		if _, err := nfs.fs.Open(index); err != nil {
			closeErr := f.Close()
			if closeErr != nil {
				return nil, closeErr
			}
			// Return path error to simulate 404 or 403
			return nil, os.ErrNotExist
		}
	}

	return f, nil
}

func main() {
	// Original FileSystem
	dir := http.Dir("./static")
	
	// Wrap it
	wrappedFS := neutronFileSystem{fs: dir}
	
	// Serve using the wrapper
	fs := http.FileServer(wrappedFS)
	http.Handle("/", fs)
	
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

1.  **Why is Directory Listing considered a security vulnerability?**
    *   *Answer:* It reveals the server's directory structure and can expose sensitive files (like `.env`, `.git`, or backup files ending in `.bak`) that were not intended to be public, facilitating further attacks.

2.  **How does Go's `http.FileServer` decide when to show a directory listing?**
    *   *Answer:* It checks if the requested path corresponds to a directory. If it is a directory, it looks for an `index.html`. If found, it serves that file. If not found, it generates an HTML list of the directory's contents.

3.  **What error should you return when directory listing is disabled?**
    *   *Answer:* Usually `404 Not Found` or `403 Forbidden`. Returning `404` is often preferred for security ("Security by Obscurity") because it doesn't reveal that the directory exists at all, whereas `403` confirms existence but denies access.

4.  **Can you disable directory listing using Nginx or Apache in front of Go?**
    *   *Answer:* Yes, and it's often the preferred method in production. For Nginx, you use `autoindex off;`. However, if the Go app is exposed directly (e.g., in a containerized environment), the application itself must handle the security.
