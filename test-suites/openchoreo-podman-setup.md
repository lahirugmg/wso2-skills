---
suite_name: "OpenChoreo Podman Setup"
description: "Complete setup and validation of OpenChoreo on local Podman environment"
skills_tested:
  - openchoreo-devops
  - openchoreo-networking
  - openchoreo-observability
  - openchoreo-security
difficulty: intermediate
estimated_duration: 45-60 minutes
prerequisites:
  - Podman Desktop installed (5.0+)
  - kubectl installed (1.28+)
  - helm installed (3.12+)
  - kind or k3d CLI installed
  - 8GB+ RAM available
  - 50GB disk space
  - macOS, Linux, or Windows with WSL2
environment:
  platform: local
  runtime: podman
  orchestrator: kind
version: 1.0.0
---

# OpenChoreo Podman Setup Test Suite

## Overview

This test suite validates the complete setup of OpenChoreo using Podman instead of Docker. It covers cluster creation, OpenChoreo installation, component validation, and basic operations.

## Architecture

```
┌─────────────────────────────────────┐
│         Podman Desktop              │
├─────────────────────────────────────┤
│                                     │
│  ┌──────────────────────────────┐  │
│  │    Kind Cluster               │  │
│  │  (Podman provider)            │  │
│  │                               │  │
│  │  ┌────────────────────────┐  │  │
│  │  │  OpenChoreo Platform   │  │  │
│  │  │  • Backstage           │  │  │
│  │  │  • Argo CD             │  │  │
│  │  │  • Cilium              │  │  │
│  │  │  • Prometheus          │  │  │
│  │  │  • Grafana             │  │  │
│  │  └────────────────────────┘  │  │
│  └──────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

## Pre-Flight Checklist

Before starting, ensure:

- [ ] Podman Desktop is running
- [ ] Podman machine has sufficient resources allocated (8GB RAM, 4 CPUs)
- [ ] All prerequisite tools are installed and in PATH
- [ ] No conflicting Docker processes running
- [ ] Ports 80, 443, 8080 are available

---

## Test 1: Verify Podman Installation

**Objective**: Confirm Podman is properly installed and configured

**Skill**: None (manual verification)

**Prerequisites**:
- Podman Desktop installed

**Prompt**:
```
Check if Podman is installed and running. Verify the version and check if the Podman machine is initialized.
```

**Validation Commands**:
```bash
# Check Podman version
podman --version
# Expected: podman version 5.0.0 or higher

# Check Podman machine status
podman machine list
# Expected: At least one machine in "Running" state

# Check Podman info
podman info
# Expected: No errors, shows system information

# Test basic podman functionality
podman run hello-world
# Expected: Success message
```

**Success Criteria**:
- [ ] Podman version 5.0.0+
- [ ] Podman machine is running
- [ ] Can pull and run containers
- [ ] No permission errors

**Troubleshooting**:
- If machine not running: `podman machine start`
- If permission errors on Linux: Add user to podman group
- If WSL issues on Windows: Restart WSL and Podman Desktop

---

## Test 2: Install and Configure Kind with Podman

**Objective**: Set up Kind to use Podman as container runtime

**Skill**: `openchoreo-devops`

**Prerequisites**:
- Podman running
- kind CLI installed

**Prompt**:
```
Use the openchoreo-devops skill to help me configure Kind to use Podman as the container runtime instead of Docker.

