---
name: openchoreo-sre
description: Use for Site Reliability Engineering on OpenChoreo - monitoring, incident response, performance tuning, capacity planning, and ensuring platform reliability
---

# OpenChoreo SRE Skill

## When to Use This Skill

Use this skill for:
- Monitoring OpenChoreo and applications
- Incident response and troubleshooting
- Performance optimization
- Capacity planning
- SLO/SLA management
- Chaos engineering
- Disaster recovery

## SLI/SLO/SLA Framework

### Define Service Level Indicators

```yaml
# SLIs for OpenChoreo Platform

slis:
  # Availability
  - name: platform_availability
    metric: (successful_requests / total_requests) * 100
    measurement_window: 30d

  - name: component_uptime
    metric: (component_up_time / total_time) * 100
    measurement_window: 30d

  # Latency
  - name: api_latency_p95
    metric: 95th_percentile(request_duration_seconds)
    measurement_window: 1h

  - name: api_latency_p99
    metric: 99th_percentile(request_duration_seconds)
    measurement_window: 1h

  # Error Rate
  - name: error_rate
    metric: (5xx_responses / total_responses) * 100
    measurement_window: 5m

  # Throughput
  - name: requests_per_second
    metric: rate(http_requests_total[5m])
    measurement_window: 5m

### Set Service Level Objectives

```yaml
slos:
  # Platform SLOs
  - name: openchoreo_availability
    sli: platform_availability
    target: 99.9%
    period: 30d

  - name: api_response_time
    sli: api_latency_p95
    target: < 200ms
    period: 1h

  - name: acceptable_error_rate
    sli: error_rate
    target: < 0.1%
    period: 1h

### Create Error Budgets

```yaml
error_budgets:
  - slo: openchoreo_availability
    budget: 43.2 minutes/month  # (100 - 99.9) * 30 * 24 * 60
    burn_rate_alerts:
      - threshold: 2x  # Consuming budget 2x faster
        alert: warning
      - threshold: 10x
        alert: critical

## Monitoring Setup

### Prometheus Configuration

```yaml
# prometheus-config.yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # OpenChoreo Platform
  - job_name: 'openchoreo-platform'
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names: ['openchoreo']
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__

  # Application Services
  - job_name: 'applications'
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names: ['production', 'staging']
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true

