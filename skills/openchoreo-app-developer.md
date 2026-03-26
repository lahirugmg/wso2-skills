---
name: openchoreo-app-developer
description: Use when developing applications on OpenChoreo - service implementation, API development, database integration, testing, and deployment workflows
---

# OpenChoreo Application Developer Skill

## When to Use This Skill

Use this skill when:
- Implementing services on OpenChoreo
- Developing REST, GraphQL, or gRPC APIs
- Integrating with databases and external services
- Writing tests for components
- Deploying applications to OpenChoreo
- Implementing CI/CD workflows

## Quick Start

### 1. Register Component in Backstage

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: customer-service
  description: Customer management microservice
  annotations:
    github.com/project-slug: myorg/customer-service
    backstage.io/kubernetes-id: customer-service
    backstage.io/kubernetes-namespace: production
    openchoreo.io/build-type: dockerfile
spec:
  type: service
  lifecycle: production
  owner: team-backend
  system: customer-management
  providesApis:
    - customer-api
  consumesApis:
    - auth-api
  dependsOn:
    - resource:postgres-db
    - resource:redis-cache
```

### 2. Develop Service

**Go Service Example:**

```go
// main.go
package main

import (
    "context"
    "database/sql"
    "encoding/json"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gorilla/mux"
    _ "github.com/lib/pq"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

type Customer struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

type Server struct {
    db     *sql.DB
    router *mux.Router
}

func main() {
    // Database connection
    dbURL := os.Getenv("DATABASE_URL")
    if dbURL == "" {
        log.Fatal("DATABASE_URL not set")
    }

    db, err := sql.Open("postgres", dbURL)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Initialize server
    srv := &Server{
        db:     db,
        router: mux.NewRouter(),
    }

    // Routes
    srv.router.HandleFunc("/customers", srv.getCustomers).Methods("GET")
    srv.router.HandleFunc("/customers/{id}", srv.getCustomer).Methods("GET")
    srv.router.HandleFunc("/customers", srv.createCustomer).Methods("POST")
    srv.router.HandleFunc("/customers/{id}", srv.updateCustomer).Methods("PUT")
    srv.router.HandleFunc("/customers/{id}", srv.deleteCustomer).Methods("DELETE")

    // Health check
    srv.router.HandleFunc("/health", srv.healthCheck).Methods("GET")

    // Metrics
    srv.router.Handle("/metrics", promhttp.Handler())

    // HTTP server
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    httpServer := &http.Server{
        Addr:         ":" + port,
        Handler:      srv.router,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    // Graceful shutdown
    done := make(chan bool)
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

    go func() {
        <-quit
        log.Println("Server is shutting down...")

        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
        defer cancel()

        httpServer.SetKeepAlivesEnabled(false)
        if err := httpServer.Shutdown(ctx); err != nil {
            log.Fatalf("Could not gracefully shutdown the server: %v\n", err)
        }
        close(done)
    }()

    log.Printf("Server is starting on port %s...", port)
    if err := httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
        log.Fatalf("Could not listen on %s: %v\n", port, err)
    }

    <-done
    log.Println("Server stopped")
}

func (s *Server) getCustomers(w http.ResponseWriter, r *http.Request) {
    rows, err := s.db.Query("SELECT id, name, email FROM customers")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    defer rows.Close()

    customers := []Customer{}
    for rows.Next() {
        var c Customer
        if err := rows.Scan(&c.ID, &c.Name, &c.Email); err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
        customers = append(customers, c)
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(customers)
}

func (s *Server) createCustomer(w http.ResponseWriter, r *http.Request) {
    var c Customer
    if err := json.NewDecoder(r.Body).Decode(&c); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    err := s.db.QueryRow(
        "INSERT INTO customers (name, email) VALUES ($1, $2) RETURNING id",
        c.Name, c.Email,
    ).Scan(&c.ID)

    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(c)
}

func (s *Server) healthCheck(w http.ResponseWriter, r *http.Request) {
    // Check database
    if err := s.db.Ping(); err != nil {
        w.WriteHeader(http.StatusServiceUnavailable)
        json.NewEncoder(w).Encode(map[string]string{
            "status": "unhealthy",
            "error":  err.Error(),
        })
        return
    }

    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]string{
        "status": "healthy",
    })
}
```

**Dockerfile:**

```dockerfile
# Build stage
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Copy dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy source
COPY . .

# Build
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Runtime stage
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder
COPY --from=builder /app/main .

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

