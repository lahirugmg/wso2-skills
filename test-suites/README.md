# OpenChoreo Test Suite Format

## Overview

Test suites are structured collections of prompts that validate skills and guide step-by-step execution of complex tasks. Each test suite is a markdown file with a specific format.

## Test Suite Structure

```yaml
---
# Metadata
suite_name: "OpenChoreo K3d Setup"
description: "Complete setup and validation of OpenChoreo on K3d"
skills_tested:
  - openchoreo-devops
  - openchoreo-networking
  - openchoreo-observability
difficulty: intermediate
estimated_duration: 30-45 minutes
prerequisites:
  - Docker Desktop installed
  - 8GB RAM available
  - kubectl installed
  - helm installed
environment:
  platform: local
  runtime: k3d
version: 1.0.0
---
```

## Test Format

Each test follows this structure:

### Test Definition

```markdown
## Test 1: [Test Name]

**Objective**: [What this test validates]

**Skill**: `skill-name`

**Prerequisites**:
- Prerequisite 1
- Prerequisite 2

**Prompt**:
```
[The exact prompt to give to Claude Code with the skill]
```

**Expected Outcome**:
- Outcome 1
- Outcome 2

**Validation Commands**:
```bash
# Commands to verify the test passed
command1
command2
```

**Success Criteria**:
- [ ] Criteria 1
- [ ] Criteria 2

**Troubleshooting**:
If X happens, do Y
```

## Test Types

1. **Setup Tests**: Install and configure components
2. **Validation Tests**: Verify components are working
3. **Integration Tests**: Test component interactions
4. **Operational Tests**: Test operational procedures
5. **Cleanup Tests**: Tear down and cleanup

## Running a Test Suite

1. Open the test suite file
2. Read the metadata and prerequisites
3. Execute tests in order
4. After each test, run validation commands
5. Check all success criteria before proceeding
6. Document any issues encountered

## Example Test Suite Structure

```
test-suites/
├── README.md                           # This file
├── openchoreo-k3d-setup.md            # Complete K3d setup
├── openchoreo-podman-setup.md         # Podman variant
├── openchoreo-app-deployment.md       # Deploy test app
├── openchoreo-monitoring-setup.md     # Setup observability
├── openchoreo-gitops-workflow.md      # Test GitOps flow
└── openchoreo-disaster-recovery.md    # DR procedures
```

## Test Suite Naming Convention

- `{component}-{scenario}-{variant}.md`
- Examples:
  - `openchoreo-k3d-setup.md`
  - `openchoreo-podman-setup.md`
  - `openchoreo-microservice-deployment.md`
  - `openchoreo-canary-deployment.md`

## Success Tracking

Each test suite should include a completion checklist:

```markdown
## Test Suite Completion

- [ ] All prerequisites met
- [ ] All tests passed
- [ ] All validation commands successful
- [ ] No errors in logs
- [ ] Performance within acceptable range
- [ ] Cleanup completed
```

## Test Data

Test suites may include sample data in accompanying files:

```
test-suites/
├── openchoreo-k3d-setup.md
└── data/
    ├── sample-app/
    │   ├── catalog-info.yaml
    │   ├── deployment.yaml
    │   └── service.yaml
    └── configs/
        └── values.yaml
```

## Reporting

After running a test suite, document results:

```markdown
## Test Run Report

- **Date**: 2025-03-26
- **Environment**: macOS, Podman 5.0
- **Duration**: 42 minutes
- **Tests Passed**: 10/10
- **Issues Found**: None
- **Notes**: All tests completed successfully
```
