---
name: wso2-business-integrator
description: Use when developing with WSO2 Business Integrator - AI-powered low-code integration development with visual editor and pro-code capabilities (v1.4.0+)
---

# WSO2 Business Integrator Development Skill

## When to Use This Skill

Use for Business Integrator operations:
- Low-code integration development
- AI-assisted development with copilot
- Visual flow design
- Rapid integration prototyping
- Business user-friendly integration

## Product Overview

WSO2 Business Integrator 1.4.0+ provides AI-powered low-code/pro-code environment for integration development with visual editor and intelligent assistance.

## Key Features

- **AI Copilot**: Natural language to integration
- **Low-Code Visual Editor**: Drag-and-drop development
- **Pro-Code Option**: Full code control when needed
- **Bidirectional**: Visual ↔ Code sync
- **400+ Connectors**: Pre-built integrations
- **AI-Assisted**: Code completion, troubleshooting, optimization

## Getting Started

**Installation:**
```bash
# Via VS Code Extension
1. Install "WSO2 Business Integrator" extension
2. Auto-configures environment
3. Start creating integrations visually
```

## AI-Assisted Development

**Natural Language to Integration:**

```
Developer: "Create integration that fetches customer data from Salesforce and saves to MySQL"

AI Copilot Generates:
1. Salesforce connector configuration
2. Data transformation logic
3. MySQL database connector
4. Error handling sequences
5. Complete integration flow (visual + code)
```

**Visual Flow Example:**

```
┌──────────────┐
│ HTTP Trigger │
│  POST /sync  │
└──────┬───────┘
       │
       ▼
┌─────────────────────┐
│ Salesforce Query    │
│ Get Contacts        │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│ ForEach Loop        │
│ Process Each Contact│
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│ Transform Data      │
│ SF → MySQL Format   │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│ Insert to MySQL     │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│ Send Response       │
│ {"status": "ok"}    │
└─────────────────────┘
```

## Low-Code Components

**Data Mapping:**
- Visual field mapping
- Auto-suggest transformations
- Test with sample data
- Generate code automatically

**Flow Control:**
- If/Else conditions
- ForEach loops
- Switch/Case routing
- Parallel execution

**Connectors:**
- Drag and drop connectors
- Auto-complete configuration
- Test connectivity
- Reusable configurations

## AI Capabilities

**Code Completion:**
- Smart suggestions as you type
- Context-aware completions
- Best practice recommendations

**Troubleshooting:**
- AI analyzes errors
- Suggests fixes
- Provides explanations

**Optimization:**
- Performance improvement suggestions
- Better error handling recommendations
- Security best practices

## Use Cases

**1. SaaS Integration:**
- Connect Salesforce, ServiceNow, Workday
- Automated data synchronization
- Real-time updates

**2. Database Integration:**
- Multi-database synchronization
- ETL processes
- Data migration

**3. API Orchestration:**
- Combine multiple APIs
- Transform and route data
- Error handling and retries

## Deployment

**Export to Micro Integrator:**
```bash
# BI generates MI-compatible artifacts
# Deploy to MI runtime for production
```

## Best Practices

- Use visual editor for rapid prototyping
- Switch to code for complex logic
- Test with sample data frequently
- Leverage AI suggestions
- Document integrations

## Resources

- **Docs**: https://wso2.com/integrator/mi/
- **Related**: `wso2-integration-platform`, `wso2-micro-integrator`

---

**Last Updated**: March 2025
