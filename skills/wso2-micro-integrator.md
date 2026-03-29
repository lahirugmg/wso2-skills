---
name: wso2-micro-integrator
description: Use when developing with WSO2 Micro Integrator - detailed ESB patterns, connector usage, mediator configuration, AI agent module, and cloud-native deployment (v4.5.0+)
---

# WSO2 Micro Integrator Development Skill

## When to Use This Skill

Use for detailed Micro Integrator operations:
- Developing integration flows with ESB patterns
- Using connectors (400+ available)
- Configuring mediators and sequences
- Building AI agents with integration
- Deploying to Kubernetes
- Performance tuning and optimization
- Troubleshooting integration issues

## Product Overview

WSO2 Micro Integrator 4.5.0+ is a lightweight ESB runtime for cloud-native deployments with AI-native capabilities.

## Key Features

- **100% Open Source**: Full ESB capabilities
- **Cloud-Native**: Kubernetes-ready, containerized
- **AI Agent Module**: Build AI agents with RAG
- **Low-Code**: Visual development with AI copilot
- **400+ Connectors**: Pre-built integrations
- **Zero-Friction Setup**: One-click VS Code installation
- **ESB Patterns**: All enterprise integration patterns

## Installation

```bash
# Download
wget https://product-dist.wso2.com/products/micro-integrator/4.5.0/wso2mi-4.5.0.zip
unzip wso2mi-4.5.0.zip

# VS Code Extension (Recommended)
1. Open VS Code
2. Install "WSO2 Micro Integrator" extension
3. Extension auto-configures Java, Maven, runtime

# Start MI
./bin/micro-integrator.sh
```

## Quick Start

**Create Integration Project:**

```bash
# Using VS Code Command Palette
Ctrl/Cmd + Shift + P → "WSO2: Create New Integration Project"

# Or using Maven
mvn archetype:generate \
  -DarchetypeGroupId=org.wso2.ei \
  -DarchetypeArtifactId=integration-project \
  -DarchetypeVersion=4.5.0
```

**Simple REST API Integration:**

```xml
<api context="/hello" name="HelloAPI" xmlns="http://ws.apache.org/ns/synapse">
    <resource methods="GET">
        <inSequence>
            <payloadFactory media-type="json">
                <format>{"message": "Hello World"}</format>
            </payloadFactory>
            <respond/>
        </inSequence>
    </resource>
</api>
```

## Common Patterns

### 1. Content-Based Routing

```xml
<api context="/orders" name="OrderRouter">
    <resource methods="POST" uri-template="/process">
        <inSequence>
            <!-- Route based on order type -->
            <switch source="json-eval($.orderType)">
                <case regex="express">
                    <call>
                        <endpoint key="ExpressOrderEndpoint"/>
                    </call>
                </case>
                <case regex="standard">
                    <call>
                        <endpoint key="StandardOrderEndpoint"/>
                    </call>
                </case>
                <default>
                    <payloadFactory media-type="json">
                        <format>{"error": "Unknown order type"}</format>
                    </payloadFactory>
                    <property name="HTTP_SC" value="400" scope="axis2"/>
                </default>
            </switch>
            <respond/>
        </inSequence>
    </resource>
</api>
```

### 2. Scatter-Gather Pattern

```xml
<sequence name="ScatterGatherExample">
    <!-- Call multiple services in parallel -->
    <clone continueParent="true">
        <target>
            <sequence>
                <call>
                    <endpoint key="InventoryService"/>
                </call>
            </sequence>
        </target>
        <target>
            <sequence>
                <call>
                    <endpoint key="PricingService"/>
                </call>
            </sequence>
        </target>
        <target>
            <sequence>
                <call>
                    <endpoint key="RecommendationService"/>
                </call>
            </sequence>
        </target>
    </clone>

    <!-- Aggregate responses -->
    <aggregate>
        <completeCondition>
            <messageCount>3</messageCount>
        </completeCondition>
        <onComplete>
            <payloadFactory media-type="json">
                <format>
                {
                    "inventory": $1,
                    "pricing": $2,
                    "recommendations": $3
                }
                </format>
                <args>
                    <arg evaluator="xml" expression="//inventory"/>
                    <arg evaluator="xml" expression="//pricing"/>
                    <arg evaluator="xml" expression="//recommendations"/>
                </args>
            </payloadFactory>
        </onComplete>
    </aggregate>
</sequence>
```

### 3. Message Transformation

