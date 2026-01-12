---
tags: ['linux', 'roadmap']
---

# TCP/IP Stack

## Summary
The TCP/IP stack is the suite of communication protocols that powers the internet and most private networks. Unlike the theoretical 7-layer OSI (Open Systems Interconnection) model, the TCP/IP model is a practical 4-layer framework (Application, Transport, Internet, and Network Access) that defines how data should be packetized, addressed, transmitted, routed, and received.

## Detailed Explanation

### The OSI Model vs. TCP/IP Model
While the OSI model is essential for academic understanding, the TCP/IP model is what is actually implemented in Linux and modern networking equipment.

| OSI Layer | TCP/IP Layer | Key Protocols | Data Unit (PDU) |
| :--- | :--- | :--- | :--- |
| 7. Application | **Application** | HTTP, DNS, SSH, SMTP | Data / Message |
| 6. Presentation | **Application** | SSL/TLS, ASCII, JPEG | Data / Message |
| 5. Session | **Application** | RPC, NetBIOS | Data / Message |
| 4. Transport | **Transport** | TCP, UDP, SCTP | Segment (TCP) / Datagram (UDP) |
| 3. Network | **Internet** | IP (IPv4/IPv6), ICMP | Packet |
| 2. Data Link | **Network Access** | Ethernet, ARP, Wi-Fi | Frame |
| 1. Physical | **Network Access** | Cables, Fiber, Hubs | Bit |

### Data Encapsulation Process
As data travels from the sender's application down to the physical wire, each layer adds its own header (and sometimes a trailer) to the data received from the layer above.

1. **Application Layer**: Generates the raw data (e.g., an HTTP GET request).
2. **Transport Layer**: Adds source/destination ports. (Result: **Segment**).
3. **Internet Layer**: Adds source/destination IP addresses. (Result: **Packet**).
4. **Network Access Layer**: Adds source/destination MAC addresses and a frame check sequence. (Result: **Frame**).

### Transport Layer: TCP vs. UDP

#### TCP (Transmission Control Protocol)
- **Characteristics**: Connection-oriented, reliable, ordered, flow control.
- **Handshake**: Uses the 3-Way Handshake (SYN -> SYN-ACK -> ACK).
- **Use Case**: Web browsing (HTTP), Secure Shell (SSH), File Transfer (FTP).

#### UDP (User Datagram Protocol)
- **Characteristics**: Connectionless, unreliable (best-effort), low overhead, fast.
- **Handshake**: None.
- **Use Case**: Real-time video streaming, Online gaming, DNS queries, DHCP.

### Linux Networking Examples (Bash)

#### 1. Capturing the TCP 3-Way Handshake
Using `tcpdump`, we can filter for the synchronization and acknowledgment flags to see the connection establishment.

```bash
# Listen on interface eth0 for TCP SYN or ACK flags
sudo tcpdump -i eth0 -n 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'
```

#### 2. Monitoring Active Sockets
The `ss` tool is the modern replacement for `netstat` in Linux, providing detailed socket statistics.

```bash
# -t (TCP), -u (UDP), -n (Numeric), -l (Listening), -p (Processes)
ss -tunlp
```

#### 3. Inspecting the Internet Layer (IP)
Use the `ip` command from the `iproute2` suite to inspect routing and addressing.

```bash
# View IP addresses and interface status
ip addr show

# View the kernel routing table
ip route show
```

## Interview Questions

**Q: Explain the TCP 3-way handshake process.**
**A:** It is the method used to establish a reliable connection:
1. **SYN**: The client sends a Synchronize packet to the server.
2. **SYN-ACK**: The server responds with a Synchronize-Acknowledgment packet.
3. **ACK**: The client sends an Acknowledgment packet back to the server.

**Q: What is the main difference between a Packet and a Frame?**
**A:** A **Packet** is the data unit at the Internet Layer (Layer 3), containing IP addresses. A **Frame** is the data unit at the Network Access Layer (Layer 2), which wraps the packet with MAC addresses for local delivery.

**Q: Why would an application choose UDP over TCP?**
**A:** For applications where speed and low latency are more important than 100% reliability, such as VOIP, live streaming, or gaming. The lack of retransmissions and handshaking reduces overhead significantly.

**Q: What happens if a TCP segment is lost during transmission?**
**A:** The sender will not receive an acknowledgment (ACK) for that segment. After a timeout or receiving multiple duplicate ACKs for previous segments, the sender will retransmit the missing segment.

**Q: At which layer does a standard network switch operate?**
**A:** A standard Layer 2 switch operates at the **Data Link (Network Access)** layer, using MAC addresses to forward frames. A "Layer 3 Switch" or Router operates at the **Internet** layer using IP addresses.
