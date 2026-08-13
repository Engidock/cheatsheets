# Prometheus Cheatsheet

Complete detailed reference guide for Prometheus monitoring and time-series database — installation, PromQL, alerting, exporters, storage, and Grafana integration.

## 🎯 Prometheus Fundamentals

### Installation & Setup

Install Prometheus:

```bash
# Download Prometheus
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz

# Extract
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64

# Run Prometheus
./prometheus --config.file=prometheus.yml

# Run with custom port
./prometheus --config.file=prometheus.yml --web.listen-address=":9091"

# Check version
./prometheus --version

# Run as systemd service
sudo systemctl start prometheus
sudo systemctl status prometheus
sudo systemctl enable prometheus
```

Basic Configuration (`prometheus.yml`):

```yaml
# Global settings
global:
  scrape_interval:     15s        # Default scrape interval
  evaluation_interval: 15s        # Default evaluation interval
  external_labels:
    monitor: 'prometheus'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - localhost:9093

# Load alerting rules
rule_files:
  - "alert.rules.yml"
  - "recording.rules.yml"

# Scrape configurations
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:9323']
```

### Prometheus Architecture

Key Components:

```text
# Prometheus Server
# - Scrapes metrics from targets
# - Stores time-series data
# - Executes alerting rules
# - Web UI on port 9090

# Exporters
# - Node Exporter (system metrics)
# - cAdvisor (container metrics)
# - Blackbox Exporter (probing)
# - Custom exporters

# Alertmanager
# - Handles alerts
# - Groups and deduplicates
# - Routes to notification channels

# Grafana
# - Visualization dashboard
# - Query builder interface
# - Plugin support

# Service Discovery
# - Kubernetes SD
# - Consul SD
# - EC2 SD
# - File SD
```

## 📈 Metrics & Labels

### Metric Types

Four Core Metric Types:

```promql
# Counter - only increases or resets
# Example: requests_total, errors_total
requests_total{method="GET",endpoint="/api"} 1234
http_requests_total{code="200"} 5000

# Gauge - can increase or decrease
# Example: temperature, disk_usage, connections
cpu_temperature_celsius 65.5
memory_usage_bytes 2147483648
active_connections 42

# Histogram - tracks distribution of values
# Creates _bucket, _count, _sum
request_duration_seconds_bucket{le="0.1"} 100
request_duration_seconds_bucket{le="0.5"} 500
request_duration_seconds_bucket{le="1.0"} 900
request_duration_seconds_sum 450
request_duration_seconds_count 1000

# Summary - similar to histogram but no buckets
response_size_bytes_sum 1024000
response_size_bytes_count 512
response_size_bytes{quantile="0.5"} 2000
response_size_bytes{quantile="0.9"} 5000
response_size_bytes{quantile="0.99"} 9000
```

Labels & Naming Conventions:

```promql
# Labels identify metrics
metric_name{label1="value1", label2="value2"} value

# Metric naming conventions
# - lowercase with underscores
# - include unit suffix
# - start with namespace

# Examples
http_requests_total{method="GET"}        # Total requests
http_request_duration_seconds            # Request duration
node_memory_MemFree_bytes                # Free memory
process_cpu_seconds_total                # CPU seconds
container_network_receive_bytes_total    # Network received

# Common labels
instance       # Target instance
job            # Job name
handler        # Handler/endpoint
method         # HTTP method
code           # HTTP status code
le             # Less than or equal (histogram bucket)
quantile       # Percentile (summary)

# Label matching operators
=              # Exact match
!=             # Not equal
=~             # Regex match
!~             # Regex not match
```

## ⚡ PromQL - Prometheus Query Language

### Basic Queries

Query Examples - Instant Queries:

```promql
# Return current value of metric
node_cpu_seconds_total

# Query with label filter
node_memory_MemFree_bytes{instance="localhost:9100"}

# Multiple label filters
http_requests_total{job="api-server", method="GET"}

# Regex label matching
http_requests_total{endpoint=~"/api/.*"}

# Not equal
http_requests_total{code!="200"}

# Exclude with regex
http_requests_total{endpoint!~"/health|/metrics"}

# Return value
up{job="prometheus"}              # 1 = up, 0 = down
```

