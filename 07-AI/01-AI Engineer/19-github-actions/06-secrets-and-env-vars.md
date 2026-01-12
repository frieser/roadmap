## Summary
Managing sensitive information (secrets) and configuration (environment variables) is critical for secure and portable AI workflows. Secrets are encrypted and never shown in logs, while environment variables are used for non-sensitive settings.

## Detailed Explanation
### **Environment Variables (`env`)**
Used for configuration that isn't sensitive.
- **Scope**: Can be defined at the workflow, job, or step level.
- **Predefined Variables**: GitHub provides defaults like `GITHUB_WORKSPACE` and `GITHUB_SHA`.

### **Secrets (`secrets`)**
Used for API keys (OpenAI, Hugging Face), database credentials, or cloud access tokens.
- **Storage**: Managed in GitHub Repository Settings -> Secrets and variables -> Actions.
- **Masking**: GitHub automatically masks secrets printed to the console with `***`.

### **AI Context: Storing Model Hub Tokens**
```yaml
steps:
  - name: Login to Hugging Face
    run: huggingface-cli login --token ${{ secrets.HF_TOKEN }}
    env:
      HF_HOME: ${{ github.workspace }}/.cache/huggingface
```

## Interview Questions
- **Q: What is the difference between a Secret and a Variable in GitHub Actions?**
- **A:** Secrets are encrypted and masked in logs, intended for sensitive data. Variables are stored as plaintext and are intended for non-sensitive configuration data.

- **Q: How can you set an environment variable that is available to all jobs in a workflow?**
- **A:** Define the `env` block at the top level of the YAML file, outside of the `jobs` block.

- **Q: Can you see the value of a secret after it has been created in GitHub Settings?**
- **A:** No, you can only update or delete it. The value is hidden once saved.
