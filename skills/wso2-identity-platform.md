---
name: wso2-identity-platform
description: Use when implementing authentication, authorization, identity management, or access control - covers Identity Server, Asgardeo (IDaaS), AI agent security, and Model Context Protocol authorization
---

# WSO2 Identity Platform Development Skill

## When to Use This Skill

Use this skill when:
- Implementing user authentication and authorization
- Setting up single sign-on (SSO) across applications
- Managing customer identity (CIAM)
- Implementing multi-factor authentication (MFA)
- Configuring adaptive authentication with risk-based rules
- Securing AI agents and non-human identities
- Implementing OAuth2, OpenID Connect, or SAML
- Managing user identities, roles, and permissions
- Implementing passwordless authentication
- Authorizing Model Context Protocol (MCP) tools
- Integrating identity management with APIs and applications

## Product Overview

WSO2 Identity Platform provides comprehensive identity and access management (IAM) across three deployment options:

### Products (Shared Codebase)

1. **WSO2 Identity Server** - Open-source, self-hosted IAM
2. **Asgardeo** - Identity as a Service (IDaaS), fully managed
3. **WSO2 Private Identity Cloud** - Single-tenant SaaS

### Key Capabilities (2025)

- **AI-Powered Features**: Auto-generate login flows, AI-driven branding
- **AI Agent Security**: First-to-market AI agent identity management
- **MCP Authorization**: Control AI agent access to MCP tools
- **Adaptive Authentication**: Risk-based, context-aware access control
- **Passwordless Auth**: FIDO2, passkeys, biometric authentication
- **Social Login**: Google, Facebook, Apple, GitHub, etc.
- **Enterprise Federation**: SAML, WS-Federation, Azure AD, Okta
- **Identity Verification**: Integration with Onfido and other providers
- **Comprehensive Protocols**: OAuth2, OIDC, SAML 2.0, WS-Trust, SCIM 2.0

## Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                  WSO2 Identity Platform                            │
├───────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌─────────────────── Identity Core ────────────────────────┐    │
│  │                                                            │    │
│  │  • User Management      • Role & Permission Management    │    │
│  │  • Authentication       • Authorization                   │    │
│  │  • Session Management   • Consent Management              │    │
│  │  • Identity Verification • Audit & Compliance             │    │
│  │                                                            │    │
│  └────────────────────────────────────────────────────────────┘   │
│                              ↓                                     │
│  ┌─────────────── Authentication Methods ───────────────────┐    │
│  │                                                            │    │
│  │  • Username/Password    • Social Login                    │    │
│  │  • MFA/2FA              • Passwordless (FIDO2)            │    │
│  │  • Biometric            • Magic Links                     │    │
│  │  • Adaptive Auth        • AI-Generated Flows              │    │
│  │                                                            │    │
│  └────────────────────────────────────────────────────────────┘   │
│                              ↓                                     │
│  ┌──────────────── Identity Types (2025) ────────────────────┐   │
│  │                                                             │   │
│  │  👤 Human Users          🤖 AI Agents                      │   │
│  │  👥 B2B Organizations    🔧 Service Accounts              │   │
│  │  👨‍💼 Employees             📱 Device Identities              │   │
│  │  🛒 Customers             🔌 API Clients                   │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘  │
│                              ↓                                     │
│  ┌────────────────── Protocols & Standards ─────────────────┐    │
│  │                                                            │    │
│  │  • OAuth 2.0 / OIDC     • SAML 2.0                        │    │
│  │  • JWT                  • WS-Federation                   │    │
│  │  • SCIM 2.0             • WS-Trust                        │    │
│  │  • MCP Authorization    • FIDO2 / WebAuthn                │    │
│  │                                                            │    │
│  └────────────────────────────────────────────────────────────┘   │
│                              ↓                                     │
│  ┌──────────────── Integrations ───────────────────────────┐     │
│  │                                                           │     │
│  │  • Applications         • APIs (WSO2 API Manager)        │     │
│  │  • AI Agents            • MCP Servers                    │     │
│  │  • Microservices        • Legacy Systems                 │     │
│  │                                                           │     │
│  └───────────────────────────────────────────────────────────┘    │
│                                                                    │
└───────────────────────────────────────────────────────────────────┘
```

## Common Tasks

### 1. Setting Up Identity Server / Asgardeo

**Identity Server (Self-Hosted):**

```bash
# Download and Install
wget https://product-dist.wso2.com/products/identity-server/7.2.0/wso2is-7.2.0.zip
unzip wso2is-7.2.0.zip
cd wso2is-7.2.0

