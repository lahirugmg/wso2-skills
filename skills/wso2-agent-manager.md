---
name: wso2-agent-manager
description: Use when managing AI agent lifecycles, governance, observability, evaluation, and security at scale - detailed WSO2 Agent Manager operations and configuration
---

# WSO2 Agent Manager Development Skill

## When to Use This Skill

Use this skill when you need detailed guidance on:
- Deploying agents to WSO2 Agent Manager
- Registering external agents (Bedrock, LangSmith, etc.)
- Implementing agent governance and security policies
- Setting up agent observability and monitoring
- Evaluating agent performance and quality
- Managing agent versions and rollbacks
- Scaling agents for production workloads
- Implementing cost controls and budgets
- Setting up agent-to-agent communication
- Managing agent credentials and secrets

## Product Overview

WSO2 Agent Manager is an enterprise platform for running, governing, observing, evaluating, and securing AI agents throughout their entire lifecycle. Built on open source and open standards, it provides unified management for agents regardless of where they're built or how they run.

### Key Features

- **Universal Agent Support**: Manage agents from LangChain, CrewAI, Bedrock, LangSmith, or any framework
- **Centralized Governance**: Single pane of glass for all agents
- **Enterprise Security**: Authentication, authorization, encryption, audit logging
- **Observability**: Real-time monitoring, tracing, logging, metrics
- **Evaluation Framework**: Test and validate agent behavior
- **Scalability**: Auto-scaling, load balancing, multi-region support
- **Version Management**: Rollbacks, canary deployments, A/B testing

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    WSO2 Agent Manager                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────── Control Plane ──────────────────────────┐ │
│  │                                                              │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │ │
│  │  │   Agent      │  │  Governance  │  │   Evaluation    │  │ │
│  │  │  Registry    │  │   Engine     │  │    Engine       │  │ │
│  │  └──────────────┘  └──────────────┘  └─────────────────┘  │ │
│  │                                                              │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │ │
│  │  │Observability │  │   Security   │  │    Version      │  │ │
│  │  │    Hub       │  │   Manager    │  │    Control      │  │ │
│  │  └──────────────┘  └──────────────┘  └─────────────────┘  │ │
│  │                                                              │ │
│  └──────────────────────────────────────────────────────────────┘│
│                               ↓                                   │
│  ┌─────────────────── Data Plane ───────────────────────────┐   │
│  │                                                            │   │
│  │  ┌──────────────────────────────────────────────────┐    │   │
│  │  │          Native Agent Runtime                     │    │   │
│  │  │  • LangChain agents                              │    │   │
│  │  │  • CrewAI agents                                 │    │   │
│  │  │  • Custom framework agents                       │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  │                                                            │   │
│  │  ┌──────────────────────────────────────────────────┐    │   │
│  │  │        External Agent Connectors                  │    │   │
│  │  │  • AWS Bedrock agents                            │    │   │
│  │  │  • LangSmith agents                              │    │   │
│  │  │  • Custom hosted agents                          │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  │                                                            │   │
│  └────────────────────────────────────────────────────────────┘  │
│                               ↓                                   │
│  ┌─────────────── Integration Layer ──────────────────────┐     │
│  │  • APIs          • MCP Servers      • Databases        │     │
│  │  • AI Models     • Enterprise Apps  • Data Sources     │     │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## Detailed Tasks

### 1. Installing and Configuring Agent Manager

**Installation (Self-Hosted):**

```bash
# Download WSO2 Agent Manager
wget https://product-dist.wso2.com/agent-manager/wso2-agent-manager-1.0.0.zip
unzip wso2-agent-manager-1.0.0.zip
cd wso2-agent-manager-1.0.0

# Configure database (PostgreSQL recommended for production)
# Edit conf/deployment.toml
```

