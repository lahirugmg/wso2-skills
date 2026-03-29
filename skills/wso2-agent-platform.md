---
name: wso2-agent-platform
description: Use when building, deploying, or managing AI agents, RAG systems, or AI workflows using WSO2's Agent Platform - covers agent development, governance, security, and lifecycle management
---

# WSO2 Agent Platform Development Skill

## When to Use This Skill

Use this skill when:
- Building AI agents using LangChain, CrewAI, or any AI framework
- Developing RAG (Retrieval-Augmented Generation) systems
- Creating AI workflows that integrate with enterprise systems
- Managing and governing AI agents at scale
- Securing and monitoring AI agent deployments
- Integrating agents with APIs, databases, and MCP servers
- Deploying agents to production environments

## Product Overview

The WSO2 Agent Platform is an enterprise-grade solution for building, running, and governing AI agents at scale. It combines:

- **WSO2 Agent Manager**: Run, govern, observe, evaluate, and secure AI agents throughout their lifecycle
- **WSO2 Integrator**: Build AI agents using low-code or pro-code with extensive connectors
- **Flexible Runtime**: Support for any AI framework (LangChain, CrewAI, custom frameworks)
- **Multi-Cloud Support**: Deploy on-premises, private cloud, public cloud, or fully managed SaaS

### Key Capabilities (2025)

- **Framework Agnostic**: Deploy agents written in any framework to WSO2 Agent Manager
- **Unified Management**: Register and manage agents from Bedrock, LangSmith, and custom runtimes
- **MCP Integration**: Seamless integration with Model Context Protocol servers
- **AI-Native Development**: Low-code visual development with AI assistance
- **Enterprise Security**: Built-in governance, access control, and compliance features
- **Scalability**: Run agents at enterprise scale with monitoring and optimization

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   WSO2 Agent Platform                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │          WSO2 Agent Manager                       │  │
│  │  • Agent Registry & Lifecycle                     │  │
│  │  • Governance & Security                          │  │
│  │  • Observability & Monitoring                     │  │
│  │  • Evaluation & Testing                           │  │
│  └──────────────────────────────────────────────────┘  │
│                          ↕                              │
│  ┌──────────────────────────────────────────────────┐  │
│  │          WSO2 Integrator (Agent Builder)          │  │
│  │  • Business Integrator (BI 1.4.0+)                │  │
│  │  • Micro Integrator (MI 4.5.0+)                   │  │
│  │  • Connector Ecosystem (400+ connectors)          │  │
│  │  • AI Agent Module                                │  │
│  └──────────────────────────────────────────────────┘  │
│                          ↕                              │
│  ┌──────────────────────────────────────────────────┐  │
│  │          Integration Layer                        │  │
│  │  • APIs & REST Services                           │  │
│  │  • MCP Servers                                    │  │
│  │  • Databases & Data Sources                       │  │
│  │  • Enterprise Applications                        │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Common Tasks

### 1. Building an AI Agent

**Using WSO2 Integrator's AI Agent Module:**

```yaml
# Prerequisites
- Install WSO2 Business Integrator or Micro Integrator
- Configure AI model access (OpenAI, Azure AI, AWS Bedrock, Anthropic)
- Set up data sources and APIs for agent context

# Development Steps
1. Launch VS Code with WSO2 MI/BI extension
2. Create new integration project
3. Add AI Agent mediator
4. Configure agent:
   - Select AI framework (LangChain, CrewAI, custom)
   - Define agent role and capabilities
   - Configure tools and data sources
   - Set up RAG if needed
5. Add connectors for enterprise integration
6. Test agent locally
7. Package for deployment
```

**RAG System Development:**

```python
# Example: RAG configuration in WSO2 Integrator
agent_config = {
    "type": "rag",
    "model": "claude-3-5-sonnet-20241022",
    "vectorStore": {
        "type": "wso2-vector-db",
        "endpoint": "https://vector-db.example.com"
    },
    "retrieval": {
        "topK": 5,
        "similarityThreshold": 0.8
    },
    "dataSources": [
        "api://internal-docs",
        "db://knowledge-base",
        "mcp://tools-server"
    ]
}
```

### 2. Deploying to WSO2 Agent Manager

**Deployment Workflow:**

