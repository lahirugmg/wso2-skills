---
name: openchoreo-local-development
description: Complete guide for setting up OpenChoreo local development environment, building from source, deploying to Kubernetes (Rancher/k3d), and verifying issue fixes
---

# OpenChoreo Local Development

This skill guides you through setting up a complete local development environment for OpenChoreo, enabling you to build from source, deploy to a local Kubernetes cluster, make code changes, and verify fixes.

## When to Use This Skill

- Setting up OpenChoreo development environment for the first time
- Contributing code or fixes to OpenChoreo
- Testing OpenChoreo changes locally before submitting PRs
- Debugging OpenChoreo issues
- Developing new features for OpenChoreo

## Prerequisites Check

Before starting, verify the user has the required tools:

```bash
# Navigate to openchoreo directory and check tools
cd openchoreo && ./check-tools.sh
```

**Required tools:**
- Go v1.24.0+
- Docker 23.0+
- Make 3.81+
- Kubernetes v1.30.0+
- kubectl v1.30.0+
- Helm v3.16.0+
- k3d 5.8+ (if using k3d)

## Development Environment Options

### Option A: Rancher Desktop (Mac/Windows Users)
- Native Kubernetes integration
- GUI for cluster management
- Good resource management

### Option B: k3d (Recommended by OpenChoreo)
- Lightweight Kubernetes in Docker
- Built-in make targets
- Faster iteration

## Complete Development Workflow

### Phase 1: Initial Setup

#### Step 1: Verify Current Kubernetes Context

```bash
# Check current Kubernetes context
kubectl config current-context

# If using Rancher Desktop, context might be: rancher-desktop
# If using k3d, context might be: k3d-openchoreo-dev
```

#### Step 2: Navigate to OpenChoreo Source

```bash
# Ensure we're in the openchoreo directory
cd openchoreo
pwd  # Should show: /path/to/openchoreo
```

#### Step 3: Choose Development Approach

**For k3d (Recommended):**
```bash
# One-command setup (creates cluster, builds, deploys)
make k3d

# This takes 5-15 minutes and includes:
# - Creating k3d cluster
# - Building all components
# - Loading images
# - Installing OpenChoreo
```

**For Rancher Desktop:**
```bash
# Build all components
make go.build

# Build Docker images
make docker-build

# Install using Helm
helm install openchoreo-control-plane ./charts/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --create-namespace

# Install data plane
helm install openchoreo-data-plane ./charts/openchoreo-data-plane \
  --namespace openchoreo-data-plane \
  --create-namespace
```

### Phase 2: Verify Installation

```bash
# Check control plane pods
kubectl get pods -n openchoreo-control-plane

# Check data plane pods
kubectl get pods -n openchoreo-data-plane

# Or use make target (k3d only)
make k3d.status
```

**Expected output:** All pods should be Running

### Phase 3: Access OpenChoreo

#### Port Forwarding Setup

```bash
# Forward Backstage UI (Control Plane)
kubectl port-forward -n openchoreo-control-plane svc/openchoreo-backstage 7007:7007

# Access at: http://localhost:7007
```

Or for k3d with built-in ingress:
- **Control Plane UI**: http://openchoreo.localhost:8080
- **Data Plane**: http://localhost:19080
- **Observability**: http://localhost:11080

### Phase 4: Making Code Changes

#### Workflow for Fixing an Issue

1. **Identify the component to change:**
   - Backend controllers: `internal/controller/`
   - APIs: `internal/server/`
   - Observer: `observer/`
   - Backstage plugins: `backstage/plugins/`

2. **Make code changes**

3. **Rebuild only changed component:**

```bash
# For k3d environment:
make k3d.update.controller    # If controller changed
make k3d.update.openchoreo-api  # If API changed
make k3d.update.observer       # If observer changed

# This rebuilds, loads image, and restarts the pod
```

```bash
# For Rancher Desktop:
# Build the component
make docker-build

# Tag for local registry
docker tag openchoreo/controller:latest openchoreo/controller:dev

# Update deployment
kubectl set image deployment/openchoreo-controller-manager \
  manager=openchoreo/controller:dev \
  -n openchoreo-control-plane

# Or delete pod to force restart
kubectl delete pod -n openchoreo-control-plane \
  -l control-plane=controller-manager
```

4. **Verify pods restarted:**

```bash
kubectl get pods -n openchoreo-control-plane -w
```

### Phase 5: Testing Changes

#### For UI Changes (Backstage):
1. Access Backstage UI
2. Navigate to affected page
3. Verify fix is visible
4. Test functionality