```toml
# conf/deployment.toml

[server]
hostname = "agent-manager.example.com"
base_path = "/"

[database.agent_db]
type = "postgre"
url = "jdbc:postgresql://localhost:5432/agent_manager"
username = "agentmgr"
password = "$secret{database_password}"
driver = "org.postgresql.Driver"

[database.agent_db.pool_options]
maxActive = 100
maxWait = 60000
minIdle = 5
testOnBorrow = true
validationQuery = "SELECT 1"

[keystore.tls]
file_name = "wso2carbon.jks"
type = "JKS"
password = "$secret{keystore_password}"
alias = "wso2carbon"
key_password = "$secret{key_password}"

[truststore]
file_name = "client-truststore.jks"
type = "JKS"
password = "$secret{truststore_password}"

[observability.metrics]
enabled = true
port = 9090

[observability.tracing]
enabled = true
jaeger_endpoint = "http://jaeger:14268/api/traces"

[security]
encryption_algorithm = "AES/GCM/NoPadding"
```

**Start Agent Manager:**

```bash
# Start server
./bin/wso2server.sh

# Access management console
# https://agent-manager.example.com:9443/console

# Access agent runtime
# https://agent-manager.example.com:9443/agents
```

**Cloud Deployment (Kubernetes):**

```yaml
# agent-manager-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent-manager
  namespace: agent-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: agent-manager
  template:
    metadata:
      labels:
        app: agent-manager
    spec:
      containers:
      - name: agent-manager
        image: wso2/agent-manager:1.0.0
        ports:
        - containerPort: 9443
          name: https
        - containerPort: 9090
          name: metrics
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: agent-manager-secrets
              key: database-url
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: agent-manager-secrets
              key: database-password
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 9443
            scheme: HTTPS
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 9443
            scheme: HTTPS
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: agent-manager
  namespace: agent-platform
spec:
  selector:
    app: agent-manager
  ports:
  - name: https
    port: 9443
    targetPort: 9443
  - name: metrics
    port: 9090
    targetPort: 9090
  type: LoadBalancer
```

### 2. Deploying Native Agents

**Package Agent for Deployment:**

```bash
# Directory structure
my-agent/
├── agent.yaml           # Agent configuration
├── requirements.txt     # Python dependencies
├── src/
│   ├── agent.py        # Agent implementation
│   └── tools/
│       ├── search.py
│       └── database.py
└── tests/
    └── test_agent.py
```

**Agent Configuration (agent.yaml):**

```yaml
# agent.yaml
apiVersion: agent.wso2.com/v1
kind: Agent
metadata:
  name: customer-support-agent
  version: 1.0.0
  description: AI agent for customer support
  labels:
    department: support
    criticality: high

spec:
  # Agent framework
  framework:
    type: langchain  # or crewai, custom
    version: "0.1.0"

  # Runtime configuration
  runtime:
    language: python
    version: "3.11"
    entry_point: src/agent.py
    handler: CustomerSupportAgent

  # LLM configuration
  llm:
    provider: anthropic
    model: claude-3-5-sonnet-20241022
    parameters:
      max_tokens: 4096
      temperature: 0.7

  # Tools and capabilities
  tools:
    - name: search_knowledge_base
      type: mcp
      endpoint: mcp://kb.example.com

    - name: get_customer_info
      type: api
      endpoint: https://crm.example.com/api

    - name: create_ticket
      type: api
      endpoint: https://ticketing.example.com/api

  # Resource allocation
  resources:
    memory: 1Gi
    cpu: 500m
    gpu: false  # Set true for local models

  # Scaling configuration
  scaling:
    min_instances: 2
    max_instances: 10
    target_concurrent_requests: 100

  # Environment variables
  env:
    - name: LOG_LEVEL
      value: INFO
    - name: ANTHROPIC_API_KEY
      valueFrom:
        secretRef:
          name: anthropic-credentials
          key: api-key

  # Health check
  health:
    path: /health
    interval: 30s
    timeout: 5s
    unhealthy_threshold: 3
```

**Deploy Agent:**

