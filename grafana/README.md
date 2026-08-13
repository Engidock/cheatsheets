# Grafana Cheatsheet

A complete, practical quick-reference for installing, configuring, and operating Grafana — datasources, dashboards, panels, templating, alerting, users/RBAC, the HTTP API, and plugins.

---

## 🎯 Grafana Fundamentals

### Installation & Setup

Install Grafana (Ubuntu/Debian package manager):

```bash
# Using package manager (Ubuntu/Debian)
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt-get update
sudo apt-get install grafana-server

# Start Grafana service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

Using Docker:

```bash
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  -e GF_SECURITY_ADMIN_PASSWORD=admin \
  grafana/grafana:latest
```

Docker Compose:

```yaml
version: '3'
services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana-storage:/var/lib/grafana
volumes:
  grafana-storage:
```

Initial configuration — access Grafana and default credentials:

```bash
# Access Grafana
http://localhost:3000

# Default credentials
Username: admin
Password: admin
```

Grafana configuration file location:

```bash
/etc/grafana/grafana.ini
```

Important `grafana.ini` settings:

```ini
[server]
  http_port = 3000
  root_url = http://grafana.example.com

[security]
  admin_user = admin
  admin_password = secretpassword
  secret_key = your-secret-key

[database]
  type = sqlite3
  path = /var/lib/grafana/grafana.db

[log]
  level = info
  mode = console
```

Docker environment variables:

```bash
GF_SECURITY_ADMIN_PASSWORD=admin          # Admin password
GF_INSTALL_PLUGINS=grafana-piechart-panel # Install plugins
GF_USERS_ALLOW_SIGN_UP=false              # Disable signup
```

### Grafana Architecture

Core components:

```text
1. Web Server
   - Hosts UI on port 3000
   - API endpoints
   - Dashboard serving

2. Database
   - SQLite (default)
   - MySQL
   - PostgreSQL
   - Stores configuration and metadata

3. Datasources
   - Prometheus
   - InfluxDB
   - Graphite
   - Elasticsearch
   - CloudWatch
   - Azure Monitor
   - Custom HTTP endpoints

4. Plugins
   - Panel plugins (visualization)
   - App plugins (applications)
   - Datasource plugins (data sources)

5. Alerting Engine
   - Alert rules evaluation
   - Notification channels
   - Alert state management

6. User Management
   - RBAC (Role-Based Access Control)
   - Organization management
   - Team management
   - Authentication (LDAP, OAuth, SAML)
```

---

## 📈 Datasources & Connections

### Adding Datasources

Add a Prometheus datasource via the UI:

```text
1. Configuration > Data Sources
2. Click "Add data source"
3. Select "Prometheus"
4. Set URL: http://prometheus-server:9090
5. Leave authentication disabled (if local)
6. Click "Save & Test"
```

Add a Prometheus datasource via the HTTP API:

```bash
curl -X POST http://localhost:3000/api/datasources \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "name": "Prometheus",
    "type": "prometheus",
    "url": "http://prometheus-server:9090",
    "access": "proxy",
    "isDefault": true,
    "jsonData": {
      "httpMethod": "GET"
    }
  }'
```

Add a Prometheus datasource via provisioning file:

```yaml
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus-server:9090
    access: proxy
    isDefault: true
```

Add an InfluxDB datasource via the UI:

```text
1. Configuration > Data Sources
2. Select "InfluxDB"
3. HTTP settings:
   - URL: http://influxdb-server:8086
   - Access: proxy
4. InfluxDB Details:
   - Database: telegraf (or your database)
   - Username: (if required)
   - Password: (if required)
5. Save & Test
```

Add an InfluxDB datasource via the API:

```bash
curl -X POST http://localhost:3000/api/datasources \
  -H "Content-Type: application/json" \
  -d '{
    "name": "InfluxDB",
    "type": "influxdb",
    "url": "http://influxdb-server:8086",
    "access": "proxy",
    "database": "telegraf",
    "jsonData": {
      "timeInterval": "10s"
    }
  }'
```

Add an Elasticsearch datasource via the UI:

```text
1. Configuration > Data Sources
2. Select "Elasticsearch"
3. HTTP settings:
   - URL: http://elasticsearch-server:9200
   - Access: proxy
4. Elasticsearch Details:
   - Index name: logstash-* (or your pattern)
   - Time field name: @timestamp
