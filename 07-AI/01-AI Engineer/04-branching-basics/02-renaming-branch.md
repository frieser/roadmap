---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Renaming branches is a common administrative task used to improve project clarity or correct typos. As an AI project evolves, an experimental branch name like `test-1` might need to be renamed to something more descriptive like `feature-multi-head-attention` to help teammates understand the intent. Git provides simple commands to rename both local and remote branches.

## Detailed Explanation

### Renaming Locally
The command depends on whether you are currently on the branch you want to rename.

```bash
# 1. Rename the branch you are currently on
git branch -m new-descriptive-name

# 2. Rename a different branch while on 'main'
git branch -m old-name new-name
```

### Renaming Remotely
Renaming a branch on GitHub or Hugging Face is a two-step process because you cannot technically "rename" a remote branch; you must delete the old one and push the new one.

```bash
# 1. Rename the local branch
git branch -m old-name new-name

# 2. Delete the old branch on the remote
git push origin --delete old-name

# 3. Push the new branch and set it as upstream
git push origin -u new-name
```

### Best Practices for AI Branch Naming
Consistent naming conventions help teams navigate large repositories:
*   `experiment/` for ML experiments: `experiment/bert-finetuning`
*   `feature/` for application code: `feature/api-endpoint`
*   `fix/` for bug fixes: `fix/dataloader-leak`
*   `data/` for data-centric changes: `data/cleaning-script-update`

## Interview Questions

**Q: How do you rename your current local branch?**
**A:** Run `git branch -m <new_name>`. The `-m` stands for "move" (similar to the Linux `mv` command).

**Q: Can you rename a branch that has already been pushed to a remote?**
**A:** Yes, but it requires extra steps. You rename the local branch, delete the old remote branch, and then push the new local branch to the remote. You should also notify your teammates, as their local tracking branches will now be pointing to a non-existent remote branch.

**Q: Why is it important to use descriptive branch names in AI projects?**
**A:** AI projects often involve dozens of experiments. Vague names like `test1` or `update` make it impossible for collaborators (or your future self) to know what was being tested without checking the code. Descriptive names like `exp/resnet-dilation-rate-4` provide immediate context.

**Q: What happens to the commits on a branch when you rename it?**
**A:** Nothing happens to the commits. Renaming a branch only changes the name of the pointer that refers to the latest commit in that series. The history and content remain identical.
