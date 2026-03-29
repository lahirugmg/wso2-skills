---
name: openchoreo-rancher-setup
description: Detailed guide for setting up OpenChoreo development environment specifically for Rancher Desktop users on Mac/Windows
---

# OpenChoreo Rancher Desktop Setup

This skill provides detailed instructions for setting up OpenChoreo development environment using Rancher Desktop, which is ideal for Mac and Windows users who want a native Kubernetes experience with GUI management.

## When to Use This Skill

- First-time setup of OpenChoreo on Rancher Desktop
- Troubleshooting Rancher Desktop-specific issues
- Configuring Rancher Desktop for OpenChoreo development
- Understanding Rancher Desktop and OpenChoreo integration

## Prerequisites

### System Requirements
- **Mac**: macOS 11 (Big Sur) or later
- **Windows**: Windows 10/11 with WSL2
- **RAM**: Minimum 8GB, Recommended 16GB+
- **Disk Space**: 20GB+ free space
- **CPU**: 4+ cores recommended

### Required Software
- Rancher Desktop (latest version)
- Docker CLI (included with Rancher)
- kubectl (included with Rancher)
- Helm 3.16+
- Git
- Go 1.24+ (for building from source)
- Make 3.81+

## Step-by-Step Setup

### Phase 1: Install Rancher Desktop

#### For Mac:
```bash
# Using Homebrew
brew install --cask rancher

# Or download from: https://rancherdesktop.io/
```

#### For Windows:
1. Download installer from https://rancherdesktop.io/
2. Run the installer
3. Follow installation wizard

### Phase 2: Configure Rancher Desktop

#### 1. Launch Rancher Desktop

**Important Settings:**

**Kubernetes Settings:**
- Kubernetes Version: 1.30.0 or higher
- Container Runtime: **dockerd (moby)** (recommended for development)
- Memory: 8GB minimum, 12GB+ recommended
- CPUs: 4 minimum, 6+ recommended

**WSL Integration (Windows only):**
- Enable integration with default WSL distro
- Enable integration with additional distros if needed

#### 2. Verify Rancher Desktop is Running

```bash
# Check Kubernetes context
kubectl config current-context
# Should show: rancher-desktop

# Check Kubernetes version
kubectl version --short

# Check nodes
kubectl get nodes
# Should show: rancher-desktop (Ready)
```

#### 3. Configure kubectl Context

```bash
# View all contexts
kubectl config get-contexts

# Ensure rancher-desktop is current
kubectl config use-context rancher-desktop

# Verify cluster access
kubectl cluster-info
```

### Phase 3: Install Additional Tools

#### Install Helm (if not already installed)

```bash
# Mac
brew install helm

# Windows (PowerShell)
choco install kubernetes-helm

# Verify
helm version
```

#### Install Go (for building from source)

```bash
# Mac
brew install go@1.24

# Windows
choco install golang --version=1.24

# Verify
go version
```

#### Install Make

```bash
# Mac (usually pre-installed)
make --version

# If not installed:
brew install make

# Windows (via chocolatey)
choco install make
```

### Phase 4: Clone OpenChoreo Repository

```bash
# Navigate to your workspace
cd ~/workspace  # or your preferred location

# Clone the repository
git clone https://github.com/openchoreo/openchoreo.git
cd openchoreo

# Verify you're on the main branch
git branch
```

### Phase 5: Build OpenChoreo Components

#### Verify Prerequisites

```bash
# Check all required tools
./check-tools.sh
```

#### Build All Components

```bash
# Build Go binaries
make go.build

# This creates binaries in bin/dist/
ls -la bin/dist/
```

#### Build Docker Images

```bash
# Build all Docker images
make docker-build

# This builds:
# - openchoreo/controller
# - openchoreo/openchoreo-api
# - openchoreo/observer
# - And other components

# Verify images
docker images | grep openchoreo
```

### Phase 6: Deploy OpenChoreo to Rancher Desktop

#### Option A: Using Helm Charts (Recommended)

```bash
# Create namespaces
kubectl create namespace openchoreo-control-plane
kubectl create namespace openchoreo-data-plane

# Install Control Plane
helm install openchoreo-control-plane ./charts/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --set controller.image.repository=openchoreo/controller \
  --set controller.image.tag=latest \
  --set controller.image.pullPolicy=IfNotPresent

# Install Data Plane
helm install openchoreo-data-plane ./charts/openchoreo-data-plane \
  --namespace openchoreo-data-plane \
  --set image.pullPolicy=IfNotPresent

# Install Workflow Plane (optional)
helm install openchoreo-workflow-plane ./charts/openchoreo-workflow-plane \
  --namespace openchoreo-workflow-plane \
  --create-namespace

# Install Observability Plane (optional)
helm install openchoreo-observability-plane ./charts/openchoreo-observability-plane \
  --namespace openchoreo-observability-plane \
  --create-namespace
```

