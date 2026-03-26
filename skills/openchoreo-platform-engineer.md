---
name: openchoreo-platform-engineer
description: Use for platform engineering on OpenChoreo - creating golden paths, developer experience, self-service capabilities, and platform templates
---

# OpenChoreo Platform Engineer Skill

## When to Use This Skill

Use this skill for:
- Creating platform templates and golden paths
- Building self-service capabilities
- Improving developer experience
- Standardizing deployment patterns
- Creating platform abstractions
- Managing platform configuration

## Golden Path Templates

### Service Template

```yaml
# templates/service-template/template.yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: golang-service
  title: Go Microservice
  description: Create a production-ready Go microservice
spec:
  owner: platform-team
  type: service

  parameters:
    - title: Service Information
      required:
        - name
        - description
        - owner
      properties:
        name:
          title: Service Name
          type: string
          description: Name of the service (lowercase, hyphenated)
          pattern: '^[a-z][a-z0-9-]*$'
        description:
          title: Description
          type: string
        owner:
          title: Owner Team
          type: string
          ui:field: OwnerPicker
          ui:options:
            allowedKinds: [Group]
        system:
          title: System
          type: string
          ui:field: EntityPicker
          ui:options:
            catalogFilter:
              kind: [System]

    - title: Database Configuration
      properties:
        database:
          title: Database
          type: string
          enum: [postgres, mysql, mongodb, none]
          default: postgres
        cache:
          title: Cache
          type: boolean
          default: true

    - title: Observability
      properties:
        metrics:
          title: Enable Prometheus Metrics
          type: boolean
          default: true
        tracing:
          title: Enable Distributed Tracing
          type: boolean
          default: true

  steps:
    - id: fetch
      name: Fetch Base Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          description: ${{ parameters.description }}
          owner: ${{ parameters.owner }}
          system: ${{ parameters.system }}
          database: ${{ parameters.database }}
          cache: ${{ parameters.cache }}
          metrics: ${{ parameters.metrics }}
          tracing: ${{ parameters.tracing }}

    - id: publish
      name: Publish to GitHub
      action: publish:github
      input:
        repoUrl: github.com?owner=myorg&repo=${{ parameters.name }}
        description: ${{ parameters.description }}

    - id: register
      name: Register Component
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'

    - id: create-argocd-app
      name: Create ArgoCD Application
      action: argocd:create-app
      input:
        name: ${{ parameters.name }}
        repoUrl: ${{ steps.publish.output.remoteUrl }}
        path: 'k8s'
        namespace: production

  output:
    links:
      - title: Repository
        url: ${{ steps.publish.output.remoteUrl }}
      - title: Open in catalog
        icon: catalog
        entityRef: ${{ steps.register.output.entityRef }}
```

### Template Skeleton

```
templates/service-template/skeleton/
├── catalog-info.yaml
├── README.md
├── Dockerfile
├── go.mod
├── main.go
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
├── .github/
│   └── workflows/
│       └── deploy.yml
└── scripts/
    └── setup-db.sh
```

## Self-Service Platform

### Platform API

```yaml
# platform-api.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: platform-capabilities
  namespace: openchoreo
data:
  capabilities: |
    - name: create_service
      description: Create a new microservice
      template: golang-service
      approval_required: false

    - name: create_database
      description: Provision a database
      template: postgres-db
      approval_required: true
      approvers: [dba-team]

    - name: create_cache
      description: Provision Redis cache
      template: redis-cache
      approval_required: false

    - name: create_kafka_topic
      description: Create Kafka topic
      template: kafka-topic
      approval_required: true
      approvers: [platform-team]

    - name: enable_monitoring
      description: Enable monitoring for service
      template: monitoring-setup
      approval_required: false

    - name: request_resources
      description: Request additional resources
      form: resource-request
      approval_required: true
      approvers: [platform-team]
```

### Resource Quotas by Team