5. Save & Test
```

Test query in Elasticsearch:

```json
GET logstash-*/_search
{
  "query": {
    "match_all": {}
  }
}
```

---

## 📊 Dashboard & Panel Types

### Dashboard Creation

Create a new dashboard via the UI:

```text
1. Click "+" icon > Dashboard
2. Click "Add new panel"
3. Select datasource (e.g., Prometheus)
4. Enter query in query editor
5. Configure visualization
6. Save dashboard
```

Create a dashboard via the API:

```bash
curl -X POST http://localhost:3000/api/dashboards/db \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "dashboard": {
      "title": "My Dashboard",
      "tags": ["monitoring", "production"],
      "timezone": "browser",
      "panels": [],
      "schemaVersion": 16,
      "version": 0
    },
    "overwrite": false
  }'
```

Dashboard settings overview:

```text
1. General
   - Title
   - Description
   - Tags
   - Timezone

2. Templating
   - Variables
   - Query parameters

3. Annotations
   - Events
   - Markers on timeline

4. Time range
   - Default time range
   - Refresh interval
```

### Panel Types & Usage

Graph Panel — line/area/bar chart over time, for trends and time-series data:

```promql
rate(requests_total[5m])
```

Stat Panel — single value with optional sparkline, for current metrics and KPIs:

```promql
up
```

Gauge Panel — circular gauge display, for percentages and health scores:

```promql
(memory_used / memory_total) * 100
```

Bar Gauge — horizontal bars for multiple series, for ranking and comparison:

```promql
topk(10, requests_total)
```

Table Panel — tabular data display, for logs and detailed data:

```promql
up
```

Heatmap Panel — 2D grid with color values, for distributions and patterns:

```promql
rate(response_time_bucket[5m])
```

Piechart Panel — pie/donut chart, for parts of a whole:

```promql
sum(requests_total) by (handler)
```

Other panel types:

```text
Alert List      # Shows active alerts — use for alert monitoring
Dashboard List  # Lists other dashboards — use for dashboard navigation
Text Panel      # Static text/markdown — use for documentation, titles
Worldmap Panel  # Geographic visualization — use for global metrics
Log Panel       # Log viewer — use for Loki/Elasticsearch logs
```

### Panel Configuration

Query and panel options:

```text
# Query Options
1. Query
   - Metric/Data source query
   - Multiple queries (A, B, C, etc.)

2. Query Inspector
   - View raw data
   - JSON response

3. Panel Options
   - Title
   - Description
   - Transparent background
```

Visualization settings (Graph panel):

```text
1. Display
   - Line width
   - Fill opacity
   - Point size
   - Staircase mode

2. Axes
   - Left Y-axis
   - Right Y-axis
   - X-axis settings

3. Legend
   - Show/hide legend
   - Legend placement
   - Legend values (min, max, avg, current)
   - Sort options

4. Alert
   - Alert state
   - Threshold
   - No data / Execution error handling
```

Threshold settings:

```text
1. Single threshold
2. Multiple thresholds
3. Color mapping
```

Value mapping — maps numeric values to text:

```text
0 => "Down"
1 => "Up"
```

Unit selection:

```text
bytes        # Bytes
percent      # Percentage
ms           # Milliseconds
s            # Seconds
ops          # Operations
```

### Templating & Variables

Variable types:

```text
1. Query        - From datasource
2. Constant     - Fixed value
3. Custom       - User defined
4. Datasource   - Datasource picker
5. Ad hoc filters - Dynamic filters
```

Create a query variable:

```text
Label: Instance
Name: instance
Datasource: Prometheus
Query: label_values(up, instance)
Multi-value: enabled
Include All option: enabled
```

Use a variable in a query:

```promql
rate(http_requests_total{instance="$instance"}[5m])
```

Multiple variable selection:

```promql
sum(requests_total{instance=~"$instance"})
```

Variable syntax examples:

```text
$instance    # Single variable
${instance}  # Escaped variable
$__interval  # Grafana internal variable
$__range     # Time range
```

Built-in variables:

```text
$__interval      # Auto interval
$__interval_ms   # Interval in milliseconds
$__range         # Time range (e.g., 1h)
$__range_s       # Range in seconds
$__from          # Start timestamp (ms)
$__to            # End timestamp (ms)
$__timezone      # Browser timezone
$__user          # Current username
```

---

## 🎨 Advanced Panel Features

### Field Overrides & Transformations

Apply a field override:

```text
1. Go to panel > Overrides
2. Add field override
3. Select field pattern (e.g., "*.value")
4. Add properties:
   - Unit
   - Decimals
   - Min/Max
   - Color
   - Custom hidden
