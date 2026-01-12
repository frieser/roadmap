---
---

## Summary
**Availability** is a quantifiable metric that measures the percentage of time a system is operational. It is commonly expressed in "Nines" (e.g., 99.9%). Each additional "Nine" represents an exponential increase in reliability and requires exponentially more engineering effort.

## Detailed Explanation

### The Formula
Availability is the ratio of Uptime to Total Time:
$$Availability = \frac{Uptime}{Uptime + Downtime}$$

### The "Nines" Table

| Availability | "Nines" | Downtime / Year | Downtime / Day | Engineering Implication |
| :--- | :--- | :--- | :--- | :--- |
| **90%** | One Nine | 36.5 days | 2.4 hours | No redundancy. Manual recovery. |
| **99%** | Two Nines | 3.65 days | 14.4 mins | Basic redundancy. Manual failover accepted. |
| **99.9%** | Three Nines | 8.76 hours | 1.44 mins | **Standard Cloud SLA**. Automated failover (e.g., RDS Multi-AZ). |
| **99.99%** | Four Nines | 52.6 mins | 8.64 secs | **High Availability**. Redundancy at every layer. Automatic healing. |
| **99.999%** | Five Nines | 5.26 mins | 0.86 secs | **Mission Critical**. Geo-replication, zero-downtime updates. |

### SLA vs SLO vs SLI
*   **SLI (Indicator)**: The specific metric measured (e.g., "Latency of GET /home").
*   **SLO (Objective)**: The goal we aim for (e.g., "99.9% of requests < 200ms").
*   **SLA (Agreement)**: The legal contract with users (e.g., "If availability < 99.9%, we refund 10%").

## Interview Questions

### Q: Is 100% availability possible?
**A:** No. In a distributed system, components fail, networks partition, and maintenance is required. Aiming for 100% is infinitely expensive. The goal is to match the availability to the business need (e.g., a banking core needs 5 nines; a blog needs 2).

### Q: How does MTTR affect Availability?
**A:** Availability is also defined as $MTBF / (MTBF + MTTR)$. Reducing the **Mean Time To Repair (MTTR)**—i.e., fixing things faster—is often easier and cheaper than increasing the Time Between Failures (MTBF).
