#AWS
#Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ecs']
---

## Summary
ECS Capacity Providers are a powerful abstraction for managing the infrastructure (EC2 or Fargate) used by Amazon ECS tasks. For EC2-based clusters, Capacity Providers integrate directly with **Auto Scaling Groups (ASG)** to automate cluster scaling based on the actual resource requirements of tasks (Task-level demand) rather than instance-level metrics.

## Detailed Explanation

ECS Capacity Providers solve the challenge of "Cluster Auto Scaling" by bridging the gap between ECS task scheduling and EC2 Auto Scaling.

### 1. Managed Scaling
Managed scaling allows ECS to manage the scale-out and scale-in actions of the Auto Scaling Group (ASG) automatically. 
- **CapacityProviderReservation Metric**: ECS publishes this metric to CloudWatch. It represents the percentage of total capacity needed. 
    - `100`: All capacity is used.
    - `> 100`: There are tasks in `PENDING` state that cannot be placed; triggers scale-out.
    - `< 100`: There is excess capacity; triggers scale-in.
- **Target Tracking**: ECS creates a target tracking scaling policy for the ASG that tracks the `CapacityProviderReservation` metric, typically aiming for a target value (e.g., 100).

### 2. Managed Termination Protection
To prevent the ASG from terminating instances that are still running tasks during a scale-in event, ECS provides **Managed Termination Protection**.
- When enabled, ECS sets "Scale-in Protection" on EC2 instances that are running at least one non-daemon task.
- Once an instance becomes empty (tasks are stopped or drained), ECS removes the protection, allowing the ASG to terminate it.

### 3. Capacity Provider Strategies
A cluster can have multiple Capacity Providers (e.g., one for On-Demand and one for Spot). A **Capacity Provider Strategy** defines how tasks are distributed:
- **Base**: The minimum number of tasks to run on a given provider (e.g., always run at least 3 tasks on On-Demand).
- **Weight**: The relative ratio for distributing tasks once the `Base` is met (e.g., a 1:3 ratio between On-Demand and Spot).

### 4. Benefits over Traditional Scaling
- **Task-Aware**: Traditional ASG scaling based on CPU/RAM utilization doesn't know about pending tasks or placement constraints (like `binpack` or `spread`). Capacity Providers scale based on whether tasks *can* be placed.
- **Simplified Operations**: Developers don't need to manually create and manage complex CloudWatch alarms for ASG scaling.

## Interview Questions

1. **What is the primary metric used by ECS Capacity Providers for managed scaling?**
   - The primary metric is `CapacityProviderReservation`. It measures the number of instances required to run all tasks (including pending ones) versus the number of instances currently available in the ASG.

2. **How does Managed Termination Protection work in an ECS Capacity Provider?**
   - When enabled, ECS automatically manages the "Instance Scale-In Protection" attribute of the EC2 instances. It protects any instance running at least one non-daemon task from being terminated by an ASG scale-in event.

3. **Can you mix Fargate and EC2 Capacity Providers in the same ECS Cluster?**
   - Yes, an ECS cluster can have multiple capacity providers of different types. You can use a Capacity Provider Strategy to distribute your tasks across Fargate, Fargate_Spot, and multiple EC2 ASG-backed providers.

4. **What happens if a task's resource requirements (CPU/Memory) exceed the largest instance type in the ASG?**
   - Even with Managed Scaling, the task will remain in `PENDING` state. Capacity Providers can only scale the *number* of instances in the ASG; they do not automatically change the instance type (vertical scaling).

5. **Explain the 'Weight' parameter in a Capacity Provider Strategy.**
   - The `Weight` parameter determines the relative percentage of tasks launched on a provider after the `Base` requirement is satisfied. For example, if Provider A has weight 1 and Provider B has weight 4, then for every 5 tasks launched, 1 goes to A and 4 go to B.
