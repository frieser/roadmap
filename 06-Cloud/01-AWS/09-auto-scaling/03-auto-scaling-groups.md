#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'auto-scaling']
---

## Summary
An **Auto Scaling Group (ASG)** is a logical collection of EC2 instances that AWS manages as a single unit to ensure application availability and cost-efficiency. It automatically adjusts the number of instances based on demand, health status, and defined scaling policies, maintaining a target capacity across multiple Availability Zones (AZs).

## Detailed Explanation

### Core Capacity Settings
The behavior of an ASG is primarily governed by three capacity parameters:
*   **Minimum Size (Min)**: The absolute minimum number of instances the group must maintain. ASG will never scale below this, ensuring a baseline level of availability even during periods of zero traffic.
*   **Maximum Size (Max)**: The upper limit for the group. This acts as a safety "ceiling" to prevent runaway costs or resource exhaustion due to aggressive scaling policies or traffic spikes.
*   **Desired Capacity**: The target number of instances the ASG aims to keep running. Scaling policies work by adjusting this value. If an instance fails, ASG launches a new one to return to the Desired Capacity.

### Health Checks and Replacements
ASG identifies and replaces instances that are no longer functioning correctly:
*   **EC2 Status Checks**: The default monitoring level. It checks the underlying hardware and the reachability of the instance.
*   **ELB Health Checks**: If the ASG is associated with a Load Balancer, it can use the LB's health checks. This is more robust as it detects application-level failures (e.g., a web server returning 503 errors).
*   **Health Check Grace Period**: A configurable time (e.g., 300 seconds) that allows new instances to finish booting and initializing before ASG starts performing health checks on them.

### Availability Zone (AZ) Rebalancing
AWS strives to keep ASG instances balanced across all enabled Availability Zones for maximum fault tolerance:
*   **Balancing Logic**: If instances become unevenly distributed (e.g., due to an AZ outage or manual termination), the ASG initiates a **Rebalancing Activity**.
*   **Mechanism**: ASG will launch a new instance in the AZ with the fewest instances before terminating one in the over-represented AZ. This "launch-before-terminate" approach ensures that the total capacity doesn't dip below the desired level during the rebalance.
*   **Suspension**: You can manually suspend the `AZRebalance` process if you need to troubleshoot or perform specific maintenance that would otherwise trigger it.

### Scaling Methods
*   **Manual**: Manually changing the Desired Capacity.
*   **Scheduled**: Scaling based on known patterns (e.g., "increase capacity every Monday at 9 AM").
*   **Dynamic**: Using CloudWatch metrics (CPU, Request Count) via policies like **Target Tracking** (keeping CPU at 50%) or **Step Scaling**.
*   **Predictive**: Using machine learning to forecast traffic and scale proactively.

## Interview Questions

**Q: What is the difference between Desired Capacity and Max Size in an ASG?**
**A:** Desired Capacity is the number of instances the ASG is actively trying to maintain at any given moment. Max Size is the hard limit that the Desired Capacity cannot exceed, serving as a cost-control and resource-safety mechanism.

**Q: If an instance is running but its application has crashed, will a default ASG replace it?**
**A:** No, not by default. Default EC2 health checks only monitor the instance status (VM reachability). To detect application crashes, you must enable **ELB Health Checks** or use custom health checks via the AWS CLI/SDK.

**Q: What happens during an AZ Rebalance if the ASG is already at its Max Size?**
**A:** ASG is allowed to temporarily exceed the Max Size by 1 instance (or 10% for large groups) during a rebalancing activity to ensure availability is maintained while the new instance is being provisioned.

**Q: What is an ASG Cooldown Period?**
**A:** It is a rest period after a scaling activity completes. During this time, the ASG will not initiate further scaling actions, allowing the previous action (e.g., adding an instance) to take effect on the CloudWatch metrics and preventing "flapping" or over-scaling.
