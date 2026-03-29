---
name: wso2-api-platform
description: Use when designing, creating, managing, securing, or monitoring APIs - covers API lifecycle management, gateway configuration, analytics, monetization, and AI/MCP API integration
---

# WSO2 API Platform Development Skill

## When to Use This Skill

Use this skill when:
- Designing and creating REST, GraphQL, WebSocket, or AsyncAPI APIs
- Managing API lifecycle (design, publish, deprecate, retire)
- Configuring API gateway, policies, and security
- Setting up API rate limiting, throttling, and quotas
- Implementing API analytics and monitoring
- Creating API monetization strategies
- Exposing AI models as APIs
- Managing Model Context Protocol (MCP) servers as APIs
- Federating API gateways across multiple environments
- Implementing APIOps and API-first development

## Product Overview

The WSO2 API Platform provides comprehensive API management capabilities for building, integrating, and exposing digital services as managed APIs in cloud, on-premise, and hybrid architectures.

### Core Components

- **API Publisher Portal**: Design, develop, and apply policies to APIs
- **Developer Portal**: Hub for discovering and subscribing to APIs
- **API Gateway**: Runtime for secure API traffic management
- **Key Manager**: Security validation and API key generation
- **Analytics & Monetization**: Usage tracking and billing
- **API Controller**: CLI for automation and CI/CD

### Key Capabilities (2025)

- **AI API Management**: Manage traffic to OpenAI, Azure AI, AWS Bedrock, Anthropic
- **MCP Support**: Convert REST APIs to MCP servers, expose MCP tools via AI Gateway
- **Modular Architecture**: Independent component scaling and updates
- **Federated Gateways**: Manage multiple gateway deployments from single control plane
- **APIOps**: Git-based API lifecycle automation
- **AI-Powered Analytics**: Moesif integration for deep insights
- **Multi-Protocol Support**: REST, GraphQL, WebSocket, AsyncAPI, gRPC

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    WSO2 API Platform                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────── API Control Plane ─────────────────────┐  │
│  │                                                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │  Publisher   │  │  Developer   │  │   Service    │   │  │
│  │  │   Portal     │  │   Portal     │  │   Catalog    │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │                                                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │ Key Manager  │  │  Analytics   │  │API Controller│   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │                                                            │  │
│  └────────────────────────────────────────────────────────────┘│
│                              ↓                                   │
│  ┌────────────────── API Gateway (Data Plane) ──────────────┐  │
│  │                                                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   Standard   │  │  AI Gateway  │  │ MCP Gateway  │   │  │
│  │  │   Gateway    │  │              │  │              │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │                                                            │  │
│  │  Policies: Rate Limiting, Security, Caching, Transform   │  │
│  │                                                            │  │
│  └────────────────────────────────────────────────────────────┘│
│                              ↓                                   │
│  ┌────────────────── Backend Services ──────────────────────┐  │
│  │  • REST APIs     • AI Models      • MCP Servers          │  │
│  │  • GraphQL       • Microservices  • Legacy Systems       │  │
│  └────────────────────────────────────────────────────────────┘│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Common Tasks

### 1. Creating an API

**REST API Creation:**

```yaml
# API Definition (OpenAPI 3.0)
openapi: 3.0.0
info:
  title: Customer API
  version: 1.0.0
  description: Customer management API
servers:
  - url: https://api.example.com/v1
paths:
  /customers:
    get:
      summary: List all customers
      operationId: listCustomers
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Customer'
    post:
      summary: Create a customer
      operationId: createCustomer
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CustomerInput'
      responses:
        '201':
          description: Customer created
components:
  schemas:
    Customer:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
    CustomerInput:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
        email:
          type: string
```

**Create API via Publisher Portal:**

1. Navigate to Publisher Portal: `https://apim.example.com/publisher`
2. Click "Create API" → "Import OpenAPI"
3. Upload OpenAPI definition or provide URL
4. Configure:
   - API Name: "Customer API"
   - Context: `/customers`
   - Version: `1.0.0`
   - Endpoint: `https://backend.example.com/api/customers`