Create a Kind cluster configuration that:
1. Uses Podman as the provider
2. Has 1 control plane node and 3 worker nodes
3. Exposes ports 80, 443, and 8080
4. Disables the default CNI (we'll install Cilium)
5. Names the cluster "openchoreo"
```

**Expected Outcome**:
- Kind cluster configuration file created
- Environment variables set for Podman
- Cluster creation command provided

**Validation Commands**:
```bash
# Verify KIND_EXPERIMENTAL_PROVIDER is set
echo $KIND_EXPERIMENTAL_PROVIDER
# Expected: podman

# Check kind version
kind version
# Expected: kind v0.20.0 or higher

# List Podman containers (should be none yet)
podman ps
# Expected: Empty or minimal output
```

**Success Criteria**:
- [ ] KIND_EXPERIMENTAL_PROVIDER=podman is set
- [ ] Kind cluster configuration file exists
- [ ] Configuration includes all required settings
- [ ] No errors when validating configuration

**Troubleshooting**:
- If Kind doesn't support Podman: Update kind to latest version
- If provider errors: Ensure Podman socket is accessible
- On macOS: May need to set DOCKER_HOST to Podman socket

---

## Test 3: Create Kind Cluster with Podman

**Objective**: Create a functional Kubernetes cluster using Kind and Podman

**Skill**: `openchoreo-devops`

**Prerequisites**:
- Kind configured for Podman
- Cluster configuration file ready

**Prompt**:
```
Using the openchoreo-devops skill, create the Kind cluster with the configuration we prepared. Monitor the creation process and verify all nodes are ready.
```

**Expected Outcome**:
- Cluster created successfully
- All nodes are in Ready state
- kubectl context switched to new cluster

**Validation Commands**:
```bash
# Verify cluster created
kind get clusters
# Expected: openchoreo

# Check cluster info
kubectl cluster-info
# Expected: Control plane URL and CoreDNS info

# Verify all nodes are ready
kubectl get nodes
# Expected: 4 nodes (1 control-plane, 3 worker) all Ready

# Check node details
kubectl get nodes -o wide
# Expected: All nodes with internal IPs, Podman runtime

# Verify system pods
kubectl get pods -n kube-system
# Expected: All pods Running or Completed

# Check Podman containers
podman ps
# Expected: 4 containers (control-plane + 3 workers)
```

**Success Criteria**:
- [ ] Cluster named "openchoreo" exists
- [ ] All 4 nodes in Ready state
- [ ] kubectl can communicate with cluster
- [ ] System pods are running
- [ ] Podman shows 4 running containers

**Troubleshooting**:
- If nodes not ready: Check `kubectl describe node <name>`
- If pod errors: Check `kubectl logs -n kube-system <pod>`
- If Podman issues: Check `podman logs <container>`
- Network issues: Verify Podman network settings

---

## Test 4: Install Cilium CNI

**Objective**: Install Cilium as the Container Network Interface

**Skill**: `openchoreo-networking`

**Prerequisites**:
- Kind cluster running
- Cilium CLI installed (optional but recommended)

**Prompt**:
```
Use the openchoreo-networking skill to install Cilium CNI on our Kind cluster. Configure it with:
- Hubble enabled for observability
- Hubble UI enabled
- Envoy integration enabled for service mesh capabilities
```

**Expected Outcome**:
- Cilium installed successfully
- All Cilium pods running
- Hubble relay and UI deployed

**Validation Commands**:
```bash
# If Cilium CLI is installed
cilium status
# Expected: All components healthy

# Check Cilium pods
kubectl get pods -n kube-system -l k8s-app=cilium
# Expected: Cilium pods on each node (4 total)

# Check Hubble relay
kubectl get pods -n kube-system -l k8s-app=hubble-relay
# Expected: Hubble relay pod Running

# Check Hubble UI
kubectl get pods -n kube-system -l k8s-app=hubble-ui
# Expected: Hubble UI pod Running

# Test network connectivity
kubectl run test-pod --image=busybox --restart=Never -- sleep 3600
kubectl exec test-pod -- ping -c 3 kubernetes.default
# Expected: Successful ping

# Cleanup test pod
kubectl delete pod test-pod
```

**Success Criteria**:
- [ ] Cilium installed in kube-system namespace
- [ ] All Cilium pods in Running state
- [ ] Hubble relay and UI pods running
- [ ] Network connectivity working
- [ ] Nodes show Cilium as network plugin

**Troubleshooting**:
- If pods pending: Check node resources
- If CrashLoopBackOff: Check pod logs
- Network issues: Verify Cilium config

---

## Test 5: Add OpenChoreo Helm Repository

**Objective**: Configure Helm to access OpenChoreo charts

**Skill**: `openchoreo-devops`

**Prerequisites**:
- Helm installed
- Cluster running

**Prompt**:
```
Use the openchoreo-devops skill to add the OpenChoreo Helm repository and verify we can access the charts.
```

**Expected Outcome**:
- Helm repository added
- Charts are listable

**Validation Commands**:
```bash
# List Helm repos
helm repo list
# Expected: openchoreo repository present

# Search for OpenChoreo charts
helm search repo openchoreo
# Expected: List of available charts

# Show chart info
helm show chart openchoreo/openchoreo
# Expected: Chart metadata displayed

# Check chart values
helm show values openchoreo/openchoreo > /tmp/openchoreo-values.yaml
cat /tmp/openchoreo-values.yaml
# Expected: Default values displayed
```

**Success Criteria**:
- [ ] Repository added successfully
- [ ] Can search for charts
- [ ] Chart metadata accessible
- [ ] Default values retrieved

**Troubleshooting**:
- If repo add fails: Check network connectivity
- If charts not found: Run `helm repo update`

---

## Test 6: Prepare OpenChoreo Configuration

**Objective**: Create customized values file for OpenChoreo installation

**Skill**: `openchoreo-devops`

**Prerequisites**:
- OpenChoreo Helm repo added
- Understanding of cluster resources

**Prompt**:
```
Using the openchoreo-devops skill, help me create a values.yaml file for installing OpenChoreo on this local Podman-based Kind cluster. The configuration should:

1. Use minimal resource settings suitable for local development
2. Configure Backstage with guest authentication (no GitHub required)
3. Enable Argo CD with a simple admin password
4. Set up Prometheus and Grafana for monitoring
5. Configure Cilium integration
6. Use local-path storage class
7. Set domain to openchoreo.local
8. Enable all core components but keep resource requests low
```

**Expected Outcome**:
- Custom values.yaml file created
- All components configured appropriately
- Resource limits suitable for local environment

**Validation Commands**:
```bash
# Verify file exists
ls -lh openchoreo-values.yaml

# Check file content
cat openchoreo-values.yaml

# Validate YAML syntax
helm lint --values openchoreo-values.yaml openchoreo/openchoreo

# Dry-run installation
helm install openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --create-namespace \
  --values openchoreo-values.yaml \
  --dry-run --debug | head -100
```

**Success Criteria**:
- [ ] values.yaml file created
- [ ] All required configurations present
- [ ] YAML syntax is valid
- [ ] Dry-run completes without errors
- [ ] Resource requests are reasonable for local setup

**Troubleshooting**:
- If lint fails: Check YAML syntax
- If dry-run errors: Review configuration values

---

## Test 7: Install OpenChoreo Platform

**Objective**: Deploy OpenChoreo to the cluster

**Skill**: `openchoreo-devops`

**Prerequisites**:
- Cluster ready
- Cilium installed
- values.yaml prepared
- openchoreo namespace can be created

**Prompt**:
```
Use the openchoreo-devops skill to install OpenChoreo using Helm with our custom values file. Monitor the installation and help troubleshoot any issues that arise.
```

**Expected Outcome**:
- OpenChoreo installed successfully
- All pods eventually reach Running state
- Services created

**Validation Commands**:
```bash
# Create namespace if not exists
kubectl create namespace openchoreo --dry-run=client -o yaml | kubectl apply -f -

# Install OpenChoreo
helm install openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --values openchoreo-values.yaml \
  --timeout 15m \
  --wait

# Watch pod creation
kubectl get pods -n openchoreo -w
# (Ctrl+C after pods are running)

# Check all resources
kubectl get all -n openchoreo

# Verify deployments
kubectl get deployments -n openchoreo
# Expected: All deployments with READY pods

# Check services
kubectl get svc -n openchoreo
# Expected: Services for Backstage, Argo CD, Grafana, etc.

# Check persistent volume claims
kubectl get pvc -n openchoreo
# Expected: PVCs for databases and storage, all Bound
```

**Success Criteria**:
- [ ] Helm installation completes successfully
- [ ] All pods reach Running state (may take 5-10 minutes)
- [ ] All deployments show READY pods
- [ ] All PVCs are Bound
- [ ] No CrashLoopBackOff or Error states
- [ ] Services created successfully

**Troubleshooting**:
- If pods pending: Check `kubectl describe pod <pod> -n openchoreo`
- If ImagePull errors: May need to configure Podman registry settings
- If PVC issues: Verify storage class availability
- Timeout issues: Increase --timeout or remove --wait and monitor manually

---

## Test 8: Configure Local DNS

**Objective**: Set up local DNS for accessing OpenChoreo services

**Skill**: `openchoreo-devops`

**Prerequisites**:
- OpenChoreo installed
- Services running

**Prompt**:
```
Use the openchoreo-devops skill to help me configure local DNS entries for accessing OpenChoreo components at:
- openchoreo.local (Backstage)
- argocd.openchoreo.local (Argo CD)
- grafana.openchoreo.local (Grafana)
- hubble.openchoreo.local (Hubble UI)

Guide me through the process for my operating system.
```

**Expected Outcome**:
- /etc/hosts entries added
- Domains resolve to 127.0.0.1

**Validation Commands**:
```bash
# Check hosts file (macOS/Linux)
cat /etc/hosts | grep openchoreo
# Expected: Entries for all domains

# Test DNS resolution
ping -c 1 openchoreo.local
ping -c 1 argocd.openchoreo.local
ping -c 1 grafana.openchoreo.local
# Expected: All resolve to 127.0.0.1

# On Windows (PowerShell)
# cat C:\Windows\System32\drivers\etc\hosts | Select-String "openchoreo"
```

**Success Criteria**:
- [ ] All domain entries in hosts file
- [ ] Domains resolve correctly
- [ ] Can ping all domains

**Troubleshooting**:
- If permission denied: Use sudo on macOS/Linux, Administrator on Windows
- Verify correct IP address (127.0.0.1 for local)

---

## Test 9: Configure Port Forwarding

**Objective**: Set up port forwarding to access OpenChoreo services

**Skill**: `openchoreo-devops`

**Prerequisites**:
- OpenChoreo running
- All pods healthy

**Prompt**:
```
Use the openchoreo-devops skill to set up port forwarding for all OpenChoreo services. Create commands or scripts to forward:
- Backstage to localhost:7007
- Argo CD to localhost:8080
- Grafana to localhost:3000
- Hubble UI to localhost:8081
```

**Expected Outcome**:
- Port forwarding commands provided
- Services accessible on localhost

**Validation Commands**:
```bash
# Port forward Backstage (in separate terminal/background)
kubectl port-forward -n openchoreo svc/backstage 7007:7007 &

# Port forward Argo CD
kubectl port-forward -n openchoreo svc/argocd-server 8080:443 &

# Port forward Grafana
kubectl port-forward -n openchoreo svc/grafana 3000:80 &

# Port forward Hubble UI
kubectl port-forward -n kube-system svc/hubble-ui 8081:80 &

# Test access with curl
curl -k http://localhost:7007/health || echo "Backstage check"
curl -k https://localhost:8080 || echo "Argo CD check"
curl -k http://localhost:3000 || echo "Grafana check"
curl -k http://localhost:8081 || echo "Hubble check"

# Or open in browser
# open http://localhost:7007
# open https://localhost:8080
# open http://localhost:3000
# open http://localhost:8081
```

**Success Criteria**:
- [ ] All port forwards established
- [ ] Services respond on localhost ports
- [ ] Can access UIs in browser
- [ ] No connection errors

**Troubleshooting**:
- If port already in use: Change local port
- If connection refused: Verify pod is running
- If timeout: Check pod logs

---

## Test 10: Verify Backstage Access

**Objective**: Confirm Backstage (Developer Portal) is accessible and functional

**Skill**: `openchoreo-devops`

**Prerequisites**:
- Backstage pod running
- Port forwarding configured

**Prompt**:
```
Guide me through accessing and verifying the Backstage developer portal. What should I see and how can I verify it's working correctly?
```

**Expected Outcome**:
- Can access Backstage UI
- Can log in (guest mode)
- See default catalog

**Validation Commands**:
```bash
# Check Backstage pod
kubectl get pods -n openchoreo -l app=backstage
# Expected: Running

# Check Backstage logs
kubectl logs -n openchoreo -l app=backstage --tail=50
# Expected: No errors, server started messages

# Access Backstage
open http://localhost:7007
# Or: curl http://localhost:7007
```

**Manual Verification**:
1. Open http://localhost:7007 in browser
2. Should see Backstage homepage
3. Click "Guest" login (if configured)
4. Should see catalog/components

**Success Criteria**:
- [ ] Backstage UI loads
- [ ] Can authenticate (guest mode)
- [ ] See navigation menu
- [ ] Can browse catalog
- [ ] No JavaScript errors in console

---

## Test 11: Verify Argo CD Access

**Objective**: Confirm Argo CD is accessible and can be logged into

**Skill**: `openchoreo-gitops`

**Prerequisites**:
- Argo CD pods running
- Port forwarding configured

**Prompt**:
```
Help me access Argo CD and retrieve the initial admin password. Guide me through the first login.
```

**Expected Outcome**:
- Can access Argo CD UI
- Can retrieve admin password
- Can log in successfully

**Validation Commands**:
```bash
# Check Argo CD pods
kubectl get pods -n openchoreo -l app.kubernetes.io/name=argocd-server
# Expected: Running

# Get initial admin password
kubectl -n openchoreo get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
# Expected: Password string

# Access Argo CD
open https://localhost:8080
# Or: curl -k https://localhost:8080
```

**Manual Verification**:
1. Open https://localhost:8080
2. Accept self-signed certificate warning
3. Login with username: admin
4. Password from command above
5. Should see Argo CD dashboard

**Success Criteria**:
- [ ] Argo CD UI loads
- [ ] Can retrieve admin password
- [ ] Can log in successfully
- [ ] See empty applications list (initially)
- [ ] No errors in UI

---

## Test 12: Verify Grafana Access

**Objective**: Confirm Grafana is accessible with monitoring data

**Skill**: `openchoreo-observability`

**Prerequisites**:
- Grafana and Prometheus running
- Port forwarding configured

**Prompt**:
```
Help me access Grafana and verify it's connected to Prometheus. Show me how to check that metrics are being collected.
```

**Expected Outcome**:
- Can access Grafana UI
- Prometheus datasource configured
- Metrics visible

**Validation Commands**:
```bash
# Check Grafana pod
kubectl get pods -n openchoreo -l app.kubernetes.io/name=grafana
# Expected: Running

# Check Prometheus pod
kubectl get pods -n openchoreo -l app=prometheus
# Expected: Running

# Get Grafana admin password (if not set in values)
kubectl get secret -n openchoreo grafana \
  -o jsonpath="{.data.admin-password}" | base64 -d && echo

# Access Grafana
open http://localhost:3000
```

**Manual Verification**:
1. Open http://localhost:3000
2. Login (admin / password from values or secret)
3. Go to Configuration → Data Sources
4. Verify Prometheus datasource exists and is working
5. Go to Explore
6. Select Prometheus
7. Run query: `up`
8. Should see metrics

**Success Criteria**:
- [ ] Grafana UI loads
- [ ] Can log in
- [ ] Prometheus datasource configured
- [ ] Can query metrics
- [ ] See OpenChoreo components in metrics

---

## Test 13: Deploy Test Application

**Objective**: Deploy a simple test application to verify the platform works end-to-end

**Skill**: `openchoreo-app-developer`

**Prerequisites**:
- OpenChoreo fully operational
- kubectl access

**Prompt**:
```
Using the openchoreo-app-developer skill, help me deploy a simple "hello world" Go service to test that applications can be deployed and accessed on this OpenChoreo platform.

The service should:
1. Return a simple JSON response
2. Have health endpoints
3. Be accessible via port-forward
4. Be registered in Backstage (optional)
```

**Expected Outcome**:
- Test app deployed
- Pod running
- Service created
- Can access the app

**Validation Commands**:
```bash
# Apply test app manifests (created by skill)
kubectl apply -f test-app.yaml -n production

# Watch deployment
kubectl rollout status deployment/hello-world -n production

# Check pod
kubectl get pods -n production -l app=hello-world
# Expected: Running

# Port forward
kubectl port-forward -n production svc/hello-world 9090:80 &

# Test app
curl http://localhost:9090
# Expected: JSON response

# Check logs
kubectl logs -n production -l app=hello-world
```

**Success Criteria**:
- [ ] Deployment successful
- [ ] Pod reaches Running state
- [ ] Service created
- [ ] Can access via port-forward
- [ ] App responds with expected output
- [ ] No errors in logs

---

## Test 14: Verify Platform Observability

**Objective**: Confirm all observability components are collecting data

**Skill**: `openchoreo-observability`

**Prerequisites**:
- Test app deployed
- Prometheus and Grafana running

**Prompt**:
```
Use the openchoreo-observability skill to verify that our test application metrics are being collected by Prometheus and visible in Grafana.
```

**Expected Outcome**:
- Prometheus scraping test app
- Metrics visible in Grafana

**Validation Commands**:
```bash
# Port forward Prometheus
kubectl port-forward -n openchoreo svc/prometheus-server 9091:80 &

# Check Prometheus targets
curl http://localhost:9091/api/v1/targets | jq
# Or open http://localhost:9091/targets

# Query metrics for test app
curl 'http://localhost:9091/api/v1/query?query=up{job="hello-world"}' | jq

# In Grafana (browser):
# 1. Go to Explore
# 2. Select Prometheus
# 3. Query: up{job="hello-world"}
# Should see metric data
```

**Success Criteria**:
- [ ] Prometheus scraping test app
- [ ] Metrics queryable in Prometheus
- [ ] Metrics visible in Grafana
- [ ] No scrape errors

---

## Test 15: Platform Cleanup

**Objective**: Properly tear down the platform and clean up resources

**Skill**: `openchoreo-devops`

**Prerequisites**:
- Completed testing
- Ready to cleanup

**Prompt**:
```
Use the openchoreo-devops skill to help me cleanly shut down and remove the OpenChoreo platform and Kind cluster, ensuring all Podman resources are cleaned up.
```

**Expected Outcome**:
- OpenChoreo uninstalled
- Cluster deleted
- Podman containers removed
- Clean state restored

**Validation Commands**:
```bash
# Uninstall OpenChoreo
helm uninstall openchoreo -n openchoreo

# Delete namespace
kubectl delete namespace openchoreo

# Delete Kind cluster
kind delete cluster --name openchoreo

# Verify cluster deleted
kind get clusters
# Expected: openchoreo not in list

# Check Podman containers
podman ps
# Expected: Kind containers gone

# Clean up any remaining containers
podman ps -a | grep openchoreo

# Remove stopped containers
podman container prune -f

# Check disk usage
podman system df
```

**Success Criteria**:
- [ ] Helm release deleted
- [ ] Namespace removed
- [ ] Kind cluster deleted
- [ ] No Podman containers related to cluster
- [ ] Disk space reclaimed

---

## Test Suite Completion Checklist

### Prerequisites
- [ ] Podman Desktop installed and running
- [ ] kubectl, helm, kind installed
- [ ] 8GB RAM allocated
- [ ] All ports available

### Setup Phase
- [ ] Test 1: Podman verification ✓
- [ ] Test 2: Kind configuration ✓
- [ ] Test 3: Cluster creation ✓
- [ ] Test 4: Cilium installation ✓
- [ ] Test 5: Helm repository ✓
- [ ] Test 6: OpenChoreo configuration ✓
- [ ] Test 7: OpenChoreo installation ✓

### Access Configuration
- [ ] Test 8: DNS configuration ✓
- [ ] Test 9: Port forwarding ✓
- [ ] Test 10: Backstage verification ✓
- [ ] Test 11: Argo CD verification ✓
- [ ] Test 12: Grafana verification ✓

### Validation Phase
- [ ] Test 13: Test app deployment ✓
- [ ] Test 14: Observability verification ✓

### Cleanup
- [ ] Test 15: Platform cleanup ✓

### Overall Results
- [ ] All tests passed
- [ ] No critical issues encountered
- [ ] Platform fully functional
- [ ] Cleanup successful

---

## Troubleshooting Guide

### Common Issues

**Issue: Podman containers won't start**
- Solution: Restart Podman machine, check resources

**Issue: Kind cluster creation fails**
- Solution: Check `KIND_EXPERIMENTAL_PROVIDER` is set, verify Podman is running

**Issue: Pods stuck in Pending**
- Solution: Check node resources, verify storage class

**Issue: ImagePull errors**
- Solution: Configure Podman registry settings, check network

**Issue: Services not accessible**
- Solution: Verify port-forwards, check service selectors

**Issue: Metrics not appearing**
- Solution: Check Prometheus scrape configs, verify pod annotations

---

## Test Run Report Template

```markdown
## OpenChoreo Podman Setup - Test Run Report

**Date**: _____________
**Tester**: _____________
**Environment**:
  - OS: _____________
  - Podman Version: _____________
  - Kind Version: _____________

**Test Results**:
  - Tests Passed: _____ / 15
  - Tests Failed: _____
  - Tests Skipped: _____

**Duration**: _____ minutes

**Issues Encountered**:
1.
2.

**Notes**:


**Overall Status**: [ ] PASS  [ ] FAIL  [ ] PARTIAL
```

---

## Next Steps

After completing this test suite:

1. **Explore OpenChoreo**: Navigate through Backstage, create components
2. **Deploy Applications**: Use `openchoreo-app-deployment.md` test suite
3. **Setup GitOps**: Follow `openchoreo-gitops-workflow.md`
4. **Advanced Monitoring**: Complete `openchoreo-monitoring-setup.md`

## Related Test Suites

- `openchoreo-k3d-setup.md` - Alternative with K3d
- `openchoreo-app-deployment.md` - Deploy sample applications
- `openchoreo-gitops-workflow.md` - GitOps workflows
- `openchoreo-disaster-recovery.md` - Backup and recovery procedures
