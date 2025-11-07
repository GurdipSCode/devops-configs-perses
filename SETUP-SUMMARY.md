# Perses Setup Complete! 🎉

## What I've Created

A complete Perses repository with **dashboards as code** that you can deploy and customize.

### 📂 Repository Structure

```
perses-dashboards/
├── 📄 QUICKSTART.md                      ← Start here! 5-minute setup
├── 📄 README.md                          ← Complete documentation
├── 📄 docker-compose.yml                 ← Run Perses with Docker
├── 📄 .gitignore                         ← Git ignore rules
│
├── config/
│   └── 📄 config.yaml                    ← Perses configuration
│
├── datasources/
│   └── 📄 prometheus.yaml                ← Your Prometheus connection
│
├── dashboards/                           ← Your dashboards (as code!)
│   ├── 📄 system-monitoring.yaml         ← CPU, memory, disk, network
│   ├── 📄 application-monitoring.yaml    ← Request rate, errors, latency
│   └── 📄 starter-template.yaml          ← Template for new dashboards
│
└── prometheus/
    └── 📄 prometheus.yml                 ← Sample scrape config
```

## 🎯 Quick Start

### 1. Update Your Prometheus URL

Edit `datasources/prometheus.yaml`:
```yaml
direct_url: "http://YOUR-PROMETHEUS-HOST:9090"
```

### 2. Start Perses

```bash
cd perses-dashboards
docker-compose up -d
```

### 3. Access UI

Open: **http://localhost:8080**

### 4. Configure Prometheus Scraping

Since your Prometheus isn't scraping yet, see `prometheus/prometheus.yml` for examples.

## 📊 What Are Data Sources?

**Data sources** tell Perses **where to get metrics from**. Think of them as connections to your monitoring systems.

### Prometheus Data Source (What You Have)

- **Type:** `PrometheusDatasource`
- **What it does:** Connects to your Prometheus server and executes PromQL queries
- **Configuration:** Defined in `datasources/prometheus.yaml`
- **Key setting:** `direct_url` - points to your Prometheus HTTP API

```yaml
kind: "Datasource"
metadata:
  name: "PrometheusDemo"       # Reference name in dashboards
spec:
  default: true                 # Make this the default
  plugin:
    kind: "PrometheusDatasource"
    spec:
      direct_url: "http://prometheus:9090"  # Your Prometheus URL
```

### How It Works

1. **Dashboard defines a query** (e.g., "show me CPU usage")
2. **Perses sends PromQL** to Prometheus via the data source
3. **Prometheus returns time-series data**
4. **Perses visualizes** the data in charts

### The Missing Piece: Prometheus Scraping

You have Prometheus, but it needs to **scrape metrics** from targets:

```yaml
# In your Prometheus config
scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['my-app:8080']  # Your service exposing metrics
```

**Common metric sources:**
- **Node Exporter** → System metrics (CPU, RAM, disk)
- **cAdvisor** → Container metrics
- **Your application** → Custom metrics via `/metrics` endpoint
- **Kubernetes** → Pod, service, node metrics

### Other Data Source Types

Perses also supports:
- **Tempo** - Distributed tracing
- **Generic HTTP** - Any JSON/CSV endpoint
- **Custom plugins** - Extend with your own

## 📈 Included Dashboards

### 1. System Monitoring
**File:** `dashboards/system-monitoring.yaml`

**Monitors:**
- CPU usage across cores
- Memory utilization
- Disk space usage
- Network traffic (TX/RX)

**Requires:** Node Exporter scraping

### 2. Application Monitoring
**File:** `dashboards/application-monitoring.yaml`

**Monitors (RED metrics):**
- **R**ate - Requests per second
- **E**rrors - Error rate percentage
- **D**uration - Response time (P95)
- Plus: Active connections, per-endpoint latency

**Requires:** Application exposing HTTP metrics

### 3. Starter Template
**File:** `dashboards/starter-template.yaml`

**Use this to:** Create your own custom dashboards

## 🔧 Dashboard Structure Explained

Each dashboard is YAML with these sections:

```yaml
kind: "Dashboard"
metadata:
  name: "unique-id"                    # Internal ID
  project: "perses"                    # Project namespace
spec:
  display:
    name: "Dashboard Title"            # What users see
  duration: "1h"                       # Default time range
  
  variables:                           # Dynamic filters
    - name: "instance"                 # Use as $instance in queries
      ...
  
  panels:                              # Chart definitions
    "panel-id":
      kind: "Panel"
      spec:
        plugin:
          kind: "TimeSeriesChart"      # Chart type
          spec:
            queries:
              - query: "up{instance=~\"$instance\"}"  # PromQL
  
  layouts:                             # Panel positioning
    - kind: "Grid"                     # 24-column grid
      spec:
        items:
          - x: 0                       # Position (0-23)
            y: 0
            width: 12                  # Columns (1-24)
            height: 6                  # Height units
```

## ✨ Key Features

### ✅ Version Control
All dashboards are YAML files → commit to Git, review changes, rollback easily

### ✅ Reusable
Copy dashboard files, modify queries, deploy across teams/projects

### ✅ Variables
Create dynamic dashboards with dropdowns (instance, environment, etc.)

### ✅ Automatic Reload
Perses detects changes every 10 seconds → edit YAML, see updates immediately

## 🚀 Next Steps

1. **Connect to your Prometheus**
   - Edit `datasources/prometheus.yaml` with your URL
   
2. **Set up Prometheus scraping**
   - See `prometheus/prometheus.yml` for examples
   - At minimum, scrape Prometheus itself
   
3. **Start Perses**
   - `docker-compose up -d`
   - Visit http://localhost:8080
   
4. **Verify data flows**
   - Check dashboards show metrics
   - Adjust time ranges if needed
   
5. **Customize dashboards**
   - Edit YAML files in `dashboards/`
   - Changes appear automatically
   
6. **Create your own**
   - Copy `starter-template.yaml`
   - Modify for your use case

## 📚 Documentation

- **QUICKSTART.md** - 5-minute setup guide
- **README.md** - Complete reference with troubleshooting
- **prometheus/prometheus.yml** - Scraping examples

## 🆘 Common Issues

**No data in dashboards?**
→ Prometheus needs scrape targets configured

**Can't connect to Prometheus?**
→ Update `direct_url` in `datasources/prometheus.yaml`

**Dashboard not appearing?**
→ Wait 10 seconds or restart: `docker-compose restart perses`

## 💡 Tips

- Start simple: Scrape Prometheus itself first
- Use the starter template for new dashboards
- Reference variables in queries: `$variable_name`
- Grid is 24 columns wide (common: 6, 8, 12, 24 width)
- Check PromQL syntax in Prometheus UI first

## 🎓 Learn More

- [Perses Docs](https://perses.dev/docs)
- [PromQL Guide](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Prometheus Exporters](https://prometheus.io/docs/instrumenting/exporters/)

---

**You're all set!** Start with QUICKSTART.md and you'll have dashboards running in minutes. 🚀
