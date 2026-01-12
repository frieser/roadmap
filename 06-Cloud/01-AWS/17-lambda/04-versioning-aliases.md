#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'lambda']
---

## Summary
AWS Lambda Versioning and Aliases are essential features for managing the lifecycle of serverless functions. **Versioning** allows you to create immutable snapshots of your code and configuration, while **Aliases** provide stable pointers to those versions. Together, they enable safe deployment patterns like Blue/Green and Canary releases by allowing controlled traffic shifting between different function iterations.

## Detailed Explanation

### 1. $LATEST and Versioning
- **$LATEST**: By default, every Lambda function has a $LATEST version. This is the **mutable** working copy of your function. Any changes to code or configuration update $LATEST.
- **Published Versions**: When you publish a version, Lambda makes a snapshot of $LATEST. This version is **immutable** (cannot be changed). It is assigned a monotonically increasing integer (e.g., version 1, 2, 3).
- **Benefits**: Ensures reproducibility. If a version works today, it will work exactly the same way forever.

### 2. Lambda Aliases
An **Alias** is a pointer to a specific Lambda version. 
- **Stable ARN**: Clients (like API Gateway or SQS) can point to an alias ARN (e.g., `arn:aws:lambda:us-east-1:123456789012:function:MyFunc:PROD`) instead of a version-specific ARN.
- **Dynamic Updates**: You can update the alias to point to a new version without changing the client configuration.
- **Common Aliases**: `DEV`, `STAGE`, `PROD`.

### 3. Shifting Traffic (Weighted Aliases)
Weighted aliases allow you to split traffic between two different published versions of a function.
- **Mechanism**: You specify a second version and a weight (percentage of traffic).
- **Example**: 90% traffic to version 1 (PROD), 10% traffic to version 2 (New Feature).
- **Constraints**:
    - Both versions must have the same IAM execution role.
    - You cannot point an alias to $LATEST when using routing weights; both must be fixed versions.
    - You cannot use the same version for both the primary and secondary routing configuration.

### 4. Deployment Patterns
- **Blue/Green Deployment**: You create a "Green" version, test it, and then flip the alias from the "Blue" version to the "Green" version instantly.
- **Canary Deployment**: You use weighted aliases to slowly increase traffic to the new version (e.g., 5%, 10%, 50%, 100%) while monitoring metrics.
- **Linear Deployment**: Traffic is shifted in equal increments over a set period (e.g., 10% every 2 minutes).

### 5. Integration with AWS CodeDeploy
CodeDeploy can automate the traffic shifting process:
- It manages the alias update.
- It can perform **Hooks** (Lambda functions) before and after the traffic shift to validate the health of the new version.
- It automatically rolls back if CloudWatch Alarms are triggered.

## Interview Questions

### Q1: What is the difference between $LATEST and a published version?
**A:** $LATEST is the current, mutable version of your Lambda function that you can edit. A published version is an immutable snapshot of $LATEST at a specific point in time, assigned a unique version number. You cannot change the code or configuration of a published version.

### Q2: Why should you use Aliases instead of pointing directly to Versions in production?
**A:** Using Aliases provides a stable ARN for event sources (like API Gateway). If you point to a specific version number, you must update every event source whenever you deploy a new version. With an Alias, you only update the pointer in Lambda, and all event sources automatically use the new version.

### Q3: How do weighted aliases support Canary deployments?
**A:** Weighted aliases allow you to specify two versions and a percentage of traffic for each. This enables you to send a small portion of production traffic (e.g., 5%) to a new version to verify its stability before fully rolling it out, minimizing the blast radius of potential bugs.

### Q4: Can you update the configuration (e.g., Environment Variables) of a published Lambda version?
**A:** No. Published versions are completely immutable, including code and all configuration settings like environment variables, memory size, and timeout. To change configuration, you must update $LATEST and publish a new version.

### Q5: What happens if a CloudWatch Alarm triggers during a CodeDeploy-managed Canary deployment?
**A:** CodeDeploy will automatically stop the deployment and roll back the alias to the previous stable version, ensuring that the system returns to a known healthy state immediately.
