---
---

# Grafana

Grafana is the world's leading open-source observability platform for **visualization**. It does not generate data; it visualizes data stored elsewhere (Prometheus, InfluxDB, SQL, Elasticsearch, etc.).

## Summary

Grafana allows you to query, visualize, alert on, and understand your metrics no matter where they are stored. It acts as the "Single Pane of Glass" for DevOps teams, combining infrastructure metrics (Prometheus), logs (Loki), and traces (Tempo) into unified dashboards.

## Detailed Explanation

### 1. Data Sources
Grafana is agnostic. It connects to **Data Sources**.
*   **Prometheus**: For time-series metrics.
*   **Loki**: For logs (LogQL query language, similar to PromQL).
*   **PostgreSQL/MySQL**: For SQL data.
*   **CloudWatch/Azure Monitor**: For cloud provider metrics.

### 2. Dashboards and Panels
*   **Dashboard**: A collection of panels arranged on a grid.
*   **Panel**: A single visualization (Graph, Gauge, Table, Heatmap).
*   **Variables**: Dropdowns (e.g., "Select Cluster") that allow dashboards to be dynamic and reusable.

---

## Go Implementation Example

In modern DevOps, we don't click buttons to create dashboards; we use **Dashboards as Code**. The `grafana-foundation-sdk` (or older tools like `grafonnet`) allows you to define dashboards in Go and deploy them via the Grafana API or Terraform.

### Generaing a Dashboard JSON with Go
```go
package main

import (
	"encoding/json"
	"fmt"
	"os"

	"github.com/grafana/grafana-foundation-sdk/go/dashboard"
	"github.com/grafana/grafana-foundation-sdk/go/timeseries"
)

func main() {
	// 1. Create Dashboard Builder
	builder := dashboard.NewDashboardBuilder("Go Service Overview").
		Uid("go-svc-overview").
		Tags([]string{"generated", "go"}).
		Refresh("5s")

	// 2. Add CPU Panel
	cpuPanel := timeseries.NewPanelBuilder().
		Title("CPU Usage").
		Unit("percent").
		Span(12). // Full width
		Height(8)

	// Add Query (Prometheus)
	cpuPanel.AddTarget(
		dashboard.NewTargetBuilder().
			Expr("rate(process_cpu_seconds_total[1m]) * 100").
			LegendFormat("{{instance}}"),
	)

	builder.WithPanel(cpuPanel)

	// 3. Build and Output JSON
	dash, err := builder.Build()
	if err != nil {
		panic(err)
	}

	jsonData, _ := json.MarshalIndent(dash, "", "  ")
	fmt.Println(string(jsonData))
	
	// In a real flow, you would POST this JSON to Grafana API /api/dashboards/db
	os.WriteFile("dashboard.json", jsonData, 0644)
}
```

## Interview Questions

**Q: What is the role of Grafana Loki?**
**A:** Loki is a log aggregation system inspired by Prometheus. Unlike Splunk or ELK which index *content* (full-text search), Loki only indexes *metadata* (labels). This makes it extremely cheap and storage-efficient. It integrates perfectly with Grafana, allowing you to split your screen: graphs on top (Prometheus), logs on bottom (Loki), correlated by the same labels.

**Q: How do Grafana Alerts differ from Prometheus Alertmanager?**
**A:**
*   **Prometheus Alertmanager**: Alerts are defined in YAML rules on the Prometheus server side based on PromQL. Best for infrastructure-as-code and reliability.
*   **Grafana Alerts**: Alerts are defined visually in the Grafana UI (or code) based on *any* data source (not just Prometheus). Best if you need to alert on SQL data or combine multiple data sources into one alert logic.

**Q: What is a "Variable" in Grafana and why is it useful?**
**A:** A Variable is a placeholder in a dashboard (e.g., `$host`). You define a query to populate it (e.g., `label_values(up, instance)`). This creates a dropdown menu at the top of the dashboard. When a user selects a host, `$host` is replaced in all panel queries. This allows you to create ONE "Host Overview" dashboard that works for 1,000 different hosts, rather than creating 1,000 dashboards.
