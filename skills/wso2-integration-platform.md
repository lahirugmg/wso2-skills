---
name: wso2-integration-platform
description: Use when integrating systems, building data pipelines, creating microservices, or developing ESB patterns - covers Business Integrator, Micro Integrator, Streaming Integrator, and Control Plane
---

# WSO2 Integration Platform Development Skill

## When to Use This Skill

Use this skill when:
- Integrating on-premise and cloud-based systems
- Building microservices with ESB patterns
- Creating data integration pipelines
- Developing real-time streaming applications
- Implementing message transformation and routing
- Building AI agents with enterprise integrations
- Connecting to databases, APIs, and enterprise applications
- Orchestrating complex business workflows
- Processing and analyzing streaming data in real-time

## Product Overview

WSO2 Integration Platform unites four core components for comprehensive integration:

### Core Components

1. **Business Integrator (BI)** - AI-powered low-code/pro-code integration development
2. **Micro Integrator (MI)** - Lightweight ESB runtime for cloud-native deployments
3. **Streaming Integrator (SI)** - Real-time data streaming and analytics
4. **Integrator Control Plane (ICP)** - Centralized management and observability

### Key Capabilities (2025)

- **AI-Native Development**: AI copilot assists at every development step
- **400+ Connectors**: Pre-built integrations for databases, SaaS apps, protocols
- **Flexible Architecture**: ESB, microservices, or hybrid deployment patterns
- **Low-Code Experience**: Visual development with full code generation
- **Cloud-Native**: Kubernetes-ready, containerized deployments
- **Real-Time Processing**: Stream processing with complex event processing (CEP)
- **AI Agent Module**: Build intelligent apps with agents and RAG

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                  WSO2 Integration Platform                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │        Integrator Control Plane (ICP 1.2.0+)             │   │
│  │  • Centralized Management                                │   │
│  │  • Observability & Monitoring                            │   │
│  │  • Deployment Management                                 │   │
│  │  • Analytics Dashboard                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            ↓  ↓  ↓                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │   Business   │  │    Micro     │  │    Streaming         │  │
│  │  Integrator  │  │  Integrator  │  │    Integrator        │  │
│  │  (BI 1.4.0+) │  │  (MI 4.5.0+) │  │    (SI 4.3.1+)       │  │
│  │              │  │              │  │                      │  │
│  │ • Low-Code   │  │ • Runtime    │  │ • Stream Processing  │  │
│  │ • AI Copilot │  │ • ESB        │  │ • CEP               │  │
│  │ • Visual Dev │  │ • Cloud-     │  │ • Real-time         │  │
│  │              │  │   Native     │  │   Analytics         │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│         ↓                 ↓                      ↓               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │               Connector Ecosystem (400+)                  │   │
│  │  • Databases  • SaaS Apps  • Protocols  • Cloud Services │   │
│  │  • AI Models  • MCP Servers  • Legacy Systems            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## Common Tasks

### 1. Setting Up Development Environment

**Install VS Code Extension:**

```bash
# Prerequisites
- Java 11 or 17
- Maven 3.x (auto-configured by extension)
- VS Code

# Install WSO2 MI/BI Extension
1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X / Cmd+Shift+X)
3. Search for "WSO2 Micro Integrator" or "WSO2 Business Integrator"
4. Click Install
5. Extension auto-configures Java, Maven, and runtime (zero-friction setup)
```

**Create Integration Project:**

```bash
# Using VS Code
1. Open Command Palette (Ctrl+Shift+P / Cmd+Shift+P)
2. Type "WSO2: Create New Integration Project"
3. Choose project type:
   - Integration Project (for MI/BI)
   - Composite Application
   - Registry Resources
   - Connector Exporter

# Using CLI
mi create project customer-integration \
  --type integration \
  --package com.example.integration
```

### 2. Building an Integration (Low-Code)

**Example: Order Processing Integration**

