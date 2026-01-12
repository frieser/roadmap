#Networking
#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
IP Routing is the mechanism that allows a Linux system to forward network packets toward their destination by consulting a **Routing Table**. This table maps destination networks to specific network interfaces and "Next Hop" gateways. It is a fundamental component of the TCP/IP stack, enabling both local communication and internet connectivity through a **Default Gateway**.

## Detailed Explanation

### 1. The Routing Table
The kernel maintains a routing table to determine the best path for every outgoing packet. When a packet is ready to be sent, the kernel compares the destination IP against the entries in the table, starting from the most specific prefix (longest prefix match).

#### Viewing the Routing Table
The modern way to view and manage routes is using the `ip route` command from the `iproute2` package.

```bash
# Display the main routing table
ip route show

# Common output format:
# default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.50 metric 100 
# 10.0.0.0/24 dev eth1 proto kernel scope link src 10.0.0.5 
```

### 2. Default Gateway
The **Default Gateway** is the IP address of the router the system uses when no specific route matches the destination address. It is effectively the "exit point" for all traffic destined for external networks (like the Internet).

```bash
# Add a default gateway
sudo ip route add default via 192.168.1.1 dev eth0

# Replace an existing default gateway
sudo ip route replace default via 192.168.1.254
```

### 3. Static Routes
Static routes are manually configured paths to specific networks. They are useful for connecting to internal subnets that aren't reachable via the default gateway.

```bash
# Syntax: ip route add <network>/<prefix> via <gateway> dev <interface>
sudo ip route add 172.16.0.0/16 via 10.0.0.1 dev eth1

# Delete a static route
sudo ip route del 172.16.0.0/16
```

### 4. Traceroute Logic
`traceroute` is used to map the path packets take to reach a destination. It exploits the **TTL (Time to Live)** field in the IP header.

1. **TTL = 1**: The first router receives the packet, decrements TTL to 0, discards the packet, and sends an **ICMP Time Exceeded** message back to the source.
2. **Identification**: The source records the IP address of the router from the ICMP message.
3. **Increment**: The process repeats with **TTL = 2**, then **TTL = 3**, until the destination is reached or a max hop limit is hit.

```bash
# Trace path to a destination
traceroute google.com

# Use ICMP Echo (ping) instead of UDP (standard in Linux)
traceroute -I google.com
```

### 5. Persistence
Changes made with `ip route` are **temporary** and will be lost on reboot. For persistence:
- **Ubuntu/Debian (Netplan)**: Edit `/etc/netplan/*.yaml`.
- **RHEL/CentOS (NetworkManager)**: Use `nmcli` or edit `/etc/sysconfig/network-scripts/route-<interface>`.

## Interview Questions

**Q: What does the "metric" value in a routing table represent?**
**A:** The metric is a cost value assigned to a route. If multiple routes match the same destination, the kernel chooses the one with the lowest metric. This is commonly used to prioritize a wired connection over a wireless one.

**Q: Explain the concept of "Longest Prefix Match" in routing.**
**A:** When the kernel looks up a destination in the routing table, it might find multiple matches (e.g., `10.0.0.0/8` and `10.1.2.0/24`). It will always select the route with the most specific prefix (the longest mask), as it represents a more precise path to the target.

**Q: How can you check which route a specific IP address will take without sending an actual packet?**
**A:** You can use the command `ip route get <destination_ip>`. The kernel will perform a table lookup and tell you exactly which interface and gateway it would use.

**Q: Why might `traceroute` show asterisks (***) for some hops?**
**A:** Asterisks indicate that the source did not receive a response within the timeout period. This usually happens because a router along the path is configured to drop "Time Exceeded" ICMP messages or ignore the incoming probe packets for security reasons.
