#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'route53']
---

## Summary
A **Route 53 Hosted Zone** is a container that stores DNS records for a specific domain (e.g., `example.com`). It serves as the authoritative source for DNS queries, translating human-readable domain names into IP addresses or other AWS resources. Route 53 distinguishes between **Public Hosted Zones**, which route internet traffic, and **Private Hosted Zones**, which route traffic within one or more Amazon VPCs.

## Detailed Explanation

### 1. Public Hosted Zones
- **Definition**: A container that holds records for how you want to route traffic on the internet.
- **Functionality**: When you register a domain or transfer DNS management to Route 53, a public hosted zone is created. It is accessible globally via the internet.
- **Name Servers**: Route 53 automatically assigns four authoritative name servers (spread across different TLDs like `.com`, `.net`, `.org`) to ensure high availability.

### 2. Private Hosted Zones
- **Definition**: A container that holds records for how you want to route traffic within one or more Amazon VPCs.
- **Functionality**: Queries stay within the AWS network and are not visible to the public internet.
- **Requirements**: Requires `enableDnsHostnames` and `enableDnsSupport` to be set to `true` in the VPC settings.
- **Use Case**: Internal service discovery (e.g., `db.internal.example.com`) and secure communication between microservices.

### 3. Record Sets
Hosted zones contain **Resource Record Sets** (RRsets):
- **A Record**: Maps a domain to an IPv4 address.
- **AAAA Record**: Maps a domain to an IPv6 address.
- **CNAME**: Maps a domain to another domain (cannot be used for the zone apex).
- **Alias Record**: An AWS-specific extension that allows mapping a domain (including the zone apex) to AWS resources (S3, ELB, CloudFront) without additional DNS queries.
- **MX Record**: Specifies mail servers for the domain.
- **TXT Record**: Often used for domain verification (e.g., SPF, DKIM).

### 4. NS and SOA Records
Every hosted zone automatically includes these two records:
- **NS (Name Server)**: Identifies the four authoritative name servers for the zone.
- **SOA (Start of Authority)**: Provides technical information about the zone, including:
    - **MNAME**: Primary name server.
    - **RNAME**: Email of the administrator.
    - **Serial Number**: Versioning of the zone file.
    - **Refresh/Retry/Expire/Minimum TTL**: Timings for secondary name servers to sync.

### 5. Split-View DNS (Overlapping Zones)
You can maintain a **Split-View DNS** by creating both a public and a private hosted zone with the **same domain name**.
- **Behavior**: An EC2 instance within an associated VPC will prioritize the **Private Hosted Zone**. If a record isn't found in the private zone, the query fails (it does not automatically "fall back" to the public zone unless configured via Resolver rules).
- **Benefit**: Allows internal users to see different content/IPs than external users (e.g., `api.example.com` points to an internal ALB for employees and a public ALB for customers).

### 6. Route 53 Resolver
- **Inbound Endpoints**: Allows on-premises DNS servers to query Route 53 private hosted zones.
- **Outbound Endpoints**: Allows Route 53 to forward DNS queries to on-premises DNS servers.

## Interview Questions

### Q1: What is the difference between a CNAME and an Alias record?
**A**: A **CNAME** maps a domain to another domain and cannot be used for the zone apex (e.g., `example.com`). An **Alias** record is an AWS-specific feature that can map the zone apex to AWS resources (like an ELB or S3 bucket) and does not incur additional charges for queries to most AWS resources.

### Q2: How does Route 53 handle queries if you have both a Public and Private hosted zone with the same name?
**A**: This is known as **Split-view DNS**. If a VPC is associated with the private hosted zone, the Route 53 Resolver will always look at the private zone first. If the record exists there, it returns it. If the record does *not* exist in the private zone, it returns `NXDOMAIN` even if the record exists in the public zone.

### Q3: What VPC settings are required for Private Hosted Zones to work?
**A**: You must enable both `enableDnsHostnames` and `enableDnsSupport` within the VPC configuration.

### Q4: Can you associate a Private Hosted Zone with a VPC in a different AWS account?
**A**: **Yes**. This requires a two-step process: the owner of the hosted zone must authorize the association using the AWS CLI/SDK, and then the owner of the VPC must accept the association.

### Q5: What is the purpose of the SOA record in Route 53?
**A**: The **SOA (Start of Authority)** record contains authoritative information about the DNS zone, such as the primary name server, the administrator's email, and various timers (TTL, refresh, retry) that control how long DNS records are cached and how often secondary servers should update.