Range Queries & Offsets:

```promql
# Range query (last 5 minutes)
node_cpu_seconds_total[5m]

# Other time ranges
http_requests_total[1h]           # Last hour
requests_total[1d]                # Last day
errors_total[30s]                 # Last 30 seconds
queries_total[7d]                 # Last 7 days

# Offset (reference past time)
http_requests_total offset 5m      # 5 minutes ago
rate(requests_total[5m] offset 1h) # 1 hour ago

# Current value vs 1 hour ago
node_memory_MemFree_bytes - node_memory_MemFree_bytes offset 1h
```

Operators & Functions:

```promql
# Arithmetic operators
node_memory_MemFree_bytes / 1024  # Convert to KB
rate(requests_total[5m]) * 60     # Convert to per-minute

# Comparison operators
node_memory_MemFree_bytes > 1000000
http_requests_total < 100
cpu_usage_percent >= 80
disk_usage_bytes <= 5000000

# Logical operators
up == 1 and job == "prometheus"
requests_total > 1000 or errors_total > 10

# Built-in functions
rate()              # Per-second average
increase()          # Total increase
irate()             # Instant rate of change
abs()               # Absolute value
round()             # Round to nearest
floor()             # Round down
ceil()              # Round up
sqrt()              # Square root
```

### Advanced PromQL

Aggregation Operators:

```promql
# Sum aggregation
sum(rate(requests_total[5m]))

# Sum by label
sum by (handler) (rate(http_requests_total[5m]))

# Sum without label
sum without (instance) (node_cpu_seconds_total)

# Average
avg(cpu_usage_percent)
avg by (job) (cpu_usage_percent)

# Count
count(up)
count by (job) (up)

# Min/Max
min(disk_free_bytes)
max(memory_usage_bytes)
max by (instance) (cpu_temperature)

# Quantile
quantile(0.95, request_duration_seconds)
quantile(0.99, http_request_duration_seconds)

# Standard deviation
stddev(response_time_seconds)

# Top-K
topk(5, by_bandwidth)
bottomk(3, least_used_services)
```

Rate & Increase Functions:

```promql
# rate() - per-second average rate
rate(http_requests_total[5m])     # Avg requests/sec over 5min
rate(errors_total[1m])            # Errors per second

# increase() - total increase over period
increase(http_requests_total[1h]) # Total requests in last hour
increase(errors_total[5m])        # Total errors in last 5min

# irate() - instant rate (last 2 points)
irate(http_requests_total[5m])    # More responsive than rate

# Using rate in expressions
rate(requests_total[5m]) > 100    # Alert if > 100 req/sec
increase(errors_total[5m]) > 10   # Alert if > 10 errors in 5min

# Combining metrics
rate(requests_total[5m]) / rate(total_operations[5m])  # Success rate
```

Joins & Binary Operators:

```promql
# One-to-one join
http_requests_total / http_requests_failed

# One-to-many join with group_left
sum by (instance) (requests_total)
  / on(instance) group_left
sum by (instance) (errors_total)

# Group modifier
sum(requests_total) by (job, handler)
avg(response_time) by (instance, method)

# Label replacement
label_replace(metric, "dst_label", "$1", "src_label", "regex")

# Vector scalar operations
vector_metric * 1.5               # Multiply all values
vector_metric + 10                # Add constant
sum(metric) / count(metric)       # Average
```

### Common Query Patterns

Real-World Query Examples:

```promql
# Request rate
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{code=~"5.."}[5m])

# Success rate percentage
(sum(rate(http_requests_total{code="200"}[5m])) /
sum(rate(http_requests_total[5m]))) * 100

# API latency (95th percentile)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# CPU usage percentage
(1 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))) * 100

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage percentage
(node_filesystem_avail_bytes{device=~"/dev/.*"} / node_filesystem_size_bytes) * 100

# Network I/O
rate(node_network_receive_bytes_total[5m]) +
rate(node_network_transmit_bytes_total[5m])

# Top 5 endpoints by request count
topk(5, sum by (handler) (rate(http_requests_total[5m])))

# Instances down
count(up == 0) by (job)
```

