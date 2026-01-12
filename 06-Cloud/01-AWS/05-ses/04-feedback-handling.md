#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ses']
---

# SES Feedback Handling (Bounces, Complaints, SNS)

## Summary
Managing email feedback is a critical requirement for maintaining a high sender reputation and ensuring delivery to the inbox. Amazon SES provides automated mechanisms to track **Bounces** (delivery failures) and **Complaints** (recipients marking mail as spam). Failure to handle these events—specifically by continuing to send to failing addresses—will lead to your account being placed under review or suspended. AWS recommends using **Amazon SNS** or **Configuration Sets** to automate the removal of these addresses from your mailing lists.

## Detailed Explanation

### 1. Types of Feedback

#### **A. Bounces**
A bounce occurs when an email cannot be delivered to the recipient.
*   **Hard Bounce**: A permanent delivery failure (e.g., the email address doesn't exist or the domain is invalid). 
    *   *Action*: You must remove the address from your list immediately.
*   **Soft Bounce**: A temporary failure (e.g., the recipient's mailbox is full or the server is temporarily down).
    *   *Action*: You can retry later, but persistent soft bounces should eventually be treated as hard bounces.

#### **B. Complaints**
A complaint occurs when a recipient clicks "Mark as Spam" in their email client (Gmail, Outlook, etc.).
*   *Action*: You must remove the address immediately. Continuing to send to a complaining recipient is a major violation of SES policy.

### 2. Notification Mechanisms

Amazon SES can notify you of these events through three primary channels:

1.  **Email Feedback Forwarding (Default)**: SES sends a notification email to the `Return-Path` address or the `Source` address. This is difficult to automate at scale.
2.  **Amazon SNS Notifications**: You can configure an SNS topic for Bounces, Complaints, and optionally Deliveries. This allows for automated processing (e.g., triggering a Lambda function to update a database).
3.  **Event Publishing (Configuration Sets)**: The most advanced method. You can send detailed JSON logs to CloudWatch, Kinesis Data Firehose, or SNS. This is preferred for complex analytics.

### 3. SES Reputation Dashboard
AWS monitors your sending activity and provides a dashboard to track health:
*   **Bounce Rate**: Should be kept **under 5%**. A rate above **10%** will result in account suspension.
*   **Complaint Rate**: Should be kept **under 0.1%**. A rate above **0.5%** will result in account suspension.

### 4. Implementation with Go
When using SNS notifications, your backend (often an AWS Lambda function written in Go) receives a JSON payload. Below is a simplified example of how to parse an SES bounce notification in Go.

```go
package main

import (
	"encoding/json"
	"fmt"
)

// SESBounceNotification represents the structure of an SES bounce event via SNS
type SESBounceNotification struct {
	EventType string `json:"eventType"`
	Bounce    struct {
		BounceType    string `json:"bounceType"`
		BounceSubType string `json:"bounceSubType"`
		BouncedRecipients []struct {
			EmailAddress string `json:"emailAddress"`
		} `json:"bouncedRecipients"`
	} `json:"bounce"`
}

func handleNotification(payload []byte) {
	var notification SESBounceNotification
	if err := json.Unmarshal(payload, &notification); err != nil {
		fmt.Printf("Error unmarshaling: %v\n", err)
		return
	}

	if notification.EventType == "Bounce" {
		for _, recipient := range notification.Bounce.BouncedRecipients {
			fmt.Printf("Action: Remove %s from database. Type: %s\n", 
				recipient.EmailAddress, notification.Bounce.BounceType)
		}
	}
}

func main() {
	// Example SNS JSON payload from SES
	rawJSON := []byte(`{
		"eventType": "Bounce",
		"bounce": {
			"bounceType": "Permanent",
			"bounceSubType": "General",
			"bouncedRecipients": [{"emailAddress": "invalid@example.com"}]
		}
	}`)
	handleNotification(rawJSON)
}
```

## Interview Questions

### **Q1: What is the difference between a Hard Bounce and a Soft Bounce in SES?**
**A**: A Hard Bounce is a permanent failure (invalid email), and the address must be removed immediately. A Soft Bounce is a temporary failure (mailbox full), which may be retried but should be monitored for persistence.

### **Q2: At what percentage does the SES Bounce Rate become a critical issue?**
**A**: A bounce rate above **5%** is considered high and may trigger a warning. If the rate exceeds **10%**, AWS will likely suspend your account's ability to send emails to protect the reputation of their IP ranges.

### **Q3: Why is using SNS better than Email Feedback Forwarding?**
**A**: Email Forwarding is manual and hard to track. SNS allows for **automated remediation**. You can hook SNS into a Lambda function or a SQS queue to automatically flag or delete failing email addresses in your database without human intervention.

### **Q4: What is a "Feedback Loop" (FBL)?**
**A**: A Feedback Loop is a mechanism where mailbox providers (like Yahoo or Microsoft) signal SES when a user marks an email as spam. SES then translates this into a "Complaint" notification for the sender.

### **Q5: If you receive a "Complaint", what is the required immediate action?**
**A**: You must immediately stop sending emails to that recipient. Failing to do so increases your complaint rate and directly impacts your deliverability to other users.