```bash
# 1. Package your agent
wso2-agent package --name customer-support-agent --version 1.0.0

# 2. Deploy to Agent Manager
wso2-agent deploy \
  --registry https://agent-manager.example.com \
  --auth-token $AUTH_TOKEN \
  --package customer-support-agent-1.0.0.car

# 3. Configure governance policies
wso2-agent policy apply \
  --agent customer-support-agent \
  --policy security-baseline.yaml

# 4. Enable monitoring
wso2-agent monitor enable \
  --agent customer-support-agent \
  --metrics all
```

**Agent Registration (External Agents):**

```yaml
# Register agents from other platforms
agent_registration:
  name: "bedrock-sales-agent"
  type: "external"
  runtime: "aws-bedrock"
  endpoint: "https://bedrock.us-east-1.amazonaws.com/agent/AGENT_ID"
  credentials:
    type: "iam-role"
    role_arn: "arn:aws:iam::ACCOUNT:role/AgentRole"
  governance:
    enabled: true
    policies:
      - "data-privacy"
      - "cost-control"
```

### 3. Agent Governance and Security

**Access Control:**

```yaml
# Agent access policy
access_policy:
  agent: "customer-support-agent"
  rules:
    - subject: "group:support-team"
      permissions: ["invoke", "monitor"]
    - subject: "group:admins"
      permissions: ["invoke", "monitor", "configure", "delete"]
    - subject: "service:api-gateway"
      permissions: ["invoke"]
      conditions:
        - rate_limit: "1000/hour"
```

**Data Privacy Controls:**

```yaml
# PII handling configuration
privacy_policy:
  agent: "customer-support-agent"
  pii_detection: enabled
  actions:
    - type: "mask"
      fields: ["email", "phone", "ssn"]
    - type: "audit"
      log_level: "detailed"
    - type: "retention"
      duration: "90days"
```

### 4. Monitoring and Observability

**Metrics to Track:**

- **Performance Metrics:**
  - Response time
  - Token usage
  - Throughput (requests/second)
  - Error rates

- **Business Metrics:**
  - Task completion rate
  - User satisfaction scores
  - Cost per interaction
  - SLA compliance

- **Security Metrics:**
  - Failed authentication attempts
  - Policy violations
  - Data access patterns
  - Anomaly detection alerts

**Monitoring Setup:**

```yaml
# Observability configuration
observability:
  agent: "customer-support-agent"
  metrics:
    enabled: true
    interval: "30s"
    exporters:
      - type: "prometheus"
        endpoint: "http://prometheus:9090"
      - type: "datadog"
        api_key: "${DATADOG_API_KEY}"

  logging:
    level: "INFO"
    format: "json"
    destinations:
      - type: "elasticsearch"
        endpoint: "https://elk.example.com"

  tracing:
    enabled: true
    sampler: "probabilistic"
    sample_rate: 0.1
    exporter:
      type: "jaeger"
      endpoint: "http://jaeger:14268"
```

### 5. Agent Evaluation and Testing

**Test Framework:**

```yaml
# Agent test suite
tests:
  - name: "functional_tests"
    type: "integration"
    scenarios:
      - name: "customer_query_handling"
        input: "What's the status of order #12345?"
        expected_behavior:
          - calls_api: "order-service"
          - returns_format: "json"
          - contains_fields: ["order_id", "status", "eta"]

      - name: "error_handling"
        input: "Invalid request data"
        expected_behavior:
          - returns_error: true
          - error_message_contains: "Invalid input"

  - name: "performance_tests"
    type: "load"
    config:
      concurrent_users: 100
      duration: "5m"
      ramp_up: "30s"
    thresholds:
      - metric: "response_time_p95"
        max: "2000ms"
      - metric: "error_rate"
        max: "1%"

  - name: "security_tests"
    type: "security"
    checks:
      - "sql_injection_resistance"
      - "prompt_injection_resistance"
      - "data_leakage_prevention"
```

## Best Practices

### Agent Development

1. **Separation of Concerns**
   - Keep business logic separate from AI model interaction
   - Use configuration files for agent behavior
   - Implement clear interfaces between components

2. **Error Handling**
   - Always handle API failures gracefully
   - Implement retry logic with exponential backoff
   - Provide fallback responses for critical scenarios
   - Log errors with sufficient context for debugging

