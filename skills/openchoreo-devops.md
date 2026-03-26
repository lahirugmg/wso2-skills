---
name: openchoreo-devops
description: Use when setting up, configuring, and managing OpenChoreo infrastructure - K3d clusters, Docker environments, Helm deployments, and cluster operations
---

# OpenChoreo DevOps Skill

## When to Use This Skill

Use this skill when:
- Setting up OpenChoreo on local Docker with K3d
- Installing OpenChoreo on production Kubernetes clusters
- Managing OpenChoreo infrastructure and upgrades
- Configuring cluster resources and networking
- Automating OpenChoreo deployments
- Scaling and maintaining OpenChoreo installations
- Performing disaster recovery and backups

## Prerequisites

### Local Development Setup
```bash
# Required tools
- Docker Desktop or Docker Engine (20.10+)
- kubectl (1.28+)
- Helm 3 (3.12+)
- K3d (5.6+)

# Optional but recommended
- k9s (terminal UI for Kubernetes)
- kubectx/kubens (context switching)
- stern (multi-pod log tailing)
```

### System Requirements

**Minimum (Development):**
- 8 GB RAM
- 4 CPU cores
- 50 GB disk space
- Docker with 4 GB memory allocated

**Recommended (Production):**
- 32 GB RAM
- 8 CPU cores
- 200 GB SSD
- High-bandwidth network

## Installation Guide

### 1. Install Required Tools

```bash
# Install K3d (Linux/Mac)
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

# Verify installation
k3d --version

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify Helm
helm version

# Install k9s (optional but highly recommended)
# Mac
brew install k9s

# Linux
curl -sL https://github.com/derailed/k9s/releases/download/v0.31.0/k9s_Linux_amd64.tar.gz | tar xz -C /tmp
sudo mv /tmp/k9s /usr/local/bin/

# Install stern (optional)
brew install stern  # Mac
# or download from https://github.com/stern/stern/releases
```

### 2. Create K3d Cluster for OpenChoreo

```bash
# Create cluster configuration file
cat > openchoreo-cluster-config.yaml <<EOF
apiVersion: k3d.io/v1alpha5
kind: Simple
metadata:
  name: openchoreo
servers: 1
agents: 3
image: rancher/k3s:v1.28.5-k3s1

# Port mappings
ports:
  - port: 80:80
    nodeFilters:
      - loadbalancer
  - port: 443:443
    nodeFilters:
      - loadbalancer
  - port: 8080:30080
    nodeFilters:
      - loadbalancer

# Registry configuration
registries:
  create:
    name: openchoreo-registry
    host: "0.0.0.0"
    hostPort: "5000"

# Resource limits
options:
  k3d:
    wait: true
    timeout: "120s"
  k3s:
    extraArgs:
      - arg: --disable=traefik  # We'll use Envoy Gateway
        nodeFilters:
          - server:*
      - arg: --disable=servicelb  # We'll use Cilium
        nodeFilters:
          - server:*
      - arg: --kube-apiserver-arg=feature-gates=MixedProtocolLBService=true
        nodeFilters:
          - server:*

# Volume mounts for persistence
volumes:
  - volume: /tmp/openchoreo-data:/var/lib/rancher/k3s/storage
    nodeFilters:
      - all
EOF

# Create cluster
k3d cluster create -c openchoreo-cluster-config.yaml

# Verify cluster
kubectl cluster-info
kubectl get nodes
```

**Alternative: Quick Start (No Config File)**

```bash
# Simple cluster creation
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

# Set context
kubectl config use-context k3d-openchoreo
```

### 3. Verify Cluster Setup

```bash
# Check nodes
kubectl get nodes -o wide

# Expected output:
# NAME                      STATUS   ROLES                  AGE   VERSION
# k3d-openchoreo-server-0   Ready    control-plane,master   2m    v1.28.5+k3s1
# k3d-openchoreo-agent-0    Ready    <none>                 2m    v1.28.5+k3s1
# k3d-openchoreo-agent-1    Ready    <none>                 2m    v1.28.5+k3s1
# k3d-openchoreo-agent-2    Ready    <none>                 2m    v1.28.5+k3s1

# Check namespaces
kubectl get namespaces

# Check storage classes
kubectl get storageclasses
```

### 4. Install OpenChoreo via Helm

