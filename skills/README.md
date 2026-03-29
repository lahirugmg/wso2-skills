# WSO2 Skills Library

This directory contains comprehensive skills for working with WSO2 products and OpenChoreo platform.

## Skills Inventory

### OpenChoreo Specialized Skills (11)

Detailed, focused skills for OpenChoreo platform operations:

| Skill | Size | Focus |
|-------|------|-------|
| `openchoreo-devops.md` | 18KB | Infrastructure setup, K3d/Docker/Podman, cluster management |
| `openchoreo-architect.md` | 17KB | System design, microservices patterns, reference architectures |
| `openchoreo-app-developer.md` | 14KB | Application development, service implementation |
| `openchoreo-sre.md` | 13KB | Site reliability, monitoring, incident response |
| `openchoreo-platform-engineer.md` | 9.4KB | Golden paths, developer experience, templates |
| `openchoreo-gitops.md` | 3.1KB | Argo CD, progressive delivery, deployments |
| `openchoreo-observability.md` | 3.3KB | Prometheus, Grafana, Jaeger, logging |
| `openchoreo-networking.md` | 3.3KB | Cilium, network policies, service mesh |
| `openchoreo-security.md` | 2.5KB | RBAC, secrets, security hardening |
| `openchoreo-troubleshooting.md` | 2.9KB | Debugging, diagnostics, problem resolution |
| `wso2-openchoreo.md` | 6.3KB | General OpenChoreo overview |

### WSO2 Platform Skills (5)

Core platform-level skills:

| Skill | Size | Focus |
|-------|------|-------|
| `wso2-agent-platform.md` | 18KB | AI agents, RAG systems, agent management |
| `wso2-api-platform.md` | 27KB | API lifecycle, AI Gateway, MCP integration |
| `wso2-integration-platform.md` | 25KB | Integration patterns, BI/MI/SI/ICP |
| `wso2-identity-platform.md` | 24KB | IAM, authentication, AI agent security |
| `wso2-engineering-platform.md` | 23KB | Choreo, Ballerina, platform engineering |

### WSO2 Product Skills (9)

Product-specific detailed skills:

| Skill | Size | Focus |
|-------|------|-------|
| `wso2-agent-manager.md` | 27KB | Agent lifecycle, governance, observability |
| `wso2-api-manager.md` | 21KB | API Manager operations, AI features |
| `wso2-micro-integrator.md` | 9.4KB | ESB patterns, connectors, deployment |
| `wso2-business-integrator.md` | 4.2KB | Low-code integration, AI copilot |
| `wso2-streaming-integrator.md` | 4.2KB | Siddhi, real-time processing, CEP |
| `wso2-identity-server.md` | 3.8KB | On-premise IAM, LDAP integration |
| `wso2-asgardeo.md` | 5.5KB | Cloud IAM, CIAM, passwordless auth |
| `wso2-choreo.md` | 5.1KB | Choreo platform operations, IDP |
| `wso2-ballerina.md` | 6.8KB | Ballerina language, network services |

**Total: 25 comprehensive WSO2 skills**

## Usage

### In Claude Code

Reference skills in your prompts:

```
"Set up OpenChoreo on Podman - use openchoreo-devops skill"
"Design a microservices architecture - use openchoreo-architect skill"
"Deploy a Go service - use openchoreo-app-developer skill"
"Configure monitoring - use openchoreo-observability skill"
```

### Skill Invocation

The skills are automatically available in Claude Code when placed in `.claude/skills/` directory. This copy in `wso2-skill/skills/` serves as:
- Documentation and reference
- Backup copy
- Version control
- Distribution to team members

## Skill Categories

### 🏗️ Infrastructure & Setup
- openchoreo-devops
- openchoreo-networking
- openchoreo-security

### 🎨 Design & Development
- openchoreo-architect
- openchoreo-app-developer
- openchoreo-platform-engineer

### 🔧 Operations & Reliability
- openchoreo-sre
- openchoreo-gitops
- openchoreo-observability
- openchoreo-troubleshooting

### 🔌 WSO2 Products
- All wso2-* skills for specific product operations

## Learning Paths

### Path 1: Platform Engineer
1. openchoreo-devops
2. openchoreo-architect
3. openchoreo-platform-engineer
4. openchoreo-gitops

### Path 2: Application Developer
1. openchoreo-devops
2. openchoreo-app-developer
3. openchoreo-gitops
4. openchoreo-observability

### Path 3: SRE
1. openchoreo-devops
2. openchoreo-sre
3. openchoreo-observability
4. openchoreo-troubleshooting

## Related Directories

- `../test-suites/` - Test suites for validating skills
- `../../.claude/skills/` - Active skills directory used by Claude Code

## Contributing

When updating skills:
1. Edit in `.claude/skills/`
2. Test with Claude Code
3. Copy to `wso2-skill/skills/` for distribution
4. Update this README if adding new skills

## Version

Last Updated: March 2025
Skills Version: 1.0.0