3. **Context Management**
   - Limit context window size to prevent token overflow
   - Implement context pruning strategies
   - Cache frequently accessed data
   - Use RAG for large knowledge bases

4. **Testing Strategy**
   - Write unit tests for individual components
   - Create integration tests for end-to-end flows
   - Perform load testing before production deployment
   - Implement continuous testing in CI/CD pipeline

### Security Best Practices

1. **Authentication & Authorization**
   - Use WSO2 Identity Server or Asgardeo for agent authentication
   - Implement role-based access control (RBAC)
   - Use session-based control for agent interactions
   - Rotate credentials regularly

2. **Data Protection**
   - Encrypt sensitive data at rest and in transit
   - Implement PII detection and masking
   - Follow data retention policies
   - Audit all data access

3. **Prompt Security**
   - Validate and sanitize all user inputs
   - Implement prompt injection detection
   - Use system prompts to enforce behavior boundaries
   - Monitor for unusual prompt patterns

### Performance Optimization

1. **Caching Strategy**
   - Cache API responses when appropriate
   - Implement vector database caching for RAG
   - Use CDN for static content
   - Cache embeddings for frequently accessed documents

2. **Scaling Considerations**
   - Use horizontal scaling for agent instances
   - Implement load balancing
   - Use async processing for non-critical tasks
   - Monitor and auto-scale based on metrics

## Integration with Other WSO2 Products

### With WSO2 API Manager

```yaml
# Expose agent as an API
api_config:
  name: "Customer Support Agent API"
  version: "1.0.0"
  context: "/agents/customer-support"
  backend:
    type: "wso2-agent"
    agent_id: "customer-support-agent"
  policies:
    - rate_limiting: "100/minute"
    - authentication: "oauth2"
    - analytics: "enabled"
```

### With WSO2 Identity Server

```yaml
# Agent identity configuration
agent_identity:
  provider: "wso2-identity-server"
  endpoint: "https://is.example.com"
  application:
    name: "customer-support-agent"
    client_id: "${CLIENT_ID}"
    client_secret: "${CLIENT_SECRET}"
  scopes:
    - "agent:invoke"
    - "data:read"
    - "api:call"
```

### With Choreo

```yaml
# Deploy agent on Choreo platform
choreo_deployment:
  project: "ai-agents"
  component:
    name: "customer-support-agent"
    type: "service"
    runtime: "ballerina"
  environment:
    - name: "AGENT_MODEL"
      value: "claude-3-5-sonnet"
    - name: "VECTOR_DB_URL"
      secret: "vector-db-credentials"
  scaling:
    min_instances: 2
    max_instances: 10
    target_cpu: 70
```

## Troubleshooting

### Common Issues

**1. Agent Response Latency**
- **Symptom**: Slow agent responses (>5s)
- **Causes**: Large context, slow API calls, inefficient RAG
- **Solutions**:
  - Reduce context window size
  - Implement parallel API calls
  - Optimize vector search (reduce topK)
  - Use faster embedding models
  - Enable response streaming

**2. Token Limit Exceeded**
- **Symptom**: Errors about context length
- **Causes**: Too much context, long conversation history
- **Solutions**:
  - Implement context pruning
  - Use summarization for long conversations
  - Switch to models with larger context windows
  - Use RAG instead of full context inclusion

**3. Agent Hallucinations**
- **Symptom**: Agent provides incorrect or fabricated information
- **Causes**: Insufficient context, poor prompts, no grounding
- **Solutions**:
  - Implement RAG with verified data sources
  - Improve system prompts with clear instructions
  - Add fact-checking layer
  - Enable citation/source attribution
  - Use higher temperature for creative tasks, lower for factual

**4. Integration Failures**
- **Symptom**: Agent cannot access required APIs/data
- **Causes**: Auth issues, network problems, config errors
- **Solutions**:
  - Check credential configuration
  - Verify network connectivity
  - Review connector configurations
  - Check API endpoint availability
  - Enable detailed logging

**5. High Costs**
- **Symptom**: Unexpected AI model costs
- **Causes**: Inefficient prompts, unnecessary API calls, poor caching
- **Solutions**:
  - Optimize prompts to reduce token usage
  - Implement aggressive caching
  - Use smaller models where appropriate
  - Set cost limits and alerts
  - Monitor token usage per interaction

