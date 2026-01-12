# Managing Tags

## Summary
Tags are used to mark specific points in history as important, usually for releases (`v1.0.0`). They are like branches that don't move.

## Detailed Explanation

### Lightweight Tags
Just a pointer to a commit.
```bash
git tag v1.0-beta
```

### Annotated Tags
Stored as full objects in the Git database. They contain a tagger name, email, date, and tagging message.
```bash
git tag -a v1.0 -m "Release version 1.0"
```
**Best Practice**: Always use annotated tags for public releases.

### Go-specific Context
**Crucial**: Go modules rely entirely on semantic version tags.
*   `v1.0.0`, `v1.2.3`: Standard releases.
*   `v0.0.0-20230101000000-abcdef`: Pseudo-versions (untagged commits).
If you don't tag your releases, `go get` has to rely on pseudo-versions, which is messy.

## Interview Questions
**Q: How do you rename a tag?**
**A:** You can't really "rename" it. You have to create a new tag alias and delete the old one.

**Q: How do you list tags?**
**A:** `git tag`.
