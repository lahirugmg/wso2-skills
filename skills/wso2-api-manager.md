---
name: wso2-api-manager
description: Use when working with WSO2 API Manager specifically - detailed operations for API design, publishing, gateway configuration, analytics, monetization, and AI/MCP features (v4.6.0+)
---

# WSO2 API Manager Development Skill

## When to Use This Skill

Use this skill for detailed WSO2 API Manager operations:
- Installing and configuring API Manager
- Creating and publishing APIs via Publisher Portal
- Managing API lifecycle states
- Configuring advanced gateway policies
- Setting up AI Gateway for multiple LLM providers
- Converting REST APIs to MCP servers
- Implementing API monetization
- Configuring analytics with Moesif
- Setting up federated gateways
- Implementing APIOps workflows
- Managing developer portal customization

## Product Overview

WSO2 API Manager 4.6.0+ is a complete platform for creating, managing, and securing APIs with built-in support for AI model management and MCP integration.

### Key Features (v4.6.0+)

- **Modular Architecture**: Independent scaling of components
- **AI Gateway**: Manage traffic to OpenAI, Anthropic, Azure AI, AWS Bedrock
- **MCP Support**: Convert REST APIs to MCP servers, proxy MCP tools
- **Federated Gateways**: Multi-gateway management
- **Moesif Analytics**: Advanced API analytics (default)
- **APIOps**: GitOps-based API lifecycle
- **Multi-Protocol**: REST, GraphQL, WebSocket, AsyncAPI, gRPC

## Installation and Setup

### Single-Node Deployment

```bash
# Download
wget https://github.com/wso2/product-apim/releases/download/v4.6.0/wso2am-4.6.0.zip
unzip wso2am-4.6.0.zip
cd wso2am-4.6.0

# Configure database (deployment.toml)
```

```toml
# repository/conf/deployment.toml

[server]
hostname = "api.example.com"
node_ip = "127.0.0.1"

[super_admin]
username = "admin"
password = "admin"
create_admin_account = true

[database.apim_db]
type = "mysql"
url = "jdbc:mysql://localhost:3306/apim_db"
username = "apimadmin"
password = "$secret{apim_db_password}"
driver = "com.mysql.jdbc.Driver"

[database.shared_db]
type = "mysql"
url = "jdbc:mysql://localhost:3306/shared_db"
username = "sharedadmin"
password = "$secret{shared_db_password}"
driver = "com.mysql.jdbc.Driver"

[apim.key_manager]
service_url = "https://localhost:9443/services/"
type = "default"

[apim.gateway]
environment = "Production"

[apim.analytics]
enable = true
type = "moesif"

[apim.analytics.moesif]
application_id = "$secret{moesif_app_id}"
```

```bash
# Start API Manager
./bin/api-manager.sh

# Access portals
# Publisher: https://api.example.com:9443/publisher
# Developer Portal: https://api.example.com:9443/devportal
# Admin Portal: https://api.example.com:9443/admin
# Carbon Console: https://api.example.com:9443/carbon
```

### Distributed Deployment (Production)

```
┌─────────────────────────────────────────┐
│          Load Balancer (Nginx)          │
└─────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
┌───────▼──────┐      ┌─────────▼────────┐
│  Gateway 1   │      │   Gateway 2      │
│  (Worker)    │      │   (Worker)       │
└──────────────┘      └──────────────────┘
        │                       │
        └───────────┬───────────┘
                    │
        ┌───────────▼───────────┐
        │   Control Plane       │
        │  (Publisher, Portal,  │
        │   Key Manager, TM)    │
        └───────────────────────┘
                    │
        ┌───────────▼───────────┐
        │      Database         │
        │   (MySQL/PostgreSQL)  │
        └───────────────────────┘
```

## Detailed Tasks

### 1. Creating APIs with Publisher Portal

**REST API from OpenAPI:**

```bash
# Using Publisher UI
1. Navigate to https://api.example.com:9443/publisher
2. Sign in (admin/admin)
3. Click "Create API" → "Import OpenAPI"
4. Provide OpenAPI definition:
   - URL: https://petstore3.swagger.io/api/v3/openapi.json
   - Or upload file
5. Review and configure:
   - Name: Petstore API
   - Context: /petstore
   - Version: 1.0.0
   - Endpoint: https://petstore3.swagger.io/api/v3
6. Click "Create"
```