## 🚨 Alerting

### Alert Rules

Alert Rules Configuration:

```yaml
# alert.rules.yml
groups:
  - name: system_alerts
    interval: 30s
    rules:
      # Alert if instance is down
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} is down for 5 minutes"

      # Alert if high CPU usage
      - alert: HighCPUUsage
        expr: |
          (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)) > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value | humanizePercentage }}"

      # Alert if high memory usage
      - alert: HighMemoryUsage
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) > 0.9
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value | humanizePercentage }}"

      # Alert if high error rate
      - alert: HighErrorRate
        expr: |
          (sum(rate(http_requests_total{code=~"5.."}[5m])) by (job) /
          sum(rate(http_requests_total[5m])) by (job)) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate in {{ $labels.job }}"
          description: "Error rate is {{ $value | humanizePercentage }}"
```

Recording Rules:

```yaml
# recording.rules.yml
groups:
  - name: recording_rules
    interval: 15s
    rules:
      # Request rate per job
      - record: job:request_rate:1m
        expr: sum by (job) (rate(http_requests_total[1m]))

      # Error rate per job
      - record: job:error_rate:5m
        expr: |
          sum by (job) (rate(http_requests_total{code=~"5.."}[5m])) /
          sum by (job) (rate(http_requests_total[5m]))

      # Latency 95th percentile
      - record: job:latency:95th
        expr: |
          histogram_quantile(0.95, sum by (le, job) (rate(http_request_duration_seconds_bucket[5m])))

      # CPU usage percentage
      - record: instance:cpu_usage:percent
        expr: |
          (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100

      # Memory usage percentage
      - record: instance:memory_usage:percent
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

## 🔧 Exporters & Scraping

### Popular Exporters

Node Exporter (System Metrics):

```bash
# Install Node Exporter
cd /opt
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.6.1.linux-amd64.tar.gz
./node_exporter-1.6.1.linux-amd64/node_exporter
```

```yaml
# Add to Prometheus config
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

```promql
# Common node_exporter metrics
node_cpu_seconds_total             # CPU time
node_memory_MemFree_bytes          # Free memory
node_memory_MemTotal_bytes         # Total memory
node_disk_free_bytes               # Disk free space
node_disk_io_time_seconds_total    # Disk I/O time
node_filesystem_avail_bytes        # Filesystem available
node_network_receive_bytes_total   # Network received
node_processes_running             # Running processes
node_load1                         # 1-minute load average
```

cAdvisor (Container Metrics):

```bash
# Run cAdvisor
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:latest
```

```yaml
# Add to Prometheus config
scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:8080']
```

```promql
# Common cAdvisor metrics
container_cpu_usage_seconds_total  # CPU usage
container_memory_usage_bytes       # Memory usage
container_network_receive_bytes    # Network received
container_network_transmit_bytes   # Network transmitted
container_fs_usage_bytes           # Filesystem usage
container_last_seen                # Container last seen
```

Blackbox Exporter (Probing):

```bash
# Install Blackbox Exporter
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.24.0/blackbox_exporter-0.24.0.linux-amd64.tar.gz
tar xvfz blackbox_exporter-0.24.0.linux-amd64.tar.gz
./blackbox_exporter-0.24.0.linux-amd64/blackbox_exporter
```

```yaml
# Prometheus config for HTTP checks
scrape_configs:
  - job_name: 'blackbox_http'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
          - https://api.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115
```

```promql
# Blackbox metrics
probe_success                      # Probe succeeded
probe_duration_seconds             # Probe duration
probe_http_status_code             # HTTP status code
probe_http_version                 # HTTP version
```

## 📊 Time Series Database

### Storage & Retention

Data Storage Configuration:

```bash
# Default storage location
/prometheus/data

# Start Prometheus with custom storage path
./prometheus --storage.tsdb.path=/custom/path

# Retention settings
./prometheus --storage.tsdb.retention.time=30d    # Retention time
./prometheus --storage.tsdb.retention.size=50GB   # Max size
./prometheus --storage.tsdb.max-block-duration=2h # Max block duration

# Check TSDB status
curl http://localhost:9090/api/v1/status/tsdb

# Compact storage
curl -X POST http://localhost:9090/api/v1/admin/tsdb/clean_tombstones
```

