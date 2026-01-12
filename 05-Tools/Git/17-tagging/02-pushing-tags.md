# Pushing Tags

## Summary
By default, `git push` does **not** transfer tags to remote servers. You must explicitly push them.

## Detailed Explanation

### Usage
```bash
# Push a single tag
git push origin v1.0.0

# Push all tags
git push origin --tags
```

### Deleting Remote Tags
```bash
git push --delete origin v1.0.0
```

### Go-specific Context
When you release a Go module:
1.  Commit changes.
2.  `git tag v1.0.0`.
3.  `git push origin v1.0.0`.
4.  The Go proxy (proxy.golang.org) will eventually see this tag and cache it, making `go get mymod@v1.0.0` available to the world.

## Interview Questions
**Q: Why doesn't `git push` include tags?**
**A:** Because you might create many local temporary tags that you don't intend to share. Git forces you to be explicit about publishing a release.
