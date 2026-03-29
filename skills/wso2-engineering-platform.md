---
name: wso2-engineering-platform
description: Use when building cloud-native applications, setting up internal developer platforms, developing with Ballerina language, or implementing platform engineering - covers Choreo (IDP), OpenChoreo, and Ballerina
---

# WSO2 Engineering Platform Development Skill

## When to Use This Skill

Use this skill when:
- Building cloud-native applications and microservices
- Setting up an Internal Developer Platform (IDP)
- Developing services with Ballerina programming language
- Implementing platform engineering practices
- Deploying applications to Kubernetes
- Creating CI/CD pipelines and APIOps workflows
- Building multi-language applications (Go, Java, Python, Ballerina, etc.)
- Implementing FinOps and cost optimization
- Managing observability and monitoring
- Self-hosting an IDP with OpenChoreo

## Product Overview

WSO2 Engineering Platform provides comprehensive developer platform capabilities:

### Core Components

1. **Choreo** - AI-native Internal Developer Platform as a Service (SaaS)
2. **OpenChoreo** - Open-source, self-hostable version of Choreo
3. **Ballerina** - Cloud-native programming language designed for integration

### Key Capabilities (2025)

- **AI-Native Development**: AI assistance throughout development lifecycle
- **Multi-Language Support**: Ballerina, Go, Java, Python, Node.js, etc.
- **Platform Engineering**: First-class support for platform engineers
- **GitOps**: Git-based deployment and configuration
- **Observability**: Built-in monitoring, logging, tracing
- **APIOps**: Automated API lifecycle management
- **FinOps**: AI-driven cost optimization
- **Self-Service**: Developers and platform engineers self-service capabilities

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                  Choreo (AI-Native IDP)                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌────────────── Platform Engineering Layer ─────────────────┐  │
│  │                                                             │  │
│  │  • Infrastructure Configuration                            │  │
│  │  • CI/CD Pipeline Management                               │  │
│  │  • Resource Provisioning                                   │  │
│  │  • Cost Management & FinOps                                │  │
│  │  • Security & Compliance                                   │  │
│  │                                                             │  │
│  └─────────────────────────────────────────────────────────────┘ │
│                              ↓                                    │
│  ┌─────────────── Developer Experience Layer ────────────────┐  │
│  │                                                             │  │
│  │  • Code Editor & IDE Integration                           │  │
│  │  • Component Development (Services, Jobs, etc.)            │  │
│  │  • Low-Code Integration (Ballerina visual editor)          │  │
│  │  • API Management (APIOps)                                 │  │
│  │  • Testing & Debugging                                     │  │
│  │                                                             │  │
│  └─────────────────────────────────────────────────────────────┘ │
│                              ↓                                    │
│  ┌───────────────── Runtime Layer ────────────────────────────┐ │
│  │                                                              │ │
│  │  • Kubernetes (GKE, EKS, AKS, Self-hosted)                 │ │
│  │  • Service Mesh (Cilium)                                   │ │
│  │  • Gateway (Envoy Gateway)                                 │ │
│  │  • Observability Stack (Prometheus, Grafana, Jaeger)      │ │
│  │                                                              │ │
│  └──────────────────────────────────────────────────────────────┘│
│                              ↓                                    │
│  ┌────────────── Data Plane (Multiple Options) ─────────────┐  │
│  │                                                             │  │
│  │  • Choreo Managed (SaaS)                                   │  │
│  │  • Private Data Plane (Your Kubernetes)                    │  │
│  │  • Multi-Region Deployment                                 │  │
│  │                                                             │  │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                      Ballerina Language                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  • Network-first programming language                            │
│  • Visual + Textual development (bidirectional)                  │
│  • Built-in support for REST, GraphQL, gRPC, WebSocket          │
│  • Native JSON support                                           │
│  • Built-in concurrency and resilience                           │
│  • Cloud-native by design                                        │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## Common Tasks

### 1. Getting Started with Choreo

**Sign Up and Create Project:**

```bash
# 1. Sign up at https://wso2.com/choreo/

# 2. Create organization
#    - Choose organization name
#    - Set up billing (free tier available)

# 3. Create first project
#    Name: "my-app"
#    Description: "My cloud-native application"

# 4. Connect GitHub repository
#    - Authorize Choreo to access your GitHub
#    - Select repositories to use
```

**Project Structure:**

```
my-app/
├── .choreo/
│   ├── component-config.yaml    # Component metadata
│   └── endpoints.yaml           # Endpoint configurations
├── src/
│   ├── service.bal              # Ballerina service (or main.go, app.py, etc.)
│   └── tests/
│       └── service_test.bal
├── Ballerina.toml               # Ballerina project config
└── README.md
```

### 2. Creating a Component in Choreo