# Start server
./bin/wso2server.sh

# Access console
# https://localhost:9443/console
# Default credentials: admin/admin
```

**Asgardeo (Cloud):**

```
1. Sign up at https://wso2.com/asgardeo/
2. Create organization
3. Access console at https://console.asgardeo.io
4. Start configuring applications and users
```

### 2. Registering an Application

**OAuth2/OIDC Application (Asgardeo/IS):**

```yaml
# Application Configuration
application:
  name: "Customer Portal"
  description: "Customer-facing web application"

  protocol:
    type: "oidc"  # or "saml", "oauth2"
    grant_types:
      - "authorization_code"
      - "refresh_token"
    callback_urls:
      - "https://app.example.com/callback"
      - "https://app.example.com/silent-callback"  # For silent refresh
    logout_urls:
      - "https://app.example.com/logout"

  access:
    allowed_origins:
      - "https://app.example.com"

  tokens:
    id_token_expiry: 3600  # seconds
    access_token_expiry: 3600
    refresh_token_expiry: 86400

  scopes:
    - "openid"
    - "profile"
    - "email"
    - "customer:read"
    - "customer:write"

  # Advanced settings
  pkce:
    required: true  # Require PKCE for public clients
    challenge_method: "S256"

  claims:
    id_token:
      - "email"
      - "given_name"
      - "family_name"
      - "groups"
    userinfo:
      - "email"
      - "phone_number"
      - "address"
```

**Via Management API:**

```bash
# Create application via API
curl -X POST https://api.asgardeo.io/t/{org}/api/server/v1/applications \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Customer Portal",
    "templateId": "6a90e4b0-fbff-42d7-bfde-1efd98f07cd7",
    "inboundProtocolConfiguration": {
      "oidc": {
        "grantTypes": ["authorization_code", "refresh_token"],
        "callbackURLs": ["https://app.example.com/callback"],
        "publicClient": false,
        "pkce": {
          "mandatory": true,
          "supportPlainTransformAlgorithm": false
        }
      }
    }
  }'
```

### 3. Configuring Authentication

**Basic Username/Password:**

```yaml
# Default configuration - username/password
authentication:
  steps:
    - step: 1
      authenticators:
        - local:
            name: "BasicAuthenticator"
            config:
              lock_after_attempts: 5
              lock_duration: 300  # seconds
              password_policy:
                min_length: 8
                require_uppercase: true
                require_lowercase: true
                require_numbers: true
                require_special_chars: true
```

**Multi-Factor Authentication (MFA):**

```yaml
# Two-step authentication
authentication:
  steps:
    - step: 1
      authenticators:
        - local:
            name: "BasicAuthenticator"

    - step: 2
      authenticators:
        - local:
            name: "TOTPAuthenticator"  # Time-based OTP
        - local:
            name: "EmailOTPAuthenticator"  # Email OTP
        - local:
            name: "SMSOTPAuthenticator"  # SMS OTP

  # Allow users to choose 2FA method
  options:
    use_subject_identifier_as_attribute: true
```

**Passwordless Authentication (FIDO2):**

```yaml
# Passwordless with FIDO2/WebAuthn
authentication:
  passwordless:
    enabled: true

  steps:
    - step: 1
      authenticators:
        - local:
            name: "FIDOAuthenticator"
            config:
              allow_usernameless: true  # Passkeys without username
              attestation: "direct"
              user_verification: "required"

  # Fallback to password if FIDO not available
  fallback:
    - authenticator: "BasicAuthenticator"
```

**Social Login:**

```yaml
# Social authentication providers
authentication:
  steps:
    - step: 1
      authenticators:
        - federated:
            name: "GoogleOIDCAuthenticator"
            config:
              client_id: "${GOOGLE_CLIENT_ID}"
              client_secret: "${GOOGLE_CLIENT_SECRET}"
              scopes:
                - "email"
                - "profile"

        - federated:
            name: "FacebookAuthenticator"
            config:
              client_id: "${FACEBOOK_APP_ID}"
              client_secret: "${FACEBOOK_APP_SECRET}"

        - federated:
            name: "GitHubAuthenticator"
            config:
              client_id: "${GITHUB_CLIENT_ID}"
              client_secret: "${GITHUB_CLIENT_SECRET}"

        - local:
            name: "BasicAuthenticator"  # Traditional login option