```

Example override patterns:

```text
Time     # Time field
Value    # Value field
Status   # Status field
*.value  # All fields ending in .value
cpu.*    # All fields starting with cpu.
```

Common override properties:

```text
Unit        # Change unit (bytes, percent, etc.)
Decimals    # Decimal places
Min value   # Minimum for gauge/bar
Max value   # Maximum for gauge/bar
Color mode  # Value, background, off
No value    # Display when no data
```

Conditional coloring:

```text
1. Thresholds
   - Define ranges
   - Assign colors

2. Value mapping
   - Map specific values to colors
   - Regular expressions supported
```

### Panel Transformations

Transform data before visualization:

```text
1. Go to panel > Transform data
2. Add transformation
3. Select transformation type
```

Transformation types:

```text
1. Organize fields
   # Rename, reorder, hide fields

2. Filter by name
   # Include/exclude fields by pattern

3. Filter data by value
   # Keep rows where field matches condition

4. Rename by regex
   # Use regex to rename fields
   # Pattern: .*_(.*)
   # Replacement: $1

5. Convert field type
   # String to number
   # Number to string
   # Etc.

6. Merge
   # Combine queries into a single result

7. Group by
   # Group data by field value

8. Histogram
   # Convert to histogram bucket data

9. Series to rows
   # Convert time series to rows

10. Extract fields
    # Extract data from JSON/Arrays
```

Example — extract a field from JSON:

```json
// Input
{"value": 42, "unit": "ms"}

// Extract "value" field
// Result: 42
```

---

## 🚨 Alerting

### Alert Rules & Notifications

Create an alert rule via the UI:

```text
1. Go to Alerting > Alert Rules
2. Click "New alert rule"
3. Configure query
4. Set condition/threshold
5. Set evaluation interval
6. Set for duration (5m, etc.)
7. Add labels and annotations
8. Save
```

Alert rule structure:

```yaml
Rule Name: HighCPUUsage
Datasource: Prometheus
Condition:
  Query: avg(cpu_usage_percent) by (instance)
  Condition: Is above 80
  For: 5 minutes

Labels:
  severity: warning
  team: platform

Annotations:
  summary: "High CPU usage on {{ $labels.instance }}"
  description: "CPU usage is {{ $value }}%"
  runbook_url: "https://wiki.example.com/high-cpu"
```

Alert notification setup:

```text
1. Set notification channel
2. Configure escalation
3. Set repeat interval
```

Common alert rules:

```text
- Instance down (up == 0)
- High CPU (cpu_usage > 80%)
- High memory (memory_used > 90%)
- High disk (disk_usage > 85%)
- High error rate (errors > threshold)
- Database slow queries
```

### Notification Channels

Configure a notification channel:

```text
1. Go to Alerting > Notification channels
2. Click "New channel"
3. Select channel type
4. Configure settings
5. Save
```

Supported channels and their key settings:

```text
# Email
- SMTP Server
- From address
- Recipient address

# Webhook
- URL
- HTTP method (POST)
- Custom headers
- Body template

# Slack
- Webhook URL: https://hooks.slack.com/services/...
- Channel: #alerts
- Mention users: @channel

# PagerDuty
- Integration key
- Incident severity

# Opsgenie
- API key
- Region (US/EU)
- Priority

# VictorOps
- API key
- Routing key

# Telegram
- Bot token
- Chat ID

# Discord
- Webhook URL
- Mention users
```

Example webhook payload:

```json
{
  "status": "firing",
  "alerts": [
    {
      "status": "firing",
      "labels": {
        "severity": "critical",
        "alertname": "HighCPU"
      },
      "annotations": {
        "description": "CPU is high"
      }
    }
  ]
}
```

---

## 👥 Users & Organizations

### User Management & RBAC

Add a new user:

```text
1. Administration > Users
2. Click "Invite"
3. Enter email
4. Select role
5. Send invite
```

User roles:

```text
Admin    # Full access to all features
Editor   # Can create dashboards/alerts
Viewer   # Read-only access
```

Organization management:

```text
1. Administration > Organizations
2. Create new organization
3. Add members
4. Assign roles
```

Organization roles:

```text
Admin    # Manage organization settings
Editor   # Create/edit dashboards
Viewer   # View dashboards
```

Team management:

```text
1. Administration > Teams
2. Create team
3. Add members
4. Set team permissions
```

Create an API token for automation:

```text
1. Administration > API Tokens
2. Create token
3. Select role (Admin/Editor/Viewer)
4. Set expiration
```

Add the token to a curl request:

```bash
curl -X POST http://localhost:3000/api/datasources \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

LDAP authentication — edit `/etc/grafana/grafana.ini`:

```ini
[auth.ldap]
enabled = true
config_file = /etc/grafana/ldap.toml
```

OAuth configuration (GitHub example):

