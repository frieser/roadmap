---
---

# Email Protocols: SMTP, IMAP, POP3

Email infrastructure is a mix of three distinct protocols. DevOps engineers usually focus on **SMTP** (sending transactional emails, alerts) rather than retrieval protocols, but understanding the ecosystem is vital for troubleshooting.

## Summary

*   **SMTP (Simple Mail Transfer Protocol)**: "Push" protocol. Used to **send** email from a client to a server, or between servers. (Port 25, 587).
*   **IMAP (Internet Message Access Protocol)**: "Pull/Sync" protocol. Used to **access** email on a server. Keeps mail on the server, syncing status (read, deleted) across multiple devices. (Port 143, 993).
*   **POP3 (Post Office Protocol v3)**: "Pull/Download" protocol. Downloads email to a single device and (usually) deletes it from the server. Legacy. (Port 110, 995).

## Detailed Explanation

### SMTP (The Delivery Truck)
*   **Port 25**: Server-to-Server communication (Relay). Often blocked by residential ISPs to prevent spam.
*   **Port 587**: Client-to-Server submission (authenticated TLS). The standard for apps sending mail.
*   **Relay**: The process of passing mail from one MTA (Mail Transfer Agent) to another until it reaches the destination.

### IMAP vs POP3
*   **IMAP**: Complex. Allows folders, server-side search, and IDLE (push notifications).
*   **POP3**: Simple. Connect, grab all messages, disconnect. Good for offline archives, bad for multi-device users.

---

## Go Implementation Example

Go's standard library `net/smtp` is excellent for sending emails.

```go
package main

import (
	"log"
	"net/smtp"
)

func main() {
	// Configuration
	from := "alerts@mycompany.com"
	password := "secret"
	to := []string{"admin@mycompany.com"}
	smtpHost := "smtp.gmail.com"
	smtpPort := "587"

	// Authentication
	auth := smtp.PlainAuth("", from, password, smtpHost)

	// Message
	msg := []byte("To: admin@mycompany.com\r\n" +
		"Subject: Server Down!\r\n" +
		"\r\n" +
		"The production database is not responding.\r\n")

	// Sending email
	err := smtp.SendMail(smtpHost+":"+smtpPort, auth, from, to, msg)
	if err != nil {
		log.Fatal(err)
	}
	log.Println("Email sent successfully!")
}
```

## Interview Questions

**Q: Why is Port 25 often blocked on cloud instances (AWS EC2)?**
**A:** Port 25 is the default port for unauthenticated relaying of spam. Cloud providers block it by default to protect their IP reputation. You usually have to request a limit increase or use an authenticated port (587) via a relay service (like SES or SendGrid).

**Q: Explain the role of an MTA vs. an MUA.**
**A:**
*   **MUA (Mail User Agent)**: The client app (Outlook, Thunderbird, `mutt`). Uses SMTP to send, IMAP to read.
*   **MTA (Mail Transfer Agent)**: The server software (Postfix, Sendmail, Exchange). Uses SMTP to route mail to other MTAs.

**Q: What is "Greylisting"?**
**A:** An anti-spam technique where the receiving MTA temporarily rejects a new email with a "4xx Temporary Error". Legitimate servers queue the mail and retry later (passing the check). Spam bots usually fire-and-forget and won't retry, so the spam is dropped.
