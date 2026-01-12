# Creating a GitHub Account

## Summary
GitHub is the largest host for Git repositories. Creating an account is free and gives you access to public/private repositories, issues, pull requests, and the wider open-source community.

## Detailed Explanation

### Steps
1.  Go to [github.com](https://github.com).
2.  Click "Sign Up".
3.  Choose a unique **Username**. This will be part of your URL (`github.com/username`), so choose wisely (professional).
4.  Enter **Email** and **Password**.
5.  Solve the puzzle (captcha) and verify email.

### Security (Critical)
*   **Enable 2FA (Two-Factor Authentication)** immediately. GitHub requires this for many contributors now.
*   **SSH Keys**: After account creation, generate an SSH key locally (`ssh-keygen`) and upload the public key to GitHub Settings -> SSH and GPG keys. This allows password-less `git push`.

### Go-specific Context
Your GitHub username is often part of your Go module path:
```go
module github.com/username/project
```
Changing your username later can break `go get` for anyone using your library, as the import path will change.

## Interview Questions
**Q: Can I have multiple GitHub accounts?**
**A:** It is against GitHub's ToS to maintain multiple free accounts to evade controls, but you can have separate personal and work accounts (though usually one account is recommended for attribution).

**Q: What is the benefit of verifying your email?**
**A:** You cannot commit with a verified "Verified" badge or recover your account effectively without it.
