## Summary
**Pull Request Guidelines** are a set of rules and best practices defined by a project to ensure code quality, consistency, and efficient review. Following these is essential for getting your AI research or implementation merged into a production codebase.

## Detailed Explanation
AI projects often have specialized guidelines due to the complexity of ML code.

### **Standard Components of an AI PR**
1. **Clear Title**: e.g., "Add support for Flash Attention 2 in Llama architecture".
2. **Context**: Why is this change needed? Link to the relevant Issue.
3. **Implementation Details**: Briefly explain the mathematical or architectural changes.
4. **Validation/Testing**:
    - **Unit Tests**: For new utility functions.
    - **Integration Tests**: Ensuring the model still loads and runs.
    - **Performance Metrics**: Evidence (plots/tables) that the model still converges or runs faster.

### **Reviewer Roles**
- **Code Reviewer**: Checks for code quality, efficiency, and Python best practices.
- **Domain Expert**: Checks the mathematical correctness of the ML algorithm.
- **MLOps Reviewer**: Checks if the change affects deployment, GPU memory usage, or CI/CD speed.

## Interview Questions
**Q: What should you include in an AI-focused Pull Request to help reviewers?**
**A:** Beyond code changes, include: 
1. A summary of the logic.
2. Link to research papers or documentation.
3. Proof of testing (logs/metrics).
4. Details on hardware used for verification (e.g., "Tested on 1x A100").

**Q: How do you handle a PR that is too large to review?**
**A:** Break it into smaller, logical PRs. For example, one PR for the data loader changes, one for the model architecture, and a final one for the training script updates.
