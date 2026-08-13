# SRE Fundamentals Cheatsheet

A field reference guide for Site Reliability Engineering: philosophy, reliability math, observability tooling, chaos engineering, and capacity planning.

## 1. SRE Fundamentals & Philosophy

**DevOps vs SRE**

- **DevOps**: The philosophy/culture (abstract). Breaks down silos between Dev and Ops.
- **SRE**: The implementation (concrete). Software engineers doing operations work.
- Mantra: *"Hope is not a strategy."*

**Toil (The Enemy)**

Toil is work that is:

- Manual
- Repetitive
- Automatable
- Tactical (no enduring value)
- Scales linearly with service growth (more users = more work)

**The Four Golden Signals**

| Signal | Question | Metric Type |
|---|---|---|
| Latency | How long does it take? | Histogram (time) |
| Traffic | How much demand? | Counter (RPS / QPS) |
| Errors | How many failed? | Counter (rate of 5xx) |
| Saturation | How "full" is it? | Gauge (queue depth / memory) |

## 2. The Mathematics of Reliability

**Availability Formula**

```text
Availability = (Total Time - Downtime) / Total Time * 100%
```

**Error Budget**

```text
Error Budget = 100% - SLO Target
```

Example: If SLO is 99.9%, the Error Budget is 0.1%.

**The Nines Table (Memorize This)**

| Availability | Downtime / Year | Downtime / Month | Downtime / Week |
|---|---|---|---|
| 99% (2 Nines) | 3.65 days | 7.31 hours | 1.68 hours |
| 99.9% (3 Nines) | 8.76 hours | 43.8 minutes | 10.1 minutes |
| 99.99% (4 Nines) | 52.6 minutes | 4.38 minutes | 1.01 minutes |
| 99.999% (5 Nines) | 5.26 minutes | 26 seconds | 6 seconds |

**Burn Rate Matrix**

How fast are we spending the error budget?

```text
Burn Rate = (Actual Error Rate) / (Allowed Error Rate for SLO)
```

- **1x Burn**: Consumes the budget in exactly 30 days. (OK — normal pace)
- **14.4x Burn**: Consumes the budget in 2 days. (Page SRE!)
- **1 Hour Burn**: Budget gone in 1 hour. (All hands on deck!)

## 3. Prometheus & PromQL Cookbook

**Metric Types**

| Type | Example | Usage Rule |
|---|---|---|
| Counter (up only) | `http_requests_total` | Must use `rate()` or `increase()` |
| Gauge (up/down) | `memory_usage_bytes` | Do NOT use `rate()`. Use `avg_over_time()` |
| Histogram (buckets) | `request_duration_seconds_bucket` | Use `histogram_quantile()` |

**Essential PromQL Queries**

Request Rate (RPS):

```promql
rate(http_requests_total[5m])
sum(rate(http_requests_total[5m])) by (service)
```

Error Rate (%):

```promql
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100
```

99th Percentile Latency (P99):

```promql
histogram_quantile(0.99, sum(rate(http_request_duration_bucket[5m])) by (le))
```

CPU Usage %:

```promql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**AlertManager Configuration Template**

```yaml
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/T0000/B0000/XXXX'
        channel: '#ops-alerts'
        send_resolved: true
```

## 4. Distributed Tracing

**The Trace Anatomy**

- **Trace**: The whole journey (User -> Frontend -> Backend -> DB).
- **Span**: One segment (e.g., "SQL Query").
- **Context**: The ID passed between services.

**W3C Trace Context Headers**

If you write a manual HTTP client, you MUST forward these:

```text
traceparent: 00-{trace-id}-{span-id}-{flags}
tracestate: vendor=data
```

**Sampling Strategies**

| Type | When Decision is Made | Pros/Cons |
|---|---|---|
| Head-Based | At start (frontend) | Fast, but may miss rare/error traces |
| Tail-Based | At end (backend) | Captures errors/outliers, but more expensive |

**Python OTel Snippet**

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("checkout_process") as span:
    span.set_attribute("user.id", "12345")
    span.set_attribute("cart.value", 99.99)
    try:
        process_payment()
    except Exception as e:
        span.record_exception(e)
        span.set_status(trace.Status(trace.StatusCode.ERROR))
```

## 5. Chaos Engineering

**The Experiment Loop**

1. **Steady State**: "System is healthy (200 OK)."
2. **Hypothesis**: "If DB Replica dies, Master handles load."
3. **Injection**: Terminate Replica Pod.
4. **Verify**: Did 5xx rate increase?
5. **Rollback**: Restore Replica.

**Chaos Mesh YAML Templates**

Pod Kill (the classic):

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-kill-example
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      "app": "nginx"
  scheduler:
    cron: "@every 10m"
```

Network Latency (the silent killer):

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay-example
spec:
  action: delay
  mode: all
  selector:
    labelSelectors:
      "app": "backend-api"
  delay:
    latency: "200ms"
    jitter: "50ms"
```

## 6. Capacity Planning & Scaling

**Little's Law**

```text
L (Concurrency) = A (RPS) * W (Latency)
```

Warning: If Latency (W) goes from 0.1s to 1.0s (10x), your RAM usage / in-flight requests (L) increases 10x instantly, even if traffic is constant.

**Cost Optimization (AWS/Cloud)**

| Pricing Model | Cost | Best For |
|---|---|---|
| On-Demand | $$$$ (Highest) | Spikes, chaos, unpredictable load |
| Reserved (RI) | $$ (Medium) | Databases, base load |
| Spot | $ (Lowest) | Stateless workers, batch jobs |

## 7. Quick Lookup Reference

**Common Ports**

| Port | Service |
|---|---|
| 9090 | Prometheus UI |
| 9100 | Node Exporter |
| 9093 | AlertManager |
| 3000 | Grafana |
| 16686 | Jaeger UI |
| 4317 | OTel gRPC |

**HTTP Status Codes (SRE View)**

| Range | Meaning | SRE Action | Impact on SLO? |
|---|---|---|---|
| 2xx | Success | Sleep well | Positive |
| 3xx | Redirect | Ignore mostly | Neutral |
| 4xx | Client Error | Check logs (bad request) | Usually no (user's fault) |
| 5xx | Server Error | WAKE UP | Yes (burns budget) |

**Linux Debugging Commands**

```bash
htop / top
df -h
netstat -tulpn
tail -f /var/log/syslog
pstree -p
curl -v telnet://hostname:port
```

---

*Source: adapted from the SRE Fundamentals cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
