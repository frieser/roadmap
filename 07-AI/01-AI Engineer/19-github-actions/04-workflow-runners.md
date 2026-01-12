## Summary
Runners are the servers that execute the jobs in your GitHub Actions workflows. Understanding the difference between GitHub-hosted and self-hosted runners is crucial for AI Engineers who often require specific hardware (like GPUs) for model training.

## Detailed Explanation
### **GitHub-Hosted Runners**
- **Provided by GitHub**: Fast to set up, secure, and maintained by GitHub.
- **Operating Systems**: Ubuntu, Windows, macOS.
- **Specifications**: Typically 2-core CPU, 7 GB RAM, and 14 GB SSD. (May vary for "Larger Runners").

### **Self-Hosted Runners**
- **Managed by You**: You provide the machine (on-prem, cloud, or edge).
- **Customization**: You can install specific dependencies, hardware (GPUs), and persistent storage.
- **AI Use Case**: Using a self-hosted runner with an NVIDIA A100 for deep learning training that wouldn't fit on standard GitHub runners.

### **Larger Runners**
GitHub offers paid, high-performance runners with more cores, RAM, and specialized features (like GPU support in some regions) for Enterprise accounts.

## Interview Questions
- **Q: When would you choose a self-hosted runner over a GitHub-hosted runner?**
- **A:** When the job requires specialized hardware (like a GPU), specific software not on standard images, or access to resources within a private network.

- **Q: What is the primary security risk of self-hosted runners on public repositories?**
- **A:** Forked repositories can run malicious code on your private infrastructure if not properly configured.

- **Q: How do you specify the OS for a GitHub-hosted runner?**
- **A:** Using the `runs-on` keyword (e.g., `runs-on: ubuntu-latest`).