5. Design API:
   - Review resources and operations
   - Add descriptions and examples
   - Configure request/response schemas
6. Configure policies (next section)
7. Publish API

**Create API via CLI (APIOps):**

```bash
# Initialize API project
apictl init customer-api --oas openapi.yaml

# Configure environment
apictl add env production \
  --apim https://apim.example.com \
  --token $ACCESS_TOKEN

# Import API
apictl import api \
  --file customer-api \
  --environment production \
  --preserve-provider=false \
  --update

# Publish API
apictl change-status api \
  --name CustomerAPI \
  --version 1.0.0 \
  --provider admin \
  --environment production \
  --action Publish
```

### 2. Applying API Policies

**Rate Limiting:**

```xml
<!-- Application-level throttling -->
<throttling>
  <policy name="Gold">
    <description>Gold tier - 1000 requests per minute</description>
    <condition>
      <limit>1000</limit>
      <time>1</time>
      <unit>minute</unit>
    </condition>
  </policy>

  <policy name="Silver">
    <description>Silver tier - 500 requests per minute</description>
    <condition>
      <limit>500</limit>
      <time>1</time>
      <unit>minute</unit>
    </condition>
  </policy>
</throttling>
```

**Advanced Throttling (Dynamic):**

```json
{
  "policyName": "DynamicRateLimit",
  "description": "Rate limit based on user tier",
  "defaultLimit": {
    "requestCount": {
      "value": 100,
      "unit": "minute"
    }
  },
  "conditionalGroups": [
    {
      "conditions": [
        {
          "type": "HeaderCondition",
          "header": "X-User-Tier",
          "value": "premium"
        }
      ],
      "limit": {
        "requestCount": {
          "value": 10000,
          "unit": "minute"
        }
      }
    }
  ]
}
```

**Security Policies:**

```yaml
# OAuth2 Authentication
security:
  - oauth2:
      type: "oauth"
      provider: "wso2is"
      endpoint: "https://is.example.com/oauth2/token"
      scopes:
        - "customer:read"
        - "customer:write"
      audience: "https://api.example.com"

# API Key Authentication
  - apiKey:
      type: "api_key"
      in: "header"
      name: "X-API-Key"

# Mutual TLS
  - mtls:
      type: "mutual_tls"
      client_certificates:
        - issuer: "CN=TrustedCA"
          serial: "12345"
```

**Request/Response Transformation:**

```xml
<!-- Transform request before backend -->
<sequence name="requestTransform">
  <header name="X-Request-ID" value="get-property('MessageID')" scope="transport"/>
  <property name="ORIGINAL_PAYLOAD" value="get-property('payload')" type="STRING"/>

  <payloadFactory media-type="json">
    <format>
      {
        "timestamp": "$1",
        "data": $2
      }
    </format>
    <args>
      <arg evaluator="xml" expression="get-property('SYSTEM_TIME')"/>
      <arg evaluator="xml" expression="get-property('ORIGINAL_PAYLOAD')"/>
    </args>
  </payloadFactory>
</sequence>

<!-- Transform response before client -->
<sequence name="responseTransform">
  <property name="RESPONSE_PAYLOAD" value="get-property('payload')" type="STRING"/>

  <filter xpath="get-property('transport', 'HTTP_SC') = '200'">
    <then>
      <payloadFactory media-type="json">
        <format>
          {
            "status": "success",
            "data": $1,
            "metadata": {
              "timestamp": "$2",
              "version": "1.0.0"
            }
          }
        </format>
        <args>
          <arg evaluator="xml" expression="get-property('RESPONSE_PAYLOAD')"/>
          <arg evaluator="xml" expression="get-property('SYSTEM_TIME')"/>
        </args>
      </payloadFactory>
    </then>
  </filter>
</sequence>
```

### 3. Configuring AI Gateway

**AI Provider Configuration:**

