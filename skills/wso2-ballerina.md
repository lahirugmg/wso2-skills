---
name: wso2-ballerina
description: Use when programming in Ballerina - cloud-native language for network services with visual and textual development (Swan Lake 2201.x)
---

# Ballerina Programming Language Skill

## When to Use This Skill

Use for Ballerina development:
- Network services and APIs
- Microservices architecture
- Integration development
- Cloud-native applications
- Visual + textual programming

## Language Overview

Ballerina is a cloud-native programming language designed specifically for network programming with bidirectional visual and textual representation.

## Key Features

- **Network-First**: Built for REST, GraphQL, gRPC, WebSocket
- **Visual + Textual**: Bidirectional code ↔ diagram
- **Native JSON**: First-class JSON support
- **Concurrency**: Built-in async and parallel execution
- **Cloud-Native**: Kubernetes, Docker ready
- **Type Safety**: Strong static typing

## Installation

```bash
# Linux/Mac
curl -sSL https://dist.ballerina.io/downloads/VERSION/ballerina-install.sh | bash

# Windows
# Download installer from ballerina.io

# Verify
bal version
```

## Hello World

```ballerina
import ballerina/io;

public function main() {
    io:println("Hello, World!");
}
```

## HTTP Service

```ballerina
import ballerina/http;

service /hello on new http:Listener(9090) {

    resource function get greeting() returns string {
        return "Hello, Ballerina!";
    }

    resource function get user/[string name]() returns json {
        return {
            message: string `Hello, ${name}!`,
            timestamp: time:utcNow()
        };
    }

    resource function post data(@http:Payload json data) returns json|error {
        // Process data
        return {
            status: "success",
            received: data
        };
    }
}
```

## Database Operations

```ballerina
import ballerinax/mysql;
import ballerinax/mysql.driver as _;

type Customer record {|
    int id;
    string name;
    string email;
|};

mysql:Client db = check new (
    host = "localhost",
    user = "dbuser",
    password = "dbpass",
    database = "customers"
);

function getCustomers() returns Customer[]|error {
    stream<Customer, error?> customerStream = db->query(
        `SELECT id, name, email FROM customers`
    );
    return from Customer customer in customerStream
           select customer;
}

function createCustomer(Customer customer) returns int|error {
    sql:ExecutionResult result = check db->execute(
        `INSERT INTO customers (name, email)
         VALUES (${customer.name}, ${customer.email})`
    );
    return <int>result.lastInsertId;
}
```

## Concurrency

```ballerina
// Concurrent calls
function getDataConcurrently() returns error? {
    worker A {
        json result1 = check fetchFromAPI1();
        result1 -> B;
    }

    worker B {
        json result2 = check fetchFromAPI2();
        json result1 = <- A;

        // Both results available, combine them
        json combined = {api1: result1, api2: result2};
    }
}

// Using futures
function parallelProcessing() returns error? {
    future<json> f1 = start fetchFromAPI1();
    future<json> f2 = start fetchFromAPI2();

    json result1 = check wait f1;
    json result2 = check wait f2;
}
```

## Error Handling

```ballerina
function divideNumbers(int a, int b) returns int|error {
    if b == 0 {
        return error("Division by zero");
    }
    return a / b;
}

function example() {
    // Using check
    int result = check divideNumbers(10, 2);

    // Manual error handling
    int|error result2 = divideNumbers(10, 0);
    if result2 is error {
        io:println("Error:", result2.message());
    } else {
        io:println("Result:", result2);
    }
}
```

## GraphQL Service

```ballerina
import ballerina/graphql;

service /graphql on new graphql:Listener(9090) {

    resource function get customer(int id) returns Customer|error {
        return getCustomerFromDB(id);
    }

    resource function get customers() returns Customer[]|error {
        return getAllCustomers();
    }

    remote function createCustomer(string name, string email)
            returns Customer|error {
        return createNewCustomer(name, email);
    }
}
```

## Testing

```ballerina
import ballerina/test;
import ballerina/http;

@test:Config {}
function testCustomerService() returns error? {
    http:Client customerClient = check new ("http://localhost:9090");

    json response = check customerClient->get("/customers");
    test:assertTrue(response is json[]);
}

@test:Config {}
function testCreateCustomer() returns error? {
    http:Client client = check new ("http://localhost:9090");

    json payload = {name: "John Doe", email: "john@example.com"};
    http:Response response = check client->post("/customers", payload);

    test:assertEquals(response.statusCode, 201);
}
```

## Package Management

```toml
# Ballerina.toml
[package]
org = "myorg"
name = "customer_service"
version = "1.0.0"

[build-options]
observabilityIncluded = true

[[dependency]]
org = "ballerinax"
name = "mysql"
version = "1.10.0"
```

```bash
# Build
bal build

# Run
bal run

# Test
bal test

# Push to Ballerina Central
bal push
```

## Deployment

**Docker:**

```bash
# Generate Dockerfile
bal build --cloud=docker

# Generated Dockerfile
docker build -t customer-service .
docker run -p 9090:9090 customer-service
```

**Kubernetes:**

```ballerina
import ballerina/cloud;

@cloud:Config {
    name: "customer-service",
    replicas: 3,
    memory: "512Mi",
    cpu: "500m"
}
service /customers on new http:Listener(9090) {
    // Service implementation
}
```

```bash
# Generate K8s artifacts
bal build --cloud=k8s

# Deploy
kubectl apply -f target/kubernetes/
```

## Best Practices

- Use type-safe operations
- Leverage built-in concurrency
- Handle errors explicitly
- Use query expressions for collections
- Write tests for all services
- Use visual editor for complex flows

## Visual Development

```
Ballerina provides bidirectional mapping:

Visual Editor shows:
┌──────────────┐
│ HTTP Listener│
│   :9090      │
└──────┬───────┘
       │
       ▼
┌─────────────┐
│  Validate   │
│   Request   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Query DB  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Response  │
└─────────────┘

Changes in visual → Updates code
Changes in code → Updates visual
```

## Resources

- **Website**: https://ballerina.io/
- **Learn**: https://ballerina.io/learn/
- **By Example**: https://ballerina.io/learn/by-example/
- **API Docs**: https://lib.ballerina.io/
- **Related**: `wso2-engineering-platform`, `wso2-choreo`

---

**Last Updated**: March 2025