```xml
<!-- Scenario: Receive order, validate, enrich with customer data, route to fulfillment -->

<!-- 1. API Definition -->
<api context="/orders" name="OrderAPI" xmlns="http://ws.apache.org/ns/synapse">
    <resource methods="POST" uri-template="/create">
        <inSequence>
            <!-- Log incoming request -->
            <log level="custom">
                <property name="Message" value="Order received"/>
                <property name="OrderID" expression="json-eval($.orderId)"/>
            </log>

            <!-- Validate order schema -->
            <validate>
                <schema key="conf:schema/order-schema.json"/>
                <on-fail>
                    <payloadFactory media-type="json">
                        <format>{"error": "Invalid order format", "details": "$1"}</format>
                        <args>
                            <arg evaluator="xml" expression="$ctx:ERROR_MESSAGE"/>
                        </args>
                    </payloadFactory>
                    <property name="HTTP_SC" value="400" scope="axis2"/>
                    <respond/>
                </on-fail>
            </validate>

            <!-- Enrich with customer data -->
            <enrich>
                <source type="inline" clone="true">
                    <customer xmlns=""/>
                </source>
                <target type="property" property="CUSTOMER_REQUEST"/>
            </enrich>

            <!-- Call customer service -->
            <call>
                <endpoint>
                    <http method="GET" uri-template="https://customer-service.example.com/api/customers/{uri.var.customerId}">
                        <suspendOnFailure>
                            <initialDuration>-1</initialDuration>
                            <progressionFactor>1</progressionFactor>
                        </suspendOnFailure>
                        <markForSuspension>
                            <retriesBeforeSuspension>0</retriesBeforeSuspension>
                        </markForSuspension>
                    </http>
                </endpoint>
            </call>

            <!-- Enrich order with customer data -->
            <enrich>
                <source type="body" clone="true"/>
                <target type="property" action="child" property="ORDER" xpath="//customer"/>
            </enrich>

            <!-- Transform to fulfillment format -->
            <payloadFactory media-type="json">
                <format>
                {
                    "orderId": "$1",
                    "customer": {
                        "id": "$2",
                        "name": "$3",
                        "email": "$4"
                    },
                    "items": $5,
                    "totalAmount": "$6",
                    "timestamp": "$7"
                }
                </format>
                <args>
                    <arg evaluator="xml" expression="json-eval($.orderId)"/>
                    <arg evaluator="xml" expression="json-eval($.customer.id)"/>
                    <arg evaluator="xml" expression="json-eval($.customer.name)"/>
                    <arg evaluator="xml" expression="json-eval($.customer.email)"/>
                    <arg evaluator="xml" expression="json-eval($.items)"/>
                    <arg evaluator="xml" expression="json-eval($.totalAmount)"/>
                    <arg evaluator="xml" expression="get-property('SYSTEM_TIME')"/>
                </args>
            </payloadFactory>

            <!-- Route to fulfillment queue -->
            <property name="OUT_ONLY" value="true"/>
            <property name="FORCE_SC_ACCEPTED" value="true" scope="axis2"/>

            <send>
                <endpoint>
                    <address uri="jms:/FulfillmentQueue?transport.jms.ConnectionFactoryJNDIName=QueueConnectionFactory&amp;java.naming.factory.initial=org.apache.activemq.jndi.ActiveMQInitialContextFactory&amp;java.naming.provider.url=tcp://localhost:61616"/>
                </endpoint>
            </send>

            <!-- Send response -->
            <payloadFactory media-type="json">
                <format>
                {
                    "status": "accepted",
                    "orderId": "$1",
                    "message": "Order submitted for processing"
                }
                </format>
                <args>
                    <arg evaluator="xml" expression="json-eval($.orderId)"/>
                </args>
            </payloadFactory>
            <respond/>
        </inSequence>

        <faultSequence>
            <log level="custom">
                <property name="Error" value="Order processing failed"/>
                <property name="Details" expression="get-property('ERROR_MESSAGE')"/>
            </log>
            <payloadFactory media-type="json">
                <format>
                {
                    "error": "Order processing failed",
                    "message": "$1"
                }
                </format>
                <args>
                    <arg evaluator="xml" expression="get-property('ERROR_MESSAGE')"/>
                </args>
            </payloadFactory>
            <property name="HTTP_SC" value="500" scope="axis2"/>
            <respond/>
        </faultSequence>
    </resource>
</api>
```

### 3. Using AI Copilot (BI Feature)

**AI-Assisted Development:**

```
# In VS Code with BI Extension

1. Natural Language to Integration:
   Developer: "Create an integration that fetches customer data from Salesforce
              and saves it to MySQL database"

   AI Copilot: Generates:
   - Salesforce connector configuration
   - Data transformation mediators
   - MySQL database connector
   - Error handling sequences
   - Complete integration flow

2. Code Completion:
   Developer types: "transform JSON to"
   AI suggests: Complete transformation templates for common formats

3. Troubleshooting:
   Developer: "Why is this integration failing?"
   AI analyzes: Logs, configuration, provides fix suggestions

4. Optimization:
   AI suggests: Performance improvements, better error handling, caching strategies
```

