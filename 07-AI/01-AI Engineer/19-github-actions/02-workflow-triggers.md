## Summary
Workflow triggers (`on`) define the events that cause a workflow to run. For AI Engineers, triggers are essential for automating model training on data updates, running tests on PRs, or manually triggering deployments via the UI.

## Detailed Explanation
### **Common Triggers**
- **`push`**: Runs when code is pushed to specific branches or tags.
- **`pull_request`**: Runs when a PR is opened, updated, or synchronized.
- **`workflow_dispatch`**: Allows manual triggering from the GitHub UI with optional inputs.
- **`repository_dispatch`**: Allows triggering via external API calls (useful for data pipeline integrations).

### **Filtering Triggers**
You can limit triggers to specific branches, tags, or paths:
```yaml
on:
  push:
    branches: [main]
    paths: ['models/**']
```
*AI Use Case*: Trigger a re-training workflow only if files in the `data/` or `models/` directory change.

### **Manual Triggers with Inputs**
```yaml
on:
  workflow_dispatch:
    inputs:
      epochs:
        description: 'Number of training epochs'
        required: true
        default: '10'
```

## Interview Questions
- **Q: How do you trigger a workflow manually from the GitHub UI?**
- **A:** By using the `workflow_dispatch` trigger in the YAML file.

- **Q: What is `repository_dispatch` used for?**
- **A:** It is used to trigger workflows via the GitHub API from external systems, such as an external data warehouse or a separate CI tool.

- **Q: How can you prevent a workflow from running when only README files are updated?**
- **A:** Use `paths-ignore: ['README.md']` under the `push` or `pull_request` trigger.