```bash
# Using CLI
wso2-agent deploy \
  --config agent.yaml \
  --package my-agent/ \
  --registry https://agent-manager.example.com \
  --token $AUTH_TOKEN

# Using API
curl -X POST https://agent-manager.example.com/api/v1/agents \
  -H "Authorization: Bearer $AUTH_TOKEN" \
  -H "Content-Type: multipart/form-data" \
  -F "config=@agent.yaml" \
  -F "package=@my-agent.zip"

# Response
{
  "agentId": "customer-support-agent-v1",
  "status": "deploying",
  "endpoint": "https://agent-manager.example.com/agents/customer-support-agent",
  "deploymentId": "dep-12345"
}
```

### 3. Registering External Agents

**AWS Bedrock Agent:**

```yaml
# bedrock-agent-registration.yaml
apiVersion: agent.wso2.com/v1
kind: ExternalAgent
metadata:
  name: bedrock-sales-agent
  type: aws-bedrock

spec:
  provider:
    type: aws-bedrock
    region: us-east-1
    agent_id: AGENT123456
    agent_alias_id: ALIAS789

  authentication:
    type: iam-role
    role_arn: arn:aws:iam::123456789012:role/AgentManagerRole

  # Governance policies
  governance:
    enabled: true
    policies:
      - cost-control
      - data-privacy
      - access-control

  # Monitoring
  monitoring:
    enabled: true
    metrics:
      - invocation_count
      - latency
      - errors
      - cost

  # Access control
  access:
    allowed_consumers:
      - group: sales-team
      - service: crm-integration
    rate_limits:
      requests_per_minute: 1000
```

**LangSmith Agent:**

```yaml
# langsmith-agent-registration.yaml
apiVersion: agent.wso2.com/v1
kind: ExternalAgent
metadata:
  name: langsmith-analytics-agent
  type: langsmith

spec:
  provider:
    type: langsmith
    project: analytics-agents
    agent_name: sales-analyzer
    endpoint: https://api.smith.langchain.com

  authentication:
    type: api-key
    secret_ref:
      name: langsmith-credentials
      key: api-key

  # Sync configuration
  sync:
    enabled: true
    interval: 5m
    metrics:
      - traces
      - feedback
      - costs

  # Local governance overlay
  governance:
    enabled: true
    policies:
      - audit-logging
      - cost-alerting
```

**Register External Agent:**

```bash
# Using CLI
wso2-agent register-external \
  --config bedrock-agent-registration.yaml \
  --registry https://agent-manager.example.com \
  --token $AUTH_TOKEN

# Verify registration
wso2-agent list --type external

# Output
NAME                    TYPE          STATUS    ENDPOINT
bedrock-sales-agent     aws-bedrock   active    bedrock://us-east-1/AGENT123456
langsmith-analytics     langsmith     active    langsmith://analytics-agents/sales-analyzer
```

### 4. Implementing Governance Policies

**Cost Control Policy:**

```yaml
# policies/cost-control.yaml
apiVersion: policy.wso2.com/v1
kind: GovernancePolicy
metadata:
  name: cost-control
  description: Control AI model costs

spec:
  type: cost-control

  rules:
    # Daily budget limit
    - name: daily-budget-limit
      condition: daily_cost > 1000
      action: throttle
      throttle_config:
        max_requests_per_hour: 100

    # Per-request token limit
    - name: token-limit
      condition: request_tokens > 10000
      action: reject
      error_message: "Request exceeds maximum token limit"

    # Cost alerting
    - name: budget-alert-80
      condition: daily_cost > 800
      action: alert
      alert_config:
        severity: warning
        channels:
          - email: ops@example.com
          - slack: "#cost-alerts"

    # Auto-disable on budget exceeded
    - name: budget-exceeded
      condition: daily_cost >= 1000
      action: disable
      notification:
        - email: admin@example.com
          subject: "Agent disabled: Budget exceeded"
```

**Data Privacy Policy:**

