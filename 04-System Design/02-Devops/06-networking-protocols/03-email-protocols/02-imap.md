---
---

## Summary
IMAP (Internet Message Access Protocol) is a standard email retrieval protocol that allows clients to access email on a remote mail server. Unlike POP3, IMAP is designed to keep messages on the server, allowing users to synchronize their email across multiple devices (desktop, mobile, web).

## Detailed Explanation

### Key Concepts
1.  **Synchronization**: Changes made on one device (read status, delete, move) are reflected on the server and all other devices.
2.  **Server-Side Folders**: IMAP supports organizing messages into folders (Mailboxes) on the server.
3.  **Statefulness**: The connection is stateful. The server keeps track of which messages the client has seen.
4.  **On-Demand Download**: Clients can download just the headers of messages without downloading the entire body/attachments, saving bandwidth.

### Ports
*   **Port 143**: Standard IMAP (insecure or STARTTLS).
*   **Port 993**: IMAPS (Implicit SSL/TLS).

## Go-Specific Context/Examples

In Go, the `github.com/emersion/go-imap` library is the standard for building IMAP clients and servers.

### Example: Connecting and Listing Folders

```go
package main

import (
	"log"

	"github.com/emersion/go-imap/client"
)

func main() {
	log.Println("Connecting to server...")

	// Connect to server
	c, err := client.DialTLS("mail.example.com:993", nil)
	if err != nil {
		log.Fatal(err)
	}
	log.Println("Connected")

	// Login
	if err := c.Login("username", "password"); err != nil {
		log.Fatal(err)
	}
	log.Println("Logged in")
	defer c.Logout()

	// List mailboxes
	mailboxes := make(chan *imap.MailboxInfo, 10)
	done := make(chan error, 1)
	go func() {
		done <- c.List("", "*", mailboxes)
	}()

	log.Println("Mailboxes:")
	for m := range mailboxes {
		log.Println("* " + m.Name)
	}
}
```

## Interview Questions

**Q: What is the main difference between IMAP and POP3?**
**A:** POP3 assumes the user accesses email from a single device; it downloads messages and typically deletes them from the server. IMAP assumes multiple devices; it keeps messages on the server and synchronizes state (read/unread/folders) across all clients.

**Q: Why is IMAP considered more complex to implement than POP3?**
**A:** IMAP requires the server to maintain complex state for every client connection (flags, folder structure, UID mapping), whereas POP3 is a simple store-and-forward protocol with minimal state.

**Q: Can you search emails on the server with IMAP?**
**A:** Yes, IMAP has server-side search capabilities (SEARCH command). You can ask the server to "Give me all unread emails from John" without downloading every email to the client first.
