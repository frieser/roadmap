---
---

# Network Sockets

## Summary
**Sockets** are the fundamental endpoints for network communication. A socket is an interface provided by the OS that allows applications to send and receive data across a network. It is uniquely identified by the combination of an **IP Address** and a **Port Number**. 

In Unix-like systems, sockets follow the "**Everything is a file**" philosophy and are represented as **File Descriptors (FD)**. This means they can be manipulated using standard file I/O operations like \`read()\` and \`write()\`.

## Detailed Explanation

### 1. Key System Calls
The socket API consists of several core system calls that define the lifecycle of a network connection:

| Call | Description | Side |
| :--- | :--- | :--- |
| \`socket()\` | Creates a new communication endpoint (returns a file descriptor). | Both |
| \`bind()\` | Assigns a local IP address and port number to the socket. | Server |
| \`listen()\` | Marks the socket as passive, ready to accept incoming connections. | Server |
| \`accept()\` | Blocks until a connection arrives, returning a *new* socket for the connection. | Server |
| \`connect()\` | Initiates a connection to a remote address (active open). | Client |
| \`send()\` / \`recv()\` | Transmits or receives data over the socket (similar to \`write\`/\`read\`). | Both |
| \`close()\` | Terminates the connection and releases the file descriptor. | Both |

### 2. Socket Types
- **\`SOCK_STREAM\` (TCP)**: 
    - Connection-oriented.
    - Guaranteed delivery, ordered, and error-checked.
    - Used for HTTP, FTP, SSH.
- **\`SOCK_DGRAM\` (UDP)**: 
    - Connectionless.
    - No guarantee of delivery or order.
    - Low latency, used for DNS, Video Streaming, VoIP.

### 3. Blocking vs. Non-blocking I/O
- **Blocking**: The process waits (sleeps) until the operation completes (e.g., \`accept\` waits for a client).
- **Non-blocking**: The call returns immediately. If data isn't ready, it returns an error (like \`EAGAIN\`). The application must poll or use notification mechanisms.

### 4. I/O Multiplexing (Select/Poll/Epoll)
When handling thousands of concurrent connections, spawning a thread per socket is inefficient. Multiplexing allows a single thread to monitor multiple sockets:
- **\`select()\`**: Old, limited to 1024 FDs, $O(N)$ complexity (must scan all FDs).
- **\`poll()\`**: Similar to select but handles more FDs. Still $O(N)$.
- **\`epoll()\` (Linux)**: Event-driven. Only returns FDs that are actually ready. $O(1)$ complexity, making it highly scalable (used by Nginx, Redis).

## Go Implementation

### Standard Library (\`net\` package)
Go provides a high-level abstraction that handles the complexity of syscalls and non-blocking I/O (using an internal netpoller based on epoll/kqueue).

\`\`\`go
package main

import (
	"fmt"
	"net"
)

func main() {
	// Server (Listen + Accept)
	ln, err := net.Listen("tcp", ":8080")
	if err != nil {
		return
	}
	fmt.Println("Server listening on :8080")
	
	for {
		conn, err := ln.Accept()
		if err != nil {
			continue
		}
		go handle(conn)
	}
}

func handle(conn net.Conn) {
	defer conn.Close()
	conn.Write([]byte("Hello from Go Socket!\n"))
}
\`\`\`

### Raw Syscall Example
For low-level control, you can use the \`syscall\` package directly. This mirrors the C API.

\`\`\`go
package main

import (
	"fmt"
	"syscall"
)

func main() {
	// 1. Create Socket (AF_INET = IPv4, SOCK_STREAM = TCP)
	fd, _ := syscall.Socket(syscall.AF_INET, syscall.SOCK_STREAM, 0)
	defer syscall.Close(fd)

	// 2. Bind
	addr := &syscall.SockaddrInet4{Port: 8081}
	copy(addr.Addr[:], []byte{0, 0, 0, 0}) // 0.0.0.0
	syscall.Bind(fd, addr)

	// 3. Listen
	syscall.Listen(fd, syscall.SOMAXCONN)
	fmt.Println("Raw syscall server listening on :8081")

	// 4. Accept
	nfd, _, _ := syscall.Accept(fd)
	syscall.Write(nfd, []byte("Hello from Raw Syscall!\n"))
	syscall.Close(nfd)
}
\`\`\`

## Interview Questions

**1. What is the difference between \`listen()\` and \`accept()\`?**
\`listen()\` marks a socket as willing to accept connections and sets a backlog queue size. \`accept()\` actually pulls the first pending connection from that queue and creates a new socket dedicated to that specific connection, leaving the original socket free to continue listening.

**2. Why does \`accept()\` return a new file descriptor?**
To allow the server to keep listening for new connections on the original socket while communicating with the current client on the new socket. This enables concurrency.

**3. What is the "Thundering Herd" problem in sockets?**
It occurs when multiple processes are waiting for an event (like \`accept()\`) on the same socket. When a connection arrives, the OS wakes up all processes, but only one can handle it, leading to wasted CPU cycles. Modern kernels (and \`EPOLLEXCLUSIVE\`) solve this.

**4. Compare \`select\`, \`poll\`, and \`epoll\`.**
\`select\` and \`poll\` require the application to pass a list of all monitored FDs to the kernel every time, and the kernel must scan them all ($O(N)$). \`epoll\` maintains the list in the kernel and only notifies the application about FDs that changed state ($O(1)$), making it much faster for high-concurrency.

**5. What is a "Zombie" or "Orphan" socket?**
An orphan socket is a socket that is no longer associated with a file descriptor in any process (e.g., after \`close()\`), but the kernel is still managing it to finish the TCP teardown (FIN/ACK sequence). If the peer doesn't respond, it can stay in \`FIN_WAIT_2\` until it times out.
