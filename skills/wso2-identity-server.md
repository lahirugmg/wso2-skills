---
name: wso2-identity-server
description: Use when deploying and configuring WSO2 Identity Server on-premise - detailed IAM setup, LDAP/AD integration, and advanced authentication (v7.2+)
---

# WSO2 Identity Server Development Skill

## When to Use This Skill

Use for on-premise Identity Server:
- Self-hosted IAM deployment
- LDAP/Active Directory integration
- Custom identity providers
- High-availability setup
- Advanced authentication flows

## Product Overview

WSO2 Identity Server 7.2+ is open-source, self-hosted IAM with AI-driven features and comprehensive protocol support.

## Installation

```bash
# Download
wget https://product-dist.wso2.com/products/identity-server/7.2.0/wso2is-7.2.0.zip
unzip wso2is-7.2.0.zip
cd wso2is-7.2.0

# Start server
./bin/wso2server.sh

# Access console
# https://localhost:9443/console
```

## Configuration

**Database Setup:**

```toml
# repository/conf/deployment.toml

[database.identity_db]
type = "mysql"
url = "jdbc:mysql://localhost:3306/identity_db"
username = "identityuser"
password = "$secret{identity_db_password}"

[database.shared_db]
type = "mysql"
url = "jdbc:mysql://localhost:3306/shared_db"
username = "shareduser"
password = "$secret{shared_db_password}"
```

## LDAP Integration

```toml
[[user_store]]
id = "PRIMARY"
type = "read_write_ldap"
class = "org.wso2.carbon.user.core.ldap.ReadWriteLDAPUserStoreManager"

[user_store.properties]
ConnectionURL = "ldap://localhost:10389"
ConnectionName = "cn=admin,dc=wso2,dc=org"
ConnectionPassword = "$secret{ldap_password}"
UserSearchBase = "ou=Users,dc=wso2,dc=org"
UserNameAttribute = "uid"
UserObjectClass = "inetOrgPerson"
```

## Advanced Authentication

**Custom Authenticator:**

```java
public class CustomAuthenticator extends AbstractApplicationAuthenticator {

    @Override
    public AuthenticatorFlowStatus process(HttpServletRequest request,
                                          HttpServletResponse response,
                                          AuthenticationContext context) {
        // Custom authentication logic
        String username = request.getParameter("username");
        String customToken = request.getParameter("token");

        if (validateCustomToken(username, customToken)) {
            context.setSubject(AuthenticatedUser.createLocalAuthenticatedUserFromSubjectIdentifier(username));
            return AuthenticatorFlowStatus.SUCCESS_COMPLETED;
        }

        return AuthenticatorFlowStatus.INCOMPLETE;
    }

    private boolean validateCustomToken(String username, String token) {
        // Validate against custom token service
        return true;
    }
}
```

## High Availability

**Cluster Configuration:**

```toml
[clustering]
membership_scheme = "wka"
domain = "wso2.is.domain"
local_member_host = "192.168.1.10"
local_member_port = "4000"

[[clustering.members]]
hostname = "192.168.1.10"
port = "4000"

[[clustering.members]]
hostname = "192.168.1.11"
port = "4000"
```

## SSO Configuration

**SAML SSO:**

```xml
<!-- Service Provider Configuration -->
<ServiceProvider>
    <ApplicationName>MyApp</ApplicationName>
    <Issuer>https://myapp.example.com</Issuer>
    <AssertionConsumerServiceURL>
        https://myapp.example.com/saml/acs
    </AssertionConsumerServiceURL>
    <DefaultAssertionConsumerServiceURL>
        https://myapp.example.com/saml/acs
    </DefaultAssertionConsumerServiceURL>
    <NameIDFormat>
        urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress
    </NameIDFormat>
    <SignResponse>true</SignResponse>
    <SignAssertion>true</SignAssertion>
</ServiceProvider>
```

## Best Practices

- Use external database for production
- Enable SSL/TLS
- Implement clustering for HA
- Regular security updates
- Monitor performance

## Resources

- **Docs**: https://is.docs.wso2.com/
- **Related**: `wso2-identity-platform`, `wso2-asgardeo`

---

**Last Updated**: March 2025