## Resources

### Official Documentation
- **Agent Platform**: https://wso2.com/agent-platform/
- **Agent Manager**: https://wso2.com/agent-platform/agent-manager/
- **WSO2 Integrator**: https://wso2.com/integrator/
- **Micro Integrator Docs**: https://mi.docs.wso2.com/
- **Business Integrator**: https://wso2.com/integrator/mi/

### Related Skills
- `wso2-agent-manager` - Detailed Agent Manager operations
- `wso2-micro-integrator` - Deep dive into MI for agent building
- `wso2-business-integrator` - Low-code agent development
- `wso2-api-platform` - Exposing agents as APIs
- `wso2-identity-platform` - Agent authentication and security

### API References
- Agent Manager API: https://wso2.com/agent-platform/docs/api
- Integrator REST API: https://mi.docs.wso2.com/en/latest/develop/using-rest-apis/
- Management API: https://wso2.com/docs/agent-manager/management-api

### Community Resources
- WSO2 Discord: https://discord.gg/wso2
- Stack Overflow: [wso2-agent-platform]
- GitHub Issues: https://github.com/wso2/

## AI/MCP Integration (2025 Features)

### Model Context Protocol Support

```yaml
# MCP server integration
mcp_integration:
  agent: "customer-support-agent"
  mcp_servers:
    - name: "internal-tools"
      endpoint: "mcp://tools.example.com"
      auth:
        type: "oauth2"
        credentials_ref: "mcp-tools-creds"
      tools:
        - "search_knowledge_base"
        - "create_ticket"
        - "escalate_to_human"

    - name: "crm-system"
      endpoint: "mcp://crm.example.com"
      tools:
        - "get_customer_info"
        - "update_customer_record"
```

### AI Gateway Integration

```yaml
# Multi-provider AI routing
ai_gateway:
  providers:
    - name: "anthropic"
      models: ["claude-3-5-sonnet-20241022"]
      endpoint: "https://api.anthropic.com"
      priority: 1

    - name: "openai"
      models: ["gpt-4"]
      endpoint: "https://api.openai.com"
      priority: 2
      fallback: true

  routing:
    strategy: "cost-optimized"
    failover: "automatic"
    load_balancing: "round-robin"
```

### Agentic Workflows

```yaml
# Multi-agent collaboration
workflow:
  name: "customer_onboarding"
  agents:
    - name: "document-processor"
      role: "Extract and validate customer documents"
      input: ["uploaded_files"]
      output: ["customer_data"]

    - name: "risk-assessor"
      role: "Evaluate customer risk profile"
      input: ["customer_data"]
      output: ["risk_score"]

    - name: "account-creator"
      role: "Create customer account"
      input: ["customer_data", "risk_score"]
      output: ["account_id"]

  orchestration:
    type: "sequential"
    error_handling: "rollback"
```

## Deployment Patterns

### Pattern 1: Centralized Agent Platform
- All agents deployed to WSO2 Agent Manager
- Centralized governance and monitoring
- Suitable for: Enterprise-wide deployment

### Pattern 2: Federated Agent Deployment
- Agents deployed across multiple regions/clouds
- Central governance via Agent Manager
- Suitable for: Multi-cloud, global deployments

### Pattern 3: Hybrid Approach
- Critical agents on WSO2 Agent Manager
- Lightweight agents on edge devices
- Suitable for: IoT, edge computing scenarios

### Pattern 4: SaaS-First
- Deploy on Choreo (WSO2's IDP)
- Managed infrastructure
- Suitable for: Fast time-to-market, small teams

## Version Compatibility

- **WSO2 Agent Manager**: Latest version (2025+)
- **WSO2 Integrator BI**: 1.4.0+
- **WSO2 Integrator MI**: 4.5.0+
- **WSO2 API Manager**: 4.6.0+ (for API exposure)
- **WSO2 Identity Server**: 7.1+ (for authentication)
- **Choreo**: Latest (for managed deployment)

---

**Last Updated**: March 2025
**Skill Type**: Flexible - Adapt principles to your specific use case
**Related Platforms**: Agent, API, Integration, Identity, Engineering
