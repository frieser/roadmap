## Summary
Artifacts allow you to share data between jobs in a workflow or store data after a workflow completes. For AI Engineers, common artifacts include trained model weights (.pth, .onnx), evaluation plots, and detailed logs.

## Detailed Explanation
### **Uploading Artifacts**
Use `actions/upload-artifact` to save files.
```yaml
- name: Upload trained model
  uses: actions/upload-artifact@v4
  with:
    name: trained-llm
    path: outputs/model.bin
```

### **Downloading Artifacts**
Use `actions/download-artifact` in a subsequent job or to download locally.
```yaml
- name: Download model for evaluation
  uses: actions/download-artifact@v4
  with:
    name: trained-llm
```

### **Retention Policy**
Artifacts are stored by GitHub for a default of 90 days (configurable). Unlike the cache, artifacts are intended to be persistent outputs of a run rather than temporary build dependencies.

## Interview Questions
- **Q: What is the difference between Cache and Artifacts?**
- **A:** Cache is used to speed up the build by reusing dependencies between runs. Artifacts are used to save files produced by a workflow (like a build binary or a model file) for long-term storage or use in other jobs.

- **Q: Can one job access an artifact created by another job in the same workflow run?**
- **A:** Yes, by using the `actions/download-artifact` action in the second job.

- **Q: How can you specify how long an artifact should be kept?**
- **A:** Using the `retention-days` parameter in the `upload-artifact` action.
