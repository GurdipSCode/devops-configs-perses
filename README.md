# Perses Dashboards as Code

This repository contains a complete setup for [Perses](https://perses.dev), a modern dashboarding solution with dashboards defined as code.

## 📋 Table of Contents

- [Quick Start](#quick-start)
- [Data Sources Explained](#data-sources-explained)
- [Dashboard Structure](#dashboard-structure)
- [Creating Your Own Dashboards](#creating-your-own-dashboards)
- [Common Queries](#common-queries)
- [Troubleshooting](#troubleshooting)

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose installed
- A Prometheus instance (you have one, but it's not scraping yet)

### Starting Perses

```bash
# Start Perses
docker-compose up -d

# View logs
docker-compose logs -f perses

# Access the UI
# Open http://localhost:8080 in your browser
```

### Configure Your Prometheus

Edit `datasources/prometheus.yaml` and update the `direct_url` to point to your Prometheus instance:

```yaml
spec:
  plugin:
    kind: "PrometheusDatasource"
    spec:
      direct_url: "http://your-prometheus-host:9090"
```

## 📊 Data Sources Explained

### What Are Data Sources?

Data sources in Perses are the systems that provide metrics data for your dashboards. They define **where** Perses should fetch data from.

### Available Data Source Types

#### 1. **Prometheus Datasource** (Primary)

Perses has first-class support for Prometheus. This is what you'll use with your existing Prometheus instance.

**Configuration:**
```yaml
kind: "PrometheusDatasource"
spec:
  direct_url: "http://prometheus:9090"  # Your Prometheus URL
```

**What it does:**
- Connects to Prometheus's HTTP API
- Executes PromQL queries
- Retrieves time-series metrics data
- Supports all Prometheus query types (instant, range, metadata)

**Since you have Prometheus but no scraping:**
You need to configure Prometheus to scrape targets before you'll see data. Here's a basic Prometheus config:

```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Example: Scrape Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Example: Scrape Node Exporter for system metrics
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
  
  # Add your own services here
  - job_name: 'my-application'
    static_configs:
      - targets: ['my-app:8080']
```

#### 2. **Other Supported Data Sources**

Perses also supports (via plugins):
- **Tempo** - Distributed tracing
- **TempoTraceQL** - Trace query language
- **Custom HTTP endpoints** - Generic JSON/CSV data sources

### Data Source Configuration Files

All data sources are defined in the `datasources/` directory as YAML files. Perses automatically loads these on startup.

**Key fields:**
- `kind: "Datasource"` - Defines it as a data source
- `metadata.name` - Unique identifier for the data source
- `metadata.project` - Project namespace (use "perses" for global)
- `spec.default: true` - Makes it the default data source
- `spec.plugin` - Defines the data source type and connection details

## 🏗️ Dashboard Structure

### Dashboard Anatomy

Each dashboard YAML file contains:

```yaml
kind: "Dashboard"              # Resource type
metadata:
  name: "dashboard-id"          # Unique identifier
  project: "perses"             # Project namespace
spec:
  display:
    name: "Dashboard Title"     # Display name in UI
    description: "Description"  # Dashboard description
  duration: "1h"                # Default time range
  refreshInterval: "30s"        # Auto-refresh interval
  variables: []                 # Dynamic filters
  panels: {}                    # Chart definitions
  layouts: []                   # Panel positioning
```

### Key Components

#### 1. **Variables**

Variables create dynamic filters for your dashboards:

```yaml
variables:
  - kind: "ListVariable"
    spec:
      name: "instance"          # Variable name (use as $instance)
      display:
        name: "Instance"        # Display name
      plugin:
        kind: "PrometheusLabelValuesVariable"
        spec:
          datasource:
            kind: "PrometheusDatasource"
            name: "PrometheusDemo"
          label_name: "instance"  # Prometheus label to query
          matchers:               # Optional: Filter the metric
            - "up"
      allow_multiple: false       # Single or multi-select
      allow_all_value: true       # Add "All" option
```

**Usage in queries:** Reference as `$instance` or `${instance}`

#### 2. **Panels**

Panels are the individual charts/visualizations:

```yaml
panels:
  "panel-id":
    kind: "Panel"
    spec:
      display:
        name: "Panel Title"
        description: "Panel description"
      plugin:
        kind: "TimeSeriesChart"  # Chart type
        spec:
          queries:
            - kind: "TimeSeriesQuery"
              spec:
                datasource:
                  kind: "PrometheusDatasource"
                  name: "PrometheusDemo"
                query: "your_prometheus_query_here"
          legend:
            position: "bottom"
          y_axis:
            format:
              unit: "percent"
            min: 0
            max: 100
```

**Available Chart Types:**
- `TimeSeriesChart` - Line/area charts for time-series data
- `StatChart` - Single value display
- `GaugeChart` - Gauge visualization
- `BarChart` - Bar charts
- `TableChart` - Data tables
- `RowPanel` - Section headers/grouping

#### 3. **Layouts**

Layouts position panels on a 24-column grid:

```yaml
layouts:
  - kind: "Grid"
    spec:
      items:
        - x: 0        # Horizontal position (0-23)
          y: 0        # Vertical position
          width: 12   # Width in columns (1-24)
          height: 6   # Height in grid units
          content:
            "$ref": "#/spec/panels/panel-id"
```

## 🎨 Creating Your Own Dashboards

### Step 1: Create a New Dashboard File

```bash
touch dashboards/my-custom-dashboard.yaml
```

### Step 2: Define the Dashboard

Start with the starter template and modify:

```yaml
kind: "Dashboard"
metadata:
  name: "my-dashboard"
  project: "perses"
spec:
  display:
    name: "My Custom Dashboard"
    description: "Description of what this monitors"
  duration: "6h"
  
  # Add your panels here
  panels:
    "my-metric":
      kind: "Panel"
      spec:
        display:
          name: "My Metric"
        plugin:
          kind: "TimeSeriesChart"
          spec:
            queries:
              - kind: "TimeSeriesQuery"
                spec:
                  datasource:
                    kind: "PrometheusDatasource"
                    name: "PrometheusDemo"
                  query: "my_metric_name"
  
  # Position your panels
  layouts:
    - kind: "Grid"
      spec:
        items:
          - x: 0
            y: 0
            width: 24
            height: 6
            content:
              "$ref": "#/spec/panels/my-metric"
```

### Step 3: Reload Perses

Perses automatically picks up new dashboards within the provisioning interval (default: 10s).

Or restart the container:
```bash
docker-compose restart perses
```

## 📈 Common Queries

### System Metrics (Node Exporter)

```promql
# CPU Usage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage Percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk Usage Percentage
100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})

# Network Traffic (bytes/sec)
rate(node_network_receive_bytes_total{device!="lo"}[5m])
rate(node_network_transmit_bytes_total{device!="lo"}[5m])
```

### Application Metrics (RED Method)

```promql
# Request Rate (requests/second)
sum(rate(http_requests_total[5m])) by (job, method, status)

# Error Rate (percentage)
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# Duration/Latency (P95)
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (job, le))
```

### Kubernetes Metrics

```promql
# Pod CPU Usage
sum(rate(container_cpu_usage_seconds_total{pod!=""}[5m])) by (pod)

# Pod Memory Usage
sum(container_memory_working_set_bytes{pod!=""}) by (pod)

# Pod Restarts
sum(rate(kube_pod_container_status_restarts_total[5m])) by (pod)
```

## 🔧 Troubleshooting

### Dashboard Not Showing Up

1. Check Perses logs:
   ```bash
   docker-compose logs perses
   ```

2. Verify YAML syntax:
   ```bash
   yamllint dashboards/your-dashboard.yaml
   ```

3. Ensure the file is in the `dashboards/` directory

### No Data Appearing

1. **Check Prometheus connection:**
   - Verify the `direct_url` in `datasources/prometheus.yaml`
   - Test Prometheus directly: `curl http://your-prometheus:9090/api/v1/query?query=up`

2. **Configure Prometheus scraping:**
   Your Prometheus needs scrape targets configured. See the example above.

3. **Verify metrics exist:**
   - Go to Prometheus UI: `http://your-prometheus:9090`
   - Run your query in the Prometheus console
   - Check if metrics are being scraped: `up` query shows all targets

4. **Check time range:**
   - Ensure the dashboard time range includes when data exists
   - Try expanding the time range (e.g., from 1h to 24h)

### Authentication Issues

If you enabled `enable_auth: true`:
- You'll need to configure authentication providers
- See [Perses authentication docs](https://perses.dev/docs/authentication)

### Common Prometheus Setup Issues

Since you have Prometheus but no scraping:

1. **Add a scrape config** for Prometheus itself:
   ```yaml
   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']
   ```

2. **Install exporters** for the metrics you want:
   - Node Exporter: System metrics
   - cAdvisor: Container metrics
   - Blackbox Exporter: Endpoint monitoring
   - Custom application metrics

3. **Verify scraping works:**
   ```bash
   # Check targets in Prometheus
   curl http://localhost:9090/api/v1/targets
   
   # Query for up metric
   curl http://localhost:9090/api/v1/query?query=up
   ```

## 📚 Additional Resources

- [Perses Documentation](https://perses.dev/docs)
- [Perses GitHub](https://github.com/perses/perses)
- [PromQL Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Prometheus Exporters](https://prometheus.io/docs/instrumenting/exporters/)

## 🎯 Next Steps

1. Configure your Prometheus to scrape targets
2. Customize the example dashboards for your use case
3. Create new dashboards for your specific services
4. Set up alerting (Perses supports Prometheus alerting rules)
5. Enable authentication for production use

## 📁 Repository Structure

```
perses-dashboards/
├── config/
│   └── config.yaml           # Perses configuration
├── dashboards/               # Dashboard definitions (as code)
│   ├── system-monitoring.yaml
│   ├── application-monitoring.yaml
│   └── starter-template.yaml
├── datasources/              # Data source definitions
│   └── prometheus.yaml
├── docker-compose.yml        # Container orchestration
└── README.md                 # This file
```

All dashboards and data sources are version-controlled and can be deployed via GitOps!
