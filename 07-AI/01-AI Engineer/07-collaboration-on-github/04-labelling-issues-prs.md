## Summary
**Labels** are metadata tags used to categorize and filter Issues and Pull Requests. In AI engineering, they are vital for distinguishing between software bugs, model performance issues, and dataset requests.

## Detailed Explanation
### **Standard Labels vs AI Labels**
While standard labels like `bug`, `enhancement`, and `documentation` are common, AI teams often use custom labels:
- **Model Labels**: `model-bug`, `accuracy-regression`, `quantization`, `fine-tuning`.
- **Data Labels**: `dataset-request`, `missing-labels`, `data-pipeline`.
- **Infrastructure**: `cuda-error`, `oom-issue`, `multi-gpu`, `tpu`.
- **Urgency**: `p0-blocker`, `p1-high`, `low-priority`.

### **Benefits**
- **Filtering**: Easily find all issues related to "CUDA" when debugging environment setup.
- **Automation**: Use GitHub Actions to assign reviewers based on labels (e.g., tag a math expert if `model-architecture` label is added).
- **Reporting**: Generate metrics on how many `performance-bug` issues were closed in a sprint.

## Interview Questions
**Q: How can labels improve the workflow of a large AI research team?**
**A:** Labels allow for specialized triage. A data scientist might filter for `dataset` labels, while a software engineer focuses on `infrastructure`. This ensures that the right experts are looking at the right problems.

**Q: What is the "Help Wanted" or "Good First Issue" label used for?**
**A:** These are used to attract new contributors. In AI repos, this might be a request to add docstrings to a model class or implement a simple utility function.
