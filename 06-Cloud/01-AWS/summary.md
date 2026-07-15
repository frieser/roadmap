---
tags: ['aws', 'roadmap', 'summary']
---

## Foundations
| Model | Customer | AWS |
| :--- | :--- | :--- |
| IaaS (EC2) | OS, runtime, app, data | HW, virt, net |
| PaaS (Lambda) | App, data | OS, runtime, infra |
| SaaS (WorkSpaces) | Data usage | Everything |

| Deployment | Ctrl | Scale | Cost |
| :--- | :---: | :---: | :--- |
| Public | Low | V.High | OpEx |
| Private/Outposts | V.High | Low | CapEx |
| Hybrid | Med | High | Balanced |

5 NIST: Self-service, Broad access, Pooling, Elasticity, Measured | Shared: AWS=security OF cloud, Customer=security IN cloud
W-A pillars: Ops Excellence, Security, Reliability, Performance, Cost Opt, Sustainability | Global: 38 Regions, 120+ AZs, 700+ Edge

## EC2
| Fam | Use | Note |
| :--- | :--- | :--- |
| T | Burstable | Credits, Unlimited default |
| M | Balanced | Fixed perf |
| C | Compute | High vCPU:RAM |
| R/X | Memory | Large RAM |
| P/G/Trn | GPU/ML | Trainium, Inferentia |

EBS (persistent, snapshots) vs Instance Store (ephemeral, local NVMe)
| Buy | Save | Interrupt |
| :--- | :--- | :--- |
| On-Demand | 0% | No |
| Spot | ≤90% | 2-min |
| Reserved | ≤75% | No |
| Savings | ≤72% | No |

EIP: $0.005/hr, 5/region; User Data: 16KB, first-boot; IMDSv2 mandatory

## VPC
CIDR `/16`–`/28`, RFC 1918; 5 IPs reserved/subnet | Public = `0.0.0.0/0→IGW`, Private = no IGW, outbound via NAT GW
SG: stateful, ENI, allow only | NACL: stateless, subnet, allow+deny | IGW: 1/VPC, 1:1 NAT | NAT GW: public subnet, EIP, 1/AZ

## IAM
Identity-based (user/group/role) vs Resource-based (explicit Principal) | Eval: Deny > Allow > Implicit
Groups: no nesting, ≤10/user | Roles: trust (who) + permissions (what); STS temp creds 15m–12h
Instance Profile: 1 role→EC2; IMDSv2 token mandatory | Cross-account: both sides need policy

## S3
Strong consistency all ops; max 5TB obj; multipart >5GB
| Class | Access | Min | Days | AZs |
| :--- | :--- | :--- | :--- | :--- |
| Standard | ms | — | — | ≥3 |
| Int-Tiering | ms | — | 30 | ≥3 |
| Standard-IA | ms | 128KB | 30 | ≥3 |
| One Zone-IA | ms | 128KB | 30 | 1 |
| Glacier Instant | ms | 128KB | 90 | ≥3 |
| Glacier Flexible | min–hr | 40KB | 90 | ≥3 |
| Glacier Deep | 12–48h | 40KB | 180 | ≥3 |

11 nines; Lifecycle = transitions + expiration; OAC for CF

## RDS
Aurora, PG, MySQL, MariaDB, Oracle, SQL Server, Db2 | gp3 (16K–64K IOPS) / io2 Block (256K, sub-ms)
Multi-AZ: sync standby, auto CNAME failover | Read Replicas: async, ≤5, cross-region
Backup: auto (0–35d PITR) vs manual snapshots (indefinite, sharable)

## DynamoDB
Item 400KB | Partition: 3K RCU / 1K WCU / 10GB | Query/Scan 1MB paginated
LSI: same PK, ≤5, 10GB coll, at-create only | GSI: diff PK, ≤20, own RCU/WCU
Streams: 24h, 4 views | Capacity: Provisioned (+AutoScale) or On-Demand | Backup: snapshots + PITR 35d
Single Table: access patterns first; 1:N=same PK; M:N=adjacency list+GSI

## Lambda
Timeout 15m | Mem 128MB–10GB | Sync 6MB/Async 256KB | /tmp ≤10GB | Pkg 50MB zip/250MB unzip | Concur 1K
Cold start = Init; Warm = reuse; Provisioned Concurrency=no cold | Layers ≤5, mount `/opt`
Versions: $LATEST(mutable)→publish(immutable); Aliases→weighted canary; API GW: REST vs HTTP (cheaper)
Lambda@Edge: 4 CF triggers, us-east-1 only, Node/Python only

## ECS
Cluster→Service(desired count)→Task(Task Def blueprint) | Task Role=container, Exec Role=agent
Fargate: serverless, awsvpc, per-task ENI, pay/vCPU-mem-sec | Capacity Provider: ASG via `CapacityProviderReservation`

## ASG & ELB
Policies: Target Tracking, Step, Simple(cooldown), Predictive(ML)
ALB: L7, path/host routing, WAF | NLB: L4, static IP/AZ, millions RPS | CLB: legacy

## R53 & CF
R53: Simple, Weighted, Latency, Failover, Geo, Geoproximity, Multivalue | Alias free for AWS, apex OK
CF: OAC for S3; Cache/OriginReq/RespHeaders policies; invalidation 1K free/mo; versioning preferred

## SES & CW & ElastiCache
SES: sandbox→production; DKIM 3 CNAME; bounce <5%/complaint <0.1% thresholds
CW: Metrics(1m/1s) + Logs(Groups→Streams) + EventBridge(schedules+rules); retention 1m=15d, 5m=63d, 1h=455d
ElastiCache: Redis(types,persist,HA,512MB) vs Memcached(strings,no persist,multi-thread,1MB); 500 nodes/reg

## Rules
1. DynamoDB: model access patterns first, not data structure
2. OAC for S3+CF, lock buckets | 3. 1 NAT GW/AZ for private subnet HA
4. GP3 > GP2: independent IOPS, 20% cheaper | 5. IMDSv2 everywhere
6. Versioning > CF invalidation | 7. Narrow CF cache key, Origin Req Policy for forwarding
8. Lambda: init outside handler, pkg<50MB | 9. Explicit Deny always wins | 10. Spot+mixed ASG for cost