```bash
# Add OpenChoreo Helm repository
helm repo add openchoreo https://charts.openchoreo.io
helm repo update

# Create namespace
kubectl create namespace openchoreo

# Create values file for customization
cat > openchoreo-values.yaml <<EOF
global:
  domain: openchoreo.local
  storageClass: local-path

# Backstage (Developer Portal)
backstage:
  enabled: true
  replicas: 2
  image:
    repository: openchoreo/backstage
    tag: latest
    pullPolicy: IfNotPresent

  resources:
    requests:
      memory: "512Mi"
      cpu: "500m"
    limits:
      memory: "1Gi"
      cpu: "1000m"

  ingress:
    enabled: true
    className: cilium
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-staging
    hosts:
      - host: openchoreo.local
        paths:
          - path: /
            pathType: Prefix

  auth:
    # Configure authentication providers
    providers:
      github:
        enabled: false  # Enable if using GitHub auth
        # clientId: YOUR_GITHUB_CLIENT_ID
        # clientSecret: YOUR_GITHUB_CLIENT_SECRET

      guest:
        enabled: true  # Allow guest access for local dev

  database:
    type: postgres
    postgres:
      host: postgresql.openchoreo.svc.cluster.local
      port: 5432
      username: backstage
      password: backstage123
      database: backstage

# Cilium (Service Mesh & Networking)
cilium:
  enabled: true
  hubble:
    enabled: true
    relay:
      enabled: true
    ui:
      enabled: true
      ingress:
        enabled: true
        hosts:
          - hubble.openchoreo.local

  operator:
    replicas: 1
    resources:
      requests:
        cpu: 100m
        memory: 128Mi

  envoyConfig:
    enabled: true

# Envoy Gateway (API Gateway)
envoyGateway:
  enabled: true
  replicas: 2
  service:
    type: LoadBalancer

# Argo CD (GitOps)
argocd:
  enabled: true
  server:
    replicas: 1
    ingress:
      enabled: true
      ingressClassName: cilium
      hosts:
        - argocd.openchoreo.local
    resources:
      requests:
        cpu: 100m
        memory: 128Mi

  controller:
    replicas: 1
    resources:
      requests:
        cpu: 250m
        memory: 512Mi

  repoServer:
    replicas: 1
    resources:
      requests:
        cpu: 100m
        memory: 256Mi

  configs:
    secret:
      argocdServerAdminPassword: \$2a\$10\$rRyBsGSHK6.uc8fntPwVIuLVHgsAhAX7TcdrqW/XGlabvu1xOR8m.  # admin

# PostgreSQL (Database)
postgresql:
  enabled: true
  auth:
    username: backstage
    password: backstage123
    database: backstage
  primary:
    persistence:
      enabled: true
      size: 8Gi
    resources:
      requests:
        cpu: 250m
        memory: 256Mi

# Prometheus (Metrics)
prometheus:
  enabled: true
  server:
    persistentVolume:
      enabled: true
      size: 8Gi
    resources:
      requests:
        cpu: 500m
        memory: 512Mi

  alertmanager:
    enabled: true
    persistentVolume:
      enabled: true
      size: 2Gi

# Grafana (Dashboards)
grafana:
  enabled: true
  adminPassword: admin123
  persistence:
    enabled: true
    size: 1Gi

  ingress:
    enabled: true
    ingressClassName: cilium
    hosts:
      - grafana.openchoreo.local

  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
        - name: Prometheus
          type: prometheus
          url: http://prometheus-server.openchoreo.svc.cluster.local
          access: proxy
          isDefault: true

# Cert Manager (TLS Certificates)
cert-manager:
  enabled: true
  installCRDs: true

# Tekton (CI/CD Pipelines) - Optional
tekton:
  enabled: false  # Enable if you want Tekton instead of Argo CD

# Crossplane (Infrastructure as Code) - Optional
crossplane:
  enabled: false  # Enable for cloud resource management
EOF

# Install OpenChoreo
helm install openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --values openchoreo-values.yaml \
  --timeout 15m \
  --wait

# Monitor installation
kubectl get pods -n openchoreo -w
```

### 5. Post-Installation Configuration

```bash
# Add local domain entries to /etc/hosts
sudo tee -a /etc/hosts <<EOF
127.0.0.1 openchoreo.local
127.0.0.1 argocd.openchoreo.local
127.0.0.1 grafana.openchoreo.local
127.0.0.1 hubble.openchoreo.local
EOF

# Verify all components are running
kubectl get pods -n openchoreo

# Check services
kubectl get svc -n openchoreo

# Check ingresses
kubectl get ingress -n openchoreo

# Get ArgoCD admin password
kubectl -n openchoreo get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

## Cluster Management Operations

### Accessing Components

```bash
# Backstage (Developer Portal)
open http://openchoreo.local

# Argo CD
open http://argocd.openchoreo.local
# Username: admin
# Password: (from secret above)

# Grafana
open http://grafana.openchoreo.local
# Username: admin
# Password: admin123

# Hubble UI (Cilium)
open http://hubble.openchoreo.local

# Port forwarding (alternative to ingress)
kubectl port-forward -n openchoreo svc/backstage 7007:7007
# Access at http://localhost:7007
```

### Cluster Scaling

```bash
# Scale agent nodes
k3d node create openchoreo-agent-3 --cluster openchoreo --role agent