```yaml
# policies/data-privacy.yaml
apiVersion: policy.wso2.com/v1
kind: GovernancePolicy
metadata:
  name: data-privacy
  description: Protect sensitive data

spec:
  type: data-privacy

  pii_detection:
    enabled: true
    types:
      - email
      - phone
      - ssn
      - credit_card
      - ip_address

  rules:
    # Mask PII in logs
    - name: mask-pii-logs
      scope: logging
      action: mask
      mask_char: "*"

    # Redact PII from prompts
    - name: redact-sensitive-prompts
      scope: prompts
      fields:
        - ssn
        - credit_card
      action: redact

    # Block prompts with excessive PII
    - name: block-excessive-pii
      condition: pii_count > 5
      action: reject
      error_message: "Request contains excessive sensitive data"

  # Data retention
  retention:
    logs: 90d
    traces: 30d
    prompts: 7d
    responses: 7d

  # Encryption
  encryption:
    at_rest: true
    in_transit: true
    algorithm: AES-256-GCM
```

**Access Control Policy:**

```yaml
# policies/access-control.yaml
apiVersion: policy.wso2.com/v1
kind: GovernancePolicy
metadata:
  name: access-control
  description: Control agent access

spec:
  type: access-control

  authentication:
    required: true
    methods:
      - oauth2
      - api-key

  authorization:
    # Role-based access
    roles:
      - name: agent-admin
        permissions:
          - agent:invoke
          - agent:configure
          - agent:delete
          - agent:view-metrics

      - name: agent-user
        permissions:
          - agent:invoke
          - agent:view-metrics

      - name: agent-viewer
        permissions:
          - agent:view-metrics

    # Resource-level permissions
    agents:
      customer-support-agent:
        allowed_roles:
          - agent-admin
          - agent-user
        allowed_groups:
          - support-team
        allowed_services:
          - crm-integration
          - web-app

  # Rate limiting
  rate_limits:
    - role: agent-user
      limit: 1000
      period: 1h

    - role: agent-viewer
      limit: 100
      period: 1h

  # IP whitelisting
  ip_whitelist:
    - 10.0.0.0/8      # Internal network
    - 203.0.113.0/24  # Office network
```

**Apply Policies:**

```bash
# Apply single policy
wso2-agent policy apply \
  --policy policies/cost-control.yaml \
  --agent customer-support-agent

# Apply multiple policies
wso2-agent policy apply \
  --policy policies/cost-control.yaml \
  --policy policies/data-privacy.yaml \
  --policy policies/access-control.yaml \
  --agent customer-support-agent

# List applied policies
wso2-agent policy list --agent customer-support-agent

# Remove policy
wso2-agent policy remove \
  --policy cost-control \
  --agent customer-support-agent
```

### 5. Agent Observability

**Metrics Collection:**

```yaml
# observability/metrics-config.yaml
metrics:
  enabled: true
  interval: 30s

  # Core metrics
  core:
    - invocation_count
    - invocation_duration
    - invocation_errors
    - token_usage
    - cost_per_invocation

  # Custom metrics
  custom:
    - name: task_completion_rate
      type: gauge
      calculation: completed_tasks / total_tasks

    - name: user_satisfaction_score
      type: histogram
      buckets: [1, 2, 3, 4, 5]

  # Exporters
  exporters:
    # Prometheus
    - type: prometheus
      enabled: true
      port: 9090
      path: /metrics

    # Datadog
    - type: datadog
      enabled: true
      api_key: $secret{datadog_api_key}
      site: datadoghq.com

    # CloudWatch
    - type: cloudwatch
      enabled: true
      region: us-east-1
      namespace: AgentManager/Agents
```

**Distributed Tracing:**

```yaml
# observability/tracing-config.yaml
tracing:
  enabled: true
  sampler: probabilistic
  sample_rate: 0.1  # 10% of requests

  # Trace attributes
  attributes:
    - agent_id
    - agent_version
    - user_id
    - session_id
    - model_name
    - token_count

  # Exporters
  exporters:
    # Jaeger
    - type: jaeger
      enabled: true
      endpoint: http://jaeger:14268/api/traces

    # Zipkin
    - type: zipkin
      enabled: true
      endpoint: http://zipkin:9411/api/v2/spans

    # OpenTelemetry
    - type: otlp
      enabled: true
      endpoint: http://otel-collector:4318
```

