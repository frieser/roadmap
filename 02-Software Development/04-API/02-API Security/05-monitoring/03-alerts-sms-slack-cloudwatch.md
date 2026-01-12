#API
---
---

# Alerts (SMS, Slack, Cloudwatch) - API Security & Monitoring

Effective alerting is the bridge between observability and incident response. In the context of API Security and Monitoring, alerts must be **actionable**, **meaningful**, and **low-noise** to prevent "alert fatigue."

---

## 1. Concept Summary: Alerting Strategies

Alerting strategies in 2025 focus on moving away from static thresholds toward **intent-based** and **anomaly-aware** systems.

*   **Threshold-Based Alerting**: Triggers when a metric crosses a pre-defined value (e.g., HTTP 5xx > 1%).
*   **Anomaly Detection**: Uses machine learning to establish a "normal" baseline and alerts on deviations (e.g., a sudden 300% spike in traffic at 3 AM).
*   **Symptom-Based Alerting**: Alerts on user-facing issues (high latency) rather than internal causes (high CPU), following the **Google SRE Golden Signals**:
    1.  **Latency**: Time taken to service a request.
    2.  **Traffic**: Demand placed on the system.
    3.  **Errors**: Rate of failed requests.
    4.  **Saturation**: How "full" your service is.

### **Security Context**
Security-specific alerting focuses on **Abuse Patterns**:
*   **Auth Failures**: Spikes in `401 Unauthorized` (Brute force).
*   **Access Violations**: Spikes in `403 Forbidden` (Insecure Direct Object Reference - IDOR).
*   **Data Exfiltration**: Unusual spikes in outgoing byte count (payload size).

---

## 2. Setting Up Meaningful Alerts

### **Thresholds & Windowing**
Avoid "flappy" alerts by using evaluation windows. Instead of alerting on a single bad request, alert if the average error rate exceeds 5% over a 5-minute window.

### **Anomaly Detection in 2025**
Platforms like **AWS CloudWatch** now support automatic baseline calculation.
*   **Evidence** ([CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Anomaly_Detection.html)):
    > "When you enable anomaly detection for a metric, CloudWatch applies machine-learning algorithms to the metric's prior data to create a model of the metric's expected values."

---

## 3. Integration Examples

### **AWS CloudWatch to PagerDuty**
The standard integration path uses **Amazon SNS** (Simple Notification Service) as the intermediary.
1.  **CloudWatch Alarm**: Monitors a metric (e.g., `UnhealthyHostCount`).
2.  **SNS Topic**: The alarm "publishes" to an SNS topic.
3.  **PagerDuty**: Subscribes to the SNS topic via a dedicated HTTPS endpoint.

### **Slack Webhooks**
The simplest way to push alerts to a team.
*   **Implementation**: Send a JSON `POST` request to a unique Slack URL.
*   **Evidence** ([source](https://github.com/dark-warlord14/CVENotifier/blob/main/internal/slack/slack.go#L20-L28)):
```go
// Example of a Slack payload structure
message := SlackMessage{
    Text: "Title: " + vulnTitle + "\nLink: " + link + "\nDate Published: " + published,
}
payload, _ := json.Marshal(message)
resp, err := http.Post(slackWebhook, "application/json", bytes.NewBuffer(payload))
```

---

## 4. Go (Golang) Code Examples

### **Triggering a Custom CloudWatch Metric**
Use the AWS SDK v2 to push custom security metrics, such as "Rate Limited Requests".

```go
import (
    "context"
    "github.com/aws/aws-sdk-go-v2/service/cloudwatch"
    "github.com/aws/aws-sdk-go-v2/service/cloudwatch/types"
    "github.com/aws/aws-sdk-go/aws"
)

func PublishSecurityMetric(cwClient *cloudwatch.Client, count float64) error {
    _, err := cwClient.PutMetricData(context.TODO(), &cloudwatch.PutMetricDataInput{
        Namespace: aws.String("MyAPI/Security"),
        MetricData: []types.MetricDatum{
            {
                MetricName: aws.String("RateLimitedRequests"),
                Value:      aws.Float64(count),
                Unit:       types.StandardUnitCount,
            },
        },
    })
    return err
}
```

### **Sending a PagerDuty Event**
Using the official `go-pagerduty` library for rich incident data.

```go
import (
    "github.com/PagerDuty/go-pagerduty"
)

func TriggerPagerDuty(routingKey string, summary string) error {
    event := pagerduty.V2Event{
        RoutingKey: routingKey,
        Action:     "trigger",
        Payload: &pagerduty.V2Payload{
            Summary:  summary,
            Source:   "api-gateway-security-monitor",
            Severity: "critical",
        },
    }
    _, err := pagerduty.ManageEvent(event)
    return err
}
```

---

## 5. Interview Preparation: Q&A

**Q: What is "Alert Fatigue" and how can you mitigate it?**
**A**: Alert Fatigue occurs when engineers are overwhelmed by frequent, non-actionable alerts, leading them to ignore critical ones. Mitigation includes:
1.  **Deduplication**: Grouping related alerts (e.g., 100 microservice errors = 1 PagerDuty incident).
2.  **Threshold Tuning**: Using percentiles (P99) instead of averages.
3.  **Actionability**: Every alert should have a linked **Runbook** explaining how to fix it.

**Q: How do you differentiate between a "Warning" and a "Critical" alert?**
**A**: Critical alerts require immediate attention (waking someone up) and usually indicate a total outage or a severe security breach. Warning alerts are "ticket-worthy" but can wait until business hours (e.g., disk space at 80% or a slow increase in memory usage).

**Q: Why use Anomaly Detection over Fixed Thresholds for API traffic?**
**A**: API traffic often has cyclical patterns (busy on weekdays, quiet on weekends). A fixed threshold that works for Monday would trigger false positives on Sunday. Anomaly detection accounts for seasonality.

**Q: In an AWS environment, how do you notify a mobile device via SMS?**
**A**: Route the CloudWatch Alarm to an SNS Topic, and add an SMS subscription to that topic. However, for production critical alerts, PagerDuty/Opsgenie is preferred because SMS lacks "acknowledgment" and "escalation" logic.