# Verify new node
kubectl get nodes

# Scale OpenChoreo components
kubectl scale deployment backstage -n openchoreo --replicas=3
kubectl scale deployment argocd-server -n openchoreo --replicas=2

# Scale with Helm
helm upgrade openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --reuse-values \
  --set backstage.replicas=3 \
  --set argocd.server.replicas=2
```

### Resource Management

```bash
# Check resource usage
kubectl top nodes
kubectl top pods -n openchoreo

# View resource requests/limits
kubectl describe nodes | grep -A 5 "Allocated resources"

# Adjust resource limits
cat > resource-patch.yaml <<EOF
spec:
  template:
    spec:
      containers:
      - name: backstage
        resources:
          requests:
            memory: "1Gi"
            cpu: "1000m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
EOF

kubectl patch deployment backstage -n openchoreo --patch-file resource-patch.yaml
```

### Backup and Restore

```bash
# Backup etcd (K3s stores data in SQLite by default for single-node)
# For production, backup PersistentVolumes and configs

# Backup Backstage catalog
kubectl exec -n openchoreo deploy/backstage -- \
  pg_dump -h postgresql -U backstage backstage > backstage-backup.sql

# Backup Argo CD applications
kubectl get applications -n openchoreo -o yaml > argocd-apps-backup.yaml

# Backup all OpenChoreo resources
kubectl get all,configmap,secret,pvc -n openchoreo -o yaml > openchoreo-backup.yaml

# Restore from backup
kubectl apply -f openchoreo-backup.yaml
kubectl apply -f argocd-apps-backup.yaml

# Restore database
kubectl exec -n openchoreo deploy/backstage -- \
  psql -h postgresql -U backstage backstage < backstage-backup.sql
```

### Upgrade OpenChoreo

```bash
# Check current version
helm list -n openchoreo

# Update Helm repo
helm repo update

# Check available versions
helm search repo openchoreo/openchoreo --versions

# Backup before upgrade
kubectl get all -n openchoreo -o yaml > pre-upgrade-backup.yaml

# Upgrade to specific version
helm upgrade openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --version 1.1.0 \
  --reuse-values \
  --wait

# Rollback if needed
helm rollback openchoreo -n openchoreo

# Verify upgrade
kubectl get pods -n openchoreo
helm history openchoreo -n openchoreo
```

## Automation Scripts

### Cluster Setup Script

```bash
#!/bin/bash
# setup-openchoreo.sh - Complete OpenChoreo setup

set -e

echo "🚀 Setting up OpenChoreo on K3d..."

# Check prerequisites
command -v k3d >/dev/null 2>&1 || { echo "k3d not found. Please install it."; exit 1; }
command -v helm >/dev/null 2>&1 || { echo "helm not found. Please install it."; exit 1; }
command -v kubectl >/dev/null 2>&1 || { echo "kubectl not found. Please install it."; exit 1; }

# Configuration
CLUSTER_NAME="openchoreo"
NAMESPACE="openchoreo"
DOMAIN="openchoreo.local"

# Create cluster
echo "📦 Creating K3d cluster..."
k3d cluster create $CLUSTER_NAME \
  --servers 1 \
  --agents 3 \
  --port "80:80@loadbalancer" \
  --port "443:443@loadbalancer" \
  --port "8080:30080@loadbalancer" \
  --registry-create ${CLUSTER_NAME}-registry:0.0.0.0:5000 \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --volume /tmp/${CLUSTER_NAME}-data:/var/lib/rancher/k3s/storage@all

# Wait for cluster
echo "⏳ Waiting for cluster to be ready..."
kubectl wait --for=condition=Ready nodes --all --timeout=300s

# Add Helm repo
echo "📚 Adding OpenChoreo Helm repository..."
helm repo add openchoreo https://charts.openchoreo.io
helm repo update

# Create namespace
echo "📁 Creating namespace..."
kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

# Install OpenChoreo
echo "🔧 Installing OpenChoreo..."
helm install openchoreo openchoreo/openchoreo \
  --namespace $NAMESPACE \
  --set global.domain=$DOMAIN \
  --set backstage.auth.providers.guest.enabled=true \
  --set grafana.adminPassword=admin123 \
  --timeout 15m \
  --wait

# Add hosts entry
echo "🌐 Adding hosts entries..."
if ! grep -q "$DOMAIN" /etc/hosts; then
    echo "127.0.0.1 $DOMAIN argocd.$DOMAIN grafana.$DOMAIN hubble.$DOMAIN" | sudo tee -a /etc/hosts
fi