#### Option B: Using Make Targets (Alternative)

```bash
# Note: Make targets are optimized for k3d
# For Rancher, manual helm installation is recommended
```

### Phase 7: Verify Installation

```bash
# Check Control Plane pods
kubectl get pods -n openchoreo-control-plane

# Expected pods:
# - openchoreo-controller-manager
# - openchoreo-api
# - openchoreo-backstage
# - postgresql (if database is included)

# Check Data Plane pods
kubectl get pods -n openchoreo-data-plane

# All pods should be in Running state
```

#### Wait for Pods to be Ready

```bash
# Watch pods until all are Running
kubectl get pods -n openchoreo-control-plane -w

# Or check status periodically
watch kubectl get pods -n openchoreo-control-plane
```

### Phase 8: Access OpenChoreo UI

#### Set Up Port Forwarding

```bash
# Forward Backstage UI (Port 7007)
kubectl port-forward -n openchoreo-control-plane \
  svc/openchoreo-backstage 7007:7007

# Keep this terminal open
```

#### Access in Browser

Open your browser and navigate to:
- **Backstage UI**: http://localhost:7007

#### Alternative: Configure Ingress (Advanced)

```bash
# Install nginx-ingress controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace

# Add to /etc/hosts (Mac) or C:\Windows\System32\drivers\etc\hosts (Windows)
# 127.0.0.1 openchoreo.local

# Access at: http://openchoreo.local
```

### Phase 9: Configure DataPlane Resource

```bash
# Create DataPlane resource
kubectl apply -f - <<EOF
apiVersion: core.choreo.dev/v1alpha1
kind: DataPlane
metadata:
  name: dev-dataplane
  namespace: openchoreo-control-plane
spec:
  clusterRef:
    name: rancher-desktop
  regions:
    - name: local
EOF

# Verify DataPlane
kubectl get dataplanes -n openchoreo-control-plane
```

## Rancher Desktop-Specific Considerations

### Image Management

Rancher Desktop uses dockerd runtime, so images built locally are immediately available:

```bash
# Build image
make docker-build

# List images
docker images | grep openchoreo

# No need to load images into cluster (unlike k3d)
```

### Resource Limits

Adjust Rancher Desktop resources if experiencing performance issues:

1. Open Rancher Desktop
2. Go to Preferences > Kubernetes
3. Increase Memory and CPU allocation
4. Apply and restart

### Networking

Rancher Desktop uses standard Kubernetes networking:

```bash
# Service access via port-forward
kubectl port-forward -n <namespace> svc/<service-name> <local-port>:<service-port>

# Or via kubectl proxy
kubectl proxy
```

## Updating Components After Code Changes

### Workflow for Iterative Development

```bash
# 1. Make code changes
vim internal/controller/project/project_controller.go

# 2. Rebuild the component
make docker-build

# 3. Tag with new version (optional)
docker tag openchoreo/controller:latest openchoreo/controller:dev-$(date +%s)

# 4. Update deployment
kubectl set image deployment/openchoreo-controller-manager \
  manager=openchoreo/controller:latest \
  -n openchoreo-control-plane

# 5. Force pod restart
kubectl delete pod -n openchoreo-control-plane \
  -l control-plane=controller-manager

# 6. Watch pod restart
kubectl get pods -n openchoreo-control-plane -w

# 7. Check logs
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-controller-manager \
  -f --tail=100
```

### Quick Update Script

Create a helper script `update-component.sh`:

```bash
#!/bin/bash
COMPONENT=$1
NAMESPACE=${2:-openchoreo-control-plane}

echo "Building $COMPONENT..."
make docker-build

echo "Restarting $COMPONENT in $NAMESPACE..."
kubectl rollout restart deployment/$COMPONENT -n $NAMESPACE

echo "Waiting for rollout..."
kubectl rollout status deployment/$COMPONENT -n $NAMESPACE

echo "Tailing logs..."
kubectl logs -n $NAMESPACE deployment/$COMPONENT -f --tail=50
```

Usage:
```bash
chmod +x update-component.sh
./update-component.sh openchoreo-controller-manager
```

## Troubleshooting Rancher Desktop Issues

### Issue: Cluster Not Starting

**Solution:**
1. Quit Rancher Desktop completely
2. Clear Kubernetes data (Preferences > Kubernetes > Reset Kubernetes)
3. Restart Rancher Desktop
4. Wait for cluster to be ready