```

### 4. Adaptive Authentication (Risk-Based)

**AI-Generated Adaptive Flow (2025 Feature):**

```
# Using AI to generate adaptive authentication flow

User prompt to AI:
"Create an authentication flow that:
- Requires MFA for high-risk logins (unusual location or device)
- Allows passwordless for trusted devices
- Blocks access from blacklisted countries
- Steps up authentication for sensitive operations"

AI generates adaptive script (JavaScript):
```

**Generated Adaptive Authentication Script:**

```javascript
// AI-generated adaptive authentication
var onLoginRequest = function(context) {
    // Get risk score
    var riskScore = calculateRiskScore(context);

    // Check user location
    var country = context.request.ip.country;
    var blacklistedCountries = ["XX", "YY"];

    if (blacklistedCountries.indexOf(country) !== -1) {
        // Block access from blacklisted countries
        context.authentication.fail();
        return;
    }

    // Check if device is trusted
    var isTrustedDevice = context.request.device.isTrusted;

    if (isTrustedDevice && riskScore < 30) {
        // Low risk + trusted device: Allow passwordless
        executeStep(1, {
            authenticatorParams: {
                local: {
                    FIDO: {}
                }
            }
        });
    } else if (riskScore > 70) {
        // High risk: Require MFA
        executeStep(1);  // Username/password
        executeStep(2, {  // MFA
            authenticatorParams: {
                local: {
                    TOTP: {}
                }
            }
        });
    } else {
        // Medium risk: Standard login
        executeStep(1);
    }
};

function calculateRiskScore(context) {
    var score = 0;

    // Check for unusual location
    var userCountry = context.currentKnownSubject.userStoreDomain.country;
    var requestCountry = context.request.ip.country;
    if (userCountry !== requestCountry) {
        score += 30;
    }

    // Check for new device
    if (!context.request.device.isTrusted) {
        score += 25;
    }

    // Check login time (outside business hours)
    var hour = new Date().getHours();
    if (hour < 6 || hour > 22) {
        score += 15;
    }

    // Check failed attempts
    var failedAttempts = context.currentKnownSubject.localClaims['http://wso2.org/claims/identity/failedLoginAttempts'];
    if (failedAttempts > 3) {
        score += 30;
    }

    return score;
}
```

### 5. AI Agent Identity Management (2025 Feature)

**Register AI Agent:**

```yaml
# AI Agent identity configuration
agent_identity:
  name: "customer-support-agent"
  type: "ai_agent"
  description: "AI agent for customer support operations"

  # Agent identity attributes
  attributes:
    agent_type: "conversational"
    framework: "langchain"
    model: "claude-3-5-sonnet-20241022"
    deployed_on: "wso2-agent-manager"

  # Authentication method for agents
  authentication:
    method: "client_credentials"  # OAuth2 client credentials
    client_id: "agent_customer_support"
    client_secret: "${AGENT_CLIENT_SECRET}"

  # Scopes and permissions
  scopes:
    - "agent:execute"
    - "customer:read"
    - "ticket:create"
    - "mcp:invoke"

  # Session-based control (agents are always-on)
  session:
    type: "long_lived"
    max_duration: "24h"
    refresh_policy: "automatic"

  # More granular control than human users
  permissions:
    api_access:
      - endpoint: "/api/customers/*"
        methods: ["GET"]
        rate_limit: "1000/minute"

      - endpoint: "/api/tickets"
        methods: ["POST"]
        rate_limit: "500/minute"

    data_access:
      - type: "customer_pii"
        allowed_fields: ["name", "email"]  # Cannot access SSN, credit cards
        masking: "enabled"

  # Audit and monitoring
  monitoring:
    enabled: true
    log_all_actions: true
    alert_on_anomaly: true
