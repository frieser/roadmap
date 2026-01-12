## Summary
Contexts are a way to access information about workflow runs, variables, runner environments, jobs, and steps. Each context is an object that contains properties, which can be accessed using expressions like `${{ context.property }}`.

## Detailed Explanation
### **Key Contexts**
- **`github`**: Information about the workflow run and the event that triggered it (e.g., `github.actor`, `github.repository`, `github.event_name`).
- **`env`**: Contains environment variables set in the workflow, job, or step.
- **`vars`**: Contains custom configuration variables defined at the repository or environment level.
- **`secrets`**: Contains sensitive data (e.g., API keys, passwords).
- **`steps`**: Information about the steps that have already run in the current job.
- **`runner`**: Information about the runner executing the job (e.g., `runner.os`, `runner.temp`).

### **Using Contexts in Expressions**
```yaml
steps:
  - name: Print actor
    run: echo "Triggered by ${{ github.actor }}"
  - name: Conditional execution
    if: ${{ github.event_name == 'push' }}
    run: echo "This was a push event"
```

## Interview Questions
- **Q: How do you access a secret named `API_KEY` in a workflow?**
- **A:** Use the expression `${{ secrets.API_KEY }}`.

- **Q: What context would you use to find the name of the branch that triggered the workflow?**
- **A:** The `github.ref_name` or `github.head_ref` (for PRs) properties.

- **Q: Is the `${{ }}` syntax required in an `if` conditional?**
- **A:** No, GitHub automatically evaluates the content of an `if` as an expression.
