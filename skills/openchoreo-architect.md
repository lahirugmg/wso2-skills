---
name: openchoreo-architect
description: Use when designing application architecture on OpenChoreo - component design, service decomposition, API strategy, data flow, and platform patterns
---

# OpenChoreo Architect Skill

## When to Use This Skill

Use this skill when:
- Designing new applications on OpenChoreo
- Decomposing monoliths into microservices
- Defining API strategy and contracts
- Planning data architecture and flows
- Choosing technology stacks
- Designing for scalability and resilience
- Creating component blueprints

## Architecture Principles

### OpenChoreo Component Model

```
Application Architecture on OpenChoreo:

┌─────────────────────────────────────────────────────────┐
│                     Application                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐   │
│  │   Service   │  │   Service   │  │    Service   │   │
│  │ Components  │  │ Components  │  │  Components  │   │
│  │             │  │             │  │              │   │
│  │ • REST API  │  │ • GraphQL   │  │ • WebSocket  │   │
│  │ • gRPC      │  │ • AsyncAPI  │  │ • Task Queue │   │
│  └─────────────┘  └─────────────┘  └──────────────┘   │
│         │                │                   │          │
│         └────────────────┴───────────────────┘          │
│                          │                              │
│  ┌──────────────────────▼───────────────────────────┐  │
│  │          Integration Components                   │  │
│  │  • Message Brokers  • Data Sync  • Connectors   │  │
│  └────────────────────────────────────────────────────┘ │
│                          │                              │
│  ┌──────────────────────▼───────────────────────────┐  │
│  │            Data Components                        │  │
│  │  • Databases  • Caches  • Object Storage         │  │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Design Patterns

### 1. Service Decomposition Pattern

**Decompose by Business Capability:**

```yaml
# E-Commerce Application Architecture

# Customer Management Domain
components:
  - name: customer-profile-service
    type: service
    responsibility: Manage customer profiles
    apis:
      - POST /customers
      - GET /customers/{id}
      - PUT /customers/{id}
    database: postgres-customers

  - name: customer-auth-service
    type: service
    responsibility: Authentication and authorization
    apis:
      - POST /auth/login
      - POST /auth/register
      - POST /auth/refresh
    integration:
      - asgardeo

# Order Management Domain
components:
  - name: order-service
    type: service
    responsibility: Order lifecycle management
    apis:
      - POST /orders
      - GET /orders/{id}
      - PUT /orders/{id}/status
    database: postgres-orders
    events:
      publishes:
        - order.created
        - order.completed
      subscribes:
        - payment.completed

  - name: order-processor
    type: worker
    responsibility: Process orders asynchronously
    triggers:
      - event: order.created
    integration:
      - inventory-service
      - payment-service

# Product Catalog Domain
components:
  - name: product-catalog-service
    type: service
    responsibility: Product information
    apis:
      - GET /products
      - GET /products/{id}
    database: mongodb-catalog
    cache: redis

  - name: product-search-service
    type: service
    responsibility: Full-text search
    apis:
      - GET /search
    database: elasticsearch

# Payment Domain
components:
  - name: payment-service
    type: service
    responsibility: Payment processing
    apis:
      - POST /payments
      - GET /payments/{id}
    integration:
      - stripe-connector
      - paypal-connector
    events:
      publishes:
        - payment.completed
        - payment.failed
```

### 2. API Gateway Pattern

```yaml
# API Gateway Architecture

api_gateway:
  name: api-gateway
  type: envoy-gateway

  routes:
    # Customer APIs
    - match:
        prefix: /api/customers
      route:
        cluster: customer-service
      policies:
        - authentication: oauth2
        - ratelimit: 1000/min
        - cors: enabled

    # Order APIs
    - match:
        prefix: /api/orders
      route:
        cluster: order-service
      policies:
        - authentication: oauth2
        - ratelimit: 500/min
        - retry: 3_attempts

    # Product APIs
    - match:
        prefix: /api/products
      route:
        cluster: product-service
      policies:
        - cache: 5min
        - ratelimit: 2000/min

    # Search APIs
    - match:
        prefix: /api/search
      route:
        cluster: search-service
      policies:
        - cache: 1min
        - ratelimit: 100/min

  observability:
    - metrics: prometheus
    - tracing: jaeger
    - logging: loki
```

### 3. Event-Driven Architecture

```yaml
# Event-Driven Communication

event_bus:
  type: kafka
  topics:
    # Order Events
    - name: orders
      partitions: 3
      replication: 3
      events:
        - order.created
        - order.updated
        - order.completed
        - order.cancelled

    # Payment Events
    - name: payments
      partitions: 3
      events:
        - payment.initiated
        - payment.completed
        - payment.failed
        - payment.refunded

    # Inventory Events
    - name: inventory
      partitions: 3
      events:
        - inventory.reserved
        - inventory.released
        - inventory.low_stock

