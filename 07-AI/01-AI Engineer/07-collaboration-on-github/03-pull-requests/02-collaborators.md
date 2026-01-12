## Summary
**Collaborators** are individuals who have been given direct write access to a repository. Unlike the fork-and-pull model, collaborators can push directly to branches and manage issues/PRs within the repository.

## Detailed Explanation
In a team of AI Engineers, most internal project work is done using the collaborator model for efficiency.

### **Permissions**
- **Read**: View code and issues.
- **Triage**: Manage issues and labels (no write access to code).
- **Write**: Push to branches, merge PRs, edit files.
- **Maintain**: Manage repository settings, secrets, and branch protection.
- **Admin**: Full control, including deleting the repo.

### **AI Team Dynamics**
- **Shared Ownership**: Collaborators can collaborate on the same branch for pair programming on a model architecture.
- **Protected Branches**: Even for collaborators, it's best practice to protect the `main` branch, requiring a PR and passing CI tests (like unit tests for data loaders) before merging.
- **Access Management**: As a lead AI Engineer, you might grant "Write" access to researchers and "Maintain" access to MLOps engineers who manage CI/CD pipelines.

## Interview Questions
**Q: How do you add a collaborator to a private AI project on GitHub?**
**A:** Go to the repository **Settings** -> **Collaborators** -> **Add people**. You search by username or email and select the appropriate permission level.

**Q: What is the risk of giving "Write" access to many collaborators without branch protection?**
**A:** Without protection, someone could accidentally push broken code or (worse) large, sensitive datasets directly to the main branch, bypassing reviews and automated checks.