**GraphQL API:**

```graphql
# schema.graphql
type Query {
  customers: [Customer]
  customer(id: ID!): Customer
}

type Customer {
  id: ID!
  name: String!
  email: String!
  orders: [Order]
}

type Order {
  id: ID!
  total: Float!
  items: [OrderItem]
}

type OrderItem {
  product: String!
  quantity: Int!
  price: Float!
}
```

```bash
# Create GraphQL API
1. Click "Create API" → "Import GraphQL Schema"
2. Upload schema.graphql
3. Configure:
   - Name: Customer GraphQL API
   - Context: /customers/graphql
   - Version: 1.0.0
   - Endpoint: https://backend.example.com/graphql
4. Click "Create"
```

**WebSocket API:**

```bash
# Create WebSocket API
1. Click "Create API" → "Design a New WebSocket API"
2. Configure:
   - Name: Chat WebSocket API
   - Context: /chat
   - Version: 1.0.0
   - Endpoint: ws://chat-backend.example.com:8080
3. Define topics:
   - /messages (pub/sub)
   - /notifications (pub only)
4. Click "Create"
```

### 2. Configuring API Policies

**Rate Limiting (Advanced Throttling):**

```xml
<!-- Custom throttling policy -->
<policy policyName="CustomRateLimit" description="Custom rate limiting">
    <defaultLimit>
        <count>1000</count>
        <timeUnit>min</timeUnit>
    </defaultLimit>

    <!-- Conditional limits -->
    <conditionalGroups>
        <conditionalGroup>
            <description>Premium users get higher limits</description>
            <condition>
                <headerCondition>
                    <headerName>X-User-Tier</headerName>
                    <headerValue>premium</headerValue>
                </headerCondition>
            </condition>
            <limit>
                <count>10000</count>
                <timeUnit>min</timeUnit>
            </limit>
        </conditionalGroup>

        <conditionalGroup>
            <description>Limit by IP for public APIs</description>
            <condition>
                <ipCondition>
                    <specificIP>192.168.1.0/24</specificIP>
                </ipCondition>
            </condition>
            <limit>
                <count>100</count>
                <timeUnit>min</timeUnit>
            </limit>
        </conditionalGroup>
    </conditionalGroups>
</policy>
```

**Request/Response Mediation:**

```xml
<!-- In sequence - Request transformation -->
<sequence xmlns="http://ws.apache.org/ns/synapse" name="CustomerAPIInFlow">
    <!-- Add correlation ID -->
    <property name="CORRELATION_ID" expression="get-property('MessageID')" scope="default"/>
    <header name="X-Correlation-ID" expression="get-property('CORRELATION_ID')" scope="transport"/>

    <!-- Validate request -->
    <filter source="get-property('REST_METHOD')" regex="POST|PUT">
        <then>
            <validate cache-schema="true">
                <schema key="conf:schema/customer-schema.json"/>
                <on-fail>
                    <payloadFactory media-type="json">
                        <format>{"error": "Invalid request format"}</format>
                    </payloadFactory>
                    <property name="HTTP_SC" value="400" scope="axis2"/>
                    <respond/>
                </on-fail>
            </validate>
        </then>
    </filter>

    <!-- Add authentication header to backend -->
    <property name="BACKEND_API_KEY" expression="$ctx:api.ut.backendKey" type="STRING"/>
    <header name="X-API-Key" expression="get-property('BACKEND_API_KEY')" scope="transport"/>
</sequence>

<!-- Out sequence - Response transformation -->
<sequence xmlns="http://ws.apache.org/ns/synapse" name="CustomerAPIOutFlow">
    <!-- Add response headers -->
    <header name="X-Correlation-ID" expression="get-property('CORRELATION_ID')" scope="transport"/>
    <header name="X-Response-Time" expression="get-property('axis2', 'BACKEND_LATENCY')" scope="transport"/>

    <!-- Transform response -->
    <filter source="get-property('axis2', 'HTTP_SC')" regex="200">
        <then>
            <payloadFactory media-type="json">
                <format>
                {
                    "status": "success",
                    "data": $1,
                    "metadata": {
                        "correlationId": "$2",
                        "timestamp": "$3"
                    }
                }
                </format>
                <args>
                    <arg evaluator="xml" expression="$body"/>
                    <arg evaluator="xml" expression="get-property('CORRELATION_ID')"/>
                    <arg evaluator="xml" expression="get-property('SYSTEM_TIME')"/>
                </args>
            </payloadFactory>
        </then>
    </filter>
</sequence>
```