#### For Backend/Controller Changes:
```bash
# Check controller logs
make k3d.logs.controller

# Or for Rancher:
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-controller-manager \
  -f --tail=100
```

#### For API Changes:
```bash
# Test API endpoint
curl http://localhost:8080/api/v1/projects
```

### Phase 6: Running Tests

```bash
# Run all unit tests
make test

# Run specific test
go test ./internal/controller/... -v -run TestSpecificFunction

# Check code quality
make lint
make code.gen-check
```

### Phase 7: Local Development Mode (Advanced)

For rapid iteration without rebuilding containers:

```bash
# Scale down cluster controller
kubectl scale deployment openchoreo-controller-manager \
  --replicas=0 \
  -n openchoreo-control-plane

# Run controller locally
make go.run.manager ENABLE_WEBHOOKS=false

# Now you can edit code and restart quickly with Ctrl+C and re-run
```

## Common Development Tasks

### Task: Fix a UI Bug in Backstage

```bash
# 1. Navigate to Backstage plugins
cd backstage/plugins/openchoreo

# 2. Make changes to React components

# 3. Rebuild Backstage image
cd ../../../  # Back to openchoreo root
make k3d.update.backstage  # or equivalent for Rancher

# 4. Access UI and verify fix
```

### Task: Fix a Controller Bug

```bash
# 1. Edit controller code
vim internal/controller/project/project_controller.go

# 2. Run tests
go test ./internal/controller/project/... -v

# 3. If tests pass, update controller
make k3d.update.controller

# 4. Check logs
make k3d.logs.controller

# 5. Verify fix by creating/updating a Project resource
```

### Task: Update API Endpoint

```bash
# 1. Edit API handler
vim internal/server/handlers/project_handler.go

# 2. Rebuild and update
make k3d.update.openchoreo-api

# 3. Test endpoint
curl -X GET http://localhost:8080/api/v1/projects

# 4. Check logs
make k3d.logs.openchoreo-api
```

## Troubleshooting

### Issue: Pods Not Starting

```bash
# Check pod status
kubectl get pods -n openchoreo-control-plane

# Describe pod for events
kubectl describe pod <pod-name> -n openchoreo-control-plane

# Check logs
kubectl logs <pod-name> -n openchoreo-control-plane
```

### Issue: Image Not Updating

```bash
# For k3d: Ensure image is loaded
make k3d.load.controller

# For Rancher: Check image pull policy
kubectl get deployment openchoreo-controller-manager \
  -n openchoreo-control-plane \
  -o yaml | grep imagePullPolicy

# Should be: IfNotPresent or Always
```

### Issue: Can't Access UI

```bash
# Check service
kubectl get svc -n openchoreo-control-plane

# Check ingress (k3d)
kubectl get ingress -A

# Try port-forward as fallback
kubectl port-forward -n openchoreo-control-plane \
  svc/openchoreo-backstage 7007:7007
```

## Cleanup

### Temporary Cleanup (Keep Cluster)

```bash
# Delete OpenChoreo installation
helm uninstall openchoreo-control-plane -n openchoreo-control-plane
helm uninstall openchoreo-data-plane -n openchoreo-data-plane

# Or for k3d
make k3d.uninstall
```

### Full Cleanup (Delete Cluster)

```bash
# For k3d
make k3d.down

# For Rancher Desktop - use GUI or:
kubectl delete namespace openchoreo-control-plane
kubectl delete namespace openchoreo-data-plane
```

## Quick Reference Commands

```bash
# Complete k3d setup
make k3d

# Update after code change
make k3d.update.<component>

# View logs
make k3d.logs.<component>

# Run tests
make test

# Check code quality
make lint

# Run controller locally
make go.run.manager ENABLE_WEBHOOKS=false

# Build binary
make go.build

# Check cluster status
make k3d.status
```

## Success Criteria

- ✅ OpenChoreo installed and running locally
- ✅ Can access Backstage UI
- ✅ Can make code changes and rebuild components
- ✅ Can verify fixes work as expected
- ✅ All tests pass
- ✅ Ready to submit PR

## Related Skills

- `openchoreo-rancher-setup` - Detailed Rancher Desktop setup
- `openchoreo-build-and-deploy` - Deep dive into build process
- `openchoreo-verify-issue-fixes` - Comprehensive testing strategies

## Additional Resources

- [OpenChoreo Contributor Guide](https://github.com/openchoreo/openchoreo/blob/main/docs/contributors/contribute.md)
- [OpenChoreo Architecture](https://openchoreo.dev/docs/category/concepts/)
- [Make Targets](https://github.com/openchoreo/openchoreo/blob/main/Makefile) - Run `make help`
