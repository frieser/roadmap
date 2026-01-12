# post-update Hook

## Summary
The `post-update` hook is a server-side hook that runs after one or more refs have been updated on the remote.

## Detailed Explanation
- **When it runs:** After a push is successfully processed.

### AI Engineering Use Case
- **CI/CD Trigger:** Trigger a Jenkins, GitHub Action, or specialized ML pipeline (like Kubeflow or Airflow) to start a training run or model evaluation.
- **Documentation Updates:** Trigger a rebuild of documentation (e.g., Sphinx or MkDocs) when code is pushed to the `main` branch.

## Interview Questions
1.  **What is the difference between `post-receive` and `post-update`?**
    `post-receive` receives the list of updated refs on `stdin`, while `post-update` receives them as command-line arguments. `post-receive` is generally more modern and preferred.
