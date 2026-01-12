# GitHub Packages

## Summary
GitHub Packages is a package hosting service. It allows you to host your software packages (Docker images, npm packages, Maven, NuGet, RubyGems) in the same place as your source code.

## Detailed Explanation

### Container Registry (ghcr.io)
The most popular feature. You can push Docker images to `ghcr.io/username/image:tag`.
*   **Auth**: Login with your Personal Access Token.
*   **Visibility**: Can be separate from repo visibility.

### Go-specific Context
Go modules are usually fetched directly from source (VCS), so there isn't a "Go Registry" in GitHub Packages in the same way there is for NPM. However, you often use **GHCR** to publish the *Dockerized* version of your Go application.

## Interview Questions
**Q: Is GitHub Packages free?**
**A:** It has a free tier for storage and transfer, but heavy usage requires billing. Public packages are free.
