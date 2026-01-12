## Summary
GitHub Issues are a powerful tool for tracking bugs, requesting features, and managing tasks within a repository. For AI Engineers, they serve as a central hub for reporting model performance degradation, dataset inconsistencies, and implementation bugs.

## Detailed Explanation
In AI development, issues are more than just "code bugs." They often involve tracking experimental results and data quality.

### **Types of AI Issues**
- **Implementation Bugs**: Standard software bugs in the training or inference code.
- **Model Performance**: Tracking issues where a model fails to meet specific benchmarks or exhibits bias.
- **Data Issues**: Reporting corrupted data samples, label noise, or missing metadata in the dataset.
- **Feature Requests**: Requesting support for new model architectures, optimizers, or integration with MLOps tools.

### **Issue Templates**
Most professional AI repositories (like `scikit-learn` or `diffusers`) use issue templates. These ensure that contributors provide:
- **Environment Details**: GPU model, CUDA version, library versions.
- **Reproducible Example**: A minimal script to reproduce the failure.
- **Expected vs Actual Results**: Performance metrics or error logs.

### **AI-Specific Workflow**
1. **Detection**: Identifying a drop in validation accuracy after a new merge.
2. **Issue Creation**: Creating an issue with the specific commit hash and experiment logs.
3. **Labels**: Applying labels like `model-bug`, `regression`, or `low-prio`.
4. **Linkage**: Linking the issue to a Pull Request that fixes the problem using keywords like "Closes #123".

## Interview Questions
**Q: How can GitHub Issues be used to manage the ML lifecycle?**
**A:** Issues can be used to track every stage: from data collection bugs and feature engineering ideas to model training failures and deployment bottlenecks. Using labels helps categorize these stages for better project management.

**Q: Why is a "Minimal Reproducible Example" critical in an AI issue?**
**A:** AI environments are complex (drivers, versions, hardware). Without a minimal script that fails in the same way, it's nearly impossible for maintainers to distinguish between a code bug, a hardware issue, or a stochastic training failure.

**Q: What is the benefit of using "Task Lists" within an issue?**
**A:** Task lists (`- [ ]`) allow you to break down a complex task (like "Implement Vision Transformer") into smaller, trackable steps that can be checked off as the work progresses.
