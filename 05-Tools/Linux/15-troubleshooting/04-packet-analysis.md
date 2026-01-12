#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
**Packet Analysis** is the process of capturing and inspecting data packets as they traverse a network interface. It is a critical skill for troubleshooting network connectivity issues, analyzing protocol behavior, and identifying security threats. Tools like `tcpdump` (CLI) and **Wireshark** (GUI) are industry standards for capturing traffic into **PCAP** (Packet Capture) files and applying filters to isolate specific communications.

## Detailed Explanation

### 1. Introduction to tcpdump
`tcpdump` is a powerful command-line packet analyzer that uses the `libpcap` library to capture network traffic. It is essential for servers where a GUI is unavailable.

#### Common Flags
- `-i [interface]`: Specifies the network interface to listen on (e.g., `eth0`, `any`).
- `-n`: Disables name resolution (IP addresses instead of hostnames). Faster and avoids extra DNS traffic.
- `-nn`: Disables both name and port resolution (e.g., `80` instead of `http`).
- `-v`, `-vv`, `-vvv`: Increases the level of verbosity.
- `-X`: Displays packet content in both Hex and ASCII.
- `-c [count]`: Captures a specific number of packets and then exits.
- `-w [file.pcap]`: Writes the captured packets to a file for later analysis.
- `-r [file.pcap]`: Reads packets from a previously saved PCAP file.

#### Bash Examples (tcpdump)
```bash
# Capture traffic on eth0 without name resolution
tcpdump -i eth0 -n

# Capture only HTTP traffic (port 80)
tcpdump -i any port 80 -n

# Capture traffic from a specific source IP
tcpdump src 192.168.1.100 -n

# Save the first 100 packets of port 443 traffic to a file
tcpdump -i eth0 port 443 -c 100 -w output.pcap

# Read from a file and display in Hex/ASCII
tcpdump -r output.pcap -X
```

### 2. Filtering with BPF (Berkeley Packet Filter)
`tcpdump` uses BPF syntax for filtering. Filters can be combined using logical operators:
- `and` (or `&&`)
- `or` (or `||`)
- `not` (or `!`)

**Example:**
```bash
# Capture traffic that is NOT SSH and comes from a specific network
tcpdump not port 22 and net 10.0.0.0/24
```

### 3. PCAP Files
The **PCAP** format is the standard for network traffic storage.
- **Interoperability**: A file captured with `tcpdump` can be opened in Wireshark, `tshark`, or any tool supporting `libpcap`.
- **pcapng**: The next-generation format which supports multiple interfaces and enhanced metadata.

### 4. Wireshark Display Filters
While `tcpdump` uses "Capture Filters" (to limit what is saved), Wireshark primarily uses "Display Filters" to hide/show packets in its GUI.

| Protocol | Filter Example | Description |
| :--- | :--- | :--- |
| **IP** | `ip.addr == 1.2.3.4` | Traffic involving specific IP |
| **TCP** | `tcp.port == 443` | HTTPS traffic |
| **HTTP** | `http.request.method == "POST"` | Show only POST requests |
| **Flags** | `tcp.flags.syn == 1` | Find connection attempts |
| **Logic** | `!(ip.addr == 10.0.0.1)` | Exclude specific host |

## Interview Questions

**Q: What is the difference between a capture filter and a display filter?**
**A:** A capture filter (used by `tcpdump` and Wireshark's capture engine) determines which packets are actually saved to the buffer/disk, reducing CPU and storage overhead. A display filter (used in Wireshark) is applied to already captured packets to narrow down the view without deleting the underlying data.

**Q: Why should you use the `-n` flag with `tcpdump` in a production environment?**
**A:** Using `-n` prevents `tcpdump` from performing reverse DNS lookups for every captured IP. This reduces the load on the system, prevents `tcpdump` from generating its own "extra" network traffic, and ensures the tool remains responsive during high traffic bursts.

**Q: How do you capture only the TCP SYN packets (connection attempts) using tcpdump?**
**A:** You can use the flag-specific filter: `tcpdump 'tcp[tcpflags] & (tcp-syn) != 0'`. This looks at the TCP header flags directly to isolate synchronization packets.

**Q: How can you transfer a capture from a remote headless server to your local machine for analysis?**
**A:** Capture the traffic to a file using `tcpdump -w remote.pcap`, then use `scp` or `sftp` to download the file to your local machine and open it with Wireshark. Alternatively, you can pipe `tcpdump` output over SSH directly into Wireshark.
