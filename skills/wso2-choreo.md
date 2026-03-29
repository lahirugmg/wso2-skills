---
name: wso2-choreo
description: Use when building cloud-native applications on Choreo - WSO2's AI-native Internal Developer Platform as a Service with platform engineering capabilities
---

# WSO2 Choreo Development Skill

## When to Use This Skill

Use for Choreo platform operations:
- Building cloud-native applications
- Deploying microservices
- Platform engineering
- CI/CD pipeline management
- Multi-language development (Go, Java, Python, Ballerina, Node.js)
- APIOps workflows
- Cost optimization (FinOps)

## Product Overview

Choreo is WSO2's AI-native Internal Developer Platform as a Service with comprehensive platform engineering features (March 2025 release).

## Getting Started

```bash
# Sign up
1. Visit https://wso2.com/choreo/
2. Create organization
3. Connect GitHub repositories
4. Start creating components
```

## Project Structure

```
my-app/
├── .choreo/
│   ├── component-config.yaml
│   └── endpoints.yaml
├── src/
│   └── service.bal  # or main.go, app.py, etc.
├── Dockerfile       # if using Docker buildpack
└── README.md
```

## Component Configuration

```yaml
# .choreo/component-config.yaml
id: customer-service
version: 1.0.0
componentType: Service
buildType: Ballerina  # or Dockerfile, Go, Python, etc.
port: 9090

runtime:
  memory: 512Mi
  cpu: 500m

env:
  - name: DATABASE_URL
    secretRef: database-credentials

healthCheck:
  port: 9090
  path: /health
  initialDelaySeconds: 10
```

## Creating Services

**Ballerina Service:**

```ballerina
import ballerina/http;

service /customers on new http:Listener(9090) {

    resource function get .() returns json|error {
        return {
            message: "Customer service running"
        };
    }

    resource function get health() returns http:Ok {
        return http:OK;
    }
}
```

**Go Service:**

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "os"
)

func main() {
    http.HandleFunc("/customers", getCustomers)
    http.HandleFunc("/health", healthCheck)

    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    log.Printf("Server starting on port %s", port)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}

func getCustomers(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(map[string]string{
        "message": "Customers list"
    })
}

func healthCheck(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("OK"))
}
```

## Platform Engineering

**Pipeline Configuration:**

```yaml
# Platform engineer defines pipeline templates
pipelines:
  - name: "standard-deployment"
    stages:
      - name: "build"
        steps:
          - type: "docker_build"
          - type: "security_scan"
          - type: "unit_tests"

      - name: "deploy-dev"
        environment: "development"
        approval_required: false

      - name: "deploy-prod"
        environment: "production"
        approval_required: true
        approvers:
          - "platform-team"
```

**Private Data Plane:**

```yaml
# Self-service data plane provisioning
data_planes:
  - name: "production-us-east"
    type: "private"
    kubernetes:
      provider: "aws"
      region: "us-east-1"
      cluster: "prod-eks-cluster"
```

## APIOps

**API Definition:**

```yaml
# apis/customer-api/api.yaml
type: REST
name: Customer API
version: 1.0.0
context: /customers
lifeCycleStatus: PUBLISHED

endpoints:
  production:
    - type: http
      url: https://customer-service.choreoapis.dev

policies:
  - type: rateLimit
    tier: Gold
    limit: 10000
    unit: minute

  - type: authentication
    oauth2:
      required: true
```

**Auto-Deployment:**

```bash
# Changes to api.yaml automatically:
# 1. Create/update API proxy
# 2. Apply policies
# 3. Publish to Developer Portal
# No manual API Manager operations needed
```

## Observability

**Built-in Monitoring:**

```yaml
observability:
  metrics:
    enabled: true
    custom_metrics:
      - name: "customer_created_total"
        type: "counter"

  alerts:
    - name: "HighErrorRate"
      condition: "error_rate > 5%"
      duration: "5m"
      severity: "critical"
      notify:
        - type: "email"
          to: "oncall@example.com"
```

## FinOps

**Cost Management:**

```yaml
cost_management:
  budgets:
    - project: "customer-portal"
      monthly_limit: 5000
      alert_threshold: 80

  optimization:
    auto_scaling: true
    right_sizing: true
    spot_instances: true
```

## Deployment

```bash
# Choreo automatically:
# 1. Builds your code
# 2. Runs tests
# 3. Scans for vulnerabilities
# 4. Deploys to Kubernetes
# 5. Configures ingress/gateway
# 6. Sets up monitoring

# Manual deployment via CLI (if available)
choreo deploy \
  --project my-app \
  --component customer-service \
  --environment production
```

## Best Practices

- Use trunk-based development
- Enable auto-scaling
- Monitor costs regularly
- Use private data planes for sensitive workloads
- Leverage APIOps for API management

## Resources

- **Docs**: https://wso2.com/choreo/docs/
- **Changelog**: https://wso2.com/choreo/changelog
- **Related**: `wso2-engineering-platform`, `wso2-ballerina`

---

**Last Updated**: March 2025