**CORS Configuration:**

```xml
<!-- cors_config.xml -->
<CORSConfiguration>
    <Access-Control-Allow-Origin>https://app.example.com</Access-Control-Allow-Origin>
    <Access-Control-Allow-Methods>GET,POST,PUT,DELETE,OPTIONS</Access-Control-Allow-Methods>
    <Access-Control-Allow-Headers>authorization,Access-Control-Allow-Origin,Content-Type,X-API-Key</Access-Control-Allow-Headers>
    <Access-Control-Allow-Credentials>true</Access-Control-Allow-Credentials>
</CORSConfiguration>
```

### 3. AI Gateway Configuration

**Multi-Provider Setup:**

```toml
# deployment.toml - AI Gateway configuration

[[apim.ai_gateway.providers]]
name = "anthropic"
type = "anthropic"
endpoint = "https://api.anthropic.com/v1"
api_key = "$secret{anthropic_api_key}"
models = ["claude-3-5-sonnet-20241022", "claude-3-opus-20240229"]
rate_limit = "1000/min"
token_limit = "100000/min"

[[apim.ai_gateway.providers]]
name = "openai"
type = "openai"
endpoint = "https://api.openai.com/v1"
api_key = "$secret{openai_api_key}"
models = ["gpt-4", "gpt-3.5-turbo"]
rate_limit = "5000/min"

[[apim.ai_gateway.providers]]
name = "azure-openai"
type = "azure"
endpoint = "https://example.openai.azure.com"
api_key = "$secret{azure_openai_key}"
deployment_id = "gpt-4-deployment"

[[apim.ai_gateway.providers]]
name = "bedrock"
type = "aws-bedrock"
region = "us-east-1"
access_key_id = "$secret{aws_access_key}"
secret_access_key = "$secret{aws_secret_key}"
models = ["anthropic.claude-v2"]

[apim.ai_gateway.routing]
strategy = "cost-optimized"  # or latency-optimized, round-robin
enable_failover = true
enable_load_balancing = true
```

**Create AI API:**

```bash
# Via Publisher Portal
1. Create API → Design AI API
2. Configure:
   - Name: Enterprise AI API
   - Context: /ai/chat
   - Version: 1.0.0
   - AI Provider: anthropic
   - Model: claude-3-5-sonnet-20241022
3. Set rate limits:
   - Gold: 10000/day
   - Silver: 1000/day
   - Bronze: 100/day
4. Enable cost tracking
5. Publish
```

**AI API with Smart Routing:**

```json
{
  "api": "Enterprise AI API",
  "routing_rules": [
    {
      "name": "high-priority",
      "condition": "request.headers['X-Priority'] == 'high'",
      "provider": "anthropic",
      "model": "claude-3-5-sonnet-20241022"
    },
    {
      "name": "cost-sensitive",
      "condition": "request.headers['X-Cost-Tier'] == 'budget'",
      "provider": "openai",
      "model": "gpt-3.5-turbo"
    },
    {
      "name": "default",
      "provider": "anthropic",
      "model": "claude-3-5-sonnet-20241022",
      "fallback": {
        "provider": "openai",
        "model": "gpt-4"
      }
    }
  ]
}
```

### 4. MCP Integration

**Convert REST API to MCP Server:**

```bash
# Via Publisher Portal
1. Select existing REST API
2. Click "Convert to MCP"
3. Review tool mappings:
   - GET /customers → list_customers tool
   - GET /customers/{id} → get_customer tool
   - POST /customers → create_customer tool
4. Configure tool descriptions and parameters
5. Publish MCP server
6. Get MCP endpoint: mcp://api.example.com/customers
```

**MCP Tool Definition (Generated):**