**Ballerina Service Component:**

```yaml
# .choreo/component-config.yaml
id: customer-service
version: 1.0.0
componentType: Service
buildType: Ballerina
port: 9090

# Runtime configuration
runtime:
  memory: 512Mi
  cpu: 500m

# Environment variables
env:
  - name: DATABASE_URL
    secretRef: database-credentials

# Health checks
healthCheck:
  port: 9090
  path: /health
  initialDelaySeconds: 10
  periodSeconds: 10
```

**Ballerina Service Code:**

```ballerina
// service.bal - Customer service in Ballerina
import ballerina/http;
import ballerina/sql;
import ballerinax/mysql;
import ballerinax/mysql.driver as _;

// Service configuration
configurable string dbHost = ?;
configurable string dbUser = ?;
configurable string dbPassword = ?;

// Database client
mysql:Client db = check new (
    host = dbHost,
    user = dbUser,
    password = dbPassword,
    database = "customers"
);

// Customer type
type Customer record {|
    int id?;
    string name;
    string email;
    string phone?;
|};

// HTTP service
service /customers on new http:Listener(9090) {

    // Get all customers
    resource function get .() returns Customer[]|error {
        stream<Customer, sql:Error?> customerStream = db->query(
            `SELECT id, name, email, phone FROM customers`
        );
        return from Customer customer in customerStream
               select customer;
    }

    // Get customer by ID
    resource function get [int id]() returns Customer|http:NotFound|error {
        Customer|sql:Error result = db->queryRow(
            `SELECT id, name, email, phone FROM customers WHERE id = ${id}`
        );

        if result is sql:NoRowsError {
            return http:NOT_FOUND;
        }

        return result;
    }

    // Create customer
    resource function post .(@http:Payload Customer customer)
            returns http:Created|error {
        sql:ExecutionResult result = check db->execute(
            `INSERT INTO customers (name, email, phone)
             VALUES (${customer.name}, ${customer.email}, ${customer.phone})`
        );

        int|string? id = result.lastInsertId;
        if id is int {
            return http:CREATED;
        }

        return error("Failed to create customer");
    }

    // Update customer
    resource function put [int id](@http:Payload Customer customer)
            returns Customer|http:NotFound|error {
        sql:ExecutionResult result = check db->execute(
            `UPDATE customers SET name = ${customer.name},
             email = ${customer.email}, phone = ${customer.phone}
             WHERE id = ${id}`
        );

        if result.affectedRowCount == 0 {
            return http:NOT_FOUND;
        }

        return self./[id]();
    }

    // Delete customer
    resource function delete [int id]() returns http:NoContent|http:NotFound|error {
        sql:ExecutionResult result = check db->execute(
            `DELETE FROM customers WHERE id = ${id}`
        );

        if result.affectedRowCount == 0 {
            return http:NOT_FOUND;
        }

        return http:NO_CONTENT;
    }

    // Health check
    resource function get health() returns http:Ok {
        return http:OK;
    }
}
```

**Deploy to Choreo:**

```bash
# Using Choreo UI
1. Go to Choreo console
2. Select project
3. Click "Create Component"
4. Choose "Service" type
5. Select GitHub repository
6. Choose Ballerina buildpack
7. Configure component settings
8. Click "Deploy"

# Using Choreo CLI (if available)
choreo deploy \
  --project my-app \
  --component customer-service \
  --environment development
```

### 3. Using Ballerina Visual Editor

**Low-Code Development Experience:**

```
Ballerina provides bidirectional mapping between text and visual:

Visual Editor:
┌─────────────────────────────────────────┐
│  [HTTP Listener] :9090                  │
│         ↓                                │
│  [Receive Request] GET /customers       │
│         ↓                                │
│  [Query Database] SELECT * FROM...      │
│         ↓                                │
│  [Transform Data] JSON response         │
│         ↓                                │
│  [Send Response] 200 OK                 │
└─────────────────────────────────────────┘

Changes in visual editor → Auto-generated code
Changes in code editor → Auto-updated visuals
```

### 4. Multi-Language Development

**Go Service Component:**

```yaml
# .choreo/component-config.yaml
componentType: Service
buildType: Dockerfile
dockerfilePath: Dockerfile
port: 8080
```

```go
// main.go - Go service
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "os"

    "github.com/gorilla/mux"
)

type Customer struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func main() {
    router := mux.NewRouter()

    router.HandleFunc("/customers", getCustomers).Methods("GET")
    router.HandleFunc("/customers/{id}", getCustomer).Methods("GET")
    router.HandleFunc("/health", healthCheck).Methods("GET")

    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    log.Printf("Server starting on port %s", port)
    log.Fatal(http.ListenAndServe(":"+port, router))
}

func getCustomers(w http.ResponseWriter, r *http.Request) {
    customers := []Customer{
        {ID: 1, Name: "John Doe", Email: "john@example.com"},
        {ID: 2, Name: "Jane Smith", Email: "jane@example.com"},
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(customers)
}

func getCustomer(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    id := vars["id"]

    customer := Customer{ID: 1, Name: "John Doe", Email: "john@example.com"}

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(customer)
}

func healthCheck(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("OK"))
}
```

