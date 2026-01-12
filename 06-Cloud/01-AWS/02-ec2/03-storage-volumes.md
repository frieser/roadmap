#AWS
#Cloud

---
tags: ['aws', 'roadmap', 'cloud', 'ec2']
---

## Summary
EC2 instances utilize two primary block storage types: **Amazon Elastic Block Store (EBS)** and **EC2 Instance Store**. EBS provides persistent, network-attached storage that can be detached and moved, offering features like snapshots and replication within an Availability Zone. Conversely, Instance Store provides high-performance, temporary (ephemeral) storage physically attached to the host server, ideal for low-latency scratch data but with the risk of data loss if the instance is stopped or terminated.

## Detailed Explanation

### 1. Amazon EBS (Elastic Block Store)
EBS is the primary persistent storage for EC2. It behaves like a network-attached disk (SAN), allowing it to persist independently of the instance's lifecycle.

*   **Persistence**: Data remains available even if the instance is stopped. It is only lost if the volume is explicitly deleted or if `DeleteOnTermination` is set to true.
*   **Availability**: Automatically replicated within its Availability Zone (AZ) to prevent data loss from single component failures.
*   **Snapshots**: Point-in-time backups stored in S3. They are incremental, meaning only the blocks that changed since the last snapshot are saved.
*   **Encryption**: Supports transparent encryption of data at rest and in transit between the instance and the volume.

#### EBS Volume Types (SSD-backed)
| Feature | gp3 (General Purpose) | io2 / io2 Block Express |
| :--- | :--- | :--- |
| **Use Case** | Most workloads (virtual desktops, dev/test) | IO-intensive databases (SAP HANA, SQL Server) |
| **Max IOPS** | 16,000 | 256,000 (Block Express) |
| **Max Throughput** | 1,000 MiB/s | 4,000 MiB/s (Block Express) |
| **Provisioning** | Scale IOPS/Throughput independently | Provisioned IOPS for sub-ms latency |
| **Durability** | 99.8% - 99.9% | 99.999% |

### 2. EC2 Instance Store
Instance Store provides temporary block-level storage. The disks are physically attached to the host computer, eliminating network latency.

*   **Performance**: Lowest latency and highest throughput (millions of IOPS on some instances) because there is no network overhead.
*   **Ephemeral Nature**: Data is lost if the instance is **stopped**, **terminated**, or if the **underlying hardware fails**. Data **survives a reboot**.
*   **Best For**: Caching, swap space, temporary files, and distributed data stores with native replication (e.g., NoSQL databases, Hadoop clusters).

### 3. Architecture Comparison
```mermaid
graph LR
    subgraph "AWS Cloud"
        subgraph "Availability Zone"
            subgraph "EC2 Host Server"
                Instance[EC2 Instance]
                IS[Instance Store<br/>(Local SSD/NVMe)]
            end
            EBS[Amazon EBS Volume<br/>(Network-Attached)]
            S3[Amazon S3<br/>(Snapshots)]
        end
    end
    
    Instance -- "SATA/NVMe Interface" --> IS
    Instance -- "Network (EBS-Optimized)" --> EBS
    EBS -. "Backup" .-> S3
```

### 4. Go Application (AWS SDK v2)
Interacting with EC2 Storage using Go. This example demonstrates how to create a `gp3` volume with custom performance settings.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ec2"
	"github.com/aws/aws-sdk-go-v2/service/ec2/types"
)

func main() {
	// 1. Load configuration
	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-east-1"))
	if err != nil {
		log.Fatalf("unable to load SDK config, %v", err)
	}

	client := ec2.NewFromConfig(cfg)

	// 2. Define gp3 volume with independent IOPS and Throughput
	input := &ec2.CreateVolumeInput{
		AvailabilityZone: aws.String("us-east-1a"),
		Size:             aws.Int32(100), // 100 GiB
		VolumeType:       types.VolumeTypeGp3,
		Iops:             aws.Int32(4000), // Scale IOPS independently of size
		Throughput:       aws.Int32(250),  // Scale throughput independently
		TagSpecifications: []types.TagSpecification{
			{
				ResourceType: types.ResourceTypeVolume,
				Tags: []types.Tag{
					{Key: aws.String("Name"), Value: aws.String("Demo-gp3-Volume")},
				},
			},
		},
	}

	// 3. Create the volume
	result, err := client.CreateVolume(context.TODO(), input)
	if err != nil {
		log.Fatalf("failed to create volume: %v", err)
	}

	fmt.Printf("Created Volume ID: %s in state: %s\n", *result.VolumeId, result.State)
}
```

## Interview Questions

**Q: What happens to the data in an Instance Store if an instance is stopped?**
**A:** The data is lost. Because Instance Store is physically attached to the host, stopping the instance releases the host hardware. When the instance is started again, it might be on a different host, and the previous local storage is wiped for security.

**Q: How does gp3 improve over gp2 in terms of cost and performance?**
**A:** `gp3` allows users to provision IOPS and Throughput independently of storage size. In `gp2`, IOPS were tied to the volume size (3 IOPS per GB), which often forced users to over-provision storage just to get better performance. `gp3` is also typically 20% cheaper per GB than `gp2`.

**Q: When should you use io2 Block Express instead of gp3?**
**A:** You should choose `io2 Block Express` for mission-critical, high-performance workloads that require up to 256,000 IOPS, 4,000 MiB/s throughput, and sub-millisecond latency (e.g., large-scale Oracle or SAP HANA databases) with 99.999% durability.

**Q: Does EBS data survive a reboot? What about Instance Store?**
**A:** Yes, both EBS and Instance Store data survive a standard OS reboot. The data loss for Instance Store only occurs during a Stop, Termination, or hardware failure.

**Q: Can a single EBS volume be attached to multiple instances?**
**A:** Yes, using the **EBS Multi-Attach** feature (supported by `io1` and `io2` volumes). This allows a volume to be attached to up to 16 Nitro-based instances in the same Availability Zone.
