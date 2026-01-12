## Summary
Caching dependencies significantly speeds up workflow execution by reusing files between runs. For AI Engineers, this often involves caching Python virtual environments, pip packages, or even small pre-trained models.

## Detailed Explanation
### **Using `actions/cache`**
The cache action allows you to persist files based on a unique key.
```yaml
- name: Cache pip dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

### **AI-Specific Caching**
- **Hugging Face Hub**: Cache downloaded models to avoid redundant downloads.
  - Path: `~/.cache/huggingface`
- **Datasets**: Cache pre-processed datasets if they fit within the cache limits (10GB per repository).

### **Cache Eviction**
GitHub removes caches that haven't been accessed in 7 days or when the total size exceeds the repository limit.

## Interview Questions
- **Q: What happens if there is a "cache miss"?**
- **A:** The job continues, the files are typically re-downloaded/re-created, and a new cache is saved at the end of the job if it completes successfully.

- **Q: Why is it important to include a hash of the `requirements.txt` file in the cache key?**
- **A:** To ensure that the cache is invalidated and updated whenever the dependencies change.

- **Q: What is the maximum storage limit for caches in a GitHub repository?**
- **A:** 10 GB total for all caches in the repository.