### 4. Building AI Agents with Integration

**RAG-Based Customer Support Agent:**

```xml
<!-- AI Agent with enterprise integrations -->
<sequence name="CustomerSupportAgent" xmlns="http://ws.apache.org/ns/synapse">
    <!-- Receive customer query -->
    <property name="QUERY" expression="json-eval($.query)"/>

    <!-- 1. Retrieve relevant context from knowledge base (RAG) -->
    <call>
        <endpoint>
            <http method="POST" uri-template="https://vector-db.example.com/search">
                <suspendOnFailure>
                    <initialDuration>-1</initialDuration>
                </suspendOnFailure>
            </http>
        </endpoint>
    </call>
    <property name="CONTEXT" expression="json-eval($.results)"/>

    <!-- 2. Get customer history from CRM -->
    <call>
        <endpoint>
            <http method="GET"
                  uri-template="https://crm.example.com/api/customers/{uri.var.customerId}/history"/>
        </endpoint>
    </call>
    <property name="CUSTOMER_HISTORY" expression="json-eval($)"/>

    <!-- 3. Build prompt with context -->
    <payloadFactory media-type="json">
        <format>
        {
            "model": "claude-3-5-sonnet-20241022",
            "max_tokens": 1024,
            "messages": [{
                "role": "user",
                "content": "Context: $1\n\nCustomer History: $2\n\nCustomer Query: $3\n\nProvide a helpful response."
            }]
        }
        </format>
        <args>
            <arg evaluator="xml" expression="get-property('CONTEXT')"/>
            <arg evaluator="xml" expression="get-property('CUSTOMER_HISTORY')"/>
            <arg evaluator="xml" expression="get-property('QUERY')"/>
        </args>
    </payloadFactory>

    <!-- 4. Call Claude API -->
    <call>
        <endpoint>
            <http method="POST" uri-template="https://api.anthropic.com/v1/messages">
                <header name="x-api-key" value="${ANTHROPIC_API_KEY}"/>
                <header name="anthropic-version" value="2023-06-01"/>
            </http>
        </endpoint>
    </call>

    <!-- 5. Extract response -->
    <property name="AI_RESPONSE" expression="json-eval($.content[0].text)"/>

    <!-- 6. Log interaction for learning -->
    <call>
        <endpoint>
            <address uri="jms:/InteractionLog?transport.jms.ConnectionFactoryJNDIName=QueueConnectionFactory"/>
        </endpoint>
    </call>

    <!-- 7. Return response -->
    <payloadFactory media-type="json">
        <format>
        {
            "response": "$1",
            "sources": $2,
            "timestamp": "$3"
        }
        </format>
        <args>
            <arg evaluator="xml" expression="get-property('AI_RESPONSE')"/>
            <arg evaluator="xml" expression="get-property('CONTEXT')"/>
            <arg evaluator="xml" expression="get-property('SYSTEM_TIME')"/>
        </args>
    </payloadFactory>
</sequence>
```

### 5. Using Connectors

**WSO2 Integration Platform provides 400+ connectors:**

**Database Connector Example (MySQL):**

```xml
<!-- MySQL connector configuration -->
<localEntry key="CustomerDB" xmlns="http://ws.apache.org/ns/synapse">
    <![CDATA[
    <connection>
        <url>jdbc:mysql://localhost:3306/customers</url>
        <username>dbuser</username>
        <password>dbpass</password>
        <driver>com.mysql.jdbc.Driver</driver>
        <maxActive>100</maxActive>
        <maxIdle>20</maxIdle>
        <minIdle>5</minIdle>
    </connection>
    ]]>
</localEntry>

<!-- Query operation -->
<dblookup>
    <connection>
        <pool>
            <dsName>CustomerDB</dsName>
        </pool>
    </connection>
    <statement>
        <sql>SELECT * FROM customers WHERE customer_id = ?</sql>
        <parameter expression="json-eval($.customerId)" type="VARCHAR"/>
        <result name="customer" column="customer_data"/>
    </statement>
</dblookup>
```

**SaaS Connector Example (Salesforce):**

```xml
<!-- Salesforce connector -->
<salesforce.init>
    <loginUrl>https://login.salesforce.com</loginUrl>
    <username>user@example.com</username>
    <password>password</password>
    <securityToken>security_token</securityToken>
</salesforce.init>

<salesforce.query>
    <queryString>SELECT Id, Name, Email FROM Contact WHERE AccountId = '{$ctx:accountId}'</queryString>
</salesforce.query>
```