# Run
CMD ["./main"]
```

### 3. Kubernetes Manifests

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: customer-service
  namespace: production
  labels:
    app: customer-service
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: customer-service
  template:
    metadata:
      labels:
        app: customer-service
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: customer-service
        image: localhost:5000/customer-service:latest
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: customer-db-secret
              key: connection-string
        - name: PORT
          value: "8080"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: customer-service
  namespace: production
spec:
  selector:
    app: customer-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
---
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: HTTPRoute
metadata:
  name: customer-service-route
  namespace: production
spec:
  parentRefs:
  - name: openchoreo-gateway
  hostnames:
  - api.openchoreo.local
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api/customers
    backendRefs:
    - name: customer-service
      port: 80
```

### 4. Testing

```go
// main_test.go
package main

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/DATA-DOG/go-sqlmock"
    "github.com/gorilla/mux"
)

func TestGetCustomers(t *testing.T) {
    // Create mock database
    db, mock, err := sqlmock.New()
    if err != nil {
        t.Fatalf("an error '%s' was not expected when opening a stub database connection", err)
    }
    defer db.Close()

    // Setup expectations
    rows := sqlmock.NewRows([]string{"id", "name", "email"}).
        AddRow(1, "John Doe", "john@example.com").
        AddRow(2, "Jane Smith", "jane@example.com")

    mock.ExpectQuery("SELECT id, name, email FROM customers").
        WillReturnRows(rows)

    // Create server
    srv := &Server{
        db:     db,
        router: mux.NewRouter(),
    }
    srv.router.HandleFunc("/customers", srv.getCustomers).Methods("GET")

    // Create request
    req, err := http.NewRequest("GET", "/customers", nil)
    if err != nil {
        t.Fatal(err)
    }

    // Record response
    rr := httptest.NewRecorder()
    srv.router.ServeHTTP(rr, req)

    // Check status code
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("handler returned wrong status code: got %v want %v",
            status, http.StatusOK)
    }

    // Check response body
    var customers []Customer
    if err := json.NewDecoder(rr.Body).Decode(&customers); err != nil {
        t.Fatal(err)
    }

    if len(customers) != 2 {
        t.Errorf("expected 2 customers, got %d", len(customers))
    }

    // Ensure all expectations were met
    if err := mock.ExpectationsWereMet(); err != nil {
        t.Errorf("there were unfulfilled expectations: %s", err)
    }
}
```

### 5. GitOps Deployment with Argo CD

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
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

## Development Workflows

### Local Development

```bash
# Run locally with Docker Compose
cat > docker-compose.yml <<EOF
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://user:pass@postgres:5432/customers?sslmode=disable
      - PORT=8080
    depends_on:
      - postgres

  postgres:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=customers
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
EOF

# Run
docker-compose up
```

### Testing in K3d

```bash
# Build and push to local registry
docker build -t localhost:5000/customer-service:latest .
docker push localhost:5000/customer-service:latest

# Deploy
kubectl apply -f k8s/

# Watch deployment
kubectl rollout status deployment/customer-service -n production

# Test
curl http://openchoreo.local/api/customers
```

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: localhost:5000
  IMAGE_NAME: customer-service

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.22'

      - name: Run tests
        run: go test -v -race -coverprofile=coverage.out ./...

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.out

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .

      - name: Push to registry
        run: docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3

      - name: Update image tag
        run: |
          sed -i 's|image: .*|image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}|' k8s/deployment.yaml

      - name: Commit and push
        run: |
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add k8s/deployment.yaml
          git commit -m "Update image to ${{ github.sha }}"
          git push
```

## Best Practices

### 1. Code Structure
```
customer-service/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handlers/
│   │   └── customer.go
│   ├── models/
│   │   └── customer.go
│   ├── repository/
│   │   └── postgres.go
│   └── service/
│       └── customer.go
├── pkg/
│   ├── database/
│   └── middleware/
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── Dockerfile
├── go.mod
└── catalog-info.yaml
```

### 2. Configuration Management
- Use environment variables
- Never commit secrets
- Use Kubernetes secrets
- External configuration for different environments

### 3. Logging and Observability
- Structured logging (JSON)
- Correlation IDs
- Prometheus metrics
- OpenTelemetry tracing
- Health checks

### 4. Error Handling
- Return appropriate HTTP status codes
- Provide meaningful error messages
- Log errors with context
- Don't expose internal errors to clients

### 5. Database
- Use connection pooling
- Implement retries
- Handle migrations properly
- Use transactions where needed
- Index appropriately

## Related Skills

- `openchoreo-architect` - Design patterns
- `openchoreo-gitops` - Deployment workflows
- `openchoreo-observability` - Monitoring
- `openchoreo-devops` - Infrastructure

---

**Last Updated**: March 2025
