---
---

## Summary
POP3S (Post Office Protocol version 3 Secure) is the encrypted version of POP3, running over SSL/TLS. It is a simple protocol used by local email clients to retrieve emails from a remote server. It follows a "store-and-forward" model, typically downloading messages to the local machine and deleting them from the server.

## Detailed Explanation

### How it Works
1.  **Connect**: Client connects to server (typically Port 995).
2.  **Auth**: Client authenticates.
3.  **Transaction**: Client asks for message list, retrieves messages, and marks them for deletion.
4.  **Update**: Server deletes the marked messages and closes connection.

### Characteristics
*   **Stateless**: The server doesn't know if you read the email on your phone. Once downloaded to your laptop, it's gone from the server (unless "Leave copy on server" is configured).
*   **Offline Access**: Good for reading emails offline after downloading.
*   **Simple**: Very basic commands (USER, PASS, LIST, RETR, DELE, QUIT).

### Ports
*   **Port 110**: Standard POP3 (Unencrypted).
*   **Port 995**: POP3S (Encrypted).

## Go-Specific Context/Examples

You can use Go's `net/textproto` for raw access or a library like `github.com/knadh/go-pop3`.

### Example: Connecting via TLS

```go
package main

import (
	"crypto/tls"
	"fmt"
	"log"
	"net/textproto"
)

func main() {
	// Connect to the POP3S server
	conn, err := tls.Dial("tcp", "pop.gmail.com:995", nil)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	text := textproto.NewConn(conn)

	// Read greeting
	_, err = text.Cmd("") 
	if err != nil {
		log.Fatal(err)
	}

	// Authenticate
	text.Cmd("USER myuser")
	text.Cmd("PASS mypassword")

	// List messages
	id, err := text.Cmd("LIST")
	fmt.Printf("Cmd id: %d\n", id)
	
	// Send QUIT
	text.Cmd("QUIT")
}
```

## Interview Questions

**Q: When would you recommend POP3 over IMAP?**
**A:** Rarely in modern contexts. It might be used if storage on the mail server is extremely limited (forcing users to download/delete), or for strict privacy/security where users want emails only on their local device and not stored in the cloud.

**Q: Does POP3 support folders?**
**A:** No. POP3 only sees the "Inbox". It has no concept of server-side folders (Sent, Trash, Custom Folders). All organization must happen locally on the client.

**Q: What is the risk of using Port 110?**
**A:** Port 110 sends credentials (username/password) and email data in **plain text**. Anyone on the network (Wi-Fi sniffing) can steal credentials. Always use POP3S (Port 995).
