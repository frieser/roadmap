#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'iam']
---

# Assuming Roles (STS & Temporary Credentials)

## Summary
Assuming an IAM Role is the process of obtaining temporary security credentials to perform actions in an AWS environment. This is handled by the **AWS Security Token Service (STS)**. Unlike IAM users, roles do not have long-term passwords or access keys; instead, they provide short-lived credentials (Access Key ID, Secret Access Key, and Session Token) that expire automatically. This mechanism is central to cross-account access, service-to-service communication, and identity federation.

## Detailed Explanation

### 1. The Role of AWS STS (Security Token Service)
STS is a global web service that enables you to request temporary, limited-privilege credentials for users. 
- **Temporary Credentials**: Consist of an Access Key ID, a Secret Access Key, and a **Session Token**.
- **No Rotation Needed**: Since credentials expire, there is no need to manually rotate them.
- **Scope**: Can be used to access resources across different AWS accounts or to grant access to federated users.

### 2. Key API Operations for Assuming Roles
- **`AssumeRole`**: Used by IAM users or other roles to assume a target role (typically for cross-account or cross-service access).
- **`AssumeRoleWithSAML`**: Used for identity federation with on-premises SAML 2.0 identity providers (e.g., Okta, AD FS).
- **`AssumeRoleWithWebIdentity`**: Used for federation with public OIDC providers (e.g., Google, Amazon, Cognito).
- **`GetSessionToken`**: Primarily used for MFA-protected requests by IAM users.

### 3. Trust Policy vs. Permissions Policy
Every IAM Role has two types of policies that govern its usage:
- **Trust Policy (Resource-based)**: Defines **WHO** is allowed to assume the role (the "Principal"). It specifies the conditions under which an entity (User, Service, or Account) can call `sts:AssumeRole`.
- **Permissions Policy (Identity-based)**: Defines **WHAT** actions the holder of the role can perform once they have assumed it.

### 4. Cross-Account Access Workflow
Cross-account access is one of the most common use cases for assuming roles:
1. **Account B (Resource Account)**: Creates a role with a **Trust Policy** allowing **Account A** to assume it.
2. **Account A (User Account)**: Grants its user the permission `sts:AssumeRole` on the ARN of the role in Account B.
3. **User in Account A**: Calls `AssumeRole` to Account B.
4. **STS**: Validates both policies and returns temporary credentials for Account B to the user.

### 5. Role Chaining and Session Duration
- **Role Chaining**: Occurs when you use the temporary credentials of one role to assume another role. 
  - **Limitation**: Sessions are restricted to a **maximum of 1 hour** during chaining.
- **Session Duration**: The default is 1 hour, but it can be configured from 15 minutes up to **12 hours** (Maximum Session Duration setting).

## Interview Questions

**Q: What are the three components of temporary credentials returned by STS?**
**A:** An **Access Key ID**, a **Secret Access Key**, and a **Session Token**. The session token is mandatory for all requests made using temporary credentials.

**Q: How does a Trust Policy differ from a Permissions Policy in an IAM Role?**
**A:** The **Trust Policy** is a resource-based policy that defines which principals (users, services, or other accounts) are authorized to assume the role. The **Permissions Policy** defines the actual AWS actions (like `s3:ListBucket`) the principal can perform *after* assuming the role.

**Q: What is "Role Chaining" and what is its primary limitation?**
**A:** Role chaining is the process of assuming one role and then using those credentials to assume a second role. Its primary limitation is that the session duration for the chained role is capped at **1 hour**, regardless of the role's maximum session duration setting.

**Q: Why is assuming a role preferred over using IAM User access keys?**
**A:** Security. IAM User access keys are long-term credentials that must be managed and rotated. Role assumption provides **temporary credentials** that expire automatically, reducing the "blast radius" if credentials are leaked and eliminating the administrative overhead of rotation.

**Q: Can an EC2 instance assume a role? If so, how?**
**A:** Yes, via an **Instance Profile**. You attach an IAM Role to the Instance Profile, which is then associated with the EC2 instance. The AWS SDKs/CLI on the instance automatically fetch temporary credentials from the Instance Metadata Service (IMDS).