```ini
[auth.github]
enabled = true
allow_sign_up = true
client_id = YOUR_CLIENT_ID
client_secret = YOUR_CLIENT_SECRET
```

---

## ⚙️ Grafana API

### API Endpoints

Authenticate and check current user:

```bash
curl -H "Authorization: Bearer YOUR_API_TOKEN" \
  http://localhost:3000/api/me
```

List datasources:

```bash
curl -H "Authorization: Bearer YOUR_API_TOKEN" \
  http://localhost:3000/api/datasources
```

Get a datasource by ID:

```bash
curl http://localhost:3000/api/datasources/1
```

Create a datasource:

```bash
curl -X POST http://localhost:3000/api/datasources \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{ "name": "Prometheus", ... }'
```

Update a datasource:

```bash
curl -X PUT http://localhost:3000/api/datasources/1 \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{ "name": "Updated Name" }'
```

Delete a datasource:

```bash
curl -X DELETE http://localhost:3000/api/datasources/1 \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

List dashboards:

```bash
curl http://localhost:3000/api/search?query=
```

Get a dashboard by slug:

```bash
curl http://localhost:3000/api/dashboards/db/dashboard-slug
```

Create a dashboard:

```bash
curl -X POST http://localhost:3000/api/dashboards/db \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{ "dashboard": {...}, "overwrite": false }'
```

Update a dashboard:

```bash
curl -X POST http://localhost:3000/api/dashboards/db \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{ "dashboard": {...}, "overwrite": true }'
```

List alerts:

```bash
curl http://localhost:3000/api/alerts
```

List alert rules:

```bash
curl http://localhost:3000/api/alert/rules
```

---

## 🔌 Plugins & Extensions

### Install & Manage Plugins

List remote plugins:

```bash
grafana-cli plugins list-remote
```

Search plugins:

```bash
grafana-cli plugins search grafana-piechart
```

Install a plugin:

```bash
grafana-cli plugins install grafana-piechart-panel
sudo systemctl restart grafana-server
```

Update a plugin:

```bash
grafana-cli plugins update grafana-piechart-panel
```

Uninstall a plugin:

```bash
grafana-cli plugins remove grafana-piechart-panel
```

Install plugins via Docker:

```bash
docker run -d \
  -e GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel \
  grafana/grafana:latest
```

Install multiple plugins (comma-separated):

```bash
GF_INSTALL_PLUGINS=plugin1,plugin2,plugin3
```

Popular plugins:

```text
1. Grafana Piechart Panel
2. Grafana Worldmap Panel
3. Grafana Status Panel
4. Grafana MultiStat Panel
5. ClickHouse datasource
6. MongoDB datasource
7. MySQL datasource
```

Plugin provisioning directory:

```bash
/etc/grafana/provisioning/plugins/
```

Enable unsigned plugins — edit `/etc/grafana/grafana.ini`:

```ini
[plugins]
allow_ui_updates = true
```

---

## 📋 Quick Reference Commands

| Operation           | Command / Step                          | Example                       |
|---------------------|------------------------------------------|--------------------------------|
| Access Grafana       | `http://localhost:3000`                  | Login with admin/admin        |
| Add datasource       | Configuration > Data Sources             | Prometheus, InfluxDB          |
| Create dashboard     | `+` > Dashboard > Add Panel              | Choose visualization type     |
| Create variable      | Dashboard > Templating                   | `$instance` variable          |
| Set alert            | Panel > Alert > Create Alert             | Threshold > 80%               |
| Add notification     | Alerting > Notifications                 | Slack, Email, Webhook         |
| Export dashboard     | Dashboard > Share > Export               | JSON format                   |
| Import dashboard     | `+` > Import Dashboard                   | From URL or JSON              |
| API call             | `curl -H "Authorization: Bearer TOKEN"`  | `GET /api/datasources`        |
| View logs            | `tail -f /var/log/grafana/grafana.log`   | Check errors                  |

---

## ✅ Best Practices

### ✓ Dashboard Design
- Use clear, descriptive titles
- Organize panels logically
- Use consistent color schemes
- Include units and labels
- Document dashboard purpose in description

### ✓ Performance
- Limit queries per dashboard
- Use appropriate time ranges
- Set refresh intervals wisely
- Use recording rules for complex queries
- Cache expensive queries

### ✓ Alerting
- Set meaningful thresholds
- Include runbook URLs
- Use clear annotations
- Test alerts before production
- Monitor alert fatigue

### ⚠️ Common Mistakes
- Too many panels per dashboard
- Unbounded time ranges
- High refresh intervals
- Missing alert descriptions
- Hardcoded values instead of variables

---

*Source: adapted from the Grafana cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
