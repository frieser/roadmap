#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'rds']
---

## Summary
General Purpose SSD storage is the default and most cost-effective storage option for Amazon RDS, suitable for a broad range of database workloads. It comes in two generations: **gp2** (legacy) and **gp3** (recommended). While gp2 ties performance (IOPS) directly to storage size, gp3 allows for independent scaling of IOPS and Throughput, offering better flexibility and often lower costs.

## Detailed Explanation

### gp2: General Purpose SSD (Previous Generation)
- **Baseline Performance**: 3 IOPS per GiB of allocated storage.
- **Minimum/Maximum**: Minimum of 100 IOPS; scales up to 16,000 IOPS at 5,334 GiB.
- **Burstable Performance**: Volumes smaller than 1,000 GiB can burst up to **3,000 IOPS** for extended periods using an I/O credit balance.
- **I/O Credits**:
    - Credits accumulate when the database uses less than its baseline IOPS.
    - Credits are spent when the workload exceeds the baseline (up to 3,000 IOPS).
    - If the burst balance reaches zero, performance is throttled back to the baseline (3 IOPS/GiB).
- **Throughput**: Between 128 MiB/s and 250 MiB/s depending on volume size.

### gp3: General Purpose SSD (Recommended)
- **Baseline Performance**: Provides a consistent baseline of **3,000 IOPS** and **125 MiB/s** throughput regardless of storage size.
- **Decoupled Performance**: Unlike gp2, you can provision additional IOPS and Throughput independently of the storage capacity.
- **Performance Limits**:
    - **IOPS**: Up to 64,000 (Engine dependent; SQL Server max 16,000).
    - **Throughput**: Up to 4,000 MiB/s (SQL Server max 1,000 MiB/s).
- **Cost**: gp3 is typically **20% cheaper** per GiB than gp2. Users only pay for provisioned IOPS/Throughput that exceeds the baseline.
- **Striping**: For many engines (MySQL, MariaDB, PostgreSQL), RDS automatically stripes data across 4 volumes when storage is **≥ 400 GiB**, increasing baseline performance to 12,000 IOPS and 500 MiB/s.

### Comparison Table
| Feature | gp2 | gp3 |
| :--- | :--- | :--- |
| **IOPS per GiB** | 3 IOPS (fixed) | 3,000 (baseline) + provisioned |
| **Max IOPS** | 16,000 (per volume) | 64,000 |
| **Max Throughput** | 250 MiB/s | 4,000 MiB/s |
| **Bursting** | Yes (< 1 TiB) | No (Consistent baseline) |
| **Price** | Standard | ~20% cheaper per GiB |

## Interview Questions
1. **What is the main advantage of gp3 over gp2 in Amazon RDS?**
   - gp3 allows users to scale IOPS and throughput independently of storage capacity, whereas gp2 ties IOPS directly to the GiB size (3 IOPS/GiB). Additionally, gp3 is generally 20% cheaper per GiB.

2. **How does the burst credit system work in gp2?**
   - gp2 volumes under 1,000 GiB accumulate I/O credits at their baseline rate. When a burst of traffic occurs, the volume can use these credits to reach 3,000 IOPS. Once the credit balance is exhausted, the performance drops to the baseline (3 IOPS/GiB).

3. **In gp2, what size must a volume be to have a baseline of 3,000 IOPS?**
   - 1,000 GiB (since 1,000 GiB * 3 IOPS/GiB = 3,000 IOPS). At this size, the volume no longer needs to "burst" to 3,000 because its baseline performance already equals the burst limit.

4. **Is it possible to switch an existing RDS instance from gp2 to gp3?**
   - Yes, RDS supports storage type modifications online. However, while the volume is in the `optimizing` state, performance may be slightly impacted, and you cannot make further storage modifications until the process completes.

5. **When would you choose gp3 over Provisioned IOPS (io1/io2)?**
   - Choose gp3 for workloads that require moderate, consistent performance at a lower cost. Provisioned IOPS (io1/io2) is reserved for high-IOPS, low-latency production workloads (up to 256,000 IOPS) where single-digit millisecond latency must be guaranteed 99.9% of the time.
