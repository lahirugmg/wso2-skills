---
name: openchoreo-observability
description: Use for setting up observability on OpenChoreo - Prometheus, Grafana, Jaeger tracing, logging with Loki, and metrics collection
---

# OpenChoreo Observability Skill

## When to Use This Skill

Use for:
- Setting up Prometheus and Grafana
- Configuring distributed tracing with Jaeger
- Log aggregation with Loki
- Creating dashboards and alerts
- Implementing observability in applications

## Prometheus Setup

```yaml
# prometheus-values.yaml
server:
  retention: 30d
  persistentVolume:
    size: 50Gi

  global:
    scrape_interval: 15s
    evaluation_interval: 15s

alertmanager:
  enabled: true

nodeExporter:
  enabled: true

kubeStateMetrics:
  enabled: true
```

## Application Instrumentation

### Go Application

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )

    httpDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
)

// Middleware
func metricsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()

        next.ServeHTTP(w, r)

        duration := time.Since(start).Seconds()
        httpDuration.WithLabelValues(r.Method, r.URL.Path).Observe(duration)
        httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, fmt.Sprintf("%d", w.StatusCode)).Inc()
    })
}
```

## Grafana Dashboards

```json
{
  "dashboard": {
    "title": "Service Overview",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total[5m])) by (service)"
        }]
      },
      {
        "title": "Error Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))"
        }]
      }
    ]
  }
}
```

## Distributed Tracing

```yaml
# jaeger-values.yaml
allInOne:
  enabled: true

storage:
  type: elasticsearch

agent:
  enabled: true
```

## Related Skills

- `openchoreo-sre`
- `openchoreo-troubleshooting`
- `openchoreo-devops`

---

**Last Updated**: March 2025
