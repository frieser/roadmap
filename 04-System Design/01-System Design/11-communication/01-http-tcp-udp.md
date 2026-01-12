---
---

## Summary
Network communication is built upon the OSI model, where different protocols operate at various layers to facilitate data exchange. **TCP (Transmission Control Protocol)** provides reliable, ordered, and error-checked delivery (L4), while **UDP (User Datagram Protocol)** prioritizes speed and low latency over reliability (L4). **HTTP (Hypertext Transfer Protocol)** is an application-layer protocol (L7) that traditionally runs over TCP, providing the foundation for web communication.

## Detailed Explanation

### 1. TCP (Transmission Control Protocol) - Layer 4
TCP is a connection-oriented protocol that ensures data reaches its destination intact and in the correct order.
*   **Mechanism**: Uses a **three-way handshake** (SYN, SYN-ACK, ACK) to establish a connection.
*   **Reliability**: Features flow control, congestion control, and error recovery (retransmission of lost packets).
*   **Use Cases**: Web browsing (HTTP), Email (SMTP), File Transfer (FTP), SSH.

### 2. UDP (User Datagram Protocol) - Layer 4
UDP is a connectionless protocol that sends packets ("datagrams") without establishing a connection or guaranteeing delivery.
*   **Mechanism**: "Fire and forget." No handshake, no acknowledgement.
*   **Efficiency**: Extremely low overhead and latency.
*   **Use Cases**: Video streaming, VoIP, Online gaming, DNS, DHCP.

### 3. HTTP (Hypertext Transfer Protocol) - Layer 7
HTTP is the backbone of the World Wide Web. It follows a request-response model.
*   **Evolution**:
    *   **HTTP/1.1**: Keep-alive connections, pipelining (limited).
    *   **HTTP/2**: Binary protocol, multiplexing (multiple requests over one TCP connection), header compression (HPACK), server push.
    *   **HTTP/3**: Uses **QUIC** (built on UDP) to solve head-of-line blocking issues in TCP.

### Go Context: `net` and `net/http`
Go provides powerful standard library support for these protocols.

#### TCP Server in Go
```go
package main

import (
	"bufio"
	"fmt"
	"net"
)

func main() {
	ln, _ := net.Listen("tcp", ":8080")
	for {
		conn, _ := ln.Accept()
		go handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
	defer conn.Close()
	message, _ := bufio.NewReader(conn).ReadString('\n')
	fmt.Print("Message received:", string(message))
}
```

#### HTTP Client in Go
```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	resp, err := http.Get("https://api.github.com")
	if err != nil {
		return
	}
	defer resp.Body.Close()
	body, _ := io.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```

## Interview Questions
*   **Q: What is the main difference between TCP and UDP?**
    *   **A:** TCP is connection-oriented and guarantees reliable, ordered delivery via handshakes and retransmissions. UDP is connectionless and prioritizes speed, offering no delivery guarantees.
*   **Q: How does HTTP/2 improve performance over HTTP/1.1?**
    *   **A:** HTTP/2 introduces multiplexing (allowing multiple requests/responses over a single TCP connection), binary framing, and header compression, reducing latency and resource usage.
*   **Q: What problem does HTTP/3 solve by using UDP (QUIC)?**
    *   **A:** HTTP/3 eliminates "head-of-line blocking" at the TCP level. In HTTP/2, if one TCP packet is lost, all subsequent packets are blocked. QUIC allows independent streams so a lost packet only affects its own stream.
