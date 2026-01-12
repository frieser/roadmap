#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ecs']
---

## Summary
AWS Fargate is a serverless compute engine for containers that works with both Amazon Elastic Container Service (ECS) and Amazon Elastic Kubernetes Service (EKS). It allows you to run containers without having to manage the underlying EC2 instances, removing the operational overhead of scaling, patching, and securing a cluster of virtual machines.

## Detailed Explanation

### Serverless Compute for Containers
AWS Fargate provides a "serverless" experience for containerized applications. In traditional ECS/EKS clusters, you are responsible for the "Data Plane" (the EC2 instances where containers run). With Fargate, AWS manages the Data Plane, and you only manage the "Control Plane" definitions (Tasks/Pods).

### No EC2 Management
- **Infrastructure Abstraction**: There are no EC2 instances to manage. You don't pick instance types, manage OS updates, or handle cluster scaling.
- **Operational Simplicity**: You define your application requirements (CPU and Memory), and Fargate launches the container in an isolated environment.
- **Security by Design**: Each Fargate task runs in its own kernel-isolated compute environment and does not share resources with other tasks, providing a higher level of security isolation.

### Pricing Model
Fargate uses a **pay-as-you-go** model based on the resources requested:
- **Metrics**: You are billed based on the amount of **vCPU** and **Memory** resources configured for the task.
- **Duration**: Billing starts when the container image starts being pulled and ends when the task terminates, calculated per second (with a 1-minute minimum).
- **Savings**: You can use **Fargate Spot** for non-critical workloads to get up to 70% discount, or **Compute Savings Plans** for committed usage.

### Networking Mode: awsvpc
Fargate tasks exclusively use the `awsvpc` network mode, which is integrated deeply with AWS VPC:
- **Dedicated ENI**: Every task gets its own Elastic Network Interface (ENI) with a private IP from your VPC subnet.
- **Task-Level Security**: Since each task has its own ENI, you apply **Security Groups** directly to the task itself, rather than the host instance.
- **Performance**: This mode provides the best networking performance and visibility, as task traffic can be monitored using VPC Flow Logs.

## Interview Questions

**1. What are the primary benefits of using AWS Fargate over the EC2 launch type?**
The primary benefits are reduced operational overhead (no server management), improved security through task-level isolation, and simplified scaling. You don't have to worry about "bin-packing" containers onto instances or managing the lifecycle of the underlying nodes.

**2. How does networking differ between Fargate and EC2 launch types?**
Fargate only supports `awsvpc` mode, meaning every task has its own ENI and private IP. In EC2 launch type, you can choose between `bridge`, `host`, `none`, or `awsvpc`. The `awsvpc` mode is generally preferred for modern architectures regardless of the launch type.

**3. Is Fargate more expensive than EC2?**
On a raw "per-unit-of-resource" basis, Fargate is typically more expensive than the equivalent EC2 instance. However, Fargate can be more cost-effective when considering "Total Cost of Ownership" (TCO), as it eliminates the cost of managing servers and the cost of idle capacity in underutilized EC2 clusters.

**4. How do you manage persistent storage in a Fargate task?**
Fargate supports **Amazon EFS (Elastic File System)** for persistent, shared storage. Each task can mount an EFS volume. Recently, Fargate also added support for **Amazon EBS (Elastic Block Store)** volumes for standalone tasks that require high-performance block storage, though it is more restricted than EFS for multi-task sharing.

**5. Can you use Fargate with Amazon EKS?**
Yes, Fargate is a supported compute option for Amazon EKS. You define which pods should run on Fargate using **Fargate Profiles**, which match pods based on namespaces and labels.
