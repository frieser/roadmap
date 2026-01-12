# GitHub Codespaces

## Summary
Codespaces provides a complete, configurable development environment in the cloud. It spins up a VS Code instance in the browser (or desktop) connected to a container running your repo.

## Detailed Explanation

### Configuration (`devcontainer.json`)
You define the environment in code.
*   **Image**: `mcr.microsoft.com/devcontainers/go:1-1.21-bullseye`
*   **Extensions**: List VS Code extensions to auto-install (e.g., Go, generic-highlighter).
*   **Port Forwarding**: Expose port 8080 to the internet so you can preview your app.

### Go-specific Context
This is amazing for onboarding. A new Go developer just clicks "Open in Codespaces" and they have Go, `gopls`, `dlv`, and all dependencies pre-installed. No "it works on my machine" issues.

## Interview Questions
**Q: Are Codespaces persistent?**
**A:** Yes, your uncommitted changes are saved if you disconnect. However, they stop running (suspend) after inactivity to save costs.
