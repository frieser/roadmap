#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'vpc']
---

## Summary
A **Route Table** is a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed. Every VPC has a **Main Route Table**, and you can create additional **Custom Route Tables**. Each subnet in your VPC must be associated with a route table; if a subnet is not explicitly associated with one, it is automatically associated with the main route table.

## Detailed Explanation

### Route Table Components
Each route in a table consists of two main parts:
- **Destination**: The CIDR block of the network you want traffic to reach (e.g., `0.0.0.0/0` for the internet).
- **Target**: The resource through which the traffic should be sent (e.g., `local`, `igw-12345`, `nat-12345`).

### Main vs. Custom Route Tables
- **Main Route Table**: 
    - Automatically created with your VPC.
    - Acts as the default for any subnet that isn't explicitly associated with another table.
    - You can modify it, but you cannot delete it.
- **Custom Route Table**:
    - Created manually to have granular control over subnet traffic.
    - Highly recommended to keep the main route table "clean" and use custom tables for public and private subnets.

### Subnet Associations
- A route table can be associated with multiple subnets.
- However, a subnet can be associated with **only one** route table at a time.
- If no association is defined, the subnet defaults to the Main Route Table.

### The "Local" Route
Every route table contains a default route for communication within the VPC (e.g., `10.0.0.0/16 -> local`). 
- This route allows all resources within the VPC to talk to each other.
- It is added automatically and **cannot be deleted**.
- It always has the highest priority unless a more specific route exists.

### Priority and Matching
AWS uses the **Longest Prefix Match** rule. If multiple routes match a destination, the most specific route (the one with the longest prefix/smallest CIDR) is chosen.

## Go Context
In Go, you can use the AWS SDK to programmatically manage route tables. This is common in Infrastructure as Code (IaC) or automation tools.

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/aws/aws-sdk-go-v2/config"
    "github.com/aws/aws-sdk-go-v2/service/ec2"
)

func main() {
    // Load the Shared AWS Configuration (~/.aws/config)
    cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
    if err != nil {
        log.Fatalf("unable to load SDK config, %v", err)
    }

    // Create an EC2 client
    client := ec2.NewFromConfig(cfg)

    // Describe Route Tables
    input := &ec2.DescribeRouteTablesInput{}
    result, err := client.DescribeRouteTables(context.TODO(), input)
    if err != nil {
        log.Fatalf("failed to describe route tables, %v", err)
    }

    fmt.Println("Route Tables:")
    for _, rt := range result.RouteTables {
        fmt.Printf("- ID: %s (VPC: %s)\n", *rt.RouteTableId, *rt.VpcId)
        for _, route := range rt.Routes {
            dest := "N/A"
            if route.DestinationCidrBlock != nil {
                dest = *route.DestinationCidrBlock
            }
            fmt.Printf("  Route: %s -> %s\n", dest, *route.State)
        }
    }
}
```

## Interview Questions

**Q: What happens if a subnet is not explicitly associated with a route table?**
**A:** It is automatically associated with the VPC's Main Route Table.

**Q: Can a subnet be associated with multiple route tables simultaneously?**
**A:** No, a subnet can only be associated with one route table at a time. However, a single route table can be associated with multiple subnets.

**Q: Can you delete the "local" route in a VPC route table?**
**A:** No, the local route is created by default to allow communication within the VPC and cannot be deleted or modified.

**Q: How does AWS decide which route to use if there are multiple matches?**
**A:** AWS uses the "Longest Prefix Match" (LPM) rule. The most specific route (the one with the smallest CIDR range that matches the destination IP) is preferred.

**Q: What is the difference between the Main Route Table and a Custom Route Table?**
**A:** The Main Route Table is created by default and serves as the catch-all for unassociated subnets. Custom Route Tables are created by users to define specific routing logic, such as directing traffic to an Internet Gateway for public subnets or a NAT Gateway for private subnets.
