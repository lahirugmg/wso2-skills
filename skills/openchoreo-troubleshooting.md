---
name: openchoreo-troubleshooting
description: Use for troubleshooting OpenChoreo issues - debugging techniques, common problems, diagnostic commands, and resolution strategies
---

# OpenChoreo Troubleshooting Skill

## When to Use This Skill

Use for:
- Diagnosing pod failures
- Debugging networking issues
- Investigating performance problems
- Resolving deployment failures
- Analyzing logs and metrics

## Diagnostic Commands

### Pod Issues

```bash
# Check pod status
kubectl get pods -n openchoreo

# Describe pod
kubectl describe pod <pod-name> -n openchoreo

# Check logs
kubectl logs <pod-name> -n openchoreo --previous
kubectl logs <pod-name> -n openchoreo --tail=100 -f

# Get events
kubectl get events -n openchoreo --sort-by='.lastTimestamp'

# Exec into pod
kubectl exec -it <pod-name> -n openchoreo -- /bin/sh
```

### Network Debugging

```bash
# Check Cilium connectivity
cilium connectivity test

# Check services
kubectl get svc -n openchoreo

# Check endpoints
kubectl get endpoints -n openchoreo

# Test connectivity
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- /bin/bash
# Inside pod:
curl http://service-name.namespace.svc.cluster.local
nslookup service-name.namespace.svc.cluster.local
traceroute service-name.namespace.svc.cluster.local
```

## Common Issues

### 1. CrashLoopBackOff

**Diagnosis:**
```bash
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

**Common Causes:**
- Application crashes
- Missing dependencies
- Configuration errors
- Resource constraints

**Resolution:**
- Fix application bugs
- Check configuration
- Increase resources
- Review startup commands

### 2. ImagePullBackOff

**Diagnosis:**
```bash
kubectl describe pod <pod> | grep -A 5 Events
```

**Resolution:**
- Verify image name and tag
- Check registry credentials
- Ensure network connectivity to registry

### 3. Pending Pods

**Diagnosis:**
```bash
kubectl describe pod <pod> | grep -A 10 Events
```

**Common Causes:**
- Insufficient resources
- Node selectors not matching
- PVC not binding

### 4. OOMKilled

**Diagnosis:**
```bash
kubectl describe pod <pod> | grep -i oom
```

**Resolution:**
- Increase memory limits
- Fix memory leaks
- Optimize application

## Performance Debugging

```bash
# Check resource usage
kubectl top nodes
kubectl top pods -n openchoreo

# Profile application
kubectl exec -it <pod> -n openchoreo -- pprof -http=:8080 /app

# Analyze slow queries (for databases)
kubectl exec -it postgres-0 -n openchoreo -- \
  psql -U postgres -c "SELECT * FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 10;"
```

## Logs Analysis

```bash
# Search logs with stern
stern -n openchoreo customer-service --since 1h | grep ERROR

# Aggregate logs
kubectl logs -l app=customer-service -n openchoreo --all-containers=true | \
  grep -E "ERROR|WARN" | wc -l
```

## Related Skills

- `openchoreo-sre`
- `openchoreo-observability`
- `openchoreo-networking`
- `openchoreo-devops`

---

**Last Updated**: March 2025
