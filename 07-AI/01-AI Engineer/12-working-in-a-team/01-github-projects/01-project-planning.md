## Summary
**GitHub Projects** is a customizable tool for planning and tracking work. For AI teams, it provides a centralized way to manage research tasks, data engineering efforts, and model deployment cycles alongside the code.

## Detailed Explanation
### **Core Concepts**
- **Items**: Issues, Pull Requests, or draft notes that represent tasks.
- **Views**: Different ways to visualize your data (Table, Board, Roadmap).
- **Fields**: Built-in (Assignee, Status) or custom (Priority, Estimate, Target Date) metadata.

### **AI Project Planning**
AI projects often require tracking non-code tasks. You can use Projects to:
- **Track Data Labeling**: Creating items for each batch of data that needs labeling.
- **Model Training Log**: Using custom fields to track the status of different training runs (e.g., "Training", "Evaluating", "Ready").
- **Resource Management**: Assigning tasks to specific GPU clusters or compute resources.

## Interview Questions
**Q: How does GitHub Projects integrate with Issues and Pull Requests?**
**A:** Projects are built directly from Issues and PRs. When you update an item in a project (like changing its status), it automatically updates the associated Issue or PR, and vice-versa.

**Q: What is a "Custom Field" in GitHub Projects and why is it useful for an AI team?**
**A:** A custom field allows you to add metadata specific to your workflow, like "Model Version", "Accuracy Metric", or "GPU Requirement". This helps in filtering and organizing tasks beyond the standard "To Do/In Progress/Done" status.