```

### 6. MCP Authorization

**Authorize AI Agent Access to MCP Tools:**

```yaml
# MCP authorization policy
mcp_authorization:
  # MCP server configuration
  server:
    name: "enterprise-tools"
    endpoint: "mcp://tools.example.com"

  # Agent-tool authorization matrix
  authorizations:
    # Customer support agent
    - agent: "customer-support-agent"
      tools:
        - name: "search_knowledge_base"
          permissions: ["execute"]
          rate_limit: "1000/hour"
          conditions:
            - working_hours: "0-23"  # Always available

        - name: "get_customer_info"
          permissions: ["execute"]
          rate_limit: "500/hour"
          conditions:
            - customer_consent_required: true
          data_access:
            - allowed_fields: ["name", "email", "account_status"]
            - denied_fields: ["ssn", "credit_card"]

        - name: "create_ticket"
          permissions: ["execute"]
          rate_limit: "200/hour"
          validation:
            - require_category: true
            - require_description: true

        # Explicitly denied tools
        - name: "delete_customer"
          permissions: []  # No access

    # Operations agent
    - agent: "operations-agent"
      tools:
        - name: "restart_service"
          permissions: ["execute"]
          rate_limit: "10/hour"
          conditions:
            - require_approval: true
            - approval_timeout: "5m"

  # Audit settings
  audit:
    log_all_invocations: true
    log_parameters: true
    log_results: false  # Results may contain sensitive data
    retention_days: 90
```

**MCP Authorization via Identity Server:**

```bash
# Create MCP authorization policy via API
curl -X POST https://is.example.com/api/identity/mcp-authz/v1/policies \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "agentId": "customer-support-agent",
    "mcpServer": "mcp://tools.example.com",
    "authorizedTools": [
      {
        "tool": "search_knowledge_base",
        "permissions": ["execute"],
        "rateLimit": "1000/hour"
      },
      {
        "tool": "get_customer_info",
        "permissions": ["execute"],
        "dataFilters": {
          "allowedFields": ["name", "email"],
          "deniedFields": ["ssn", "credit_card"]
        }
      }
    ]
  }'
```

### 7. User Management

**Create User Programmatically:**

```bash
# Via SCIM 2.0 API
curl -X POST https://api.asgardeo.io/t/{org}/scim2/Users \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "userName": "john.doe@example.com",
    "name": {
      "givenName": "John",
      "familyName": "Doe"
    },
    "emails": [{
      "primary": true,
      "value": "john.doe@example.com",
      "type": "work"
    }],
    "phoneNumbers": [{
      "value": "+1234567890",
      "type": "mobile"
    }],
    "password": "TemporaryPassword123!",
    "active": true
  }'
```

**Bulk User Import:**

```yaml
# CSV import format
username,email,firstName,lastName,department,role
john.doe@example.com,john.doe@example.com,John,Doe,Engineering,Developer
jane.smith@example.com,jane.smith@example.com,Jane,Smith,Sales,Manager
```

### 8. Role-Based Access Control (RBAC)

**Define Roles and Permissions:**

```yaml
# Role configuration
roles:
  - name: "Administrator"
    permissions:
      - "user:create"
      - "user:read"
      - "user:update"
      - "user:delete"
      - "role:manage"
      - "application:manage"

  - name: "Customer Manager"
    permissions:
      - "customer:read"
      - "customer:update"
      - "order:read"
      - "ticket:create"

  - name: "Customer"
    permissions:
      - "profile:read"
      - "profile:update"
      - "order:read:own"  # Can only read own orders

# Assign roles to users
user_roles:
  - user: "john.doe@example.com"
    roles:
      - "Administrator"

  - user: "jane.smith@example.com"
    roles:
      - "Customer Manager"
```

**API-Level Authorization:**

```yaml
# Protect API with scopes/permissions
api_authorization:
  api: "Customer API"
  resources:
    - path: "/customers"
      method: "GET"
      required_scopes: ["customer:read"]
      required_roles: ["Customer Manager", "Administrator"]

    - path: "/customers/{id}"
      method: "DELETE"
      required_scopes: ["customer:delete"]
      required_roles: ["Administrator"]
