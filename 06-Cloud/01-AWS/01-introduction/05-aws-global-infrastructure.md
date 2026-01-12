---
tags: ['aws', 'roadmap']
---

## Summary
The **AWS Global Infrastructure** is a massive, multi-layered architecture designed to provide high availability, fault tolerance, and low-latency access to cloud services worldwide. It is primarily composed of **Regions**, which contain multiple **Availability Zones (AZs)**, and a vast network of **Points of Presence** (Edge Locations). This structure allows customers to deploy applications globally while maintaining strict data residency and compliance standards.

## Detailed Explanation

### 1. AWS Regions
An **AWS Region** is a physical geographic area that hosts clusters of data centers. Each region is completely independent and isolated to ensure fault tolerance.
- **Isolation**: Failures in one region do not impact others.
- **Selection Factors**:
    - **Compliance**: Adherence to local data sovereignty laws (e.g., GDPR in Europe).
    - **Latency**: Deploying resources closest to the end-users to reduce round-trip time.
    - **Service Availability**: Some new or specialized services might only be available in specific regions (e.g., `us-east-1`).
    - **Pricing**: Costs for compute and storage vary by region due to local taxes, electricity, and operational costs.
- **Quantity**: As of early 2026, AWS spans 38+ regions with over 120 Availability Zones globally.

### 2. Availability Zones (AZs)
An **Availability Zone** is one or more discrete data centers with redundant power, networking, and connectivity.
- **Design**: Every region has at least **3 AZs**.
- **Connectivity**: AZs within a region are connected via high-speed, ultra-low-latency private fiber (allowing synchronous replication).
- **Fault Isolation**: They are physically separated (typically by many kilometers) to protect against localized disasters like fires or floods, but close enough to support low-latency applications.
- **Best Practice**: Always architect for **Multi-AZ** to ensure high availability (HA).

### 3. Points of Presence (Edge)
To optimize content delivery, AWS uses a global network of **Points of Presence**.
- **Edge Locations**: Data centers used by **CloudFront** (CDN) to cache content and **Route 53** (DNS) to resolve queries.
- **Regional Edge Caches**: Mid-tier cache layers located between the Origin and Edge Locations to reduce the load on origins and speed up content delivery.
- **Security**: Services like **AWS Shield** and **WAF** operate at the edge to block attacks before they reach the main infrastructure.

### 4. Extensions of the Global Infrastructure
- **AWS Local Zones**: Brings AWS compute, storage, and database services closer to large population centers for single-digit millisecond latency (e.g., for gaming or media rendering).
- **AWS Wavelength**: Embeds AWS services into **5G networks** to provide ultra-low latency for mobile applications.
- **AWS Outposts**: Hardware racks provided by AWS to run AWS services **on-premises** or in a co-location facility for hybrid cloud consistency.

## Interview Questions

**Q: What is the main difference between a Region and an Availability Zone?**
**A:** A Region is a physical geographic location (like Ireland or Tokyo) that contains multiple Availability Zones. An AZ is a cluster of data centers within a Region. You choose a Region for proximity or compliance, and you use multiple AZs within that Region for high availability and fault tolerance.

**Q: Why does AWS recommend a multi-AZ architecture?**
**A:** By distributing resources across multiple AZs, you eliminate single points of failure. If one data center (AZ) suffers an outage (power, hardware, or natural disaster), the other AZs in the same Region continue to operate, ensuring the application remains available.

**Q: How do Edge Locations improve application performance?**
**A:** Edge Locations are used by Amazon CloudFront to cache static and dynamic content closer to end-users. This reduces the physical distance data must travel, significantly lowering latency and improving the user experience for global audiences.

**Q: When would you use an AWS Local Zone instead of a standard AWS Region?**
**A:** You use Local Zones when your application requires ultra-low latency (single-digit milliseconds) for users in a specific geographic area where a full AWS Region is not physically close enough. Common use cases include real-time gaming, financial trading, and live video production.

**Q: What are the four main criteria for selecting an AWS Region?**
**A:** 1. **Data Sovereignty/Compliance** (legal requirements), 2. **Proximity to users** (Latency), 3. **Service Availability** (feature set), and 4. **Cost** (variable pricing across regions).
