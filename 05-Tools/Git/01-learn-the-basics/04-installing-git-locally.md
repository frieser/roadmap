# Installing Git Locally

## Summary
Installing Git is the first step to version control. The process varies by operating system: Linux users typically use a package manager (apt, yum), macOS users use the Xcode command line tools or Homebrew, and Windows users often use the Git for Windows installer which includes Git Bash.

## Detailed Explanation

### Linux
For Debian/Ubuntu-based distributions, use `apt`:
```bash
sudo apt update
sudo apt install git
```

For RHEL/CentOS/Fedora, use `dnf` or `yum`:
```bash
sudo dnf install git
```

### macOS
The easiest way is to install the **Xcode Command Line Tools**. On Mavericks (10.9) or above, simply run `git` from the Terminal the very first time.
```bash
git --version
```
If not installed, it will prompt you to install it. Alternatively, if you use **Homebrew**:
```bash
brew install git
```

### Windows
Download the official installer from [git-scm.com](https://git-scm.com/download/win).
*   This installs **Git Bash**, a terminal emulator that provides a bash-like environment on Windows, which is highly recommended over using the standard Command Prompt.

### First-Time Configuration
After installing, you **must** configure your identity. Git embeds this information into every commit you make.

```bash
# Set your name
git config --global user.name "John Doe"

# Set your email (should match your GitHub email)
git config --global user.email "johndoe@example.com"

# Verify configuration
git config --list
```

### Go-specific Context
You cannot effectively develop Go without Git installed.
*   **Dependency Fetching**: The `go` command relies on the system's `git` binary to download module source code. If Git is not in your system `$PATH`, `go get` or `go mod tidy` will fail.
*   **Private Modules**: If you are working with private Go modules (e.g., in a corporate environment), you need to configure Git to authenticate (often via SSH or a credential helper) so the Go toolchain can access those repos.

```bash
# Example error if Git is not installed or configured correctly when running Go:
# go: missing git command. See https://golang.org/s/gogetcmd
```

## Interview Questions
**Q: What is the difference between `git config --global`, `--system`, and `--local`?**
**A:**
*   `--local` (default): Applies only to the current repository (`.git/config`).
*   `--global`: Applies to the current user (stored in `~/.gitconfig`).
*   `--system`: Applies to all users on the system (stored in `/etc/gitconfig`).

**Q: How do you check which version of Git is installed?**
**A:** Run `git --version` in your terminal.

**Q: Why do we need to configure `user.name` and `user.email`?**
**A:** Every Git commit contains the author's name and email address. This metadata is immutable once baked into the commit and is essential for tracking who made which changes in the project history.
