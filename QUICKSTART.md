# Quick Setup Guide

## 🚀 Get Started in 5 Minutes

### Step 1: Update Prometheus Connection

Edit `datasources/prometheus.yaml` and change the URL to your Prometheus instance:

```yaml
spec:
  plugin:
    kind: "PrometheusDatasource"
    spec:
      direct_url: "http://YOUR-PROMETHEUS-HOST:9090"
```

### Step 2: Configure Prometheus Scraping (If Not Already Done)

Since your Prometheus isn't scraping yet, you need to add scrape targets.

**Option A: If Prometheus is running in Docker**

1. Edit your Prometheus config file (usually `prometheus.yml`)
2. Add scrape configs (see `prometheus/prometheus.yml` in this repo for examples)
3. Restart Prometheus:
   ```bash
   docker restart prometheus
   ```

**Option B: Quick Test Setup**

Add Prometheus to scrape itself:

```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

**Option C: Add Node Exporter for System Metrics**

```bash
# Start Node Exporter
docker run -d \
  --name node-exporter \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  prom/node-exporter \
  --path.rootfs=/host

# Add to Prometheus config
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

### Step 3: Start Perses

```bash
# From the repository directory
docker-compose up -d

# Check it's running
docker-compose ps

# View logs
docker-compose logs -f perses
```

### Step 4: Access Perses UI

Open your browser to: **http://localhost:8080**

You should see the Perses dashboard with your dashboards listed.

### Step 5: Verify Data

1. Click on "System Monitoring" dashboard
2. Select an instance from the dropdown
3. You should see data if Prometheus is scraping

**If no data appears:**
- Check Prometheus has data: Visit `http://YOUR-PROMETHEUS:9090` and query `up`
- Verify the datasource URL is correct in `datasources/prometheus.yaml`
- Check Perses logs: `docker-compose logs perses`

## 📝 What's Included

This repository includes:

✅ **Pre-configured dashboards:**
- System Monitoring (CPU, memory, disk, network)
- Application Monitoring (RED metrics: Rate, Errors, Duration)
- Starter Template (for creating your own)

✅ **Prometheus datasource** ready to connect

✅ **Docker setup** for easy deployment

✅ **Dashboard-as-code** - all dashboards version controlled in YAML

## 🎯 Next Steps

1. **Customize dashboards** - Edit the YAML files in `dashboards/`
2. **Add more metrics** - Configure Prometheus to scrape more targets
3. **Create dashboards** - Copy `starter-template.yaml` and modify
4. **Set up alerts** - Add Prometheus alerting rules

## 🆘 Troubleshooting

**Problem: Dashboard shows but no data**
- ✓ Check Prometheus is reachable: `curl http://YOUR-PROMETHEUS:9090/api/v1/query?query=up`
- ✓ Verify datasource URL in `datasources/prometheus.yaml`
- ✓ Check Prometheus is scraping: Go to Prometheus UI → Status → Targets

**Problem: Perses won't start**
- ✓ Check Docker logs: `docker-compose logs perses`
- ✓ Verify port 8080 is available: `netstat -an | grep 8080`
- ✓ Check YAML syntax: `yamllint config/config.yaml`

**Problem: Dashboard not appearing**
- ✓ Wait 10 seconds (auto-reload interval)
- ✓ Check file is in `dashboards/` directory
- ✓ Validate YAML syntax
- ✓ Restart Perses: `docker-compose restart perses`

## 📚 Learn More

- Full README with detailed documentation
- Example dashboards in `dashboards/` directory
- Sample Prometheus config in `prometheus/prometheus.yml`
- [Perses Documentation](https://perses.dev/docs)
