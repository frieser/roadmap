---
---

## Summary
Layer 4 load balancing operates at the **Transport Layer** (TCP/UDP) of the OSI model. It makes routing decisions based on network-layer information such as IP addresses and port numbers, without inspecting the actual payload of the packets. This makes it extremely fast and efficient for high-throughput traffic.

## Detailed Explanation

### What is Layer 4 Load Balancing?
In Layer 4 load balancing, the load balancer receives a connection request and directs it to a healthy backend server based on a simple algorithm (like Round Robin or Least Connections) using only the source and destination IP addresses and TCP/UDP ports.

### Key Characteristics
*   **Packet-Level Processing**: It only looks at the packet headers (IP and Port). It does not "open" the packet to see what's inside (e.g., HTTP headers, cookies).
*   **High Performance**: Because it requires minimal CPU cycles for inspection, it can handle a massive number of connections with very low latency.
*   **Protocol Agnostic**: Since it operates at the transport layer, it can balance any protocol built on top of TCP/UDP, including HTTP, SMTP, DNS, and proprietary game protocols.
*   **SSL Pass-through**: It can pass encrypted traffic directly to the backend servers without decrypting it, maintaining end-to-end encryption.

### Pros and Cons
*   **Pros**:
    *   Faster and more efficient (lower CPU/Memory usage).
    *   Simpler to manage.
    *   Secure (supports SSL pass-through).
*   **Cons**:
    *   **Dumb Routing**: Cannot route based on URL path, headers, or cookies.
    *   **Uneven Load**: May lead to uneven distribution if many requests come from a single long-lived TCP connection (e.g., a proxy).

### Go Application
In Go, you can implement a simple Layer 4 load balancer using the `net` package. Here is a basic TCP proxy concept:

```go
package main

import (
	"io"
	"log"
	"net"
)

func main() {
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatal(err)
	}

	for {
		clientConn, err := listener.Accept()
		if err != nil {
			log.Println(err)
			continue
		}

		// Simplified: Always forward to the same backend
		// In a real LB, you'd pick from a pool of backends
		go forward(clientConn, "127.0.0.1:8081")
	}
}

func forward(client net.Conn, backendAddr string) {
	defer client.Close()

	backend, err := net.Dial("tcp", backendAddr)
	if err != nil {
		log.Println(err)
		return
	}
	defer backend.Close()

	// Bidirectional copy
	go io.Copy(backend, client)
	io.Copy(client, backend)
}
```

## Interview Questions
*   **Q: At which OSI layer does L4 load balancing operate?**
*   **A:** It operates at the Transport Layer (Layer 4), utilizing TCP and UDP protocols.
*   **Q: Why is L4 load balancing faster than L7?**
*   **A:** It only inspects the network headers (IP and Port) and does not need to parse or decrypt the application data (like HTTP headers), which significantly reduces CPU overhead.
*   **Q: Can a Layer 4 load balancer perform path-based routing (e.g., /api vs /static)?**
*   **A:** No. Path-based routing requires inspecting the HTTP request, which only happens at Layer 7.
