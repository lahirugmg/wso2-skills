---
name: openchoreo-gitops
description: Use for GitOps workflows on OpenChoreo - Argo CD configuration, deployment automation, progressive delivery, and Git-based operations
---

# OpenChoreo GitOps Skill

## When to Use This Skill

Use for:
- Argo CD configuration and management
- Git-based deployment workflows
- Progressive delivery (canary, blue-green)
- Automated rollbacks
- Multi-environment deployments

## Argo CD Setup

### Application Configuration

```yaml
# argocd/apps/customer-service.yaml
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
    path: k8s/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 3m
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas  # Ignore HPA changes
```

### Kustomize Structure

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
└── overlays/
    ├── development/
    │   └── kustomization.yaml
    ├── staging/
    │   └── kustomization.yaml
    └── production/
        └── kustomization.yaml
```

## Progressive Delivery

### Canary Deployment

```yaml
# argocd/rollouts/canary.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: customer-service
spec:
  replicas: 5
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {duration: 5m}
        - setWeight: 40
        - pause: {duration: 10m}
        - setWeight: 60
        - pause: {duration: 10m}
        - setWeight: 80
        - pause: {duration: 10m}
      analysis:
        templates:
          - templateName: success-rate
        startingStep: 2
      trafficRouting:
        cilium:
          virtualService:
            name: customer-service
  template:
    metadata:
      labels:
        app: customer-service
    spec:
      containers:
        - name: customer-service
          image: customer-service:v2
```

## Automation

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Update image tag
        run: |
          cd k8s/overlays/production
          kustomize edit set image customer-service=registry.io/customer-service:${{ github.sha }}

      - name: Commit changes
        run: |
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add .
          git commit -m "Update image to ${{ github.sha }}"
          git push
```

## Related Skills

- `openchoreo-devops`
- `openchoreo-app-developer`
- `openchoreo-sre`

---

**Last Updated**: March 2025