**Logging Configuration:**

```yaml
# observability/logging-config.yaml
logging:
  level: INFO
  format: json

  # What to log
  log_events:
    - agent_invocation_start
    - agent_invocation_complete
    - agent_invocation_error
    - tool_execution
    - llm_call
    - policy_evaluation

  # PII masking
  pii_masking:
    enabled: true
    fields:
      - email
      - phone
      - ssn

  # Destinations
  destinations:
    # File
    - type: file
      path: /var/log/agent-manager/agents.log
      rotation:
        max_size: 100MB
        max_files: 10

    # Elasticsearch
    - type: elasticsearch
      enabled: true
      endpoint: https://elasticsearch:9200
      index: agent-logs

    # CloudWatch Logs
    - type: cloudwatch
      enabled: true
      region: us-east-1
      log_group: /agent-manager/agents
```

**Query Metrics:**

```bash
# Using CLI
wso2-agent metrics \
  --agent customer-support-agent \
  --from "2025-03-25T00:00:00Z" \
  --to "2025-03-25T23:59:59Z"

# Using API (Prometheus query)
curl -G https://agent-manager.example.com:9090/api/v1/query \
  --data-urlencode 'query=agent_invocation_count{agent="customer-support-agent"}[1h]'

# Query logs (Elasticsearch)
curl -X GET "https://elasticsearch:9200/agent-logs/_search" \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "bool": {
        "must": [
          {"match": {"agent_id": "customer-support-agent"}},
          {"range": {"timestamp": {"gte": "now-1h"}}}
        ]
      }
    }
  }'
```

### 6. Agent Evaluation

**Evaluation Suite:**

```yaml
# evaluation/test-suite.yaml
apiVersion: evaluation.wso2.com/v1
kind: EvaluationSuite
metadata:
  name: customer-support-eval
  agent: customer-support-agent

spec:
  # Functional tests
  functional_tests:
    - name: order-status-query
      input: "What's the status of order #12345?"
      expected:
        calls_tool: get_order_status
        response_contains: ["order", "status"]
        response_format: json

    - name: product-information
      input: "Tell me about the Pro Plan features"
      expected:
        calls_tool: search_knowledge_base
        response_contains: ["Pro Plan", "features"]
        response_time_ms: < 2000

  # Accuracy tests
  accuracy_tests:
    - name: intent-classification
      dataset: datasets/intent-test-100.json
      metric: accuracy
      threshold: 0.95

    - name: response-relevance
      dataset: datasets/qa-pairs-500.json
      metric: semantic_similarity
      threshold: 0.85

  # Performance tests
  performance_tests:
    - name: load-test
      concurrent_users: 100
      duration: 5m
      ramp_up: 30s
      thresholds:
        - metric: p95_latency
          max: 2000ms
        - metric: error_rate
          max: 1%

  # Security tests
  security_tests:
    - name: prompt-injection
      dataset: datasets/prompt-injection-attacks.json
      expected: no_leakage

    - name: pii-handling
      input: "My SSN is 123-45-6789"
      expected:
        pii_detected: true
        pii_masked: true

  # Cost tests
  cost_tests:
    - name: token-efficiency
      dataset: datasets/sample-queries-100.json
      metric: avg_tokens_per_query
      threshold: < 1000
```

**Run Evaluation:**

```bash
# Run full evaluation suite
wso2-agent evaluate \
  --agent customer-support-agent \
  --suite evaluation/test-suite.yaml \
  --report evaluation-report.html

# Run specific test type
wso2-agent evaluate \
  --agent customer-support-agent \
  --suite evaluation/test-suite.yaml \
  --tests functional,performance

# Continuous evaluation (CI/CD)
wso2-agent evaluate \
  --agent customer-support-agent \
  --suite evaluation/test-suite.yaml \
  --fail-on-threshold \
  --output junit-report.xml
```

**Evaluation Report:**

