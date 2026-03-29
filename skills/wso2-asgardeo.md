---
name: wso2-asgardeo
description: Use when working with Asgardeo - WSO2's cloud IDaaS for CIAM, AI agent identity, and passwordless authentication
---

# WSO2 Asgardeo Development Skill

## When to Use This Skill

Use for Asgardeo cloud IAM:
- Customer Identity and Access Management (CIAM)
- AI agent authentication and authorization
- Passwordless authentication (FIDO2, passkeys)
- Social login integration
- SaaS-first identity management

## Product Overview

Asgardeo is WSO2's Identity as a Service (IDaaS) offering with AI-powered features, supporting human users, AI agents, and applications.

## Getting Started

```bash
# Sign up
1. Go to https://wso2.com/asgardeo/
2. Create organization
3. Access console at https://console.asgardeo.io
```

## Quick Setup

**Register Application:**

```bash
# Via Console
1. Applications → New Application
2. Choose type: Web, SPA, Mobile, M2M
3. Configure:
   - Name: My App
   - Callback URLs: https://app.example.com/callback
   - Allowed origins: https://app.example.com
4. Get credentials:
   - Client ID
   - Client Secret
```

## Integration

**React SPA:**

```javascript
import { AuthProvider, useAuthContext } from "@asgardeo/auth-react";

const config = {
    signInRedirectURL: "https://app.example.com",
    signOutRedirectURL: "https://app.example.com/logout",
    clientID: "YOUR_CLIENT_ID",
    baseUrl: "https://api.asgardeo.io/t/yourorg",
    scope: [ "openid", "profile", "email" ]
};

function App() {
    return (
        <AuthProvider config={config}>
            <MyApp />
        </AuthProvider>
    );
}

function MyApp() {
    const { state, signIn, signOut } = useAuthContext();

    return (
        <div>
            {state.isAuthenticated ? (
                <>
                    <p>Welcome, {state.username}!</p>
                    <button onClick={signOut}>Logout</button>
                </>
            ) : (
                <button onClick={signIn}>Login</button>
            )}
        </div>
    );
}
```

## AI Agent Identity

**Register AI Agent:**

```javascript
// Using Asgardeo Management API
const registerAgent = async () => {
    const response = await fetch(
        'https://api.asgardeo.io/t/yourorg/api/server/v1/applications',
        {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${accessToken}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                name: 'customer-support-agent',
                description: 'AI agent for customer support',
                templateId: 'ai-agent-template',
                inboundProtocolConfiguration: {
                    oidc: {
                        grantTypes: ['client_credentials'],
                        publicClient: false
                    }
                }
            })
        }
    );

    const agent = await response.json();
    return agent;
};
```

## Passwordless Auth

**FIDO2/Passkeys:**

```javascript
// Enable passkey authentication
const enablePasskey = async () => {
    // User registers passkey
    await authClient.requestPasskeyRegistration();

    // User authenticates with passkey
    await authClient.signInWithPasskey();
};
```

## Social Login

**Configure Providers:**

```bash
# Via Console
1. Connections → New Connection
2. Choose provider:
   - Google
   - Facebook
   - GitHub
   - Apple
3. Configure credentials
4. Enable for applications
```

## MCP Authorization

**Authorize AI Agent Access:**

```javascript
// Define MCP authorization policy
const mcpPolicy = {
    agentId: 'customer-support-agent',
    mcpServer: 'mcp://tools.example.com',
    authorizedTools: [
        {
            tool: 'search_knowledge_base',
            permissions: ['execute'],
            rateLimit: '1000/hour'
        },
        {
            tool: 'get_customer_info',
            permissions: ['execute'],
            dataFilters: {
                allowedFields: ['name', 'email'],
                deniedFields: ['ssn', 'credit_card']
            }
        }
    ]
};
```

## User Management

**Create Users (SCIM API):**

```javascript
const createUser = async (userData) => {
    const response = await fetch(
        'https://api.asgardeo.io/t/yourorg/scim2/Users',
        {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${accessToken}`,
                'Content-Type': 'application/scim+json'
            },
            body: JSON.stringify({
                schemas: ['urn:ietf:params:scim:schemas:core:2.0:User'],
                userName: userData.email,
                name: {
                    givenName: userData.firstName,
                    familyName: userData.lastName
                },
                emails: [{
                    primary: true,
                    value: userData.email
                }]
            })
        }
    );

    return await response.json();
};
```

## Adaptive Authentication

**AI-Generated Flows:**

```
# Use AI to generate authentication flow

User prompt: "Require MFA for logins from new devices,
             allow passwordless for trusted devices"

AI generates adaptive script automatically
```

## Best Practices

- Use passwordless for better UX
- Enable MFA for sensitive operations
- Leverage AI for authentication flows
- Monitor usage via analytics dashboard
- Use session management features

## Resources

- **Docs**: https://wso2.com/asgardeo/docs/
- **APIs**: https://wso2.com/asgardeo/docs/apis/
- **SDKs**: https://github.com/asgardeo
- **Related**: `wso2-identity-platform`, `wso2-identity-server`

---

**Last Updated**: March 2025
