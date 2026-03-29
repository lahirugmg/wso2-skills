---
name: wso2-openchoreo
description: Use when setting up self-hosted Internal Developer Platform with OpenChoreo - Kubernetes-native IDP with Backstage, Cilium, and GitOps
---

# OpenChoreo Development Skill

## When to Use This Skill

Use for self-hosted IDP:
- Building internal developer platform
- Kubernetes-native deployment
- Full control over infrastructure
- On-premise or private cloud IDP
- GitOps-based workflows

## Product Overview

OpenChoreo is the open-source, self-hostable version of Choreo, providing complete IDP capabilities with Kubernetes-native architecture.

## Architecture

```
OpenChoreo Stack:
├── Backstage (Developer Portal)
├── Cilium (Service Mesh)
├── Envoy Gateway (API Gateway)
├── Argo CD (GitOps)
├── Kubernetes (Runtime)
└── Prometheus/Grafana (Observability)
```

## Prerequisites

```bash
# Requirements
- Kubernetes cluster 1.28+
- kubectl configured
- Helm 3
- 8GB+ RAM for cluster
- Domain name (for ingress)
```

## Installation

```bash
# Add Helm repo
helm repo add openchoreo https://charts.openchoreo.io
helm repo update

# Install OpenChoreo
helm install openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --create-namespace \
  --set global.domain=idp.example.com \
  --set backstage.enabled=true \
  --set cilium.enabled=true \
  --set envoyGateway.enabled=true \
  --set argocd.enabled=true
```

## Configuration

**Values File:**

```yaml
# openchoreo-values.yaml
global:
  domain: idp.example.com

backstage:
  enabled: true
  replicas: 2
  resources:
    requests:
      memory: "512Mi"
      cpu: "500m"

  auth:
    providers:
      - id: github
        clientId: YOUR_GITHUB_CLIENT_ID
        clientSecret: YOUR_GITHUB_CLIENT_SECRET

cilium:
  enabled: true
  hubble:
    enabled: true
    ui:
      enabled: true

envoyGateway:
  enabled: true

argocd:
  enabled: true
  server:
    ingress:
      enabled: true
      hosts:
        - argocd.idp.example.com

prometheus:
  enabled: true

grafana:
  enabled: true
  adminPassword: admin123
```

## Component Registration

**Catalog Info:**

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: customer-service
  description: Customer management service
  annotations:
    github.com/project-slug: myorg/customer-service
    backstage.io/kubernetes-id: customer-service
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  lifecycle: production
  owner: team-backend
  system: customer-platform
  providesApis:
    - customer-api
```

## Service Deployment

**Kubernetes Manifests:**

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: customer-service
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: customer-service
  template:
    metadata:
      labels:
        app: customer-service
    spec:
      containers:
      - name: customer-service
        image: myorg/customer-service:v1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
```

**GitOps (Argo CD):**

```yaml
# argocd/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: customer-service
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/customer-service
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Platform Templates

**Service Template:**

```yaml
# templates/service-template.yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: service-template
  title: Create a New Service
  description: Create a new microservice with best practices
spec:
  owner: platform-team
  type: service

  parameters:
    - title: Service Information
      required:
        - name
        - description
      properties:
        name:
          type: string
          description: Service name
        description:
          type: string
          description: Service description
        language:
          type: string
          enum: [go, python, java, nodejs]

  steps:
    - id: fetch-base
      name: Fetch Base Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}

    - id: create-repo
      name: Create GitHub Repository
      action: github:create
      input:
        repoName: ${{ parameters.name }}

    - id: register
      name: Register Component
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.create-repo.output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'
```

## Observability

**Access Dashboards:**

```bash
# Backstage (Developer Portal)
https://idp.example.com

# Argo CD (GitOps)
https://argocd.idp.example.com

# Grafana (Metrics)
https://grafana.idp.example.com

# Hubble UI (Service Mesh)
https://hubble.idp.example.com
```

## Platform Engineering

**Golden Paths:**

```yaml
# Define golden paths for common patterns
golden_paths:
  - name: "create-rest-api"
    template: "service-template"
    defaults:
      language: "go"
      includes:
        - database
        - redis
        - monitoring

  - name: "create-batch-job"
    template: "job-template"
    schedule: "0 0 * * *"
```

## Security

**Network Policies:**

```yaml
# Cilium network policy
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: customer-service-policy
spec:
  endpointSelector:
    matchLabels:
      app: customer-service
  ingress:
    - fromEndpoints:
      - matchLabels:
          app: gateway
  egress:
    - toEndpoints:
      - matchLabels:
          app: database
```

## Best Practices

- Use GitOps for all deployments
- Define platform templates for common patterns
- Implement proper RBAC
- Monitor resource usage
- Regular backups
- Update components regularly

## Troubleshooting

**Check Component Status:**

```bash
# Backstage
kubectl -n openchoreo logs -l app=backstage

# Argo CD
kubectl -n argocd get applications

# Cilium
cilium status
```

## Resources

- **Website**: https://openchoreo.io/
- **Docs**: https://openchoreo.io/docs/
- **GitHub**: https://github.com/wso2/openchoreo
- **Related**: `wso2-engineering-platform`, `wso2-choreo`

---

**Last Updated**: March 2025
