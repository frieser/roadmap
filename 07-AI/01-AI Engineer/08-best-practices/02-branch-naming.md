## Summary
**Branch Naming** is the practice of giving descriptive and consistent names to Git branches. A good naming convention helps team members understand the purpose of a branch at a glance and aids in organizing the workflow.

## Detailed Explanation
### **Common Naming Patterns**
- **feature/** or **feat/**: New features or research implementations. (`feat/transformer-xl`)
- **bugfix/** or **fix/**: Fixing bugs in the code. (`fix/dataloader-index-error`)
- **hotfix/**: Urgent fixes for production issues.
- **research/**: Experimental branches that might not be merged. (`research/test-new-loss-function`)
- **docs/**: Documentation-only changes. (`docs/update-installation-guide`)

### **Naming Tips for AI Engineers**
1. **Use Kebab-case**: Use hyphens to separate words (`feature/new-model-head`).
2. **Include Ticket IDs**: If using a task tracker like Jira or GitHub Issues, include the ID (`feat/AI-123-add-wandb-logging`).
3. **Be Specific**: Avoid generic names like `update-code` or `fix-bug`. Use `fix/adam-optimizer-convergence` instead.
4. **User Prefixes**: In large teams, you might prefix with your name or initials (`js/feat/vision-transformer`).

## Interview Questions
**Q: Why is a consistent branch naming convention important in a collaborative AI project?**
**A:** It reduces confusion during code reviews and helps in identifying the status of various features. It also allows for automation, where certain branch name patterns trigger specific CI/CD workflows (e.g., `research/*` branches skip the expensive production tests).

**Q: What information should a good branch name convey?**
**A:** It should convey the **type** of change (feature, bugfix, etc.), the **specific component** being modified, and optionally a **task ID** for traceability.