# Event Choreography
workflows:
  order_fulfillment:
    trigger: order.created
    flow:
      - service: inventory-service
        action: reserve_inventory
        on_success: inventory.reserved
        on_failure: inventory.insufficient

      - service: payment-service
        waits_for: inventory.reserved
        action: process_payment
        on_success: payment.completed
        on_failure: payment.failed

      - service: shipping-service
        waits_for: payment.completed
        action: create_shipment
        on_success: order.completed
```

### 4. CQRS Pattern

```yaml
# Command Query Responsibility Segregation

# Write Side (Commands)
command_service:
  name: order-command-service
  type: service
  responsibility: Handle write operations
  apis:
    - POST /orders
    - PUT /orders/{id}
    - DELETE /orders/{id}
  database: postgres-orders-write
  events:
    publishes:
      - order.created
      - order.updated
      - order.deleted

# Read Side (Queries)
query_service:
  name: order-query-service
  type: service
  responsibility: Handle read operations
  apis:
    - GET /orders
    - GET /orders/{id}
    - GET /orders/search
  database: mongodb-orders-read  # Denormalized for fast reads
  cache: redis
  events:
    subscribes:
      - order.created
      - order.updated
      - order.deleted

# Event Projection
projector:
  name: order-projector
  type: worker
  responsibility: Update read model from events
  subscribes:
    - order.created → insert_to_read_db
    - order.updated → update_in_read_db
    - order.deleted → delete_from_read_db
```

### 5. Backend for Frontend (BFF) Pattern

```yaml
# BFF Architecture

# Mobile BFF
mobile_bff:
  name: mobile-api
  type: service
  responsibility: Optimized APIs for mobile apps
  apis:
    - GET /mobile/home  # Aggregates multiple backend calls
    - GET /mobile/orders/{id}  # Returns mobile-optimized data
  aggregates:
    - customer-service
    - order-service
    - product-service
  features:
    - data_minimization: true  # Less data for mobile
    - offline_support: true
    - push_notifications: true

# Web BFF
web_bff:
  name: web-api
  type: service
  responsibility: Optimized APIs for web apps
  apis:
    - GET /web/dashboard  # Rich data for web
    - GET /web/orders  # Paginated results
  aggregates:
    - customer-service
    - order-service
    - product-service
    - analytics-service
  features:
    - server_side_rendering: true
    - real_time_updates: websocket
```

## Technology Stack Selection

### Decision Matrix

```yaml
# Service Implementation Technologies

use_cases:
  # High-Performance REST APIs
  rest_api_performance:
    options:
      - technology: Go
        pros: [fast, low_memory, good_concurrency]
        cons: [verbose, less_libraries]
        when: High throughput, low latency requirements

      - technology: Ballerina
        pros: [network_native, visual_dev, built_in_integrations]
        cons: [smaller_ecosystem]
        when: Integration-heavy services

      - technology: Node.js
        pros: [fast_development, large_ecosystem, async_io]
        cons: [single_threaded, callback_hell]
        when: I/O intensive, rapid development

  # Data Processing
  data_processing:
    options:
      - technology: Python
        pros: [ml_libraries, data_science_tools, readable]
        cons: [slower, gil_limitations]
        when: ML, data analysis, scripting

      - technology: Java
        pros: [mature, robust, enterprise_libraries]
        cons: [verbose, slower_startup]
        when: Complex business logic, enterprise integration

  # Real-Time Systems
  realtime:
    options:
      - technology: Ballerina + WebSocket
        pros: [native_websocket, async_support]
        when: Chat, notifications, live updates

      - technology: Go + gRPC
        pros: [efficient_streaming, low_latency]
        when: Service-to-service streaming

# Database Selection
databases:
  # Transactional Data
  transactional:
    options:
      - database: PostgreSQL
        when: Complex queries, ACID, relational data

      - database: MySQL
        when: Read-heavy, mature ecosystem

  # Document Storage
  document:
    options:
      - database: MongoDB
        when: Flexible schema, nested documents

      - database: CouchDB
        when: Offline-first, mobile sync

  # Cache
  cache:
    options:
      - database: Redis
        when: Session storage, rate limiting, queues

      - database: Memcached
        when: Simple key-value caching

  # Search
  search:
    options:
      - database: Elasticsearch
        when: Full-text search, analytics

      - database: Typesense
        when: Typo-tolerant search, smaller scale

  # Time-Series
  timeseries:
    options:
      - database: InfluxDB
        when: Metrics, IoT data

      - database: TimescaleDB
        when: Time-series + SQL
```

## Reference Architectures

### 1. E-Commerce Platform

```yaml
# Complete E-Commerce Architecture on OpenChoreo

system: ecommerce-platform

# Frontend Layer
frontends:
  - name: customer-web-app
    type: spa
    tech: React + Next.js
    deploys_to: openchoreo
    apis: web-bff

  - name: customer-mobile-app
    type: mobile
    tech: React Native
    apis: mobile-bff

  - name: admin-panel
    type: spa
    tech: React + TypeScript
    apis: admin-bff

# BFF Layer
bffs:
  - name: web-bff
    tech: Node.js + Express
    aggregates: [customer, order, product, cart]

  - name: mobile-bff
    tech: Go + Fiber
    aggregates: [customer, order, product, cart]

  - name: admin-bff
    tech: Ballerina
    aggregates: [customer, order, product, inventory, analytics]

