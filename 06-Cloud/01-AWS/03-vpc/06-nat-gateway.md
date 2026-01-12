#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'vpc']
---

## Summary
A **NAT (Network Address Translation) Gateway** is a managed AWS service that enables instances in a private subnet to connect to the internet or other AWS services (outbound traffic) while preventing the internet from initiating a connection with those instances (inbound traffic). It is a highly available, scalable, and more reliable alternative to self-managed NAT instances.

## Detailed Explanation

### **Public vs. Private NAT Gateway**
*   **Public NAT Gateway**: Used for outbound internet access. It must be created in a **public subnet** (a subnet with a route to an Internet Gateway). It requires an **Elastic IP (EIP)** address. Traffic from private instances is routed to the NAT Gateway, which translates the private IP to its public EIP before sending it to the Internet Gateway.
*   **Private NAT Gateway**: Used for outbound access to other VPCs or on-premises networks (via Transit Gateway or Virtual Private Gateway) without using the public internet. It does **not** require an EIP and cannot connect to an Internet Gateway (traffic will be dropped).

### **Cost**
*   NAT Gateway pricing involves two main components: **Hourly charge** (per NAT Gateway created and active) and **Data processing charge** (per GB of data that passes through the gateway).
*   Because charges are per NAT Gateway, it is often a significant cost factor in VPC design, especially with multi-AZ setups.

### **Redundancy and High Availability**
*   A NAT Gateway is redundant within a single **Availability Zone (AZ)**. It can handle up to 45 Gbps of bandwidth and scales automatically.
*   **Critical Best Practice**: For multi-AZ fault tolerance, you should create one NAT Gateway in each AZ where you have private subnets and route the traffic from those subnets to the NAT Gateway in the same AZ. If the AZ where the NAT Gateway resides fails, resources in other AZs will still have outbound access through their respective local NAT Gateways.

### **NAT Gateway vs. NAT Instance**
*   **Managed**: AWS manages NAT Gateway (patching, scaling, HA). NAT Instances are EC2 instances managed by the user.
*   **Bandwidth**: NAT Gateway scales up to 100 Gbps (burst); NAT Instance depends on the instance type.
*   **Security Groups**: NAT Gateways do not support Security Groups; they use Network ACLs. NAT Instances support Security Groups.

## Interview Questions

1.  **Q: What is the main difference between a Public and a Private NAT Gateway?**
    *   **A**: A Public NAT Gateway requires an Elastic IP and is placed in a public subnet to provide internet access to private instances. A Private NAT Gateway does not use an Elastic IP and is used to route traffic to other VPCs or on-premises networks without passing through the public internet.

2.  **Q: How do you achieve high availability for NAT Gateways across multiple Availability Zones?**
    *   **A**: Since a NAT Gateway is only highly available within a single AZ, you must deploy one NAT Gateway in each AZ where you have private subnets. You then configure the route tables of each private subnet to point to the NAT Gateway in its own AZ.

3.  **Q: Why would you choose a NAT Gateway over a NAT Instance?**
    *   **A**: NAT Gateway is a managed service, meaning AWS handles scaling, availability, and maintenance. It offers higher bandwidth (up to 45-100 Gbps) and is generally more reliable than a single NAT Instance which is a single point of failure unless manually configured for HA.

4.  **Q: Do NAT Gateways support Security Groups?**
    *   **A**: No, NAT Gateways do not support Security Groups. You must use Network Access Control Lists (NACLs) to control traffic at the subnet level where the NAT Gateway is located, or Security Groups on the source instances.
