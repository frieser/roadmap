---
tags: ['linux', 'roadmap']
---

# Ethernet and ARP

## Summary
Ethernet is the standard physical and data link layer technology for wired local area networks (LANs), identifying devices via unique hardware MAC addresses. The **Address Resolution Protocol (ARP)** is a critical networking protocol used to map an IP address (Layer 3) to a physical MAC address (Layer 2) within a local network segment.

## Detailed Explanation

### Ethernet and the MAC Address
Ethernet operates at **Layer 1 (Physical)** and **Layer 2 (Data Link)** of the OSI model. It defines how data is formatted and transmitted over the wire.

*   **MAC Address (Media Access Control)**: A 48-bit unique identifier burned into the Network Interface Card (NIC). It is usually represented as six groups of two hexadecimal digits (e.g., `52:54:00:12:34:56`).
*   **Ethernet Frame**: The unit of data at Layer 2. It contains the Source MAC, Destination MAC, EtherType (to identify the Layer 3 protocol, usually 0x0800 for IPv4), the payload (data), and a Frame Check Sequence (FCS) for error detection.

#### Identifying Interfaces in Linux
To see the MAC address of your network interfaces:

```bash
# View link-layer information
ip link show
```

### Address Resolution Protocol (ARP)
In an IPv4 network, when a host wants to communicate with another host on the same subnet, it needs the destination MAC address. ARP performs this discovery.

#### The ARP Process:
1.  **ARP Request**: The sender sends a broadcast frame (`ff:ff:ff:ff:ff:ff`) asking: *"Who has 192.168.1.10? Tell 192.168.1.5."*
2.  **ARP Reply**: The host with the matching IP responds with a unicast frame: *"I have 192.168.1.10, my MAC is 00:11:22:33:44:55."*
3.  **Caching**: To improve efficiency, the mapping is stored in the **ARP Cache** (or Neighbor Table).

#### Managing the ARP Table with `ip neigh`
The `iproute2` suite provides the `ip neigh` command to manage the neighbor table.

```bash
# List all neighbors (ARP table)
ip neigh show

# Output flags:
# REACHABLE: The entry is valid and recently verified.
# STALE: The entry is valid but hasn't been verified recently.
# DELAY: Waiting for a reachability confirmation.

# Manually add a static ARP entry (useful for security or debugging)
sudo ip neigh add 192.168.1.100 lladdr 00:11:22:33:44:55 dev eth0

# Delete an ARP entry
sudo ip neigh del 192.168.1.100 dev eth0

# Flush the entire ARP table for an interface
sudo ip neigh flush dev eth0
```

### RARP and Modern Alternatives
**RARP (Reverse ARP)** was used by diskless systems to find their IP address based on their MAC address.
*   **Legacy**: RARP is obsolete and has been replaced by **BOOTP** and eventually **DHCP**.
*   **DHCP Advantages**: Unlike RARP, DHCP operates at the Application Layer (over UDP) and can provide the gateway, DNS servers, and other configuration parameters.

## Interview Questions

**Q: In which OSI layer does ARP operate?**
**A:** ARP is often described as operating at **Layer 2 (Data Link Layer)** because it uses Ethernet broadcasts and works with MAC addresses, but it is effectively the "glue" that interfaces between Layer 2 and Layer 3 (Network Layer).

**Q: What is "ARP Spoofing" (or ARP Poisoning)?**
**A:** It is a technique where an attacker sends falsified ARP messages onto a local network. This links the attacker's MAC address with the IP address of a legitimate server (like the default gateway), allowing the attacker to intercept, modify, or stop data traffic (Man-in-the-Middle attack).

**Q: How does a host determine if it needs to use ARP for a destination IP?**
**A:** The host performs a bitwise AND operation on its own IP and subnet mask, and then on the destination IP and its subnet mask. If the resulting network addresses match, the destination is on the **local** network, and the host uses ARP. If they don't match, the host sends the packet to its **default gateway** (using ARP to find the gateway's MAC).

**Q: What happens if an ARP request is sent but no device replies?**
**A:** The sending host will be unable to encapsulate the IP packet into an Ethernet frame. This usually results in a "No route to host" or "Host unreachable" error at the application layer after a timeout.