# Service Layer
services:
  # Customer Domain
  - name: customer-service
    tech: Go
    database: postgres
    apis: [REST]
    cache: redis

  # Order Domain
  - name: order-service
    tech: Java + Spring Boot
    database: postgres
    apis: [REST]
    events: kafka

  - name: order-processor
    tech: Python
    triggers: kafka
    database: postgres

  # Product Domain
  - name: product-service
    tech: Node.js
    database: mongodb
    cache: redis
    apis: [REST, GraphQL]

  - name: product-search
    tech: Python + FastAPI
    database: elasticsearch

  # Cart Domain
  - name: cart-service
    tech: Go
    database: redis
    apis: [REST]

  # Payment Domain
  - name: payment-service
    tech: Java
    integrations: [stripe, paypal]
    events: kafka

  # Inventory Domain
  - name: inventory-service
    tech: Go
    database: postgres
    events: kafka

  # Shipping Domain
  - name: shipping-service
    tech: Python
    integrations: [fedex, ups, usps]
    events: kafka

  # Notification Domain
  - name: notification-service
    tech: Node.js
    integrations: [sendgrid, twilio, fcm]
    triggers: kafka

# Data Layer
data:
  - postgres-cluster:
      databases: [customers, orders, inventory]
      replicas: 3
      backup: daily

  - mongodb-cluster:
      databases: [products, reviews]
      replicas: 3
      sharding: enabled

  - redis-cluster:
      use: [cache, sessions, cart]
      replicas: 3

  - elasticsearch:
      indices: [products, orders]
      replicas: 2

  - kafka-cluster:
      topics: [orders, payments, inventory, notifications]
      partitions: 3
      replication: 3

# Integration Layer
integrations:
  - asgardeo:  # Identity
      features: [authentication, authorization, mfa]

  - api-manager:  # API Gateway
      features: [rate_limiting, analytics, monetization]

  - stripe:  # Payments
      apis: [payments, refunds, subscriptions]

  - sendgrid:  # Email
      apis: [transactional_email, marketing]

# Observability
observability:
  - prometheus:  # Metrics
      scrape_interval: 15s

  - jaeger:  # Tracing
      sampling: 0.1

  - loki:  # Logs
      retention: 30d

  - grafana:  # Dashboards
      dashboards: [services, infrastructure, business]
```

### 2. SaaS Multi-Tenant Platform

```yaml
# Multi-Tenant SaaS Architecture

system: saas-platform

# Tenancy Strategy
tenancy:
  model: shared-database-shared-schema
  isolation: row-level-security
  tenant_id: organization_id

# Service Layer
services:
  # Tenant Management
  - name: tenant-service
    responsibility: Tenant provisioning and management
    database: postgres
    features:
      - tenant_creation
      - subscription_management
      - feature_flags
      - usage_tracking

  # Core Application Services
  - name: api-service
    responsibility: Main application APIs
    database: postgres
    tenant_aware: true
    features:
      - multi_tenancy
      - tenant_isolation
      - resource_quotas

  # Analytics Service
  - name: analytics-service
    responsibility: Per-tenant analytics
    database: clickhouse
    tenant_aware: true

  # Billing Service
  - name: billing-service
    responsibility: Usage-based billing
    database: postgres
    integrations: [stripe]
    features:
      - metering
      - invoicing
      - subscriptions

# Data Isolation
data_isolation:
  strategy: shared-database
  tables:
    all_tables:
      columns: [organization_id]
      index: btree(organization_id)
      rls_policy: organization_id = current_setting('app.current_organization')::int

# Resource Quotas
quotas:
  by_tier:
    free:
      api_calls: 1000/day
      storage: 1GB
      users: 5

    pro:
      api_calls: 100000/day
      storage: 100GB
      users: 50

    enterprise:
      api_calls: unlimited
      storage: 1TB
      users: unlimited
```

## Best Practices

### 1. Service Design
- Single Responsibility Principle
- Design for failure
- API-first development
- Loose coupling
- High cohesion

### 2. Data Architecture
- Database per service
- Event sourcing for audit trails
- CQRS for read-heavy systems
- Cache strategically
- Plan for data migration

### 3. Scalability
- Horizontal scaling by default
- Stateless services
- Async processing for long operations
- Connection pooling
- Load balancing

### 4. Resilience
- Circuit breakers
- Retry with exponential backoff
- Timeouts on all external calls
- Graceful degradation
- Health checks

### 5. Security
- Zero trust architecture
- Service-to-service authentication
- API gateway for external access
- Encrypt data at rest and in transit
- Regular security audits

## Related Skills

- `openchoreo-app-developer` - Implementation of architectures
- `openchoreo-platform-engineer` - Platform configuration
- `openchoreo-devops` - Infrastructure setup
- `openchoreo-sre` - Operations and reliability

---

**Last Updated**: March 2025
**Skill Type**: Flexible - Adapt patterns to your requirements