### 5. Platform Engineering with Choreo

**Platform Engineering Perspective (March 2025 Feature):**

```yaml
# Platform configuration for the organization
platform_config:
  # CI/CD Pipeline Templates
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

        - name: "deploy-staging"
          environment: "staging"
          approval_required: true
          approvers:
            - "platform-team"

        - name: "deploy-prod"
          environment: "production"
          approval_required: true
          approvers:
            - "platform-team"
            - "security-team"

    - name: "hotfix-deployment"
      fast_track: true
      stages:
        - name: "build"
        - name: "deploy-prod"
          approval_required: true
          approvers:
            - "on-call-engineer"

  # Private Data Plane Configuration
  data_planes:
    - name: "production-us-east"
      type: "private"
      kubernetes:
        provider: "aws"
        region: "us-east-1"
        cluster: "prod-eks-cluster"

    - name: "production-eu-west"
      type: "private"
      kubernetes:
        provider: "aws"
        region: "eu-west-1"
        cluster: "prod-eks-cluster-eu"

  # Cost Management
  cost_management:
    budgets:
      - project: "customer-portal"
        monthly_limit: 5000  # USD
        alert_threshold: 80  # percent

    optimization:
      auto_scaling: true
      right_sizing_recommendations: true
      spot_instances: true

  # Security & Compliance
  security:
    container_scanning: true
    vulnerability_threshold: "high"
    secrets_management: "vault"

    policies:
      - name: "no-root-containers"
        enabled: true
      - name: "require-resource-limits"
        enabled: true
```

**Platform Engineer Dashboard:**

- View all projects and components
- Monitor costs across organization
- Configure CI/CD pipelines
- Manage private data planes
- Set security policies
- Review resource utilization

### 6. APIOps with Choreo

**Git-Based API Management:**

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
      url: https://customer-service.prod.choreoapis.dev

  sandbox:
    - type: http
      url: https://customer-service.dev.choreoapis.dev

operations:
  - target: /
    verb: GET
    authType: Application & Application User
    throttlingPolicy: Unlimited

  - target: /{id}
    verb: GET
    authType: Application & Application User
    throttlingPolicy: Unlimited

  - target: /
    verb: POST
    authType: Application & Application User
    throttlingPolicy: 10KPerMin

policies:
  - type: rateLimit
    tier: Gold
    limit: 10000
    unit: minute

  - type: authentication
    oauth2:
      required: true

  - type: cors
    allowOrigins:
      - "*"
    allowMethods:
      - GET
      - POST
      - PUT
      - DELETE
```

**APIOps Workflow:**

```bash
# 1. Developer updates API definition in Git
git add apis/customer-api/api.yaml
git commit -m "Add POST /customers endpoint"
git push origin main

# 2. Choreo automatically:
#    - Detects changes in api.yaml
#    - Creates/updates API proxy
#    - Applies policies
#    - Publishes to Developer Portal
#    - No manual API Manager operations needed
```

### 7. Observability in Choreo

**Built-in Monitoring:**

```yaml
# Component observability configuration
observability:
  metrics:
    enabled: true
    custom_metrics:
      - name: "customer_created_total"
        type: "counter"
        help: "Total customers created"

      - name: "customer_fetch_duration"
        type: "histogram"
        help: "Time to fetch customer data"
        buckets: [0.1, 0.5, 1, 2, 5]

  logging:
    level: "INFO"
    format: "json"
    # Logs automatically aggregated in Choreo

  tracing:
    enabled: true
    sample_rate: 0.1
    # Distributed tracing with OpenTelemetry

  alerts:
    - name: "HighErrorRate"
      condition: "error_rate > 5%"
      duration: "5m"
      severity: "critical"
      notify:
        - type: "email"
          to: "oncall@example.com"
        - type: "slack"
          channel: "#alerts"

    - name: "HighLatency"
      condition: "p95_latency > 2s"
      duration: "10m"
      severity: "warning"
```

**Choreo Observability Dashboard:**

- Real-time metrics (requests, errors, latency)
- Distributed tracing
- Log aggregation and search
- Cost analytics
- Custom dashboards
- Alerting and notifications

### 8. OpenChoreo (Self-Hosted IDP)

**OpenChoreo Architecture:**

```
OpenChoreo Components:
- Backstage (Developer Portal)
- Cilium (Service Mesh)
- Envoy Gateway (API Gateway)
- GitOps (Argo CD)
- Kubernetes (Runtime)
```

**Deploy OpenChoreo:**

```bash
# Prerequisites
- Kubernetes cluster (1.28+)
- kubectl configured
- Helm 3

