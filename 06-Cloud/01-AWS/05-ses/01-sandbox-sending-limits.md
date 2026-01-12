#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ses']
---

## Summary
The **Amazon SES Sandbox** is a restricted environment for new AWS accounts designed to prevent fraud and protect the reputation of the service. In the sandbox, users can test SES features but are limited in sending volume, rate, and recipient types. Moving to **Production Access** removes these restrictions, allowing emails to be sent to any recipient and enabling higher throughput.

## Detailed Explanation

### The SES Sandbox
By default, all new Amazon SES accounts are placed in the **sandbox** environment. This status is region-specific and applies to each AWS Region individually.

#### Sandbox Restrictions:
1. **Verified Recipients Only**: You can only send emails to email addresses or domains that you have explicitly verified in your SES account, or to the Amazon SES mailbox simulator.
2. **Sending Quota**: Limited to **200 messages** per 24-hour period.
3. **Sending Rate**: Limited to **1 message per second**.
4. **Suppression List**: Certain account-level suppression list management features via API are disabled.

### Moving to Production
To send emails to unverified recipients and increase sending limits, you must request **Production Access**.

#### Steps to Request Access:
1. **Console**: Navigate to the SES Account Dashboard and select "Request production access".
2. **Use Case**: Specify whether the mail is **Transactional** (e.g., password resets) or **Marketing** (e.g., newsletters).
3. **Website URL**: Provide your website URL to help AWS understand your content.
4. **Content Description**: Explain how you build your mailing list and how you handle bounces and complaints.
5. **Approval**: AWS Support usually reviews and responds within 24 hours.

### Sending Limits and Quotas
Once in production, SES applies two main quotas to regulate sending:
- **Sending Quota**: The maximum number of emails you can send in a 24-hour period. This is a rolling window.
- **Sending Rate**: The maximum number of emails SES accepts per second.
- **Message Size**: The default maximum message size is 10 MB (including attachments and MIME encoding).

#### Managing Limits:
- **Scaling**: AWS automatically increases your limits as you consistently send high-quality mail near your current limits.
- **Manual Increase**: You can request a limit increase by opening a case in the AWS Support Center.
- **Monitoring**: Use the SES console dashboard or CloudWatch metrics (`SendingQuotas`) to monitor usage.

## Interview Questions

### 1. What are the primary restrictions of the Amazon SES sandbox?
In the sandbox, you can only send emails to verified addresses or domains, you are limited to 200 messages per 24 hours, and your sending rate is capped at 1 message per second.

### 2. How do you move an SES account from the sandbox to production?
You must submit a "Production Access" request via the SES console. This requires providing details about your use case (Transactional vs. Marketing), your website, and your strategy for handling bounces and complaints.

### 3. What is the difference between Sending Quota and Sending Rate?
The **Sending Quota** is the total number of emails allowed in a rolling 24-hour period, while the **Sending Rate** is the maximum number of emails that can be sent per second.

### 4. Are SES sending quotas based on the number of messages or the number of recipients?
They are based on **recipients**. For example, a single email sent to 10 recipients counts as 10 towards your sending quota.

### 5. What happens if you exceed your SES sending limits?
SES will reject the request with a `ThrottlingException` or `AccountSendingPaused` error. It is best practice to monitor these limits using CloudWatch and implement exponential backoff in your application.
