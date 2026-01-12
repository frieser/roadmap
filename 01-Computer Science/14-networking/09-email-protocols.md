---
---

## Summary
Email depends on a suite of protocols for sending (**SMTP**) and receiving (**IMAP/POP3**) messages. In DevOps, configuring these correctly and implementing security standards (**SPF, DKIM, DMARC**) is vital for deliverability and preventing spoofing.

## Core Protocols

### 1. SMTP (Simple Mail Transfer Protocol)
- **Purpose**: Sending emails.
- **Ports**: 25 (Legacy/Relay), 587 (Submission with STARTTLS), 465 (SMTPS/SSL).
- **Function**: Transfers mail from a client to a server or between servers.

### 2. IMAP (Internet Message Access Protocol)
- **Purpose**: Retrieving emails.
- **Port**: 143 (Default), 993 (SSL).
- **Feature**: Syncs mail across multiple devices. Actions (like marking as read) are reflected everywhere.

### 3. POP3 (Post Office Protocol v3)
- **Purpose**: Retrieving emails.
- **Port**: 110 (Default), 995 (SSL).
- **Feature**: Downloads mail to a single device and typically deletes it from the server. Legacy.

---

## Email Security (Prevention of Spoofing)

### SPF (Sender Policy Framework)
- **Mechanism**: A **DNS TXT record** that lists the IP addresses/domains authorized to send email on behalf of your domain.
- **Check**: The receiving server checks if the sending IP is in the SPF record.

### DKIM (DomainKeys Identified Mail)
- **Mechanism**: Adds a **digital signature** to the email header.
- **Check**: The sender signs the email with a private key. The receiver uses the public key (found in a **DNS TXT record**) to verify the signature.

### DMARC (Domain-based Message Authentication, Reporting, and Conformance)
- **Mechanism**: A policy that tells the receiver what to do if SPF or DKIM fails (e.g., `p=none`, `p=quarantine`, `p=reject`).
- **Benefit**: Provides reporting back to the domain owner about who is sending mail as them.

---

## Go Implementation: Sending Email
Using the standard library `net/smtp`.

```go
package main

import (
	"log"
	"net/smtp"
)

func main() {
	// Configuration
	from := "sender@example.com"
	password := "your-app-password"
	to := []string{"recipient@example.com"}
	smtpHost := "smtp.gmail.com"
	smtpPort := "587"

	// Message
	message := []byte("Subject: DevOps Test Email\n\nThis is a test email sent from Go.")

	// Authentication
	auth := smtp.PlainAuth("", from, password, smtpHost)

	// Send
	err := smtp.SendMail(smtpHost+":"+smtpPort, auth, from, to, message)
	if err != nil {
		log.Fatal(err)
	}
	log.Println("Email sent successfully!")
}
```

### Validating SPF/DKIM (Conceptual)
For advanced validation in Go, libraries like `github.com/emersion/go-msgauth` are used to verify signatures and check SPF records programmatically.

---

## Interview Questions
- **Q: What is the difference between IMAP and POP3?**
  - **A:** IMAP syncs the state of emails across all devices (multi-device support). POP3 downloads and usually removes email from the server, making it suitable for single-device access.
- **Q: How does DKIM prevent tampering?**
  - **A:** DKIM hashes the email body and specific headers and signs that hash with a private key. If an attacker changes the content, the hash won't match the signature when verified with the public key.
- **Q: What happens if a domain has no SPF or DKIM records?**
  - **A:** Most modern mail providers (Gmail, Outlook) will either mark the email as Spam or reject it entirely, as there is no way to verify the sender's identity.
