---
tags: ['aws', 'roadmap', 'rds']
---

# RDS DB Instances (Engines, Classes, Multi-AZ)

## Summary
Amazon Relational Database Service (Amazon RDS) is a managed service that simplifies the setup, operation, and scaling of relational databases in the AWS Cloud. A **DB Instance** is an isolated database environment that provides cost-efficient, resizable capacity while managing common database administration tasks like backups, software patching, and automatic failure detection.

## Detailed Explanation

### Managed Database Service
Amazon RDS is a Platform-as-a-Service (PaaS) offering. AWS handles the "undifferentiated heavy lifting" of database management, including:
- **Infrastructure**: Power, cooling, networking, and hardware maintenance.
- **Maintenance**: OS and database software installation and patching.
- **Availability**: Automatic failure detection and recovery.
- **Backups**: Automated daily snapshots and transaction logs for point-in-time recovery (PITR).

### Supported DB Engines
RDS supports several popular relational database engines, allowing users to migrate existing applications easily:
1. **Amazon Aurora**: A MySQL and PostgreSQL-compatible relational database built for the cloud.
2. **PostgreSQL**: Open-source object-relational database.
3. **MySQL**: Widely used open-source relational database.
4. **MariaDB**: Community-developed fork of MySQL.
5. **Oracle Database**: Enterprise-grade relational database.
6. **Microsoft SQL Server**: Microsoft's relational database management system.
7. **IBM Db2**: Enterprise-grade data management software.

### DB Instance Classes
The compute and memory capacity of a DB instance is determined by its **DB Instance Class**. These are categorized into:
- **General Purpose**: Balanced compute, memory, and networking (e.g., `db.m7g`, `db.t3`).
- **Memory Optimized**: Designed for memory-intensive applications (e.g., `db.r7g`, `db.x2g`).
- **Burstable Performance**: Ideal for workloads with low average CPU utilization but occasional spikes (e.g., `db.t3.micro`).
- **Compute Optimized**: Best for applications that require high CPU performance (e.g., `db.c6g`).

### High Availability: Multi-AZ vs. Read Replicas
RDS provides two main mechanisms for availability and scaling:

| Feature | Multi-AZ (Standby) | Read Replicas |
| :--- | :--- | :--- |
| **Replication** | Synchronous | Asynchronous |
| **Purpose** | High Availability & Disaster Recovery | Scaling Read Performance |
| **Usage** | Passive standby (No read/write) | Active read-only instances |
| **Failover** | Automatic (via DNS CNAME change) | Manual (can be promoted to primary) |
| **Location** | Separate AZ (Same Region) | Same/Separate AZ or Cross-Region |

#### Multi-AZ with Two Readable Standbys
A newer deployment option for MySQL and PostgreSQL that provides one writer and two reader instances across three AZs. It offers:
- Typically under 35 seconds failover.
- Up to 2x faster transaction commit latency.
- High availability with read scaling on the standby nodes.

### Go Implementation Example
In Go, you interact with an RDS instance using the standard `database/sql` package and a driver compatible with your DB engine.

```go
package main

import (
	"database/sql"
	"fmt"
	"log"

	_ "github.com/lib/pq" // PostgreSQL driver
)

func main() {
	// RDS Connection details (should be stored in environment variables or Secrets Manager)
	host := "my-rds-db.cluster-xxxx.us-east-1.rds.amazonaws.com"
	port := 5432
	user := "dbuser"
	password := "securepassword"
	dbname := "inventory"

	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=require",
		host, port, user, password, dbname)

	// Open connection
	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		log.Fatalf("Error opening database: %v", err)
	}
	defer db.Close()

	// Verify connection
	err = db.Ping()
	if err != nil {
		log.Fatalf("Error connecting to RDS: %v", err)
	}

	fmt.Println("Successfully connected to RDS instance!")
}
```

## Interview Questions

**Q: What is the main difference between RDS Multi-AZ and Read Replicas?**
**A:** Multi-AZ is for High Availability (HA) and uses synchronous replication to a passive standby instance in another AZ. Read Replicas are for scaling read traffic and use asynchronous replication to active instances that can be read from.

**Q: Does RDS Multi-AZ provide zero data loss failover?**
**A:** Yes, because replication to the Multi-AZ standby is synchronous, the data is guaranteed to be consistent, ensuring zero data loss during an automatic failover.

**Q: How does RDS handle failover in a Multi-AZ deployment?**
**A:** RDS monitors the health of the primary instance. If a failure is detected, it automatically promotes the standby to primary and updates the DNS CNAME record to point to the new primary. Applications typically resume service within 60-120 seconds.

**Q: When would you use RDS on EC2 instead of Amazon RDS?**
**A:** You would choose EC2 if you need full control over the operating system, need to install custom database features/plugins not supported by RDS, or require a specific DB version/engine that RDS doesn't provide.

**Q: What is the benefit of "Multi-AZ with two readable standbys"?**
**A:** It combines high availability with read scaling. It provides faster failover (typically <35s), improved write performance (up to 2x), and allows the two standby instances to serve read traffic, which was not possible with the traditional single-standby Multi-AZ.
