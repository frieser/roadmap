---
tags: ['cloud', 'roadmap', 'aws', 'ec2']
---

# Burstable Performance Instances (CPU Credits)

## Summary
**Burstable Performance Instances** (T4g, T3, T3a, and T2) are a category of EC2 instances designed to provide a cost-effective baseline level of CPU performance with the ability to "burst" above that baseline when the workload demands it. This is managed through a **CPU Credit** system, making these instances ideal for workloads that are typically idle or have low-to-moderate average CPU utilization but experience occasional spikes, such as web servers, developer environments, and small databases.

## Detailed Explanation

### 1. The Baseline Performance
Each burstable instance has a pre-defined **baseline CPU performance** level, expressed as a percentage of a vCPU.
- **Below Baseline**: When the instance uses less CPU than its baseline, it earns CPU credits.
- **Above Baseline**: When the instance needs more performance than its baseline allows, it spends earned CPU credits to burst.
- **Sustained Baseline**: If an instance has no credits, its performance is capped at the baseline (in Standard mode).

### 2. CPU Credits: Earning and Spending
- **What is a CPU Credit?**: One CPU credit equals **one vCPU running at 100% utilization for one minute**. Alternatively, it could be one vCPU at 50% for two minutes, or 2 vCPUs at 25% for two minutes.
- **Earning Credits**: Credits are earned continuously at a fixed hourly rate, which varies by instance size. For example, a `t3.micro` earns 6 credits per hour (baseline 10% per vCPU).
- **Accrual Limit**: There is a maximum number of credits an instance can accumulate (the "Accrued Limit"). Once reached, newly earned credits are discarded.
- **Credit Balance**: This is the current pool of "banked" credits available for bursting.

### 3. Unlimited vs. Standard Mode
AWS T-family instances can operate in two distinct credit management modes:

| Feature | Standard Mode | Unlimited Mode |
| :--- | :--- | :--- |
| **Bursting** | Limited by earned credit balance. | Unlimited duration. |
| **Exhaustion** | Performance is throttled to baseline. | Performance never throttled. |
| **Cost** | Fixed hourly price. | Potential extra charges for "Surplus Credits." |
| **Best For** | Non-critical, low-cost workloads. | Business-critical workloads with unpredictable spikes. |

- **Unlimited Mode (Default for T3/T3a/T4g)**: If the instance exhausts its credit balance, it uses **Surplus Credits**. If the average CPU usage over a 24-hour period remains below the baseline, no extra charge is incurred. If it stays above, you are charged a flat additional fee per vCPU-hour.

### 4. Instance Families
- **T4g**: Powered by **AWS Graviton2** (Arm-based) processors. Offers up to 40% better price-performance than T3 instances.
- **T3 / T3a**: Next-generation x86 instances (Intel / AMD) built on the **AWS Nitro System**, allowing for better resource allocation and performance.
- **T2**: Previous generation instances. They do not support Unlimited mode by default and lack the performance benefits of the Nitro system.

## Interview Questions

### Q1: What is the fundamental difference between a T3 and an M5 instance regarding CPU performance?
**A:** A T3 instance is a **burstable** instance that provides a baseline performance and uses a credit system for spikes. An M5 instance provides **fixed, dedicated performance**; it has no baseline or credit system and can run at 100% CPU indefinitely without extra costs or throttling.

### Q2: If a T3 instance in Unlimited mode has a baseline of 20% and averages 30% CPU usage over 24 hours, how is it billed?
**A:** The instance will be billed for its standard hourly rate plus an additional charge for the **Surplus Credits** used. Since the average usage (30%) exceeded the baseline (20%) over the 24-hour window, the extra 10% usage is billed at a flat rate per vCPU-hour.

### Q3: What happens to the CPU Credit Balance when an instance is stopped and started?
**A:** For T2 Standard instances, the credit balance is **lost** when the instance is stopped. For T3, T3a, and T4g instances, the credit balance persists for a period of time after the instance is stopped but is eventually lost if not restarted. In Unlimited mode, any surplus credits are charged immediately upon stopping.

### Q4: Why would you choose a T4g instance over a T3 instance?
**A:** T4g instances are generally preferred because they use **AWS Graviton2 (Arm)** processors, which provide significantly better price-performance (up to 40%) and lower costs compared to the x86-based T3 instances for compatible workloads.

### Q5: Define exactly what "1 CPU Credit" represents.
**A:** One CPU credit provides the performance of one vCPU running at 100% utilization for one minute.