# Install OpenChoreo
helm repo add openchoreo https://charts.openchoreo.io
helm repo update

helm install openchoreo openchoreo/openchoreo \
  --namespace openchoreo \
  --create-namespace \
  --set global.domain=idp.example.com \
  --set backstage.enabled=true \
  --set cilium.enabled=true \
  --set envoyGateway.enabled=true \
  --set argocd.enabled=true

# Access Backstage
kubectl port-forward -n openchoreo svc/backstage 7007:7007
# Open http://localhost:7007
```

**Create Component in OpenChoreo:**

```yaml
# catalog-info.yaml (Backstage format)
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: customer-service
  description: Customer management service
  annotations:
    github.com/project-slug: myorg/customer-service
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  lifecycle: production
  owner: team-backend
  system: customer-platform
  providesApis:
    - customer-api
```

## Best Practices

### Development Workflow

1. **Trunk-Based Development**
   - Commit frequently to main branch
   - Use feature flags for incomplete features
   - Short-lived feature branches (< 2 days)
   - Automated testing on every commit

2. **GitOps**
   - Store all configuration in Git
   - Declarative infrastructure and app config
   - Automated sync with Git repository
   - Audit trail through Git history

3. **Testing Strategy**
   - Unit tests (80%+ coverage)
   - Integration tests
   - Contract testing for APIs
   - Load testing before production
   - Chaos engineering for resilience

### Platform Engineering

1. **Self-Service**
   - Provide templates for common patterns
   - Clear documentation and examples
   - Automated provisioning
   - Empower developers to self-serve

2. **Golden Paths**
   - Define recommended ways to build and deploy
   - Automate the golden path
   - Make it easier to do the right thing
   - Allow deviation when needed

3. **Cost Optimization**
   - Set resource limits and quotas
   - Enable autoscaling
   - Use spot instances where appropriate
   - Monitor and alert on budget thresholds

## Integration with Other WSO2 Products

### With WSO2 API Manager

```yaml
# API Manager backend on Choreo
api_backend:
  component: "customer-service"
  endpoint: "https://customer-service.choreoapis.dev"

  api_proxy:
    name: "Customer API"
    context: "/customers"
    version: "1.0.0"
```

### With WSO2 Identity Server / Asgardeo

```yaml
# Authentication for Choreo components
authentication:
  provider: "asgardeo"
  organization: "my-org"

  application:
    client_id: "${CLIENT_ID}"
    client_secret: "${CLIENT_SECRET}"
```

### With WSO2 Integrator

```yaml
# Deploy integrations on Choreo
integration_component:
  type: "integration"
  runtime: "ballerina"
  source: "github.com/myorg/customer-integrations"
```

## Troubleshooting

### Common Issues

**1. Build Failures**
- **Symptom**: Component build fails
- **Solutions**:
  - Check build logs in Choreo
  - Verify Dockerfile/buildpack configuration
  - Check for dependency issues
  - Ensure tests pass locally

**2. Deployment Issues**
- **Symptom**: Component fails to deploy
- **Solutions**:
  - Check pod logs
  - Verify resource limits
  - Check health check configuration
  - Verify secrets and config

**3. Performance Issues**
- **Symptom**: Slow response times
- **Solutions**:
  - Check metrics dashboard
  - Enable tracing to find bottlenecks
  - Review resource utilization
  - Implement caching

## Resources

### Official Documentation
- **Choreo**: https://wso2.com/choreo/docs/
- **Ballerina**: https://ballerina.io/learn/
- **OpenChoreo**: https://openchoreo.io/
- **Choreo API**: https://wso2.com/choreo/docs/api/

### Related Skills
- `wso2-choreo` - Detailed Choreo operations
- `wso2-ballerina` - Deep dive into Ballerina language
- `wso2-openchoreo` - Self-hosted IDP setup

### Learning Resources
- Ballerina by Example: https://ballerina.io/learn/by-example/
- Choreo Tutorials: https://wso2.com/choreo/docs/tutorials/
- Platform Engineering Guide: https://wso2.com/choreo/docs/platform-engineering/

## Version Compatibility

- **Choreo**: Latest (SaaS, always up-to-date)
- **Ballerina**: 2201.x (Swan Lake)
- **OpenChoreo**: 1.x
- **Kubernetes**: 1.28+

---

**Last Updated**: March 2025
**Skill Type**: Flexible - Adapt to your platform needs
**Related Platforms**: Engineering, Integration, API, Identity