```yaml
# team-quotas.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-backend-quota
  namespace: team-backend
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "50"
    services: "20"
    persistentvolumeclaims: "10"
---
apiVersion: v1
kind: LimitRange
metadata:
  name: team-backend-limits
  namespace: team-backend
spec:
  limits:
    - max:
        cpu: "2"
        memory: 4Gi
      min:
        cpu: "100m"
        memory: 128Mi
      default:
        cpu: "500m"
        memory: 512Mi
      defaultRequest:
        cpu: "250m"
        memory: 256Mi
      type: Container
```

## Platform Standards

### Deployment Standards

```yaml
# standards/deployment-standard.yaml
apiVersion: templates.openchoreo.io/v1
kind: DeploymentStandard
metadata:
  name: production-deployment
spec:
  required:
    # Resource Management
    resources:
      required: true
      minimum:
        requests:
          cpu: 100m
          memory: 128Mi
        limits:
          cpu: 1000m
          memory: 1Gi

    # Health Checks
    healthChecks:
      required: true
      liveness:
        required: true
        minimum_initial_delay: 10s
      readiness:
        required: true
        minimum_initial_delay: 5s

    # Labels
    labels:
      required:
        - app
        - version
        - team
        - environment

    # Security
    securityContext:
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false

    # Replicas
    minReplicas: 2

    # HPA
    autoscaling:
      recommended: true
      minReplicas: 2
      maxReplicas: 10

  recommended:
    # Pod Disruption Budget
    podDisruptionBudget:
      enabled: true
      minAvailable: 1

    # Service Mesh
    serviceMesh:
      enabled: true
      type: cilium

    # Monitoring
    monitoring:
      prometheus:
        enabled: true
      jaeger:
        enabled: true

  forbidden:
    - hostNetwork: true
    - hostPID: true
    - privileged: true
```

### Policy Enforcement

```yaml
# gatekeeper-constraints.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: must-have-labels
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
    namespaces: ["production", "staging"]
  parameters:
    labels:
      - key: "app"
      - key: "version"
      - key: "team"
      - key: "environment"
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredResources
metadata:
  name: must-have-resources
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    limits:
      - cpu
      - memory
    requests:
      - cpu
      - memory
```

## Developer Portal Customization

### Custom Plugins

```typescript
// plugins/resource-usage/src/components/ResourceUsageCard.tsx
import React from 'react';
import { InfoCard } from '@backstage/core-components';
import { useEntity } from '@backstage/plugin-catalog-react';

export const ResourceUsageCard = () => {
  const { entity } = useEntity();
  const namespace = entity.metadata.annotations?.['backstage.io/kubernetes-namespace'];

  // Fetch resource usage
  const [usage, setUsage] = React.useState(null);

  React.useEffect(() => {
    fetch(`/api/platform/resource-usage/${namespace}`)
      .then(res => res.json())
      .then(setUsage);
  }, [namespace]);

  return (
    <InfoCard title="Resource Usage">
      {usage && (
        <div>
          <p>CPU: {usage.cpu}%</p>
          <p>Memory: {usage.memory}%</p>
          <p>Storage: {usage.storage} GB</p>
        </div>
      )}
    </InfoCard>
  );
};
```

## Platform Metrics

```yaml
# platform-metrics-dashboard.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: platform-metrics-dashboard
data:
  dashboard.json: |
    {
      "title": "Platform Metrics",
      "panels": [
        {
          "title": "Service Creation Rate",
          "expr": "rate(platform_services_created_total[1h])"
        },
        {
          "title": "Time to Deploy",
          "expr": "histogram_quantile(0.95, platform_deployment_duration_seconds_bucket)"
        },
        {
          "title": "Developer Satisfaction",
          "expr": "avg(developer_satisfaction_score)"
        },
        {
          "title": "Platform Adoption",
          "expr": "count(backstage_components_total) by (type)"
        }
      ]
    }
```

## Related Skills

- `openchoreo-devops` - Infrastructure
- `openchoreo-architect` - System design
- `openchoreo-app-developer` - Using platform
- `openchoreo-security` - Security standards

---

**Last Updated**: March 2025
