## Summary
GitHub Actions workflows are defined using YAML. A workflow is an automated process consisting of one or more jobs, triggered by specific events. Understanding the basic syntax—`name`, `on`, `jobs`, and `steps`—is fundamental for building CI/CD pipelines in AI projects.

## Detailed Explanation
### **Core Components**
- **`name`**: (Optional) The name of the workflow as it appears in the Actions tab.
- **`on`**: The event that triggers the workflow (e.g., `push`, `pull_request`).
- **`jobs`**: A collection of tasks that run on the same runner. Jobs run in parallel by default.
- **`steps`**: A sequence of tasks within a job. Steps can run commands or use actions.

### **Anatomy of a Basic Workflow**
```yaml
name: AI Model CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Run tests
        run: python -m pytest
```

### **YAML Best Practices**
- **Indentation**: Use 2 spaces; never use tabs.
- **Quotes**: Quotes are optional for strings unless they contain special characters.
- **Lists**: Use a hyphen followed by a space for list items.

## Interview Questions
- **Q: Where must workflow files be stored in a repository?**
- **A:** In the `.github/workflows` directory.

- **Q: What is the difference between a `job` and a `step`?**
- **A:** A job is a group of steps that execute on the same runner. Jobs run in parallel unless dependencies are specified, while steps run sequentially within a job.

- **Q: How do you make one job depend on the completion of another?**
- **A:** Use the `needs` keyword in the dependent job definition.
