# Managing Submodules

## Summary
Working with submodules requires a specific set of commands to ensure they are in sync with the parent repository.

## Detailed Explanation

### Adding
```bash
git submodule add https://github.com/example/lib.git
```

### Updating
When you pull the parent repo, submodules are not automatically updated.
```bash
git submodule update --init --recursive
```
*   `--init`: Initialize if new.
*   `--recursive`: Update nested submodules.

### Updating the pointer
To upgrade a submodule to a newer version:
1.  `cd` into submodule.
2.  `git pull origin main`.
3.  `cd ..` (back to parent).
4.  `git add submodule_folder` (records the new SHA).
5.  `git commit`.

## Interview Questions
**Q: What is the `.gitmodules` file?**
**A:** A text file that maps the submodule path to its remote URL.

**Q: Why does `git status` show "dirty" for a submodule?**
**A:** It means there are uncommitted changes or untracked files inside the submodule directory.
