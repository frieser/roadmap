---
tags: ['tools', 'roadmap']
---

## Summary
Terraform **parallelism** refers to the engine's ability to perform multiple operations simultaneously by walking the **Dependency Graph**. By default, Terraform processes up to **10 resources** at once. Mastering parallelism is key to optimizing deployment speed for large-scale infrastructure while avoiding provider-side **API rate limiting** (throttling).

## Detailed Explanation

### 1. The Dependency Graph (DAG)
Terraform builds a **Directed Acyclic Graph (DAG)** of all resources in your configuration:
- **Implicit Dependencies:** Automatically detected via resource attributes (e.g., `subnet_id = aws_subnet.main.id`).
- **Explicit Dependencies:** Defined using the `depends_on` meta-argument.
- **Parallel Walk:** Terraform identifies resources that have no unmet dependencies and executes them concurrently.

### 2. The `-parallelism=n` Flag
You can control the maximum number of concurrent operations using the `-parallelism` flag during `plan`, `apply`, or `destroy`.
- **Default:** 10
- **Setting:** `terraform apply -parallelism=30`

### 3. Tuning Parallelism
#### When to Increase (> 10)
- **Massive Independence:** You are creating hundreds of independent resources (e.g., S3 buckets, IAM users, DNS records).
- **Fast APIs:** The cloud provider's API is highly performant and has high rate limits.

#### When to Decrease (< 10)
- **API Throttling:** You encounter errors like `RequestLimitExceeded` (AWS) or `429 Too Many Requests`.
- **Resource Contention:** When many resources are competing for the same limited pool (e.g., multiple databases trying to join the same security group simultaneously).
- **Network/CPU Limits:** On local machines or small CI runners with limited resources.

### 4. API Rate Limiting (Throttling)
Cloud providers limit the number of API calls you can make in a given timeframe.
- **Symptom:** Terraform logs show retries with exponential backoff or hard failures due to "Rate Limit Exceeded".
- **Solution:** Lower the parallelism (`-parallelism=2` or `5`) to reduce the burst of requests and stay within provider quotas.

## Interview Questions

**Q: How does Terraform decide which resources to create at the same time?**
**A:** It builds a dependency graph (DAG) and identifies resources that don't depend on any others (or whose dependencies have already been met). These independent resources are then processed in parallel up to the limit set by the `-parallelism` flag.

**Q: What is the default parallelism value in Terraform, and how do you change it?**
**A:** The default is **10**. It can be changed by passing the `-parallelism=n` flag to commands like `terraform plan` or `terraform apply`.

**Q: You are seeing "Rate Limit Exceeded" errors from AWS while running an apply. What is your first troubleshooting step?**
**A:** Decrease the parallelism (e.g., `terraform apply -parallelism=3`) to reduce the frequency of API calls. If the issue persists, consider splitting the state file to reduce the total number of resources managed in a single run.

**Q: Can you set the parallelism value inside the Terraform HCL code?**
**A:** No. Parallelism is a CLI-level setting and must be passed as a flag during command execution. It cannot be defined in `.tf` files.
