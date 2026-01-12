# Handling Conflicts

## Summary
Merge conflicts occur when Git cannot automatically resolve differences in code between two commits (usually when the same line is modified differently). User intervention is required to choose the correct version.

## Detailed Explanation

### Recognizing a Conflict
Git stops the merge and says:
`CONFLICT (content): Merge conflict in file.go`.

### Markers
Git inserts markers into the file:
```go
<<<<<<< HEAD
fmt.Println("Local Change")
=======
fmt.Println("Incoming Change")
>>>>>>> branch-name
```

### Resolution Flow
1.  Open the file.
2.  Decide which code to keep (or combine them).
3.  Delete the markers (`<<<`, `===`, `>>>`).
4.  `git add file.go`.
5.  `git commit` (to finalize the merge).

### Go-specific Context
**`go.sum` Conflicts**: These are very common.
*   **Don't** try to manually edit the hashes in `go.sum`.
*   **Do**:
    1.  Delete `go.sum` (or the conflict block).
    2.  Run `go mod tidy`.
    3.  Go will regenerate the correct checksums based on `go.mod`.

## Interview Questions
**Q: How can you see which files have conflicts?**
**A:** `git status` lists them as "both modified".

**Q: What tool can help with conflicts?**
**A:** `git mergetool` opens a GUI (like VS Code, KDiff3) to help visualize 3-way merges.
