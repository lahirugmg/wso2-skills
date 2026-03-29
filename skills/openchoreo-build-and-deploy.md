---
name: openchoreo-build-and-deploy
description: Deep dive into OpenChoreo build system, component architecture, and deployment strategies for local development
---

# OpenChoreo Build and Deploy

Comprehensive guide to building OpenChoreo components from source and deploying them to local Kubernetes for development and testing.

## When to Use This Skill

- Understanding OpenChoreo's build system and Make targets
- Building specific components instead of full rebuild
- Optimizing build times during development
- Troubleshooting build issues
- Creating custom deployment configurations

## OpenChoreo Component Architecture

### Components Overview

```
openchoreo/
├── cmd/
│   ├── manager/          # Controller manager (Kubernetes operator)
│   ├── openchoreo-api/   # REST API server
│   └── occ/              # CLI tool
├── observer/             # Observability agent (separate Go module)
├── backstage/            # Backstage UI and plugins
└── charts/               # Helm charts for deployment
```

### Component Responsibilities

1. **Controller Manager** (`cmd/manager`)
   - Reconciles OpenChoreo CRDs
   - Manages Project, Component, Environment resources
   - Deploys workloads to data planes

2. **OpenChoreo API** (`cmd/openchoreo-api`)
   - REST API for UI and CLI
   - Authentication and authorization
   - Aggregates data from Kubernetes

3. **Observer** (`observer/`)
   - Collects logs and metrics
   - Processes observability data
   - Provides query APIs

4. **Backstage** (`backstage/`)
   - Developer portal UI
   - OpenChoreo plugins
   - Service catalog

5. **OCC CLI** (`cmd/occ`)
   - Command-line interface
   - Project/component management
   - Local development tools

## Build System

### Makefile Targets

```bash
# View all available targets
make help

# Key categories:
# - go.*       : Go build and run targets
# - docker.*   : Docker image build targets
# - k3d.*      : k3d-specific development targets
# - helm.*     : Helm chart operations
# - test.*     : Testing targets
# - lint.*     : Code quality checks
```

### Building Go Binaries

```bash
# Build all binaries
make go.build

# Output: bin/dist/<os>/<arch>/manager, openchoreo-api, occ

# Build specific binary
make go.build.manager
make go.build.openchoreo-api
make go.build.occ

# Cross-compile for different platforms
GOOS=linux GOARCH=amd64 make go.build
```

### Building Docker Images

```bash
# Build all images
make docker-build

# Images created:
# - openchoreo/controller:latest
# - openchoreo/openchoreo-api:latest
# - openchoreo/observer:latest

# Build specific component
make docker-build IMG=openchoreo/controller:latest

# Custom tag
make docker-build IMG=openchoreo/controller:dev-$(date +%s)
```

### Building for k3d

```bash
# Complete k3d build
make k3d.build

# This runs:
# 1. make go.build
# 2. make docker-build
# 3. Optimizes for k3d

# Build specific component for k3d
make k3d.build.controller
make k3d.build.openchoreo-api
make k3d.build.observer
```

## Deployment Strategies

### Strategy 1: Full Deployment (k3d)

**Best for:** Initial setup, testing complete system

```bash
# One command - creates cluster and deploys everything
make k3d

# Or step by step:
make k3d.up          # Create cluster
make k3d.build       # Build all components
make k3d.load        # Load images to cluster
make k3d.install     # Install via Helm
make k3d.configure   # Configure DataPlane
```

### Strategy 2: Incremental Updates (k3d)

**Best for:** Iterative development, quick testing

```bash
# Update single component (rebuild + load + restart)
make k3d.update.controller
make k3d.update.openchoreo-api
make k3d.update.observer

# This is equivalent to:
make k3d.build.<component>
make k3d.load.<component>
kubectl rollout restart deployment/<component>
```

### Strategy 3: Manual Deployment (Rancher/Any K8s)

**Best for:** Custom configurations, non-k3d environments

```bash
# 1. Build images
make docker-build

# 2. Create namespaces
kubectl create namespace openchoreo-control-plane
kubectl create namespace openchoreo-data-plane

# 3. Install via Helm with custom values
helm install openchoreo-control-plane ./charts/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --set controller.image.tag=latest \
  --set controller.image.pullPolicy=IfNotPresent \
  --set api.image.tag=latest \
  --values custom-values.yaml

# 4. Install data plane
helm install openchoreo-data-plane ./charts/openchoreo-data-plane \
  --namespace openchoreo-data-plane
```

### Strategy 4: Local Controller Development

**Best for:** Rapid controller development, debugging

```bash
# 1. Deploy everything except controller
make k3d.install

# 2. Scale down controller in cluster
kubectl scale deployment openchoreo-controller-manager \
  --replicas=0 \
  -n openchoreo-control-plane

# 3. Run controller locally
make go.run.manager ENABLE_WEBHOOKS=false

# 4. Make changes, Ctrl+C, and re-run
# No need to rebuild Docker images!
```

## Component-Specific Build Details

### Controller Manager

**Source:** `cmd/manager/main.go`, `internal/controller/`

```bash
# Build binary only
make go.build.manager

# Run locally (requires kubeconfig)
make go.run.manager ENABLE_WEBHOOKS=false

# Build Docker image
make docker-build IMG=openchoreo/controller:latest

# Update in k3d
make k3d.update.controller
```

**Key Files:**
- `cmd/manager/main.go` - Entry point
- `internal/controller/*` - Controller reconcilers
- `api/v1alpha1/*` - CRD definitions

### OpenChoreo API

**Source:** `cmd/openchoreo-api/main.go`, `internal/server/`

