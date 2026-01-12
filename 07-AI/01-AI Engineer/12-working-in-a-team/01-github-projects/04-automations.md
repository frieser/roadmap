## Summary
**Project Automations** allow you to automatically update item fields based on events. In an AI project, this reduces manual work and ensures the project board is always up-to-date with the code's status.

## Detailed Explanation
### **Built-in Workflows**
- **Auto-add**: Automatically add new Issues or PRs to a project when they are created in a specific repository.
- **Status Update**: Move an item to "In Progress" when a branch is created, or to "Done" when a PR is merged.
- **Reopen**: Move an item back to "In Progress" if an Issue is reopened.

### **Advanced Automation with GitHub Actions**
AI teams can create custom Actions to:
- **Label by Performance**: Automatically tag a PR with "High-Performance" if it passes a specific benchmark in CI.
- **Assign Experts**: Mention a specific team member if a PR modifies a sensitive part of the model architecture.
- **Clean up**: Archive items that have been in "Done" for more than 30 days.

## Interview Questions
**Q: Give an example of how automation can save time for an AI Engineer.**
**A:** An automation can automatically move a "Model Training" task from "Todo" to "In Progress" the moment a Pull Request is opened with the new training script, and then to "Done" when that PR is merged. This keeps the project board accurate without manual intervention.

**Q: What is the benefit of "Auto-add" workflows in a multi-repository organization?**
**A:** It ensures that every single bug report or feature request across all organization repos is captured in a central "Master Project" for visibility, preventing tasks from falling through the cracks.