```xml
<sequence name="TransformCustomerData">
    <!-- JSON to JSON transformation -->
    <payloadFactory media-type="json">
        <format>
        {
            "customerId": "$1",
            "profile": {
                "fullName": "$2",
                "contact": {
                    "email": "$3",
                    "phone": "$4"
                }
            },
            "metadata": {
                "createdAt": "$5",
                "source": "api"
            }
        }
        </format>
        <args>
            <arg evaluator="xml" expression="json-eval($.id)"/>
            <arg evaluator="xml" expression="json-eval($.name)"/>
            <arg evaluator="xml" expression="json-eval($.email)"/>
            <arg evaluator="xml" expression="json-eval($.phone)"/>
            <arg evaluator="xml" expression="get-property('SYSTEM_TIME')"/>
        </args>
    </payloadFactory>
</sequence>
```

## Connector Usage

**Database Connector:**

```xml
<localEntry key="CustomerDB">
    <![CDATA[
    <connection>
        <url>jdbc:mysql://localhost:3306/customers</url>
        <username>dbuser</username>
        <password>dbpass</password>
        <driver>com.mysql.jdbc.Driver</driver>
        <maxActive>100</maxActive>
    </connection>
    ]]>
</localEntry>

<dblookup>
    <connection>
        <pool><dsName>CustomerDB</dsName></pool>
    </connection>
    <statement>
        <sql>SELECT * FROM customers WHERE id = ?</sql>
        <parameter expression="json-eval($.customerId)" type="INTEGER"/>
        <result name="customer" column="customer_data"/>
    </statement>
</dblookup>
```

**Salesforce Connector:**

```xml
<salesforce.init>
    <loginUrl>https://login.salesforce.com</loginUrl>
    <username>user@example.com</username>
    <password>password</password>
    <securityToken>token</securityToken>
</salesforce.init>

<salesforce.query>
    <queryString>SELECT Id, Name FROM Account WHERE Industry = 'Technology'</queryString>
</salesforce.query>
```

## AI Agent Module

**RAG-Based Agent:**

```xml
<sequence name="CustomerSupportAgent">
    <!-- 1. Vector search for context -->
    <call>
        <endpoint>
            <http method="POST" uri-template="https://vector-db.example.com/search"/>
        </endpoint>
    </call>
    <property name="CONTEXT" expression="json-eval($.results)"/>

    <!-- 2. Get customer history -->
    <call>
        <endpoint>
            <http method="GET" uri-template="https://crm.example.com/api/customers/{uri.var.id}"/>
        </endpoint>
    </call>
    <property name="HISTORY" expression="json-eval($)"/>

    <!-- 3. Build prompt -->
    <payloadFactory media-type="json">
        <format>
        {
            "model": "claude-3-5-sonnet-20241022",
            "max_tokens": 1024,
            "messages": [{
                "role": "user",
                "content": "Context: $1\n\nHistory: $2\n\nQuery: $3"
            }]
        }
        </format>
        <args>
            <arg evaluator="xml" expression="get-property('CONTEXT')"/>
            <arg evaluator="xml" expression="get-property('HISTORY')"/>
            <arg evaluator="xml" expression="json-eval($.query)"/>
        </args>
    </payloadFactory>

    <!-- 4. Call Claude -->
    <call>
        <endpoint>
            <http method="POST" uri-template="https://api.anthropic.com/v1/messages">
                <header name="x-api-key" value="${ANTHROPIC_API_KEY}"/>
                <header name="anthropic-version" value="2023-06-01"/>
            </http>
        </endpoint>
    </call>
</sequence>
```

## Deployment

**Kubernetes Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: micro-integrator
  template:
    metadata:
      labels:
        app: micro-integrator
    spec:
      containers:
      - name: micro-integrator
        image: wso2/wso2mi:4.5.0
        ports:
        - containerPort: 8290
        - containerPort: 8253
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mi-secrets
              key: database-password
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
```

## Best Practices

- Use connection pooling for databases
- Implement proper error handling
- Enable caching where appropriate
- Monitor performance metrics
- Use async processing for long operations

## Troubleshooting

**Memory Issues:**
- Increase heap size: `-Xmx2048m`
- Use streaming for large payloads
- Monitor GC logs

**Performance:**
- Enable pass-through mode
- Optimize database queries
- Use connection pooling

## Resources

- **Docs**: https://mi.docs.wso2.com/
- **Connectors**: https://store.wso2.com/store/
- **Related**: `wso2-integration-platform`

---

**Last Updated**: March 2025