```json
{
  "name": "customer-management",
  "version": "1.0.0",
  "description": "MCP server for customer management",
  "tools": [
    {
      "name": "list_customers",
      "description": "Retrieve list of customers",
      "inputSchema": {
        "type": "object",
        "properties": {
          "limit": {
            "type": "integer",
            "description": "Maximum customers to return"
          },
          "offset": {
            "type": "integer",
            "description": "Pagination offset"
          }
        }
      }
    },
    {
      "name": "create_customer",
      "description": "Create a new customer",
      "inputSchema": {
        "type": "object",
        "required": ["name", "email"],
        "properties": {
          "name": {"type": "string"},
          "email": {"type": "string", "format": "email"}
        }
      }
    }
  ]
}
```

**MCP Proxy Configuration:**

```toml
# Expose MCP tools via AI Gateway

[[apim.mcp.servers]]
name = "internal-tools"
endpoint = "mcp://tools.internal.example.com"
auth_type = "oauth2"
client_id = "$secret{mcp_client_id}"
client_secret = "$secret{mcp_client_secret}"

[apim.mcp.authorization]
enable = true
policy_endpoint = "https://is.example.com/api/mcp-authz/v1"

[[apim.mcp.servers.tools]]
name = "search_knowledge_base"
allowed_agents = ["customer-support-agent"]
rate_limit = "1000/hour"

[[apim.mcp.servers.tools]]
name = "create_ticket"
allowed_agents = ["customer-support-agent", "operations-agent"]
rate_limit = "200/hour"
```

### 5. Analytics and Monitoring

**Moesif Configuration:**

```toml
# deployment.toml

[apim.analytics.moesif]
application_id = "$secret{moesif_app_id}"

# Data capture settings
capture_request_body = true
capture_response_body = true
capture_request_headers = ["Authorization", "X-API-Key", "User-Agent"]
capture_response_headers = ["X-RateLimit-Remaining", "X-Response-Time"]

# User identification
identify_user_from = "token"  # Extract from JWT
user_id_claim = "sub"

# Company identification
identify_company_from = "header"
company_id_header = "X-Organization-ID"

# Sampling (for high-volume APIs)
sampling_percentage = 100  # 100% for all APIs
```

**Custom Dashboards:**

```javascript
// Moesif Dashboard Configuration
{
  "name": "API Performance Overview",
  "charts": [
    {
      "title": "Requests Over Time",
      "type": "timeseries",
      "metric": "request_count",
      "group_by": "api_name"
    },
    {
      "title": "P95 Latency by API",
      "type": "bar",
      "metric": "response_time",
      "aggregation": "p95",
      "group_by": "api_name"
    },
    {
      "title": "Error Rate",
      "type": "timeseries",
      "metric": "error_rate",
      "filter": "status_code >= 400"
    },
    {
      "title": "AI API Costs",
      "type": "timeseries",
      "metric": "ai_cost_usd",
      "group_by": ["provider", "model"]
    },
    {
      "title": "Top Consumers",
      "type": "table",
      "metrics": ["request_count", "total_cost"],
      "group_by": "user_id",
      "limit": 10
    }
  ]
}
```

### 6. API Monetization

**Subscription Plans:**

```bash
# Via Admin Portal
1. Navigate to https://api.example.com:9443/admin
2. Go to "Monetization" → "Subscription Plans"
3. Create plan:
```

```json
{
  "name": "Professional Plan",
  "description": "For growing businesses",
  "billing_cycle": "monthly",
  "price": 99.00,
  "currency": "USD",
  "quotas": {
    "requests_per_month": 1000000,
    "rate_limit": "1000/minute"
  },
  "apis": [
    "Customer API v1.0.0",
    "Order API v1.0.0"
  ],
  "features": [
    "Email support",
    "99.9% SLA",
    "Analytics dashboard"
  ],
  "billing_provider": "stripe",
  "trial_period_days": 14
}
```

**Usage-Based Pricing (AI APIs):**

```json
{
  "name": "AI API Pay-Per-Token",
  "billing_model": "usage-based",
  "rates": [
    {
      "tier": "input_tokens",
      "price_per_1k": 0.003,
      "currency": "USD"
    },
    {
      "tier": "output_tokens",
      "price_per_1k": 0.015,
      "currency": "USD"
    }
  ],
  "volume_discounts": [
    {
      "min_tokens": 1000000,
      "discount_percentage": 10
    },
    {
      "min_tokens": 10000000,
      "discount_percentage": 20
    }
  ]
}
```

