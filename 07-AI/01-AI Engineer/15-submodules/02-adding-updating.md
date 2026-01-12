# Adding and Updating Submodules

## Summary
Managing submodules involves specific commands for initialization, fetching, and updating. Because submodules are effectively separate repositories, their state must be managed explicitly alongside the parent repository.

## Detailed Explanation

### Adding a Submodule
To add a new external repo:
```bash
git submodule add https://github.com/owner/repo path/to/subdir
```
This creates the `.gitmodules` file and clones the repo into the specified path.

### Cloning a Repo with Submodules
If you clone a repo that already has submodules, they will be empty by default. You must run:
```bash
git clone --recursive https://github.com/owner/main-repo
# OR if already cloned:
git submodule update --init --recursive
```

### Updating a Submodule
If someone else updates the submodule commit in the main repo, you need to sync:
```bash
git submodule update --remote --merge
```
To update to a new version of the submodule yourself:
1.  Go into the submodule directory: `cd path/to/subdir`
2.  Pull the latest: `git pull origin main`
3.  Go back to main repo: `cd ..`
4.  Commit the change: `git commit -am "Update submodule to latest version"`

### AI Engineering Workflow
1.  **Reproducibility:** When you release a model, ensure your submodules are pinned to the exact commits used during training.
2.  **Development:** If you find a bug in the shared data-loader submodule while training your model, you can fix it inside the submodule directory, commit/push there, and then update the main repo's reference.

## Interview Questions
1.  **How do you initialize submodules in a freshly cloned repository?**
    `git submodule update --init --recursive`
2.  **What happens if you change code inside a submodule but forget to commit/push there before committing the main repo?**
    The main repo will point to a commit hash that doesn't exist on the submodule's remote server. Others who pull your main repo will encounter errors when trying to update the submodule.
3.  **How do you remove a submodule?**
    Modern Git (`>1.8.5`):
    1. `git rm path/to/submodule`
    2. Remove relevant sections from `.git/config` and `.gitmodules` (usually handled by `git rm`).
    3. Remove the `.git/modules/path/to/submodule` directory.