```yaml
# AI Gateway configuration for multiple providers
ai_gateway:
  name: "Enterprise AI Gateway"
  version: "4.6.0"

  providers:
    # Anthropic Claude
    - name: "anthropic"
      type: "anthropic"
      endpoint: "https://api.anthropic.com/v1"
      authentication:
        type: "api_key"
        key_ref: "anthropic-api-key"
      models:
        - "claude-3-5-sonnet-20241022"
        - "claude-3-opus-20240229"
      rate_limits:
        requests_per_minute: 1000
        tokens_per_minute: 100000

    # OpenAI
    - name: "openai"
      type: "openai"
      endpoint: "https://api.openai.com/v1"
      authentication:
        type: "api_key"
        key_ref: "openai-api-key"
      models:
        - "gpt-4"
        - "gpt-3.5-turbo"
      rate_limits:
        requests_per_minute: 5000
        tokens_per_minute: 500000

    # Azure OpenAI
    - name: "azure-openai"
      type: "azure"
      endpoint: "https://example.openai.azure.com"
      authentication:
        type: "api_key"
        key_ref: "azure-openai-key"
      deployment_id: "gpt-4-deployment"
      models:
        - "gpt-4"

    # AWS Bedrock
    - name: "aws-bedrock"
      type: "bedrock"
      region: "us-east-1"
      authentication:
        type: "aws_iam"
        role_arn: "arn:aws:iam::ACCOUNT:role/BedrockRole"
      models:
        - "anthropic.claude-v2"
        - "anthropic.claude-instant-v1"

  # Routing configuration
  routing:
    strategy: "cost-optimized"  # or "latency-optimized", "round-robin"

    rules:
      - name: "high-priority-claude"
        condition: "request.headers['X-Priority'] == 'high'"
        target: "anthropic"
        model: "claude-3-5-sonnet-20241022"

      - name: "cost-sensitive-gpt"
        condition: "request.headers['X-Cost-Tier'] == 'budget'"
        target: "openai"
        model: "gpt-3.5-turbo"

      - name: "default"
        target: "anthropic"
        model: "claude-3-5-sonnet-20241022"
        fallback:
          - target: "openai"
            model: "gpt-4"

  # Load balancing
  load_balancing:
    enabled: true
    algorithm: "weighted_round_robin"
    weights:
      anthropic: 60
      openai: 30
      azure-openai: 10

  # Failover
  failover:
    enabled: true
    max_retries: 3
    retry_delay: "1s"
    backoff: "exponential"
```

**Expose AI Model as API:**

```yaml
# Create API for Claude
api:
  name: "Claude AI API"
  version: "1.0.0"
  context: "/ai/claude"
  type: "HTTP"

  backend:
    type: "ai_gateway"
    provider: "anthropic"
    model: "claude-3-5-sonnet-20241022"

  resources:
    - path: "/chat"
      methods: ["POST"]
      operation: "chat_completion"

    - path: "/embed"
      methods: ["POST"]
      operation: "embeddings"

  policies:
    authentication:
      - oauth2:
          scopes: ["ai:chat"]

    rate_limiting:
      - tier: "gold"
        limit: "10000/day"
      - tier: "silver"
        limit: "1000/day"

    monitoring:
      - token_usage: true
      - cost_tracking: true
      - latency_tracking: true

    cost_control:
      - max_tokens_per_request: 4096
      - daily_budget: 1000  # USD
      - alert_threshold: 80  # percent
```

### 4. MCP Server Integration

**Convert REST API to MCP Server:**

