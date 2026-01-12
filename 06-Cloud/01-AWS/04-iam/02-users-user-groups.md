#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'iam']
---

## Summary
**IAM Users** are entities created in AWS to represent individuals or applications that interact with AWS resources. **IAM Groups** are collections of IAM users that allow administrators to manage permissions for multiple users at once, simplifying the implementation of security policies and organizational structure.

## Detailed Explanation

### **Root User vs. IAM User**
*   **Root User**: The identity created when the AWS account is first opened. It has complete, unrestricted access to all resources and billing information. It cannot be restricted by IAM policies. 
    *   *Best Practice*: Use the Root user only to create your first IAM user with admin permissions, then lock away the Root credentials and use MFA.
*   **IAM User**: A specific identity within your AWS account with its own credentials and permissions. IAM users start with **no permissions** by default (Implicit Deny) and must be granted access through policies.

### **IAM Groups**
Groups are a way to manage permissions for a set of users. Instead of attaching policies to individual users, you attach them to a group.
*   **Organization**: Helps in mirroring the company's organizational structure (e.g., `Admins`, `Developers`, `Testers`).
*   **Efficiency**: Adding a user to a group automatically grants them all the permissions defined for that group. Removing them revokes those permissions instantly.

### **Group Inheritance and Limitations**
*   **Permissions**: A user inherits permissions from all groups they belong to. If a user is in both "Developers" and "Admins", they have the combined permissions of both.
*   **No Nesting**: IAM Groups **cannot** be nested. A group cannot contain another group; it can only contain users.
*   **Direct Attachment**: While possible, attaching policies directly to users is discouraged in favor of group-based management for better scalability.

### **Authentication & Access**
1.  **AWS Management Console**: Access via Username and Password. MFA (Multi-Factor Authentication) should always be enabled.
2.  **Programmatic Access**: Access via **Access Key ID** and **Secret Access Key**. Used for the AWS CLI, SDKs, and APIs. These are long-term credentials and should be rotated regularly.

## Interview Questions

### **1. What is the primary difference between an IAM User and an IAM Group?**
An **IAM User** is an individual identity (person or service) with unique credentials, while an **IAM Group** is a collection of users. Groups are used to manage permissions collectively; policies attached to a group apply to all its members.

### **2. Can an IAM user belong to multiple groups?**
Yes, an IAM user can belong to multiple groups (up to 10 by default). The user will inherit the permissions from all the groups they are a member of.

### **3. Is it possible to nest IAM groups (put a group inside another group)?**
No. AWS IAM does not support nested groups. You can only add users to groups, not groups to other groups.

### **4. Why is it a security risk to use the AWS Root user for daily administrative tasks?**
The Root user has absolute power and its permissions cannot be limited. If the credentials are compromised, the entire account is at risk. Best practice is to use an IAM user with `AdministratorAccess` and keep the Root user protected with MFA for emergency use only.

### **5. What is "Least Privilege" in the context of IAM users?**
The principle of **Least Privilege** means granting users only the minimum permissions they need to perform their specific job functions. This reduces the "blast radius" in case of a credential leak or accidental misuse.
