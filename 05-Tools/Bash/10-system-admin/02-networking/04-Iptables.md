---
---

## Summary
`iptables` is a user-space utility program that allows a system administrator to configure the IP packet filter rules of the Linux kernel firewall. It organizes rules into **Chains** (INPUT, OUTPUT, FORWARD) and **Tables** (Filter, NAT).

## Detailed Explanation

### Chains
*   **INPUT**: Packets coming *into* the local machine.
*   **OUTPUT**: Packets leaving the local machine.
*   **FORWARD**: Packets passing *through* the machine (routing/gateway).

### Basic Usage
*   **List Rules**: `iptables -L -n -v`.
*   **Allow Port 80**: `iptables -A INPUT -p tcp --dport 80 -j ACCEPT`.
*   **Drop Everything**: `iptables -P INPUT DROP` (Default Policy).
*   **Stateful**: `iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT` (Allow responses to outgoing traffic).

### Persistence
Rules are lost on reboot. You need `iptables-save` and `iptables-restore`, or a service like `iptables-persistent`.

## Go-Specific Context/Examples

Go libraries (like `github.com/coreos/go-iptables`) wrap the `iptables` binary command to manipulate rules programmatically (often used by Kubernetes CNI plugins and Docker).

### Example: Adding a Rule in Go (Wrapper)
```go
ipt, _ := iptables.New()
err := ipt.Append("filter", "INPUT", "-p", "tcp", "--dport", "80", "-j", "ACCEPT")
```

## Interview Questions

**Q: What is the difference between `DROP` and `REJECT`?**
**A:**
*   **DROP**: Silently discards the packet. The sender gets no response and eventually times out. Good for security (stealth).
*   **REJECT**: Discards the packet but sends an ICMP "Port Unreachable" error back. Good for friendly debugging.

**Q: Why does `iptables` order matter?**
**A:** Rules are processed sequentially (top to bottom). The *first* rule that matches wins. If you have a rule that DROPS everything at the top, allowing Port 80 below it will never be reached.

**Q: What is NAT (Network Address Translation) in iptables?**
**A:** The `nat` table is used to modify Source or Destination IPs.
*   **SNAT (Source NAT)**: Changing the source IP of outgoing packets (Masquerading, used by Routers/Gateways).
*   **DNAT (Destination NAT)**: Changing the dest IP of incoming packets (Port Forwarding).