# Alerting rules
rule_files:
  - /etc/prometheus/rules/*.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

### Alert Rules

```yaml
# alerts/openchoreo-alerts.yml
groups:
  - name: openchoreo_platform
    interval: 30s
    rules:
      # High Error Rate
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) * 100 > 1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }}% (threshold: 1%)"

      # API Latency
      - alert: HighAPILatency
        expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)) > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "API latency is high"
          description: "P95 latency for {{ $labels.service }} is {{ $value }}s"

      # Pod Restarts
      - alert: PodRestarting
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod is restarting"
          description: "Pod {{ $labels.pod }} has restarted {{ $value }} times"

      # Memory Usage
      - alert: HighMemoryUsage
        expr: |
          (container_memory_usage_bytes / container_spec_memory_limit_bytes) * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Container {{ $labels.container }} using {{ $value }}% memory"

      # CPU Usage
      - alert: HighCPUUsage
        expr: |
          rate(container_cpu_usage_seconds_total[5m]) * 100 > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "Container {{ $labels.container }} using {{ $value }}% CPU"

      # Disk Space
      - alert: DiskSpaceLow
        expr: |
          (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space is low"
          description: "Only {{ $value }}% disk space available on {{ $labels.instance }}"

      # Certificate Expiry
      - alert: CertificateExpiringSoon
        expr: (cert_exporter_cert_expires_in_seconds / 86400) < 30
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "Certificate expiring soon"
          description: "Certificate {{ $labels.subject }} expires in {{ $value }} days"
```

### Grafana Dashboards

```json
// openchoreo-overview-dashboard.json
{
  "dashboard": {
    "title": "OpenChoreo Platform Overview",
    "panels": [
      {
        "title": "Platform Availability",
        "targets": [
          {
            "expr": "(sum(up{job=\"openchoreo-platform\"}) / count(up{job=\"openchoreo-platform\"})) * 100"
          }
        ],
        "type": "gauge"
      },
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (service)"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "(sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))) * 100"
          }
        ],
        "type": "graph"
      },
      {
        "title": "P95 Latency",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

## Incident Response

### Incident Management Process

```yaml
# incident-response-runbook.yaml
runbook:
  - phase: detection
    steps:
      - Monitor alerts
      - Check dashboards
      - Review logs

  - phase: triage
    steps:
      - Assess severity
      - Determine impact
      - Assign incident commander
      - Create incident channel (Slack)

  - phase: investigation
    steps:
      - Gather symptoms
      - Check recent changes
      - Review logs and metrics
      - Form hypothesis

  - phase: mitigation
    steps:
      - Execute fix
      - Rollback if needed
      - Verify resolution
      - Monitor stability

  - phase: recovery
    steps:
      - Restore full service
      - Verify SLOs met
      - Close incident

  - phase: postmortem
    steps:
      - Write postmortem (within 48h)
      - Identify root cause
      - Create action items
      - Share learnings
```

### Common Incident Scenarios

**Scenario 1: High Pod Restart Rate**

```bash
# Investigate
kubectl get pods -n openchoreo | grep -v Running
kubectl describe pod <pod-name> -n openchoreo
kubectl logs <pod-name> -n openchoreo --previous

# Common causes:
# - OOMKilled: Increase memory limits
# - CrashLoopBackOff: Fix application bug
# - ImagePullBackOff: Check image availability

# Mitigation
kubectl edit deployment <deployment-name> -n openchoreo
# Increase resources or fix configuration
```

**Scenario 2: Database Connection Pool Exhaustion**

```bash
# Check connections
kubectl exec -it <postgres-pod> -- psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"

# Check application logs
kubectl logs -l app=<service> -n production --tail=100 | grep "connection"

# Mitigation
# 1. Increase pool size in application
# 2. Scale database
# 3. Fix connection leaks
```

**Scenario 3: High Latency**

```bash
# Check current latency
# Query Prometheus
curl -G 'http://prometheus:9090/api/v1/query' \
  --data-urlencode 'query=histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))'

# Check traces
# Access Jaeger UI and find slow requests

# Common causes:
# - Database slow queries
# - External API latency
# - Resource constraints
# - Network issues

# Mitigation
# - Add caching
# - Optimize queries
# - Scale resources
# - Add circuit breakers
```

## Performance Tuning

### Application Optimization

```yaml
# performance-checklist.yaml
optimizations:
  - category: code
    checks:
      - Use connection pooling
      - Implement caching (Redis)
      - Async processing for long tasks
      - Batch database operations
      - Use appropriate data structures
      - Profile and optimize hot paths

  - category: database
    checks:
      - Add indexes on frequently queried columns
      - Use EXPLAIN to analyze queries
      - Implement query caching
      - Use read replicas for read-heavy loads
      - Partition large tables
      - Regular VACUUM (PostgreSQL)

  - category: kubernetes
    checks:
      - Set appropriate resource requests/limits
      - Use HorizontalPodAutoscaler
      - Enable cluster autoscaling
      - Use pod disruption budgets
      - Implement affinity/anti-affinity
      - Use local storage for ephemeral data

  - category: networking
    checks:
      - Enable HTTP/2
      - Use connection keep-alive
      - Implement gRPC for service-to-service
      - Configure proper timeouts
      - Use service mesh for traffic management
```

### Load Testing

```bash
# Load test with k6
cat > load-test.js <<EOF
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up
    { duration: '5m', target: 100 },  // Stay at 100
    { duration: '2m', target: 200 },  // Ramp to 200
    { duration: '5m', target: 200 },  // Stay at 200
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% < 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function () {
  const res = http.get('http://openchoreo.local/api/customers');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
EOF

# Run load test
k6 run load-test.js

# With Prometheus output
k6 run --out prometheus load-test.js
```

## Capacity Planning

```yaml
# capacity-plan.yaml
current_state:
  nodes: 4
  total_cpu: 16 cores
  total_memory: 64 GB
  current_utilization:
    cpu: 60%
    memory: 70%

growth_projection:
  period: 6 months
  traffic_growth: 200%  # 2x current
  new_services: 5

required_capacity:
  nodes: 8  # 2x for growth + buffer
  total_cpu: 32 cores
  total_memory: 128 GB
  buffer: 30%  # For spikes and maintenance

scaling_strategy:
  - Implement HPA for all services
  - Add cluster autoscaling
  - Use spot instances for non-critical workloads
  - Implement resource quotas per team
```

## Disaster Recovery

```bash
# Backup script
#!/bin/bash
# backup-openchoreo.sh

BACKUP_DIR="/backups/openchoreo/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup etcd (if using external etcd)
ETCDCTL_API=3 etcdctl snapshot save $BACKUP_DIR/etcd-snapshot.db

# Backup persistent volumes
kubectl get pv -o yaml > $BACKUP_DIR/pv.yaml
kubectl get pvc -A -o yaml > $BACKUP_DIR/pvc.yaml

# Backup all Kubernetes resources
for ns in openchoreo production staging; do
    kubectl get all,configmap,secret,pvc -n $ns -o yaml > $BACKUP_DIR/$ns.yaml
done

# Backup Backstage catalog
kubectl exec -n openchoreo deploy/backstage -- \
    pg_dump -h postgresql -U backstage backstage | \
    gzip > $BACKUP_DIR/backstage-catalog.sql.gz

# Upload to S3
aws s3 sync $BACKUP_DIR s3://openchoreo-backups/$(date +%Y%m%d)/

echo "Backup completed: $BACKUP_DIR"
```

## Related Skills

- `openchoreo-devops` - Infrastructure management
- `openchoreo-observability` - Monitoring setup
- `openchoreo-troubleshooting` - Problem diagnosis
- `openchoreo-security` - Security operations

---

**Last Updated**: March 2025
**Skill Type**: Rigid - Follow SRE practices exactly
