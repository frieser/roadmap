---
tags: ['ai', 'roadmap']
---

## Summary
Installing Git is the first step in setting up a professional AI engineering environment. It is available across all major operating systems (Linux, macOS, and Windows) and can be managed via command-line package managers or standalone installers. Once installed, a few basic configuration steps are required to identify your work, ensuring that your contributions to AI projects are properly attributed.

## Detailed Explanation

### 1. Installation by Platform

#### Linux (Debian/Ubuntu)
Git is usually available in the default package manager.
```bash
sudo apt update
sudo apt install git -y
```

#### Linux (Fedora/RHEL)
```bash
sudo dnf install git -y
```

#### macOS
*   **Via Homebrew (Recommended):** `brew install git`
*   **Via Xcode:** Simply type `git` in the terminal; if not installed, macOS will prompt you to install the Command Line Tools.

#### Windows
*   **Git for Windows:** Download from [git-scm.com](https://git-scm.com/). It includes **Git Bash**, which provides a Unix-like command-line experience.
*   **Via Winget:** `winget install --id Git.Git -e --source winget`

### 2. Basic Configuration
After installation, you must set your identity. This information is embedded in every commit you make.

```bash
# Set your global username
git config --global user.name "John Doe"

# Set your global email (use the same one as your GitHub/GitLab account)
git config --global user.email "johndoe@example.com"

# Set the default branch name to 'main' (modern standard)
git config --global init.defaultBranch main
```

### 3. Verification
To ensure Git is correctly installed and accessible from your path, run:
```bash
git --version
# Should output something like: git version 2.40.0
```

### 4. Why AI Engineers need the CLI
While there are many GUI clients (GitKraken, Sourcetree, VS Code integration), AI Engineers should be comfortable with the CLI because:
*   **Remote Servers:** Much AI training happens on remote headless Linux servers where GUIs are unavailable.
*   **Automation:** Scripting Git commands (e.g., in a Python deployment script or a Dockerfile) requires CLI knowledge.

## Interview Questions
**Q: How do you install Git on a Ubuntu Linux system?**
**A:** By running `sudo apt update && sudo apt install git`.

**Q: Why do you need to configure `user.name` and `user.email`?**
**A:** Git uses this information to "sign" every commit. It identifies who made which changes, which is critical for collaboration and accountability in a team.

**Q: What is "Git Bash" and why is it useful on Windows?**
**A:** Git Bash is an application for Windows that provides an emulation of the Bash shell. It allows Windows users to run Git commands and shell scripts exactly like they would on Linux or macOS.

**Q: What command do you use to check your current Git configuration?**
**A:** `git config --list` or `git config -l`.

**Q: How can you check which version of Git is installed?**
**A:** By running the command `git --version`.