**File Connector Example:**

```xml
<!-- Read file -->
<fileconnector.read>
    <source>/data/orders/order_{$ctx:orderId}.json</source>
    <contentType>application/json</contentType>
</fileconnector.read>

<!-- Write file -->
<fileconnector.write>
    <destination>/data/processed/processed_{$ctx:orderId}.json</destination>
    <content>{$body}</content>
    <append>false</append>
</fileconnector.write>
```

### 6. Streaming Integration (SI)

**Real-Time Order Analytics:**

```sql
-- Siddhi App for real-time order processing
@App:name("OrderAnalytics")
@App:description("Real-time order monitoring and alerting")

-- Define order stream
@source(type='kafka',
        topic.list='orders',
        partition.no.list='0,1,2',
        threading.option='single.thread',
        group.id='order-analytics',
        bootstrap.servers='localhost:9092',
        @map(type='json'))
define stream OrderStream (
    orderId string,
    customerId string,
    amount double,
    status string,
    timestamp long
);

-- Define alert stream
@sink(type='http',
      publisher.url='https://alerts.example.com/api/alerts',
      method='POST',
      @map(type='json'))
define stream AlertStream (
    alertType string,
    customerId string,
    totalAmount double,
    orderCount long,
    timestamp long
);

-- Detect high-value customers (>$10k in 1 hour)
@info(name='HighValueCustomer')
from OrderStream#window.time(1 hour)
select
    'HIGH_VALUE_CUSTOMER' as alertType,
    customerId,
    sum(amount) as totalAmount,
    count() as orderCount,
    timestamp
group by customerId
having totalAmount > 10000
insert into AlertStream;

-- Detect unusual order patterns (>10 orders in 5 min)
@info(name='UnusualOrderPattern')
from OrderStream#window.time(5 min)
select
    'UNUSUAL_PATTERN' as alertType,
    customerId,
    sum(amount) as totalAmount,
    count() as orderCount,
    timestamp
group by customerId
having orderCount > 10
insert into AlertStream;

-- Calculate rolling averages
@info(name='OrderMetrics')
from OrderStream#window.timeBatch(1 min)
select
    count() as orderCount,
    sum(amount) as totalRevenue,
    avg(amount) as avgOrderValue,
    timestamp
insert into OrderMetricsStream;
```

**Deploy Streaming App:**

```bash
# Using VS Code SI Extension
1. Create new Siddhi application (.siddhi file)
2. Write Siddhi queries
3. Test locally with sample data
4. Deploy to Streaming Integrator runtime

# Using CLI
si deploy --file OrderAnalytics.siddhi --runtime si-server-01

# Monitor
si status --app OrderAnalytics
si logs --app OrderAnalytics --follow
```

### 7. Using Control Plane (ICP)

**Centralized Management:**

```yaml
# ICP Configuration
control_plane:
  endpoint: "https://icp.example.com"

  # Register MI instances
  micro_integrator:
    instances:
      - name: "mi-node-01"
        url: "https://mi-01.example.com:9164"
        group: "production"

      - name: "mi-node-02"
        url: "https://mi-02.example.com:9164"
        group: "production"

  # Register SI instances
  streaming_integrator:
    instances:
      - name: "si-node-01"
        url: "https://si-01.example.com:9443"
        group: "streaming"

  # Register BI instances
  business_integrator:
    instances:
      - name: "bi-dev-01"
        url: "https://bi-dev.example.com:9164"
        group: "development"

  # Monitoring
  monitoring:
    metrics:
      enabled: true
      interval: 30s
      exporters:
        - prometheus
        - datadog

    logging:
      level: INFO
      aggregation: elasticsearch

    tracing:
      enabled: true
      sampler: probabilistic
      rate: 0.1
```

**View Dashboard:**
- Navigate to: `https://icp.example.com/dashboard`
- View all deployments, health status, metrics
- Manage artifacts across all instances
- View logs and traces in unified interface

## Best Practices

### Integration Design

1. **Loose Coupling**
   - Use message queues for async communication
   - Implement circuit breakers for resilience
   - Avoid tight dependencies between systems
   - Use event-driven architecture where appropriate

2. **Error Handling**
   - Always implement fault sequences
   - Use dead letter queues for failed messages
   - Log errors with sufficient context
   - Implement retry logic with exponential backoff