### 7. APIOps Workflow

**Git Repository Structure:**

```
api-definitions/
├── apis/
│   ├── customer-api/
│   │   ├── api.yaml
│   │   ├── swagger.yaml
│   │   └── policies/
│   │       ├── rate-limit.xml
│   │       └── mediation.xml
│   ├── order-api/
│   │   └── api.yaml
│   └── product-api/
│       └── api.yaml
├── .github/
│   └── workflows/
│       └── api-deployment.yml
└── README.md
```

**API Definition (api.yaml):**

```yaml
# apis/customer-api/api.yaml
id: customer-api-v1
name: Customer API
version: 1.0.0
context: /customers
type: REST
lifeCycleStatus: PUBLISHED

provider: admin
visibility: PUBLIC

endpointConfig:
  production:
    endpoint: https://backend.example.com/api/customers
    type: http
  sandbox:
    endpoint: https://sandbox.backend.example.com/api/customers
    type: http

transport:
  - https

tier: Unlimited

policies:
  - type: RateLimiting
    tier: Gold
  - type: Authentication
    oauth2: true

cors:
  enabled: true
  allowOrigins:
    - "*"
  allowMethods:
    - GET
    - POST
    - PUT
    - DELETE

operations:
  - target: /
    verb: GET
    throttlingPolicy: Unlimited
  - target: /
    verb: POST
    throttlingPolicy: 10KPerMin
  - target: /{id}
    verb: GET
    throttlingPolicy: Unlimited
  - target: /{id}
    verb: PUT
    throttlingPolicy: 10KPerMin
  - target: /{id}
    verb: DELETE
    throttlingPolicy: 1KPerMin
```

**CI/CD Pipeline:**

```yaml
# .github/workflows/api-deployment.yml
name: API Deployment

on:
  push:
    branches:
      - main
    paths:
      - 'apis/**'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup apictl
        run: |
          wget https://github.com/wso2/product-apim-tooling/releases/download/v4.3.0/apictl-4.3.0-linux-x64.tar.gz
          tar -xvf apictl-4.3.0-linux-x64.tar.gz
          sudo mv apictl /usr/local/bin/

      - name: Configure environment
        run: |
          apictl add env production \
            --apim https://api.example.com \
            --token ${{ secrets.APIM_ACCESS_TOKEN }}

      - name: Import APIs
        run: |
          for api_dir in apis/*/; do
            apictl import api \
              --file "$api_dir" \
              --environment production \
              --preserve-provider=false \
              --update \
              --skip-cleanup
          done

      - name: Publish APIs
        run: |
          # Parse API definitions and publish
          for api_file in apis/*/api.yaml; do
            api_name=$(yq e '.name' "$api_file")
            api_version=$(yq e '.version' "$api_file")

            apictl change-status api \
              --name "$api_name" \
              --version "$api_version" \
              --provider admin \
              --environment production \
              --action Publish
          done
```

## Best Practices

### API Design
- Use resource-based URLs
- Implement proper versioning
- Provide comprehensive documentation
- Use standard HTTP status codes
- Implement pagination for lists

### Security
- Always use HTTPS in production
- Implement OAuth2/OpenID Connect
- Use API keys for service-to-service
- Rate limit all APIs
- Validate all inputs

### Performance
- Enable response caching
- Implement connection pooling
- Use async processing for long operations
- Monitor and optimize gateway performance
- Set appropriate timeouts

## Troubleshooting

**Gateway Timeout:**
- Increase backend timeout in API definition
- Check backend service health
- Review gateway logs
- Scale gateway horizontally

**Rate Limit Errors:**
- Review throttling policies
- Check subscriber tier limits
- Implement retry with backoff
- Upgrade subscription if needed

**Authentication Failures:**
- Verify OAuth2 configuration
- Check token expiration
- Review Key Manager connectivity
- Validate scopes and permissions

## Resources

- **Documentation**: https://apim.docs.wso2.com/
- **API Controller**: https://apim.docs.wso2.com/en/latest/install-and-setup/setup/api-controller/
- **Related Skill**: `wso2-api-platform`

---

**Last Updated**: March 2025
**Skill Type**: Flexible - Adapt to API requirements