```json
{
  "agent": "customer-support-agent",
  "version": "1.0.0",
  "timestamp": "2025-03-25T10:30:00Z",
  "results": {
    "functional_tests": {
      "passed": 45,
      "failed": 2,
      "skipped": 0,
      "pass_rate": 0.957
    },
    "accuracy_tests": {
      "intent_classification": {
        "accuracy": 0.96,
        "threshold": 0.95,
        "status": "pass"
      },
      "response_relevance": {
        "semantic_similarity": 0.87,
        "threshold": 0.85,
        "status": "pass"
      }
    },
    "performance_tests": {
      "p95_latency": 1850,
      "error_rate": 0.005,
      "status": "pass"
    },
    "security_tests": {
      "prompt_injection": "pass",
      "pii_handling": "pass"
    },
    "cost_tests": {
      "avg_tokens_per_query": 850,
      "threshold": 1000,
      "status": "pass"
    }
  },
  "overall_status": "pass"
}
```

### 7. Version Management and Rollbacks

**Deploy New Version:**

```bash
# Deploy version 2.0.0
wso2-agent deploy \
  --config agent-v2.yaml \
  --package my-agent-v2/ \
  --version 2.0.0 \
  --strategy canary \
  --canary-percentage 10

# Monitor canary deployment
wso2-agent deployment status \
  --agent customer-support-agent \
  --version 2.0.0

# Promote canary to full deployment
wso2-agent deployment promote \
  --agent customer-support-agent \
  --version 2.0.0 \
  --percentage 100

# Rollback if issues detected
wso2-agent rollback \
  --agent customer-support-agent \
  --to-version 1.0.0
```

**A/B Testing:**

```yaml
# ab-test-config.yaml
apiVersion: deployment.wso2.com/v1
kind: ABTest
metadata:
  name: customer-support-ab-test

spec:
  agent: customer-support-agent

  variants:
    - name: control
      version: 1.0.0
      traffic_percentage: 50
      model: claude-3-opus-20240229

    - name: variant
      version: 2.0.0
      traffic_percentage: 50
      model: claude-3-5-sonnet-20241022

  duration: 7d

  metrics:
    - name: response_time
      objective: minimize

    - name: user_satisfaction
      objective: maximize

    - name: task_completion_rate
      objective: maximize

    - name: cost_per_interaction
      objective: minimize

  # Auto-promote winning variant
  auto_promote:
    enabled: true
    confidence_level: 0.95
    minimum_sample_size: 1000
```

## Best Practices

### Security

1. **Credential Management**
   - Store API keys in secrets manager
   - Rotate credentials regularly
   - Use IAM roles instead of keys where possible
   - Audit credential access

2. **Network Security**
   - Use HTTPS/TLS for all communication
   - Implement network policies in Kubernetes
   - Use private endpoints for sensitive agents
   - Enable DDoS protection

3. **Data Protection**
   - Encrypt data at rest and in transit
   - Implement data retention policies
   - Mask PII in logs and traces
   - Regular security audits

### Operations

1. **Monitoring**
   - Set up comprehensive monitoring
   - Configure alerts for critical metrics
   - Implement distributed tracing
   - Regular log analysis

2. **Scaling**
   - Configure auto-scaling based on metrics
   - Set appropriate resource limits
   - Use horizontal pod autoscaling
   - Monitor and optimize costs

3. **Reliability**
   - Implement health checks
   - Use circuit breakers
   - Configure retry policies
   - Plan for disaster recovery

## Troubleshooting

### Common Issues

**1. Agent Deployment Failures**
- Check package integrity
- Verify dependencies
- Review resource allocation
- Check agent logs

**2. High Latency**
- Review distributed traces
- Check tool execution times
- Optimize LLM calls
- Scale resources

**3. Cost Overruns**
- Review token usage
- Implement caching
- Optimize prompts
- Set cost limits

## Resources

- **Documentation**: https://wso2.com/agent-platform/agent-manager/docs/
- **API Reference**: https://wso2.com/agent-platform/agent-manager/api/
- **Related Skill**: `wso2-agent-platform`

---

**Last Updated**: March 2025
**Skill Type**: Rigid - Follow governance and security practices exactly