```bash
# Build binary
make go.build.openchoreo-api

# Run locally (requires kubeconfig and database)
./bin/dist/$(go env GOOS)/$(go env GOARCH)/openchoreo-api

# Build Docker image
make docker-build IMG=openchoreo/openchoreo-api:latest

# Update in k3d
make k3d.update.openchoreo-api
```

**Environment Variables:**
```bash
# Required for local run
export KUBECONFIG=~/.kube/config
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=openchoreo
export DB_USER=openchoreo
export DB_PASSWORD=password
```

### Observer

**Source:** `observer/` (separate Go module)

```bash
# Build observer
cd observer
make build

# Run locally
./bin/observer

# Build Docker image
make docker-build

# Update in k3d (from root)
cd ..
make k3d.update.observer
```

### Backstage

**Source:** `backstage/` (Node.js/TypeScript)

```bash
# Install dependencies
cd backstage
yarn install

# Run locally for development
yarn dev

# Build production
yarn build

# Build Docker image (from root)
cd ..
make docker-build-backstage

# Update in k3d
make k3d.update.backstage
```

### OCC CLI

**Source:** `cmd/occ/main.go`

```bash
# Build CLI
make go.build.occ

# Install to $GOPATH/bin
go install ./cmd/occ

# Run directly
make go.run.occ GO_RUN_ARGS="version"
make go.run.occ GO_RUN_ARGS="project list"

# Build for multiple platforms
GOOS=linux GOARCH=amd64 make go.build.occ
GOOS=darwin GOARCH=arm64 make go.build.occ
GOOS=windows GOARCH=amd64 make go.build.occ
```

## Advanced Build Configurations

### Custom Image Tags

```bash
# Use custom tag
VERSION=v0.1.0-dev make docker-build

# With custom repository
IMG_REPO=myregistry.io/myorg make docker-build
```

### Build with Debug Symbols

```bash
# Disable optimizations for debugging
CGO_ENABLED=0 go build -gcflags="all=-N -l" \
  -o bin/manager \
  ./cmd/manager
```

### Multi-Architecture Builds

```bash
# Build for ARM (Apple Silicon)
GOARCH=arm64 make go.build

# Build for AMD64
GOARCH=amd64 make go.build

# Docker multi-arch build
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t openchoreo/controller:latest \
  -f Dockerfile .
```

## Helm Chart Customization

### Using Custom Values

```yaml
# custom-values.yaml
controller:
  image:
    repository: openchoreo/controller
    tag: dev-latest
    pullPolicy: IfNotPresent

  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 512Mi

  replicas: 1

  env:
    - name: LOG_LEVEL
      value: debug

api:
  enabled: true
  replicas: 1
```

```bash
# Deploy with custom values
helm install openchoreo-control-plane ./charts/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --values custom-values.yaml
```

### Upgrading Deployments

```bash
# Upgrade after rebuilding
helm upgrade openchoreo-control-plane ./charts/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --reuse-values

# Force recreation of pods
helm upgrade openchoreo-control-plane ./charts/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --recreate-pods

# For k3d
make k3d.upgrade.control-plane
```

## Optimization Tips

### Faster Builds

```bash
# Use Go build cache
export GOCACHE=$HOME/.cache/go-build

# Parallel builds
make -j4 go.build

# Skip unnecessary steps
SKIP_TESTS=true make docker-build
```

### Smaller Images

```bash
# Use multi-stage Dockerfile
# Already implemented in OpenChoreo Dockerfiles

# Verify image size
docker images | grep openchoreo
```

### Faster Image Loading (k3d)

```bash
# Load only changed component
make k3d.load.controller

# Parallel loads
make k3d.load.controller & make k3d.load.openchoreo-api & wait
```

## Troubleshooting Build Issues

### Issue: Go Build Fails

```bash
# Clean Go cache
go clean -cache -modcache

# Update dependencies
go mod tidy
go mod download

# Verify Go version
go version  # Should be 1.24+
```

### Issue: Docker Build Fails

```bash
# Check Docker daemon
docker ps

# Clean Docker build cache
docker builder prune

# Check disk space
df -h
```

### Issue: Make Target Not Found

```bash
# Ensure you're in openchoreo root
pwd  # Should end with /openchoreo

# Update Make
make --version

# View available targets
make help
```

### Issue: Image Not Loading to k3d

```bash
# Verify cluster exists
k3d cluster list

# Check cluster context
kubectl config current-context

# Manually load image
k3d image import openchoreo/controller:latest -c openchoreo-dev
```

## Quick Reference

### Essential Build Commands

```bash
# Full build
make go.build docker-build

# Single component
make k3d.update.controller

# Local development
make go.run.manager ENABLE_WEBHOOKS=false

# Tests
make test

# Code generation
make code.gen
```

### Deployment Commands

```bash
# k3d full setup
make k3d

# Manual install
helm install openchoreo-control-plane ./charts/openchoreo-control-plane -n openchoreo-control-plane --create-namespace

# Upgrade
make k3d.upgrade.control-plane

# Uninstall
helm uninstall openchoreo-control-plane -n openchoreo-control-plane
```

## Success Criteria

- ✅ Understand OpenChoreo component architecture
- ✅ Can build individual components from source
- ✅ Can deploy to local Kubernetes
- ✅ Can update components incrementally
- ✅ Know how to optimize build times
- ✅ Can troubleshoot build and deployment issues

## Related Skills

- `openchoreo-local-development` - Complete workflow
- `openchoreo-rancher-setup` - Rancher-specific setup
- `openchoreo-verify-issue-fixes` - Testing and verification