```yaml
# MCP conversion configuration
mcp_conversion:
  source_api: "customer-api-v1"
  mcp_server:
    name: "Customer Management MCP"
    description: "MCP server for customer operations"
    version: "1.0.0"

  tool_mapping:
    - api_operation: "GET /customers"
      mcp_tool:
        name: "list_customers"
        description: "Retrieve list of customers"
        parameters:
          - name: "limit"
            type: "integer"
            description: "Maximum number of customers to return"
            required: false
          - name: "offset"
            type: "integer"
            description: "Pagination offset"
            required: false

    - api_operation: "POST /customers"
      mcp_tool:
        name: "create_customer"
        description: "Create a new customer"
        parameters:
          - name: "name"
            type: "string"
            description: "Customer name"
            required: true
          - name: "email"
            type: "string"
            description: "Customer email"
            required: true

    - api_operation: "GET /customers/{id}"
      mcp_tool:
        name: "get_customer"
        description: "Get customer by ID"
        parameters:
          - name: "id"
            type: "string"
            description: "Customer ID"
            required: true
```

**Expose MCP Tools via AI Gateway:**

```yaml
# MCP proxy configuration
mcp_proxy:
  name: "Enterprise MCP Gateway"

  servers:
    - name: "internal-tools"
      endpoint: "mcp://tools.internal.example.com"
      authentication:
        type: "oauth2"
        token_endpoint: "https://is.example.com/oauth2/token"
        client_id: "${MCP_CLIENT_ID}"
        client_secret: "${MCP_CLIENT_SECRET}"

      tools:
        - "search_knowledge_base"
        - "create_ticket"
        - "get_customer_info"

      authorization:
        # Control which AI agents can access tools
        rules:
          - agent: "customer-support-agent"
            allowed_tools: ["search_knowledge_base", "get_customer_info"]

          - agent: "operations-agent"
            allowed_tools: ["create_ticket"]

      rate_limits:
        calls_per_minute: 1000
        calls_per_day: 100000

  # Expose MCP tools to AI agents
  exposure:
    protocol: "mcp"
    endpoint: "mcp://gateway.example.com"
    discovery: "enabled"  # Allow agents to discover available tools
```

### 5. API Analytics and Monitoring

**Analytics Configuration:**

```yaml
# Moesif Analytics Integration (Default in 4.6.0+)
analytics:
  provider: "moesif"

  configuration:
    application_id: "${MOESIF_APP_ID}"

    # Data collection
    capture:
      request_body: true
      response_body: true
      request_headers:
        - "Authorization"
        - "X-API-Key"
        - "User-Agent"
      response_headers:
        - "X-RateLimit-Remaining"
        - "X-Response-Time"

    # User identification
    identify_user:
      from: "token"  # Extract from JWT or API key
      claim: "sub"   # JWT claim for user ID

    # Company identification
    identify_company:
      from: "header"
      header: "X-Organization-ID"

    # Sampling (for high-volume APIs)
    sampling:
      rate: 100  # 100% for critical APIs
      rules:
        - path: "/health"
          rate: 1  # Sample 1% of health checks

    # Event filtering
    filters:
      - skip_path: "/health"
      - skip_path: "/metrics"
      - skip_status_codes: [301, 302]

  # Custom dashboards
  dashboards:
    - name: "API Performance"
      metrics:
        - "response_time_p95"
        - "error_rate"
        - "throughput"

    - name: "Usage by Customer"
      dimensions:
        - "user_id"
        - "organization_id"
      metrics:
        - "request_count"
        - "unique_users"

    - name: "AI API Costs"
      metrics:
        - "token_usage"
        - "cost_per_request"
        - "daily_spend"
```

**Real-Time Monitoring:**

