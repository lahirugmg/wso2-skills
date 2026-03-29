---
name: wso2-streaming-integrator
description: Use when working with real-time data streaming and analytics using WSO2 Streaming Integrator - Siddhi queries, stream processing, CEP (v4.3.1+)
---

# WSO2 Streaming Integrator Development Skill

## When to Use This Skill

Use for Streaming Integrator operations:
- Real-time data processing
- Complex Event Processing (CEP)
- Stream analytics
- Real-time alerting
- Data integration pipelines

## Product Overview

WSO2 Streaming Integrator 4.3.1+ provides real-time stream processing with Siddhi query language and VS Code extension.

## Key Features

- **Siddhi Query Language**: SQL-like stream processing
- **Real-Time Analytics**: Process data as it arrives
- **Complex Event Processing**: Pattern detection
- **VS Code Extension**: Development and testing
- **Multiple Sources**: Kafka, HTTP, File, Database, etc.

## Installation

```bash
# Download
wget https://github.com/wso2/streaming-integrator/releases/download/v4.3.1/wso2si-4.3.1.zip
unzip wso2si-4.3.1.zip

# VS Code Extension
Install "WSO2 Streaming Integrator" extension
```

## Siddhi Basics

**Simple Stream Processing:**

```sql
@App:name("OrderAnalytics")

-- Define input stream
@source(type='kafka', topic.list='orders', bootstrap.servers='localhost:9092', @map(type='json'))
define stream OrderStream (
    orderId string,
    customerId string,
    amount double,
    timestamp long
);

-- Define output stream
@sink(type='log')
define stream HighValueOrders (
    orderId string,
    customerId string,
    amount double
);

-- Filter high-value orders
from OrderStream[amount > 1000]
select orderId, customerId, amount
insert into HighValueOrders;
```

## Common Patterns

**1. Windowing:**

```sql
-- Sliding window (last 5 minutes)
from OrderStream#window.time(5 min)
select customerId, sum(amount) as totalAmount
group by customerId
insert into CustomerSpending;

-- Tumbling window (batch every minute)
from OrderStream#window.timeBatch(1 min)
select count() as orderCount, avg(amount) as avgAmount
insert into OrderMetrics;
```

**2. Pattern Detection:**

```sql
-- Detect 3 failed login attempts in 5 minutes
from every (e1=LoginStream) -> e2=LoginStream[e1.userId == e2.userId and e1.status == 'failed']<3>
    within 5 min
select e1.userId, count() as failedAttempts
insert into FailedLoginAlert;
```

**3. Join Streams:**

```sql
-- Join order and customer streams
from OrderStream#window.time(1 hour) as o
    join CustomerStream#window.time(1 day) as c
    on o.customerId == c.customerId
select o.orderId, c.name, c.email, o.amount
insert into EnrichedOrders;
```

**4. Real-Time Alerts:**

```sql
-- High-value customer alert
from OrderStream#window.time(1 hour)
select customerId, sum(amount) as totalAmount, count() as orderCount
group by customerId
having totalAmount > 10000
insert into HighValueCustomerAlert;

-- Unusual pattern alert
from OrderStream#window.time(5 min)
select customerId, count() as orderCount
group by customerId
having orderCount > 10
insert into UnusualPatternAlert;
```

## Data Sources

**Kafka:**
```sql
@source(type='kafka',
        topic.list='orders',
        partition.no.list='0,1,2',
        bootstrap.servers='localhost:9092',
        @map(type='json'))
define stream OrderStream (...);
```

**HTTP:**
```sql
@source(type='http',
        receiver.url='http://0.0.0.0:8080/orders',
        @map(type='json'))
define stream OrderStream (...);
```

**Database (CDC):**
```sql
@source(type='cdc',
        url='jdbc:mysql://localhost:3306/orders',
        username='user',
        password='pass',
        table.name='orders',
        @map(type='keyvalue'))
define stream OrderStream (...);
```

## Deployment

**Kubernetes:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: si-deployment
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: streaming-integrator
        image: wso2/wso2si:4.3.1
        ports:
        - containerPort: 9443
```

## Best Practices

- Use appropriate window types
- Optimize queries for performance
- Monitor memory usage
- Implement error handling
- Test with realistic data volumes

## Resources

- **Docs**: https://si.docs.wso2.com/
- **Siddhi**: https://siddhi.io/
- **Related**: `wso2-integration-platform`

---

**Last Updated**: March 2025
