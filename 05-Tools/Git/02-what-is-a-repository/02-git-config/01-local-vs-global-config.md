# Local vs Global Config

## Summary
Git configuration determines how Git looks and operates. These settings can be defined at three levels: System (all users), Global (current user), and Local (current repository). Local settings override Global, which override System.

## Detailed Explanation

### Configuration Levels
1.  **System**: `/etc/gitconfig`. Applies to every user on the system and all their repositories.
    *   `git config --system`
2.  **Global**: `~/.gitconfig` or `~/.config/git/config`. Applies to you (the user) for all your repositories.
    *   `git config --global`
3.  **Local**: `.git/config` in the specific repository. Applies only to that single project.
    *   `git config --local` (default if no flag is passed inside a repo)

### Common Settings
```bash
# Identity
git config --global user.name "Alice"
git config --global user.email "alice@example.com"

# Editor (e.g., for commit messages)
git config --global core.editor "vim"

# Default branch name
git config --global init.defaultBranch main
```

### Viewing Config
```bash
# List all settings
git config --list

# Show origin of a setting
git config --list --show-origin
```

### Go-specific Context
*   **Private Repositories**: Go developers often need to configure Git to use SSH instead of HTTPS for private modules to avoid password prompts during `go get`.

```bash
# Force Git to use SSH for GitHub
git config --global url."git@github.com:".insteadOf "https://github.com/"
```
This is a critical global config for Go developers working in organizations with private dependencies.

## Interview Questions
**Q: How do you override the global username for a specific project?**
**A:** Go into the project directory and run `git config user.name "New Name"`. This writes to `.git/config` which takes precedence over the global config.

**Q: Where is the local configuration stored?**
**A:** In the `.git/config` file inside the repository's root directory.
