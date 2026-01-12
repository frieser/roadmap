#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ses']
---

# SES Identity Verification (Domain vs Email Address)

## Summary
Amazon SES requires identity verification before you can send email. You can verify either a specific **Email Address** or an entire **Domain**. Verifying an email address is quick for individual testing, while verifying a domain provides scalability by authorizing any email address or subdomain under that domain.

## Detailed Explanation

### **1. Email Address Identity**
- **Process**: You enter an email address in the SES console. AWS sends a verification email to that address containing a link.
- **Verification Method**: Clicking the verification link within 24 hours.
- **Scope**: Only the specific verified email address can be used as a "From", "Source", "Sender", or "Return-Path" address.
- **Use Case**: Quick setup, small-scale sending, or when you don't have access to DNS settings.
- **Case Sensitivity**: Email addresses are case-sensitive in SES (e.g., `User@example.com` is different from `user@example.com`).

### **2. Domain Identity**
- **Process**: You enter a domain name (e.g., `example.com`).
- **Verification Method**: Adding DNS records to your domain's DNS configuration.
- **Records**:
    - **Easy DKIM**: AWS generates three **CNAME** records. This is the modern and recommended method.
    - **BYODKIM (Bring Your Own DKIM)**: You provide your own public-private key pair. You add the public key as a **TXT** record.
- **Scope**: Once a domain is verified, any email address (e.g., `support@example.com`, `sales@example.com`) or subdomain (e.g., `marketing.example.com`) is automatically authorized.
- **Inheritance**: Subdomains and email addresses inherit the verification status of the parent domain for "straightforward sending".
- **Use Case**: Production environments, branding, and sending from multiple addresses without individual verification.

### **3. Verification Status & Inheritance**
| Feature | Email Identity | Domain Identity |
| :--- | :--- | :--- |
| **Setup Time** | Instant (Link click) | Propagation dependent (up to 72h) |
| **Scalability** | Low (Per address) | High (Entire domain) |
| **DNS Access** | Not required | **Required** |
| **Method** | Email Link | DNS Records (DKIM/CNAME) |

### **4. Advanced Sending**
While email addresses inherit verification from the domain, "Advanced Sending" features (like using specific **Configuration Sets**, **Delegate Sending Policies**, or overriding domain-level feedback settings) require the email address to be explicitly verified as an individual identity.

## Interview Questions

### **Q1: What is the primary difference between verifying a domain vs. an email address in SES?**
**A**: Verifying an email address authorizes only that specific address to send mail via a link confirmation. Verifying a domain authorizes the root domain, all subdomains, and any email address associated with that domain using DNS records (DKIM).

### **Q2: Which DNS records are used for Easy DKIM domain verification in SES?**
**A**: SES provides three **CNAME** records that must be added to your DNS provider. These records point to AWS-managed DKIM keys.

### **Q3: If I verify `example.com`, do I need to verify `no-reply@example.com`?**
**A**: No. Verification is inherited. Once the domain `example.com` is verified, any email address under it is authorized for sending. However, if you need to apply a specific Configuration Set or Policy to `no-reply@example.com`, you should verify it individually.

### **Q4: Can I use SES to send email from a domain I don't own?**
**A**: Only if you can verify an individual email address on that domain (by clicking the link). To verify a domain identity, you must have access to its DNS settings.

### **Q5: What happens if a domain verification is stuck in "Pending" for more than 72 hours?**
**A**: Common issues include:
1. Incorrect record values (copy-paste errors).
2. DNS provider automatically appending the domain name (resulting in `record.example.com.example.com`).
3. Not including underscores in the record names (some providers struggle with this).
4. The region in which the records were generated does not match where you are checking.