### Issue: Image Pull Errors

**Solution:**
```bash
# Ensure imagePullPolicy is set correctly
kubectl get deployment openchoreo-controller-manager \
  -n openchoreo-control-plane \
  -o yaml | grep imagePullPolicy

# Should be: IfNotPresent or Never for local images

# Update if needed
kubectl patch deployment openchoreo-controller-manager \
  -n openchoreo-control-plane \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"manager","imagePullPolicy":"IfNotPresent"}]}}}}'
```

### Issue: Pods Stuck in Pending

**Solution:**
```bash
# Check pod events
kubectl describe pod <pod-name> -n openchoreo-control-plane

# Common causes:
# 1. Insufficient resources - increase Rancher Desktop limits
# 2. Image not found - rebuild with make docker-build
# 3. PVC issues - check storage

# Check node resources
kubectl top nodes
kubectl describe node rancher-desktop
```

### Issue: Can't Access Services

**Solution:**
```bash
# Verify service exists
kubectl get svc -n openchoreo-control-plane

# Check endpoints
kubectl get endpoints -n openchoreo-control-plane

# Test service from within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Then: wget -O- http://openchoreo-backstage.openchoreo-control-plane:7007

# Use port-forward as fallback
kubectl port-forward -n openchoreo-control-plane \
  svc/openchoreo-backstage 7007:7007
```

### Issue: Performance Problems

**Solution:**
1. Increase Rancher Desktop resources (Memory/CPU)
2. Disable unused Rancher features
3. Clean up unused images:
   ```bash
   docker image prune -a
   ```
4. Restart Rancher Desktop

## Cleanup and Reset

### Uninstall OpenChoreo (Keep Cluster)

```bash
# Uninstall Helm releases
helm uninstall openchoreo-control-plane -n openchoreo-control-plane
helm uninstall openchoreo-data-plane -n openchoreo-data-plane
helm uninstall openchoreo-workflow-plane -n openchoreo-workflow-plane
helm uninstall openchoreo-observability-plane -n openchoreo-observability-plane

# Delete namespaces
kubectl delete namespace openchoreo-control-plane
kubectl delete namespace openchoreo-data-plane
kubectl delete namespace openchoreo-workflow-plane
kubectl delete namespace openchoreo-observability-plane
```

### Reset Rancher Desktop

1. Open Rancher Desktop
2. Go to Preferences > Kubernetes
3. Click "Reset Kubernetes"
4. Confirm and wait for reset

### Clean Docker Images

```bash
# Remove OpenChoreo images
docker images | grep openchoreo | awk '{print $3}' | xargs docker rmi -f

# Prune unused images
docker image prune -a
```

## Quick Reference

### Common Commands

```bash
# Check cluster status
kubectl get nodes

# View all pods
kubectl get pods -A

# Access Backstage
kubectl port-forward -n openchoreo-control-plane svc/openchoreo-backstage 7007:7007

# Rebuild and update controller
make docker-build && kubectl rollout restart deployment/openchoreo-controller-manager -n openchoreo-control-plane

# View logs
kubectl logs -n openchoreo-control-plane deployment/openchoreo-controller-manager -f

# List images
docker images | grep openchoreo

# Check resource usage
kubectl top nodes
kubectl top pods -A
```

### Useful Aliases

Add to your `~/.zshrc` or `~/.bashrc`:

```bash
# Kubernetes shortcuts
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods -A'
alias kl='kubectl logs -f'
alias kd='kubectl describe'

# OpenChoreo specific
alias occ-logs='kubectl logs -n openchoreo-control-plane deployment/openchoreo-controller-manager -f'
alias occ-status='kubectl get pods -n openchoreo-control-plane'
alias occ-ui='kubectl port-forward -n openchoreo-control-plane svc/openchoreo-backstage 7007:7007'
```

## Success Criteria

- ✅ Rancher Desktop installed and running
- ✅ Kubernetes cluster accessible (rancher-desktop context)
- ✅ OpenChoreo components built successfully
- ✅ All pods running in openchoreo-control-plane namespace
- ✅ Backstage UI accessible at http://localhost:7007
- ✅ Can rebuild and update components

## Related Skills

- `openchoreo-local-development` - Main development workflow
- `openchoreo-build-and-deploy` - Build process details
- `openchoreo-verify-issue-fixes` - Testing and verification

## Additional Resources

- [Rancher Desktop Documentation](https://docs.rancherdesktop.io/)
- [OpenChoreo Documentation](https://openchoreo.dev/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
