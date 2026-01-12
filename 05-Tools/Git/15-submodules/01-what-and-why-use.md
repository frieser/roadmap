# Git Submodules

## Summary
Submodules allow you to keep a Git repository as a subdirectory of another Git repository. This lets you clone another project into your project and keep your commits separate.

## Detailed Explanation

### How it works
The parent repo doesn't track the *files* of the submodule. It only tracks a **pointer** (SHA-1 hash) to a specific commit of the submodule.
This is stored in a `.gitmodules` file and the tree object.

### Pros & Cons
*   **Pros**: Share code between projects without a package registry.
*   **Cons**: Complex to manage. Users often forget to initialize or update them.

### Go-specific Context
Go Modules (`go.mod`) have largely replaced the need for submodules for dependency management in Go. However, submodules are still used for non-Go dependencies (like C libraries) or shared test fixtures.

## Interview Questions
**Q: Does `git clone` download submodules automatically?**
**A:** No, you must use `git clone --recurse-submodules` or run `git submodule update --init` afterwards.
