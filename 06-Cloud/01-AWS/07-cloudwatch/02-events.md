---
tags: ['cloud', 'roadmap', 'aws', 'cloudwatch']
---

# CloudWatch Events / EventBridge (Rules, Targets, Schema Registry)

## Summary
Amazon EventBridge (formerly CloudWatch Events) is a serverless event bus service that facilitates the creation of event-driven architectures. It allows applications to communicate asynchronously by routing events from AWS services, custom applications, and SaaS partners to various targets. By decoupling producers and consumers, it simplifies the scaling and maintenance of complex distributed systems.

## Detailed Explanation

### 1. Event Buses
An event bus is the pipeline that receives events. EventBridge supports three types of buses:
- **Default Event Bus**: Automatically created in every account; it receives events from nearly all AWS services.
- **Custom Event Bus**: Created for your own applications to send and receive custom events.
- **SaaS Event Bus**: Specifically for events from integrated third-party partners (e.g., Auth0, Zendesk, Datadog).

### 2. Rules and Event Patterns
Rules filter incoming events and route them to targets. A rule can match based on:
- **Event Patterns**: JSON objects that define the structure of events to match.
  
  **Example Rule Pattern:**
  ```json
  {
    "source": ["aws.ec2"],
    "detail-type": ["EC2 Instance State-change Notification"],
    "detail": {
      "state": ["running", "stopped"]
    }
  }
  ```
- **Filter Logic**: Supports prefix matching, numeric ranges, "anything-but" matching, and suffix matching.

### 3. Schedule Expressions (Cron & Rate)
Rules can also be triggered on a schedule, acting as a serverless "cron" service.
- **Rate expressions**: Simple intervals like `rate(1 minute)` or `rate(2 hours)`.
- **Cron expressions**: Follow a 6-field format: `cron(Minutes Hours Day-of-month Month Day-of-week Year)`.
  - **Example**: `cron(0 8 1 * ? *)` - Runs at 08:00 AM on the 1st day of every month.
  - **Constraint**: You must specify a wildcard (`*`) or `?` for either `Day-of-month` or `Day-of-week`.

### 4. Targets
Targets are the AWS services or endpoints that process the events. A single rule can have up to **5 targets**. Common targets include:
- AWS Lambda functions
- Amazon SQS queues
- Amazon SNS topics
- Step Functions state machines
- API Destinations (invoking external HTTP endpoints)

### 5. Schema Registry and Discovery
- **Schema Registry**: Stores the structure (schema) of events. This helps developers understand the data format without manual documentation.
- **Schema Discovery**: An automated feature that listens to events on a bus and automatically generates schemas in the registry.
- **Code Bindings**: Allows you to download SDKs for schemas in languages like Java, Python, TypeScript, and **Go**, enabling type-safe event handling.

## Interview Questions

**Q: What is the relationship between CloudWatch Events and Amazon EventBridge?**
**A:** EventBridge is the evolution of CloudWatch Events. It uses the same underlying API but adds advanced features like Custom Event Buses, SaaS integrations, Schema Registry, and the ability to archive and replay events.

**Q: How does EventBridge handle failed event delivery to a target?**
**A:** EventBridge retries delivery for up to 24 hours with exponential backoff. You can also configure a **Dead Letter Queue (DLQ)** using SQS to capture events that failed all retry attempts.

**Q: Can EventBridge route events across different AWS Regions or Accounts?**
**A:** Yes. You can configure an Event Bus in another account or region as a target for a rule in the source account/region, enabling global and multi-account event-driven architectures.

**Q: What is the benefit of using API Destinations in EventBridge?**
**A:** API Destinations allow EventBridge to trigger any HTTP endpoint (external to AWS) as a target. It handles authentication (Basic, OAuth, API Key) and can perform rate limiting to protect the target endpoint.

**Q: Explain the difference between 'rate' and 'cron' schedule expressions.**
**A:** `rate` is used for simple, recurring intervals (e.g., every 5 minutes), whereas `cron` provides fine-grained control over specific times, dates, and days of the week (e.g., 9:00 AM every Monday through Friday).
