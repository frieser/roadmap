#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ses']
---

## Summary
**Sender Reputation** is a critical metric in Amazon SES that determines the deliverability of your emails and the health of your AWS account. It is primarily measured through **Bounce Rates** and **Complaint Rates**. AWS monitors these metrics to protect its IP address reputation and ensure high deliverability for all users. If these rates exceed specific thresholds, AWS may place your account under review or suspend your ability to send emails.

## Detailed Explanation

### 1. Key Reputation Metrics
Amazon SES tracks two primary metrics to evaluate your sender reputation:

*   **Bounce Rate**: The percentage of sent emails that could not be delivered to the recipient's mail server.
    *   **Hard Bounce**: Permanent delivery failure (e.g., non-existent email address).
    *   **Soft Bounce**: Temporary delivery failure (e.g., full mailbox). SES reputation focuses primarily on hard bounces.
*   **Complaint Rate**: The percentage of recipients who marked your email as spam. This is reported via Feedback Loops (FBL) from Mailbox Providers (ISPs).

### 2. AWS SES Thresholds
AWS enforces strict limits to maintain service quality:

| Metric | Goal (Healthy) | Warning / Review | Suspension Risk |
| :--- | :--- | :--- | :--- |
| **Bounce Rate** | < 2% | > 5% | > 10% |
| **Complaint Rate** | < 0.05% | > 0.1% | > 0.5% |

### 3. Account Status and Probation
*   **Healthy**: Metrics are within the desired range.
*   **Under Review (Probation)**: If thresholds are exceeded, AWS sends a notification. You are typically given a period (e.g., 30 days) to fix the underlying issue. You must provide a **Root Cause Analysis (RCA)** and an action plan.
*   **Sending Paused (Suspension)**: If the issue persists or the rates are extremely high, AWS will disable your ability to send emails until the issue is resolved and a formal appeal is accepted.

### 4. Reputation Dashboard
The SES Console provides a **Reputation Metrics** dashboard that visualizes:
*   Account-level bounce and complaint rates.
*   Notifications for specific issues (e.g., "Spam Trap" hits).
*   History of metrics over time.

### 5. Best Practices for High Reputation
*   **Use Double Opt-in**: Ensure subscribers explicitly want your emails.
*   **Clean Lists Regularly**: Remove bounced addresses immediately (Automate this using SNS notifications).
*   **Authentication**: Implement **SPF**, **DKIM**, and **DMARC** to prove you are a legitimate sender.
*   **Easy Unsubscribe**: Provide a clear "Unsubscribe" link to reduce spam complaints.
*   **Monitor SNS Notifications**: Set up SNS topics for Bounces and Complaints to programmatically update your database.

## Interview Questions

**Q: What is the difference between a Hard Bounce and a Soft Bounce in the context of SES reputation?**
**A:** A Hard Bounce is a permanent failure (e.g., the email address doesn't exist), while a Soft Bounce is temporary (e.g., the recipient's inbox is full). AWS SES reputation metrics primarily track Hard Bounces because they indicate poor list hygiene.

**Q: What are the specific bounce rate thresholds that trigger a warning and suspension in AWS SES?**
**A:** AWS expects you to keep your bounce rate below 5%. If it exceeds 5%, your account may be placed under review. If it exceeds 10%, AWS may pause your ability to send emails.

**Q: How can you programmatically handle bounces and complaints in AWS SES?**
**A:** You can configure Amazon SES to send notifications to an **Amazon SNS** topic whenever a bounce or complaint occurs. You can then trigger a Lambda function or an SQS queue to process these notifications and automatically remove the offending email addresses from your mailing list.

**Q: If your SES account is placed "Under Review," what steps should you take?**
**A:** You should immediately identify the source of the high bounce or complaint rate (e.g., a specific campaign or an old list). Then, provide AWS with a detailed response explaining the root cause and the specific technical or procedural changes you are implementing to prevent it from happening again.

**Q: Why is the Complaint Rate threshold much lower than the Bounce Rate threshold?**
**A:** Because a spam complaint is a direct signal from a user that the email is unwanted, which has a much more severe impact on the reputation of the sending IP addresses and the "from" domain across the entire internet.
