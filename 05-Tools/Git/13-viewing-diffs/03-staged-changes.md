# Staged Changes

## Summary
Staged changes are modifications that you have added to the index (`git add`) and are ready to be committed. `git diff` does not show these by default.

## Detailed Explanation

### Usage
```bash
git diff --staged
# or
git diff --cached
```

### Workflow
1.  Edit files.
2.  `git add .`
3.  `git diff --staged` (Final Review).
4.  `git commit`.

### Go-specific Context
This is the perfect time to catch debug print statements (`fmt.Println`) or accidental inclusion of secrets before they get baked into a commit.

## Interview Questions
**Q: If I run `git diff` after `git add .`, what will I see?**
**A:** Nothing, because `git diff` only shows unstaged changes. You must use `--staged`.