# Get access info
echo ""
echo "✅ OpenChoreo installed successfully!"
echo ""
echo "📍 Access Points:"
echo "  - Backstage:  http://$DOMAIN"
echo "  - Argo CD:    http://argocd.$DOMAIN"
echo "  - Grafana:    http://grafana.$DOMAIN"
echo "  - Hubble UI:  http://hubble.$DOMAIN"
echo ""
echo "🔑 Credentials:"
echo "  - Backstage: Guest access enabled"
echo "  - Grafana:   admin / admin123"
echo "  - Argo CD:   admin / (run: kubectl -n $NAMESPACE get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d)"
echo ""
echo "🎉 Happy developing!"
```

### Cluster Teardown Script

```bash
#!/bin/bash
# teardown-openchoreo.sh

CLUSTER_NAME="openchoreo"

echo "🗑️  Tearing down OpenChoreo..."

# Delete cluster
k3d cluster delete $CLUSTER_NAME

# Remove hosts entries
sudo sed -i '' '/openchoreo.local/d' /etc/hosts

# Clean up volumes
rm -rf /tmp/${CLUSTER_NAME}-data

echo "✅ Cleanup complete!"
```

## Production Deployment

### On Existing Kubernetes Cluster

```bash
# Prerequisites: Kubernetes 1.28+ cluster with:
# - Storage provisioner (e.g., EBS, GCE PD, Azure Disk)
# - LoadBalancer support (or Ingress controller)
# - DNS configured

# Create production values
cat > openchoreo-production.yaml <<EOF
global:
  domain: openchoreo.example.com
  storageClass: gp3  # or your storage class

backstage:
  enabled: true
  replicas: 3
  resources:
    requests:
      memory: "2Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "2000m"

  ingress:
    enabled: true
    className: nginx  # or your ingress class
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
    tls:
      - secretName: backstage-tls
        hosts:
          - openchoreo.example.com

  database:
    type: postgres
    postgres:
      host: postgres.db.svc.cluster.local  # External database
      port: 5432
      username: backstage
      # Use external secrets operator
      passwordSecret:
        name: backstage-db-secret
        key: password

cilium:
  enabled: true
  operator:
    replicas: 2
  hubble:
    enabled: true
    relay:
      enabled: true
      replicas: 2

argocd:
  enabled: true
  server:
    replicas: 3
    ingress:
      enabled: true
      tls:
        - secretName: argocd-tls
          hosts:
            - argocd.example.com

  controller:
    replicas: 3

  repoServer:
    replicas: 3

postgresql:
  enabled: false  # Use external managed database

prometheus:
  enabled: true
  server:
    persistentVolume:
      enabled: true
      size: 50Gi
      storageClass: gp3
    replicaCount: 2

grafana:
  enabled: true
  replicas: 2
  persistence:
    enabled: true
    storageClass: gp3
EOF

# Install on production
helm install openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --create-namespace \
  --values openchoreo-production.yaml \
  --timeout 20m \
  --wait
```

## Troubleshooting

### Common Issues

**1. Pods Not Starting**
```bash
# Check pod status
kubectl get pods -n openchoreo

# Describe problematic pod
kubectl describe pod <pod-name> -n openchoreo

# Check logs
kubectl logs <pod-name> -n openchoreo

# Check events
kubectl get events -n openchoreo --sort-by='.lastTimestamp'
```

**2. Storage Issues**
```bash
# Check PVCs
kubectl get pvc -n openchoreo

# Check storage class
kubectl get storageclass

# Describe PVC
kubectl describe pvc <pvc-name> -n openchoreo
```

**3. Network Issues**
```bash
# Check Cilium status
kubectl exec -n kube-system ds/cilium -- cilium status

# Test connectivity
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- /bin/bash
# Inside container: curl http://backstage.openchoreo.svc.cluster.local:7007

# Check DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup backstage.openchoreo.svc.cluster.local
```

## Best Practices

1. **Resource Management**
   - Set appropriate resource requests and limits
   - Use HPA for auto-scaling
   - Monitor resource usage regularly

2. **High Availability**
   - Run multiple replicas of critical components
   - Use anti-affinity rules for pod distribution
   - Configure PodDisruptionBudgets

3. **Security**
   - Use RBAC for access control
   - Enable network policies
   - Rotate secrets regularly
   - Keep components updated

4. **Monitoring**
   - Set up alerts for critical metrics
   - Monitor component health
   - Track resource utilization
   - Log aggregation

5. **Backup Strategy**
   - Regular backups of etcd/data
   - Backup PersistentVolumes
   - Document restore procedures
   - Test restore regularly

## Related Skills

- `openchoreo-architect` - Platform architecture and design
- `openchoreo-sre` - Site reliability engineering
- `openchoreo-security` - Security configuration
- `openchoreo-troubleshooting` - Problem diagnosis and resolution

---

**Last Updated**: March 2025
**Skill Type**: Rigid - Follow installation and configuration procedures exactly