Storage layout:

```text
data/
  wal/                            # Write-Ahead Log
  block_time1/                    # Data block 1
  block_time2/                    # Data block 2
```

Performance Tuning:

```bash
# Query timeout
./prometheus --query.timeout=2m

# Query max concurrency
./prometheus --query.max-concurrency=20

# Max samples limit
./prometheus --query.max-samples=10000000

# Lookback duration (for instant queries)
./prometheus --query.lookback-delta=5m

# Block duration
./prometheus --storage.tsdb.min-block-duration=2h
./prometheus --storage.tsdb.max-block-duration=31d
```

```yaml
# Scrape configuration tuning
# Lower scrape interval = more storage
# Higher scrape interval = less storage but less granularity
global:
  scrape_interval: 15s            # Default 15 seconds
  scrape_timeout: 10s             # Default 10 seconds
  evaluation_interval: 15s        # Default 15 seconds
```

## 🔌 Grafana Integration

### Grafana Datasource & Dashboards

Setup Prometheus in Grafana:

```text
1. Go to Configuration > Data Sources
2. Click "Add data source"
3. Select "Prometheus"
4. Set URL: http://prometheus-server:9090
5. Click "Save & Test"
```

Grafana HTTP request to add datasource:

```bash
curl -X POST http://localhost:3000/api/datasources \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Prometheus",
    "type": "prometheus",
    "url": "http://localhost:9090",
    "access": "proxy",
    "isDefault": true
  }'
```

Common dashboard queries in Grafana:

```promql
# CPU Usage
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100

# Memory Available
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024

# Disk Usage
100 - (node_filesystem_avail_bytes / node_filesystem_size_bytes * 100)

# Request Rate
sum(rate(http_requests_total[5m])) by (job)
```

Dashboard Panel Types:

```promql
# Graph Panel
# Shows metrics over time
# Query: rate(http_requests_total[5m])

# Stat Panel
# Shows current value
# Query: up

# Gauge Panel
# Shows value on gauge
# Query: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Bar Gauge
# Horizontal bars for multiple series
# Query: topk(10, sum by (handler) (http_requests_total))

# Table
# Tabular data display
# Query: up

# Heatmap
# Shows distribution over time
# Query: rate(http_request_duration_seconds_bucket[5m])

# Logs Panel
# For log visualization
# Requires Loki datasource
```

## 📋 Quick Reference Commands

| Operation | Command | Example |
|---|---|---|
| Current metric value | `metric_name` | `up` |
| Filter by label | `metric{label="value"}` | `http_requests_total{code="200"}` |
| Rate calculation | `rate(metric[range])` | `rate(requests_total[5m])` |
| Total increase | `increase(metric[range])` | `increase(errors_total[1h])` |
| Aggregation | `sum/avg/max/min(metric)` | `sum(requests_total)` |
| Group by label | `sum by (label)(metric)` | `sum by (job)(requests_total)` |
| Quantile | `histogram_quantile(0.95, metric)` | `histogram_quantile(0.95, request_duration)` |
| Comparison | `metric > value` | `cpu_usage > 80` |
| Range vector | `metric[time_range]` | `cpu_usage[5m]` |
| Label matching | `metric{label=~"regex"}` | `up{job=~"api.*"}` |

## ✅ Best Practices

### ✓ Metric Design

- Use descriptive metric names
- Include units in metric names
- Use lowercase with underscores
- Add helpful labels for filtering
- Avoid high cardinality labels

### ✓ Scraping Strategy

- Set appropriate scrape intervals
- Use service discovery when possible
- Monitor your exporters
- Set reasonable timeout values
- Use static configs for testing only

### ✓ Query Performance

- Use recording rules for complex queries
- Limit query range when possible
- Use appropriate time ranges
- Aggregate early in queries
- Cache expensive queries

### ⚠️ Common Mistakes

- High cardinality labels (like IDs)
- Unbounded time ranges in alerts
- Too frequent scraping intervals
- Mixing count and gauge metrics
- Not using recording rules for dashboards

---

*Source: adapted from the Prometheus cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
