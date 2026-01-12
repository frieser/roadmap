---
tags: ['aws', 'roadmap']
---

# Instance Profiles (Passing roles to EC2)

## Summary
An **Instance Profile** is a container for an IAM role that allows you to pass role information to an Amazon EC2 instance at launch. It acts as a bridge, enabling applications on the instance to securely access AWS services by automatically providing temporary security credentials through the **Instance Metadata Service (IMDS)**, eliminating the need to manage or store long-term access keys.

## Detailed Explanation

### The Role-Profile Relationship
While often conflated, IAM Roles and Instance Profiles serve distinct purposes:
- **IAM Role**: Defines a set of permissions (via policies) and a trust policy (who can assume it).
- **Instance Profile**: A container that holds **exactly one** IAM role. It is the specific object that is attached to an EC2 instance.

**Key Rule**: An instance profile can contain only one IAM role, but one IAM role can be associated with multiple instance profiles.

### How Credentials are Provided
When an EC2 instance is associated with an instance profile, AWS handles the "heavy lifting" of security:
1. **Assumption**: The EC2 service assumes the role associated with the instance profile.
2. **Delivery**: Temporary credentials (Access Key, Secret Key, and Session Token) are delivered to the instance via the **Instance Metadata Service (IMDS)**.
3. **Consumption**: Applications using the AWS SDK, CLI, or specialized libraries automatically check the metadata service for credentials if none are found in environment variables or configuration files.

### Instance Metadata Service (IMDS)
The credentials and other instance-specific data are available at a link-local IP address: `http://169.254.169.254/latest/meta-data/`.
- **IMDSv1**: A simple request-response model. Vulnerable to SSRF (Server-Side Request Forgery).
- **IMDSv2**: Uses session-oriented requests. Requires a "token" obtained via a `PUT` request before the metadata can be retrieved via `GET`. This is the current security best practice.

### Creation Workflow
- **AWS Management Console**: When you create a role for EC2, the console automatically creates an instance profile with the same name.
- **AWS CLI / Terraform / CloudFormation**: These tools require you to explicitly:
    1. Create the **IAM Role**.
    2. Create the **Instance Profile**.
    3. Add the Role to the Instance Profile (`aws iam add-role-to-instance-profile`).
    4. Attach the Instance Profile to the EC2 instance.

### Security Benefits
- **No Hardcoded Secrets**: You never store `AWS_ACCESS_KEY_ID` or `AWS_SECRET_ACCESS_KEY` on the instance or in the code.
- **Automatic Rotation**: AWS rotates the temporary credentials automatically before they expire.
- **Least Privilege**: Roles can be easily updated to grant or revoke specific permissions without touching the instance itself.

## Interview Questions

### 1. What is the difference between an IAM Role and an Instance Profile?
An **IAM Role** is a set of permissions that defines what actions are allowed. An **Instance Profile** is a container for that role that specifically allows the role to be "passed" to an EC2 instance. In the AWS Console, this distinction is mostly hidden, but when using the CLI or IaC tools, you must create both separately.

### 2. How does an application on an EC2 instance know which credentials to use?
AWS SDKs and the CLI follow a **provider chain** to look for credentials. If they don't find them in environment variables or local config files, they automatically query the **Instance Metadata Service (IMDS)** at `http://169.254.169.254`. If an instance profile is attached, the IMDS returns temporary credentials for that role.

### 3. Can you attach more than one IAM role to a single EC2 instance?
No. An EC2 instance can only have **one** instance profile attached at a time, and an instance profile can contain only **one** IAM role. If you need multiple sets of permissions, you must combine them into a single role.

### 4. What is the primary security advantage of IMDSv2 over IMDSv1?
IMDSv2 protects against **SSRF (Server-Side Request Forgery)** attacks. It requires a session-based token obtained via a `PUT` request with a header that a simple SSRF vulnerability usually cannot bypass, whereas IMDSv1 allows simple `GET` requests which are easier for attackers to exploit.

### 5. If you update the policy attached to a role already used by an EC2 instance, do you need to restart the instance?
No. Changes to the IAM policy take effect almost immediately (within seconds to minutes). The instance does not need to be restarted because the metadata service will simply start providing credentials that reflect the new permissions.
