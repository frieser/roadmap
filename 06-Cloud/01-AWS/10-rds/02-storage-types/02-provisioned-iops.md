#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'rds']
---

## Summary
Amazon RDS Provisioned IOPS (PIOPS) storage is an SSD-backed tier designed for I/O-intensive, business-critical database workloads. It provides predictable, low-latency performance by allowing users to provision a specific number of I/O operations per second (IOPS) independently of storage capacity. The current recommendation is **io2 Block Express**, which offers sub-millisecond latency and 100x higher durability than the legacy **io1** type.

## Detailed Explanation

### io2 Block Express vs. io1
*   **io2 Block Express (Recommended)**: The next-generation PIOPS storage. It provides up to **256,000 IOPS** and **4,000 MiB/s** throughput. It features **sub-millisecond average latency** and is designed for 99.999% durability. It supports a significantly higher IOPS-to-GiB ratio (up to 1,000:1 on Nitro instances).
*   **io1 (Previous Generation)**: The legacy provisioned IOPS tier. While it also supports up to 256,000 IOPS on specific instance types (like R5b), it typically provides single-digit millisecond latency and 99.9% durability. The maximum IOPS-to-GiB ratio is 50:1.

### Performance Characteristics
| Feature | io2 Block Express | io1 |
| :--- | :--- | :--- |
| **Max IOPS** | 256,000 | 256,000 |
| **Max Throughput** | 4,000 MiB/s | 4,000 MiB/s |
| **Latency** | Sub-millisecond | Single-digit millisecond |
| **Durability** | 99.999% | 99.9% |
| **IOPS:GiB Ratio** | Up to 1,000:1 (Nitro) | Up to 50:1 |

### Critical Considerations

#### 1. Instance Class Limits
Storage performance is strictly governed by the **DB instance class**. Each instance type has a maximum "EBS-optimized" bandwidth and IOPS limit. If you provision 100,000 IOPS on a `db.m5.large`, the instance will bottleneck at its much lower hardware limit (approx. 18,750 IOPS). Always verify the instance specs before over-provisioning storage.

#### 2. The Nitro System Requirement
To unlock the full potential of **io2 Block Express** (including sub-millisecond latency and the 1,000:1 ratio), the DB instance must be based on the **AWS Nitro System** (e.g., M5, M6g, R5, R6g, X2g).

#### 3. Dedicated Log Volume (DLV)
For write-heavy OLTP workloads, PIOPS supports a **Dedicated Log Volume**. This feature moves PostgreSQL transaction logs or MySQL/MariaDB redo/binary logs to a separate 1 TiB volume with 3,000 IOPS. This reduces I/O contention on the primary data volume and ensures more consistent write latencies.

#### 4. Cost Model
Unlike General Purpose (gp3), where a baseline performance is included in the storage price, PIOPS follows a "pay for what you provision" model. You are charged for 100% of the Provisioned IOPS and the storage GiB, regardless of whether the database is actively using those resources.

### Go Implementation Example
The following snippet uses the **AWS SDK for Go v2** to programmatically inspect the storage type and IOPS configuration of your RDS instances.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/rds"
)

func main() {
	ctx := context.TODO()
	// Load AWS configuration (default credentials/region)
	cfg, err := config.LoadDefaultConfig(ctx)
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := rds.NewFromConfig(cfg)

	// Describe all DB instances
	input := &rds.DescribeDBInstancesInput{}
	result, err := client.DescribeDBInstances(ctx, input)
	if err != nil {
		log.Fatalf("failed to describe instances, %v", err)
	}

	for _, db := range result.DBInstances {
		fmt.Printf("DB Identifier: %s\n", *db.DBInstanceIdentifier)
		fmt.Printf("  Storage Type: %s\n", *db.StorageType)
		
		if db.Iops != nil {
			fmt.Printf("  Provisioned IOPS: %d\n", *db.Iops)
		}
		
		if db.StorageThroughput != nil {
			fmt.Printf("  Throughput: %d MiB/s\n", *db.StorageThroughput)
		}
		fmt.Println("--------------------------------")
	}
}
```

## Interview Questions

**Q: When should you choose io2 Block Express over gp3 storage for an RDS instance?**
**A:** Choose io2 Block Express when your workload requires **sub-millisecond latency**, sustained IOPS exceeding **64,000**, or if the application is highly sensitive to I/O variance. It is also the correct choice for business-critical systems where the **99.999% durability** of io2 is a requirement over the 99.9% of gp3.

**Q: What is the "IOPS to GiB ratio" and why is it superior in io2?**
**A:** This ratio determines how much IOPS you can provision relative to the volume size. **io1** is capped at **50:1**, meaning you need a large (and expensive) volume to get high IOPS. **io2 Block Express** allows up to **1,000:1**, enabling you to provision high performance on small data volumes, significantly optimizing costs for small but high-throughput databases.

**Q: Why might an RDS instance fail to reach its provisioned IOPS target?**
**A:** The primary reason is **Instance Level Limits**. Each DB instance class has a maximum throughput and IOPS cap. Other factors include:
1. **Network Bandwidth**: The instance may reach its EBS-optimized throughput limit first.
2. **I/O Size**: If I/O operations are larger than 16 KB, they may hit throughput limits before IOPS limits.
3. **Database Contention**: Row-level locks or index contention within the DB engine can prevent the storage from being fully utilized.

**Q: How does a Dedicated Log Volume (DLV) improve performance for PIOPS users?**
**A:** A DLV separates sequential transaction/redo log writes from the random I/O pattern of data files. By placing logs on a dedicated volume with guaranteed IOPS (3,000), it minimizes "head-of-line blocking" where a burst of data writes delays a critical log commit, thereby ensuring more consistent and lower commit latencies.
