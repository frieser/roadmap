#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'vpc']
---

## Summary
Classless Inter-Domain Routing (CIDR) is a method for allocating IP addresses and IP routing. In AWS, every Virtual Private Cloud (VPC) must be associated with an IPv4 CIDR block. This block defines the range of private IP addresses that can be assigned to resources within the VPC. AWS supports VPC CIDR block sizes between `/16` (65,536 addresses) and `/28` (16 addresses).

## Detailed Explanation
AWS Virtual Private Clouds use CIDR notation to define the network boundaries. When creating a VPC, you specify a primary IPv4 CIDR block.

### RFC 1918 Private Ranges
AWS recommends using private IP address ranges as specified in **RFC 1918** for VPC CIDR blocks to avoid conflicts with the public internet:
- **10.0.0.0/8**: Used for large networks (e.g., `10.0.0.0/16`).
- **172.16.0.0/12**: Often used for medium-sized environments (e.g., `172.16.0.0/16`).
- **192.168.0.0/16**: Commonly used for smaller or home-like networks (e.g., `192.168.0.0/24`).

### VPC Sizing and Subnets
- **Size Limits**: The allowed block size is between `/16` (65,536 IPs) and `/28` (16 IPs).
- **Secondary CIDRs**: If the primary CIDR is exhausted, you can associate secondary CIDR blocks (up to 5 per VPC by default).
- **AWS Reserved Addresses**: In every subnet CIDR block, AWS reserves 5 IP addresses:
    1.  `x.x.x.0`: Network address.
    2.  `x.x.x.1`: VPC router.
    3.  `x.x.x.2`: Amazon DNS server (AmazonProvidedDNS).
    4.  `x.x.x.3`: Reserved by AWS for future use.
    5.  `x.x.x.255`: Network broadcast address (AWS does not support broadcast, but reserves it).

### Go Context
In Go, the `net` package is used to handle CIDR blocks and IP parsing. This is useful when writing custom networking tools or automation scripts using the AWS SDK for Go.

```go
package main

import (
    "fmt"
    "net"
)

func main() {
    cidr := "10.0.0.0/16"
    ip, ipnet, err := net.ParseCIDR(cidr)
    if err != nil {
        fmt.Printf("Error parsing CIDR: %v\n", err)
        return
    }
    
    fmt.Printf("IP Address: %s\n", ip)
    fmt.Printf("Network: %s\n", ipnet.IP)
    fmt.Printf("Subnet Mask: %s\n", ipnet.Mask)
    
    // Check if an IP is within the CIDR block
    testIP := net.ParseIP("10.0.1.5")
    if ipnet.Contains(testIP) {
        fmt.Printf("%s is inside %s\n", testIP, cidr)
    }
}
```

## Interview Questions
**Q: What is the minimum and maximum size for a VPC CIDR block in AWS?**
**A:** The allowed block size is between a `/28` netmask (16 IP addresses) and a `/16` netmask (65,536 IP addresses).

**Q: How many IP addresses does AWS reserve in each subnet?**
**A:** AWS reserves 5 IP addresses in each subnet: `.0` (Network address), `.1` (VPC router), `.2` (DNS server), `.3` (Reserved by AWS for future use), and `.255` (Network broadcast address).

**Q: Can you change the primary CIDR block of a VPC after it has been created?**
**A:** No, the primary CIDR block cannot be changed. However, you can associate secondary CIDR blocks with the VPC if additional address space is needed.

**Q: Why should you avoid overlapping CIDR blocks when connecting VPCs?**
**A:** Overlapping CIDR blocks cause routing conflicts. If you use VPC Peering, Transit Gateway, or VPN/Direct Connect to connect networks, each network must have a unique IP range to ensure packets reach the correct destination.
