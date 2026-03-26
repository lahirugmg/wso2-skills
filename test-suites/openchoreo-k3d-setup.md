---
suite_name: "OpenChoreo K3d Setup"
description: "Complete setup and validation of OpenChoreo on K3d (Docker/Podman)"
skills_tested:
  - openchoreo-devops
  - openchoreo-networking
  - openchoreo-observability
difficulty: intermediate
estimated_duration: 30-45 minutes
prerequisites:
  - Docker Desktop OR Podman Desktop
  - kubectl installed (1.28+)
  - helm installed (3.12+)
  - k3d installed (5.6+)
  - 8GB+ RAM available
environment:
  platform: local
  runtime: k3d
version: 1.0.0
---

# OpenChoreo K3d Setup Test Suite

## Overview

Faster alternative to Kind - uses K3d which is optimized for local development. Works with both Docker and Podman.

## Quick Start

This test suite follows a similar structure to the Podman suite but uses K3d for faster cluster creation and better local development experience.

## Test Sequence

1. ✓ Verify container runtime (Docker/Podman)
2. ✓ Install K3d if needed
3. ✓ Create K3d cluster with configuration
4. ✓ Verify cluster health
5. ✓ Add OpenChoreo Helm repo
6. ✓ Install OpenChoreo
7. ✓ Configure access
8. ✓ Verify all components
9. ✓ Deploy test application
10. ✓ Cleanup

For detailed test steps, see `openchoreo-podman-setup.md` and adapt commands for K3d.

## Key Differences from Podman/Kind Suite

- Uses K3d instead of Kind
- Faster cluster creation (~2 minutes vs ~5 minutes)
- Built-in registry support
- Traefik disabled (using Cilium/Envoy)
- Better suited for rapid iteration

## Sample K3d Cluster Creation

```bash
k3d cluster create openchoreo \
  --servers 1 \
  --agents 3 \
  --port "80:80@loadbalancer" \
  --port "443:443@loadbalancer" \
  --port "8080:30080@loadbalancer" \
  --registry-create openchoreo-registry:0.0.0.0:5000 \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --volume /tmp/openchoreo-data:/var/lib/rancher/k3s/storage@all
```

---

**Note**: For complete test details, follow the structure in `openchoreo-podman-setup.md` but replace Kind commands with K3d equivalents.
