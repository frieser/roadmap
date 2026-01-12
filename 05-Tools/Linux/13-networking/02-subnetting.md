---
tags: ['linux', 'roadmap']
---

# Subnetting (CIDR, Netmask, private IP ranges)

## Summary
Subnetting is the process of partitioning a single physical network into multiple logical sub-networks (subnets). It allows organizations to organize their internal networks, improve routing efficiency, and enhance security by isolating traffic. Key concepts include **CIDR** (Classless Inter-Domain Routing) notation, **Netmasks**, and the identification of **Private IP ranges** defined by RFC 1918.

## Detailed Explanation

### **1. CIDR and Netmask**
IP addresses are 32-bit numbers (IPv4). Subnetting defines which part of the address represents the **Network ID** and which part represents the **Host ID**.

*   **Netmask**: A bitmask that "masks" the network portion. Example: `255.255.255.0`.
*   **CIDR Notation**: A shorthand way to write the netmask by counting the number of leading '1' bits. Example: `/24` is equivalent to `255.255.255.0`.

### **2. Comparing /24 and /32**
In Linux administration, you frequently encounter different subnet sizes:

*   **Standard LAN (/24)**:
    *   **Mask**: `255.255.255.0`
    *   **Capacity**: 256 total addresses.
    *   **Usable Hosts**: 254 (Total - Network ID - Broadcast).
    *   **Usage**: Common for home routers and small office networks.
*   **Single Host (/32)**:
    *   **Mask**: `255.255.255.255`
    *   **Capacity**: 1 address.
    *   **Usage**: Used for loopback interfaces, VPN end-points, or specific host routes where no routing to other neighbors is needed on that specific interface.

### **3. Private IP Ranges (RFC 1918)**
These ranges are reserved for internal use and are not routable on the public internet:
*   **10.0.0.0/8**: Large networks (16.7 million IPs).
*   **172.16.0.0/12**: Medium networks (1 million IPs).
*   **192.168.0.0/16**: Small/Home networks (65,536 IPs).

### **4. Using `ipcalc` in Linux**
The `ipcalc` tool is invaluable for calculating network boundaries.

**Example: Analyzing a /24 network**
```bash
ipcalc 192.168.1.0/24
```
**Output:**
```text
Network:    192.168.1.0/24
Netmask:    255.255.255.0 = 24
Broadcast:  192.168.1.255
Address space:  Private Use
HostMin:    192.168.1.1
HostMax:    192.168.1.254
Hosts/Net:  254
```

**Example: Checking a single host (/32)**
```bash
ipcalc 10.0.0.1/32
```

### **5. Manual Calculation Steps**
1.  **Network ID**: Set all host bits to 0.
2.  **Broadcast Address**: Set all host bits to 1.
3.  **Usable Range**: All addresses between the Network ID and Broadcast.
4.  **Number of Hosts**: $2^{(32 - \text{prefix})} - 2$.

---

## Interview Questions

**Q: What is the purpose of a Subnet Mask?**
**A:** A subnet mask is used to distinguish between the network portion and the host portion of an IP address. It determines the size of the network and how many hosts can reside within it.

**Q: How many usable hosts are available in a /27 subnet?**
**A:** A /27 subnet has 5 bits for hosts ($32 - 27 = 5$). The total number of addresses is $2^5 = 32$. Subtracting the network and broadcast addresses gives **30 usable hosts**.

**Q: What is the difference between a /31 and a /32 subnet?**
**A:** A /32 represents a single host (no broadcast or network overhead). A /31 is specifically used for point-to-point links (e.g., between two routers) where there are only two addresses, effectively using one for each end without a traditional broadcast address (RFC 3021).

**Q: What are the three main private IP address ranges?**
**A:** 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.

**Q: If a host has an IP of 192.168.1.50/24, what is its broadcast address?**
**A:** 192.168.1.255.