3. **Performance**
   - Use pass-through mode for simple routing
   - Implement caching for frequently accessed data
   - Use connection pooling for databases
   - Optimize payload transformations

4. **Security**
   - Secure endpoints with OAuth2/JWT
   - Encrypt sensitive data
   - Use secure vault for credentials
   - Implement audit logging

### Development Workflow

1. **Local Development**
   - Use VS Code extensions for development
   - Test with mock endpoints
   - Use hot deployment during development
   - Leverage AI copilot for faster development

2. **Version Control**
   - Store integration projects in Git
   - Use branching strategy (gitflow)
   - Tag releases
   - Document changes in commit messages

3. **Testing**
   - Write unit tests for mediators
   - Integration tests for end-to-end flows
   - Load testing before production
   - Monitor test coverage

4. **Deployment**
   - Use CI/CD pipelines
   - Automate deployments with scripts
   - Use blue-green or canary deployments
   - Monitor after deployment

## Integration Patterns

### Enterprise Integration Patterns Supported:

- **Message Routing**: Content-based routing, message filter, dynamic router
- **Message Transformation**: Message translator, envelope wrapper, content enricher
- **Message Endpoints**: Message endpoint, event-driven consumer, polling consumer
- **System Management**: Wire tap, message store, message processor
- **Messaging Channels**: Point-to-point, publish-subscribe, dead letter channel

### Example: Scatter-Gather Pattern

```xml
<!-- Scatter-Gather: Query multiple services and aggregate results -->
<scatter-gather parallel-execution="true">
    <!-- Call inventory service -->
    <sequence key="CallInventoryService"/>

    <!-- Call pricing service -->
    <sequence key="CallPricingService"/>

    <!-- Call recommendation service -->
    <sequence key="CallRecommendationService"/>

    <!-- Aggregation -->
    <aggregator class="org.wso2.carbon.mediator.aggregate.JSONAggregator">
        <completionTimeout>5000</completionTimeout>
        <onComplete>
            <payloadFactory media-type="json">
                <format>
                {
                    "inventory": $1,
                    "pricing": $2,
                    "recommendations": $3
                }
                </format>
            </payloadFactory>
        </onComplete>
    </aggregator>
</scatter-gather>
```

## Troubleshooting

### Common Issues

**1. Integration Timeout**
- **Symptom**: Integration fails with timeout error
- **Solutions**:
  - Increase timeout in endpoint configuration
  - Implement async processing
  - Use caching for slow backends
  - Check network connectivity

**2. Memory Issues**
- **Symptom**: OutOfMemoryError, performance degradation
- **Solutions**:
  - Increase JVM heap size (-Xmx)
  - Use streaming for large payloads
  - Implement proper message cleanup
  - Monitor memory usage

**3. Connector Issues**
- **Symptom**: Connector operations fail
- **Solutions**:
  - Verify connector version compatibility
  - Check credentials and permissions
  - Review connector documentation
  - Enable debug logs for connector

**4. Message Transformation Errors**
- **Symptom**: Invalid payload or transformation failures
- **Solutions**:
  - Validate input schemas
  - Use try-catch in transformations
  - Log original and transformed payloads
  - Test transformations with sample data

## Resources

### Official Documentation
- **Integration Platform**: https://wso2.com/integration-platform/
- **Micro Integrator**: https://mi.docs.wso2.com/
- **Business Integrator**: https://wso2.com/integrator/mi/
- **Streaming Integrator**: https://si.docs.wso2.com/
- **Connector Store**: https://store.wso2.com/store/

### Related Skills
- `wso2-micro-integrator` - Deep dive into MI runtime
- `wso2-business-integrator` - Low-code development
- `wso2-streaming-integrator` - Real-time data processing
- `wso2-agent-platform` - Building AI agents with integrations

### Tools
- **VS Code Extensions**: WSO2 MI, BI, SI extensions
- **CLI Tools**: MI CLI, SI CLI
- **Testing**: SoapUI, Postman
- **Monitoring**: Prometheus, Grafana, ELK

## Version Compatibility

- **Business Integrator**: 1.4.0+
- **Micro Integrator**: 4.5.0+
- **Streaming Integrator**: 4.3.1+
- **Integrator Control Plane**: 1.2.0+

---

**Last Updated**: March 2025
**Skill Type**: Flexible - Adapt to your integration requirements
**Related Platforms**: Integration, Agent, API, Engineering
