#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'iam']
---

## Summary
**Resource-based policies** are JSON policy documents that you attach directly to an AWS resource (such as an S3 bucket, SQS queue, or KMS key) rather than to an IAM identity (user, group, or role). They define which principals (who) have permission to perform specific actions on that resource and under what conditions.

## Detailed Explanation

### Core Concept
Unlike identity-based policies which specify what an identity can do, resource-based policies specify **who can access the resource**. These policies are always **inline policies**; there are no managed resource-based policies.

### The `Principal` Element
The most critical difference in JSON structure is the mandatory `Principal` element. Since the policy is attached to the resource, AWS needs to know which entity is being granted (or denied) permissions.
*   **Format**: Can be an AWS account ID, an IAM user ARN, a role ARN, or a service principal (e.g., `s3.amazonaws.com`).
*   **Anonymous Access**: Using `"Principal": "*"` allows public access (if not restricted by other layers like Block Public Access in S3).

### Cross-Account Access
Resource-based policies are the primary mechanism for granting cross-account access:
1.  **Trust**: The resource-based policy in Account A must specify the principal from Account B.
2.  **Permission**: The identity in Account B must also have an identity-based policy allowing the action on the resource in Account A.
*Exception*: If the principal and resource are in the same account, a permission in either the identity-based policy **OR** the resource-based policy is sufficient to grant access.

### Comparison with Identity-Based Policies
| Feature | Identity-Based Policy | Resource-Based Policy |
| :--- | :--- | :--- |
| **Attached to** | IAM User, Group, or Role | AWS Resource (S3, SQS, etc.) |
| **Principal** | Implicitly the identity attached to | Explicitly defined in `Principal` element |
| **Use Case** | Managing what a user can do | Managing who can access a resource |
| **Visibility** | Centralized in IAM | Distributed across resources |

### Common Services Supporting Resource-Based Policies
*   **Amazon S3**: Bucket policies.
*   **Amazon SQS**: Queue policies.
*   **Amazon SNS**: Topic policies.
*   **AWS KMS**: Key policies (mandatory for KMS keys).
*   **AWS Lambda**: Function policies (e.g., to allow S3 to trigger a function).
*   **IAM Roles**: The **Trust Policy** is technically a resource-based policy for the role.

## Interview Questions

### 1. What is the main difference between an identity-based policy and a resource-based policy?
Identity-based policies are attached to IAM identities (users, roles) and define what they can do. Resource-based policies are attached to resources (S3 buckets, SQS) and define who (Principal) can access them.

### 2. Is the `Principal` element required in identity-based policies?
No. In identity-based policies, the principal is implicitly the user or role to which the policy is attached. It is, however, mandatory in resource-based policies.

### 3. How do you grant an IAM user in Account B access to an S3 bucket in Account A?
You need two permissions: 1) A resource-based policy (Bucket Policy) in Account A that lists the user (or account) from Account B as a `Principal`. 2) An identity-based policy in Account B that grants the user permission to perform actions on the specific bucket ARN in Account A.

### 4. What happens if a resource-based policy grants access but an identity-based policy denies it?
An explicit `Deny` always overrides an `Allow`. The request will be denied regardless of which policy contains the deny.

### 5. Why are KMS Key Policies unique compared to other resource-based policies?
KMS keys *must* have a key policy. If you don't provide one, KMS creates a default one. Unlike S3 where you can rely solely on IAM policies, KMS requires the key policy to explicitly delegate permission to the account's IAM policies if you want to use identity-based permissions.