```yaml
# Prometheus metrics export
monitoring:
  prometheus:
    enabled: true
    port: 9090
    path: /metrics

    metrics:
      - name: "api_requests_total"
        type: "counter"
        help: "Total API requests"
        labels:
          - api_name
          - api_version
          - method
          - status_code

      - name: "api_request_duration_seconds"
        type: "histogram"
        help: "API request duration"
        buckets: [0.1, 0.5, 1, 2, 5, 10]
        labels:
          - api_name
          - method

      - name: "api_gateway_token_usage"
        type: "counter"
        help: "AI API token usage"
        labels:
          - provider
          - model

      - name: "api_cost_usd"
        type: "counter"
        help: "API costs in USD"
        labels:
          - provider
          - model

  # Alerting
  alerts:
    - name: "HighErrorRate"
      condition: "error_rate > 5%"
      duration: "5m"
      severity: "critical"
      notifications:
        - type: "email"
          to: "ops@example.com"
        - type: "slack"
          channel: "#api-alerts"

    - name: "HighLatency"
      condition: "p95_latency > 2s"
      duration: "10m"
      severity: "warning"
      notifications:
        - type: "pagerduty"
          service_key: "${PAGERDUTY_KEY}"

    - name: "BudgetExceeded"
      condition: "daily_ai_cost > 1000"
      severity: "critical"
      actions:
        - "disable_ai_apis"
        - "notify_admin"
```

### 6. API Monetization

**Subscription Plans:**

```yaml
# Monetization configuration
monetization:
  enabled: true
  currency: "USD"

  billing_provider: "stripe"
  stripe:
    api_key: "${STRIPE_API_KEY}"
    webhook_secret: "${STRIPE_WEBHOOK_SECRET}"

  plans:
    - name: "Free"
      description: "Free tier for developers"
      price: 0
      billing_cycle: "monthly"
      quotas:
        requests_per_month: 10000
        rate_limit: "100/minute"
      features:
        - "Basic APIs"
        - "Community support"

    - name: "Professional"
      description: "For growing businesses"
      price: 99
      billing_cycle: "monthly"
      quotas:
        requests_per_month: 1000000
        rate_limit: "1000/minute"
      features:
        - "All APIs"
        - "Email support"
        - "99.9% SLA"

    - name: "Enterprise"
      description: "For large organizations"
      price: 999
      billing_cycle: "monthly"
      quotas:
        requests_per_month: -1  # Unlimited
        rate_limit: "10000/minute"
      features:
        - "All APIs"
        - "Priority support"
        - "99.99% SLA"
        - "Dedicated account manager"

  # Usage-based pricing (for AI APIs)
  usage_pricing:
    - api: "claude-ai-api"
      model: "pay-per-token"
      rates:
        - tier: "input_tokens"
          price: 0.003  # per 1K tokens
        - tier: "output_tokens"
          price: 0.015  # per 1K tokens

      volume_discounts:
        - min_tokens: 1000000
          discount: 10  # percent
        - min_tokens: 10000000
          discount: 20  # percent
```

## Best Practices

### API Design

1. **RESTful Principles**
   - Use nouns for resources, not verbs
   - Use HTTP methods correctly (GET, POST, PUT, DELETE)
   - Use proper HTTP status codes
   - Implement HATEOAS where appropriate

2. **Versioning**
   - Use URL versioning: `/v1/customers`
   - Document deprecation timeline
   - Maintain backwards compatibility within major version
   - Provide migration guides for breaking changes

3. **Documentation**
   - Use OpenAPI/Swagger for REST APIs
   - Provide code examples in multiple languages
   - Document authentication requirements
   - Include example requests and responses
   - Keep documentation in sync with implementation

4. **Error Handling**
   - Use consistent error response format
   - Provide meaningful error messages
   - Include error codes for programmatic handling
   - Don't expose sensitive information in errors

### Security

1. **Authentication & Authorization**
   - Always use HTTPS in production
   - Implement OAuth2/OpenID Connect for user APIs
   - Use API keys for service-to-service communication
   - Implement least privilege access
   - Rotate credentials regularly

2. **Input Validation**
   - Validate all input data
   - Sanitize data to prevent injection attacks
   - Implement request size limits
   - Use schema validation (JSON Schema)

3. **Rate Limiting**
   - Implement rate limiting at multiple levels (application, user, IP)
   - Provide clear rate limit headers
   - Implement exponential backoff for retries
   - Monitor for abuse patterns

### Performance

1. **Caching**
   - Implement response caching where appropriate
   - Use ETags for conditional requests
   - Cache at multiple levels (gateway, CDN, client)
   - Set appropriate cache headers