```

## Best Practices

### Security

1. **Credential Management**
   - Never store plaintext passwords
   - Use strong password policies
   - Implement password expiry
   - Enable account lockout after failed attempts
   - Use secure credential storage (HSM, key vaults)

2. **Token Security**
   - Use short-lived access tokens (1 hour)
   - Use refresh tokens for long sessions
   - Implement token rotation
   - Validate tokens on every request
   - Use PKCE for public clients

3. **Session Management**
   - Implement session timeout
   - Use secure, httpOnly cookies
   - Implement single logout (SLO)
   - Monitor concurrent sessions
   - Invalidate sessions on password change

### Identity Lifecycle

1. **Onboarding**
   - Implement email/phone verification
   - Use self-registration with approval workflow
   - Provide clear privacy policy and consent
   - Support identity verification (KYC)

2. **Access Review**
   - Regularly review user access
   - Remove inactive accounts
   - Audit role assignments
   - Monitor privilege escalation

3. **Offboarding**
   - Disable accounts promptly
   - Revoke all access tokens
   - Remove from all groups/roles
   - Archive user data per policy

### Compliance

1. **Data Privacy**
   - Implement GDPR/CCPA compliance
   - Provide data export functionality
   - Support right to be forgotten
   - Obtain and manage user consent

2. **Audit Logging**
   - Log all authentication events
   - Log authorization decisions
   - Log administrative actions
   - Retain logs per compliance requirements

## Integration with Other WSO2 Products

### With WSO2 API Manager

```yaml
# API Manager integration
api_security:
  key_manager: "wso2-identity-server"
  endpoint: "https://is.example.com"

  oauth2:
    token_endpoint: "/oauth2/token"
    revoke_endpoint: "/oauth2/revoke"
    introspect_endpoint: "/oauth2/introspect"

  jwt_validation:
    issuer: "https://is.example.com/oauth2/token"
    audience: "https://api.example.com"
    jwks_endpoint: "https://is.example.com/oauth2/jwks"
```

### With WSO2 Agent Manager

```yaml
# Agent authentication via Identity Server
agent_security:
  identity_provider: "wso2-identity-server"

  agent_authentication:
    method: "oauth2_client_credentials"
    token_endpoint: "https://is.example.com/oauth2/token"

  mcp_authorization:
    policy_endpoint: "https://is.example.com/api/mcp-authz/v1"
```

### With Choreo

```yaml
# Choreo application authentication
choreo_auth:
  identity_provider: "asgardeo"
  organization: "my-org"

  application:
    client_id: "${CLIENT_ID}"
    client_secret: "${CLIENT_SECRET}"
    redirect_uri: "https://app.choreoapis.dev/callback"
```

## Troubleshooting

### Common Issues

**1. Login Failures**
- **Symptom**: Users cannot log in
- **Causes**: Wrong credentials, locked account, expired password
- **Solutions**:
  - Check account status
  - Verify credentials
  - Check password policy
  - Review audit logs
  - Reset password if needed

**2. Token Validation Errors**
- **Symptom**: API returns 401 Unauthorized
- **Causes**: Expired token, invalid signature, wrong audience
- **Solutions**:
  - Verify token expiration
  - Check JWT signature with JWKS
  - Validate issuer and audience claims
  - Ensure clock synchronization

**3. MFA Not Working**
- **Symptom**: MFA codes not accepted
- **Causes**: Time drift, wrong secret, used code
- **Solutions**:
  - Synchronize server time (NTP)
  - Re-enroll MFA device
  - Check TOTP secret configuration
  - Verify code window settings

## Resources

### Official Documentation
- **Identity Platform**: https://wso2.com/identity-platform/
- **Identity Server Docs**: https://is.docs.wso2.com/
- **Asgardeo Docs**: https://wso2.com/asgardeo/docs/
- **Asgardeo APIs**: https://wso2.com/asgardeo/docs/apis/

### Related Skills
- `wso2-identity-server` - On-premise deployment
- `wso2-asgardeo` - Cloud IAM specifics
- `wso2-api-platform` - API security
- `wso2-agent-platform` - AI agent security

### Standards
- OAuth 2.0: RFC 6749
- OpenID Connect: https://openid.net/specs/openid-connect-core-1_0.html
- SAML 2.0: http://docs.oasis-open.org/security/saml/
- SCIM 2.0: RFC 7643, RFC 7644

## Version Compatibility

- **WSO2 Identity Server**: 7.2+ (latest features)
- **Asgardeo**: Latest (always up-to-date)
- **Private Identity Cloud**: Latest

---

**Last Updated**: March 2025
**Skill Type**: Flexible - Adapt to your security requirements
**Related Platforms**: Identity, API, Agent, Engineering