2. **Pagination**
   - Always paginate list endpoints
   - Use cursor-based pagination for large datasets
   - Provide links to next/previous pages
   - Document maximum page size

3. **Compression**
   - Enable gzip/brotli compression
   - Compress responses > 1KB
   - Support Accept-Encoding header

## Integration with Other WSO2 Products

### With WSO2 Identity Server

```yaml
# API security with WSO2 IS
api_security:
  identity_provider: "wso2-identity-server"
  endpoint: "https://is.example.com"

  oauth2:
    authorization_endpoint: "/oauth2/authorize"
    token_endpoint: "/oauth2/token"
    introspection_endpoint: "/oauth2/introspect"

  applications:
    - name: "customer-api"
      client_id: "${CLIENT_ID}"
      client_secret: "${CLIENT_SECRET}"
      grant_types:
        - "client_credentials"
        - "authorization_code"
      scopes:
        - "customer:read"
        - "customer:write"
```

### With Choreo

```yaml
# Deploy API on Choreo
choreo_deployment:
  project: "enterprise-apis"
  component:
    name: "customer-api"
    type: "rest-api"

  backend:
    type: "ballerina-service"
    source: "github.com/example/customer-service"

  environment:
    - name: "DATABASE_URL"
      secret: "database-credentials"

  gateway:
    domain: "api.example.com"
    rate_limits:
      gold: "10000/hour"
      silver: "1000/hour"
```

## Troubleshooting

### Common Issues

**1. Gateway Timeout**
- **Symptom**: 504 Gateway Timeout errors
- **Causes**: Slow backend, network issues, insufficient resources
- **Solutions**:
  - Increase gateway timeout settings
  - Optimize backend performance
  - Implement caching
  - Scale gateway horizontally
  - Use async processing for long operations

**2. Rate Limit Errors**
- **Symptom**: 429 Too Many Requests
- **Causes**: Exceeded quota, aggressive client polling
- **Solutions**:
  - Review rate limit configuration
  - Implement client-side backoff
  - Use webhooks instead of polling
  - Upgrade to higher tier if legitimate traffic

**3. Authentication Failures**
- **Symptom**: 401 Unauthorized or 403 Forbidden
- **Causes**: Invalid credentials, expired tokens, insufficient permissions
- **Solutions**:
  - Verify credentials are correct
  - Check token expiration
  - Review scope/permission configuration
  - Check Key Manager connectivity

**4. High Latency**
- **Symptom**: Slow API responses (>2s)
- **Causes**: Slow backend, no caching, network latency
- **Solutions**:
  - Enable response caching
  - Optimize backend queries
  - Use CDN for static content
  - Implement connection pooling
  - Monitor database performance

## Resources

### Official Documentation
- **API Platform**: https://wso2.com/api-platform/
- **API Manager**: https://apim.docs.wso2.com/
- **API Controller**: https://apim.docs.wso2.com/en/latest/install-and-setup/setup/api-controller/
- **REST API Reference**: https://apim.docs.wso2.com/en/latest/develop/product-apis/

### Related Skills
- `wso2-api-manager` - Detailed API Manager operations
- `wso2-identity-platform` - API security and authentication
- `wso2-agent-platform` - Exposing agents as APIs
- `wso2-engineering-platform` - API development with Choreo

### Tools
- **API Controller (apictl)**: CLI for API management
- **Postman**: API testing and documentation
- **Swagger Editor**: OpenAPI design
- **Moesif**: Analytics platform (integrated)

## Version Compatibility

- **WSO2 API Manager**: 4.6.0+ (recommended for AI/MCP features)
- **API Controller**: 4.3.0+
- **WSO2 Identity Server**: 7.0+ (for OAuth2/OIDC)
- **Choreo**: Latest (for managed deployment)

---

**Last Updated**: March 2025
**Skill Type**: Flexible - Adapt to your API requirements
**Related Platforms**: API, Identity, Agent, Engineering
