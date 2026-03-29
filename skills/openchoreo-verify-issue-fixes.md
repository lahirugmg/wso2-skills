---
name: openchoreo-verify-issue-fixes
description: Comprehensive guide for verifying bug fixes and feature implementations in OpenChoreo, including testing strategies and verification workflows
---

# OpenChoreo Verify Issue Fixes

This skill provides structured approaches for verifying that bug fixes and feature implementations work correctly in OpenChoreo, ensuring changes are ready for PR submission.

## When to Use This Skill

- After fixing a bug in OpenChoreo
- After implementing a new feature
- Before submitting a pull request
- Testing specific GitHub issues
- Validating changes work as expected

## Verification Workflow

```
1. Understand the Issue
   ↓
2. Identify Affected Components
   ↓
3. Deploy Changes Locally
   ↓
4. Execute Test Scenarios
   ↓
5. Verify Fix Works
   ↓
6. Run Automated Tests
   ↓
7. Check for Regressions
   ↓
8. Document Verification
```

## Phase 1: Understanding the Issue

### Analyze the GitHub Issue

```bash
# Use gh CLI to fetch issue details
gh issue view <issue-number> --repo openchoreo/openchoreo

# Example:
gh issue view 2909 --repo openchoreo/openchoreo
```

### Key Questions to Answer:

1. **What is broken?** - Identify the bug or missing feature
2. **Where is it broken?** - UI, API, Controller, Observer?
3. **How to reproduce?** - Steps to trigger the issue
4. **Expected behavior?** - What should happen instead?
5. **Affected users?** - Who is impacted by this issue?

### Document Test Plan

Create a simple test plan:

```markdown
## Test Plan for Issue #2909

### Issue: Add back button to API list page

### Affected Component: Backstage UI

### Test Scenarios:
1. Navigate to API registration page
2. Verify back button is present
3. Click back button
4. Verify navigates to API list page

### Expected Results:
- Back button visible on API registration page
- Clicking back button returns to API list
- Navigation is smooth with no errors
```

## Phase 2: Identify Affected Components

### Component Mapping

| Issue Type | Likely Component | Files to Check |
|------------|------------------|----------------|
| UI Bug | Backstage | `backstage/plugins/*` |
| API Issue | OpenChoreo API | `internal/server/*` |
| Controller Bug | Manager | `internal/controller/*` |
| CRD Issue | API Definitions | `api/v1alpha1/*` |
| Observability | Observer | `observer/*` |
| CLI Issue | OCC | `cmd/occ/*` |

### Find Related Code

```bash
# Search for relevant files
cd openchoreo

# For UI issues (#2909 - back button)
find backstage -name "*.tsx" -o -name "*.ts" | xargs grep -l "API.*register"

# For controller issues
find internal/controller -name "*.go" | xargs grep -l "Project"

# For API issues
find internal/server -name "*.go" | xargs grep -l "handler"
```

## Phase 3: Deploy Changes Locally

### Option A: Full Redeployment

```bash
# For k3d
make k3d.down
make k3d

# For Rancher
helm uninstall openchoreo-control-plane -n openchoreo-control-plane
make docker-build
helm install openchoreo-control-plane ./charts/openchoreo-control-plane -n openchoreo-control-plane
```

### Option B: Update Specific Component

```bash
# For k3d
make k3d.update.controller      # Backend controller changes
make k3d.update.openchoreo-api  # API changes
make k3d.update.backstage       # UI changes

# For Rancher
make docker-build
kubectl rollout restart deployment/<component-name> -n openchoreo-control-plane
```

### Verify Deployment

```bash
# Check pods are running
kubectl get pods -n openchoreo-control-plane

# Wait for pod to be ready
kubectl wait --for=condition=ready pod \
  -l app=openchoreo-backstage \
  -n openchoreo-control-plane \
  --timeout=300s

# Check logs for errors
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-backstage \
  --tail=50
```

## Phase 4: Verification by Component Type

### UI Bug Fixes (Backstage)

**Example: Issue #2909 - Add back button**

```bash
# 1. Access Backstage UI
# k3d: http://openchoreo.localhost:8080
# Rancher: kubectl port-forward -n openchoreo-control-plane svc/openchoreo-backstage 7007:7007
#          Then: http://localhost:7007

# 2. Navigate to the affected page
# - Open browser
# - Go to API registration page

# 3. Verify the fix
# - Check back button is visible
# - Click the back button
# - Verify navigation works

# 4. Test edge cases
# - Try keyboard navigation (Esc key, browser back button)
# - Test on different screen sizes (responsive design)
# - Check browser console for errors (F12)

# 5. Check logs
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-backstage \
  -f --tail=100
```

**Verification Checklist for UI:**
- [ ] Visual change is visible
- [ ] Functionality works as expected
- [ ] No console errors in browser
- [ ] No regression in related features
- [ ] Responsive design intact
- [ ] Accessibility considerations met

### Controller Bug Fixes

**Example: Issue #2914 - Component deletion redirect**

```bash
# 1. Create test resources
kubectl apply -f - <<EOF
apiVersion: core.choreo.dev/v1alpha1
kind: Project
metadata:
  name: test-project
  namespace: default
spec:
  displayName: "Test Project"
---
apiVersion: core.choreo.dev/v1alpha1
kind: Component
metadata:
  name: test-component
  namespace: default
spec:
  projectRef:
    name: test-project
  type: Service
EOF

# 2. Watch controller logs
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-controller-manager \
  -f --tail=100 &

# 3. Trigger the fix (delete component via UI)
# - Access Backstage
# - Navigate to test-component
# - Delete the component
# - Verify redirects to parent project

# 4. Verify in Kubernetes
kubectl get components -n default
kubectl get projects -n default

# 5. Check controller handled deletion properly
# - Check logs for deletion events
# - Verify finalizers were processed
# - Check related resources cleaned up
```

**Verification Checklist for Controller:**
- [ ] Resource reconciliation successful
- [ ] No error logs
- [ ] Status conditions updated correctly
- [ ] Finalizers handled properly
- [ ] Related resources updated/deleted as expected
- [ ] No infinite reconciliation loops

### API Bug Fixes

**Example: API endpoint returns correct data**

```bash
# 1. Port-forward API service (if needed)
kubectl port-forward -n openchoreo-control-plane \
  svc/openchoreo-api 8080:8080 &

# 2. Test the API endpoint
curl -X GET http://localhost:8080/api/v1/projects \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  | jq .

# 3. Verify response
# - Check HTTP status code (should be 200)
# - Verify response structure matches spec
# - Check all expected fields are present
# - Verify data accuracy

# 4. Test edge cases
# - Invalid input
curl -X GET http://localhost:8080/api/v1/projects/nonexistent

# - Missing auth
curl -X GET http://localhost:8080/api/v1/projects

# - Malformed request
curl -X POST http://localhost:8080/api/v1/projects \
  -H "Content-Type: application/json" \
  -d '{"invalid": json}'

# 5. Check API logs
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-api \
  -f --tail=100
```

**Verification Checklist for API:**
- [ ] Correct HTTP status codes
- [ ] Response format matches API spec
- [ ] Data accuracy verified
- [ ] Error handling works correctly
- [ ] Authentication/authorization enforced
- [ ] API logs show expected behavior

### Configuration/Secrets Bug Fixes

**Example: Issue #2142 - Remove unused secrets**

```bash
# 1. Fresh install without Jenkins
helm uninstall openchoreo-control-plane -n openchoreo-control-plane

# 2. Install without Jenkins configuration
helm install openchoreo-control-plane ./charts/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --set jenkins.enabled=false

# 3. Verify installation succeeds
kubectl get pods -n openchoreo-control-plane

# 4. Check no dummy secrets required
kubectl get secrets -n openchoreo-control-plane

# 5. Verify Backstage starts without Jenkins secrets
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-backstage \
  --tail=100 | grep -i jenkins

# Should not see errors about missing Jenkins secrets
```

**Verification Checklist for Configuration:**
- [ ] Installation succeeds without dummy values
- [ ] No error messages about missing config
- [ ] Optional features truly optional
- [ ] Documentation updated to reflect changes
- [ ] Backward compatibility maintained

### Documentation Updates

**Example: Issue #2969 - PostgreSQL documentation**

```bash
# 1. Follow the documentation step-by-step
# - Open the new documentation
# - Execute each command
# - Verify each step works

# 2. Test setup from scratch
# - Fresh environment
# - Follow docs exactly as written
# - Note any issues or unclear steps

# 3. Verify configuration works
# - PostgreSQL connection successful
# - Observer can query database
# - Data is being stored correctly

# 4. Test troubleshooting section
# - Intentionally create common issues
# - Follow troubleshooting steps
# - Verify solutions work
```

**Verification Checklist for Documentation:**
- [ ] All steps are clear and accurate
- [ ] Commands work as written
- [ ] No missing prerequisites
- [ ] Examples are complete and tested
- [ ] Troubleshooting section covers common issues
- [ ] Links are valid

## Phase 5: Automated Testing

### Run Unit Tests

```bash
# Run all tests
make test

# Run specific package tests
go test ./internal/controller/project/... -v

# Run with coverage
make test-coverage

# Check coverage report
go tool cover -html=coverage.out
```

### Run Integration Tests

```bash
# If integration tests exist
make test-integration

# Or specific integration tests
go test ./test/integration/... -v -tags=integration
```

### Run End-to-End Tests

```bash
# If e2e tests exist
make test-e2e

# Manual e2e testing
# - Create project
# - Create component
# - Deploy component
# - Access via endpoint
# - Delete component
# - Delete project
```

## Phase 6: Regression Testing

### Check Related Functionality

```bash
# Identify related features that might be affected
# Test them to ensure no regressions

# Example: If you fixed component deletion:
# - Test component creation still works
# - Test component updates still work
# - Test project deletion still works
# - Test component listing still works
```

### Common Regression Scenarios

1. **UI Changes:**
   - Test other pages using similar components
   - Check navigation still works everywhere
   - Verify styling is consistent

2. **Controller Changes:**
   - Test other CRD reconciliations
   - Verify watches still trigger correctly
   - Check status updates work

3. **API Changes:**
   - Test related API endpoints
   - Verify authentication still works
   - Check pagination, filtering, sorting

## Phase 7: Performance Verification

### Check Performance Impact

```bash
# Monitor resource usage
kubectl top pods -n openchoreo-control-plane

# Check response times (for API changes)
time curl http://localhost:8080/api/v1/projects

# Monitor logs for performance issues
kubectl logs -n openchoreo-control-plane \
  deployment/openchoreo-controller-manager \
  | grep -i "slow\|timeout\|performance"
```

### Load Testing (if applicable)

```bash
# Simple load test for API
for i in {1..100}; do
  curl -s http://localhost:8080/api/v1/projects > /dev/null &
done
wait

# Check API still responsive
curl http://localhost:8080/api/v1/health
```

## Phase 8: Documentation Verification

### Verify Changes Documented

```bash
# Check if documentation was updated
git diff main -- docs/ README.md

# If user-facing change, ensure:
# - README updated (if needed)
# - User guide updated
# - API docs updated (if API changed)
# - Migration guide (if breaking change)
```

## Verification Report Template

Create a verification report in your PR description:

```markdown
## Verification Report for Issue #<number>

### Issue Description
<!-- Brief description of the issue -->

### Changes Made
<!-- List of changes made to fix the issue -->

### Verification Steps Performed

#### Manual Testing
- [x] Verified fix works as expected
- [x] Tested on Backstage UI at http://localhost:7007
- [x] No console errors observed
- [x] Navigation works correctly

#### Automated Testing
- [x] Unit tests pass: `make test`
- [x] Linting passes: `make lint`
- [x] Code generation up to date: `make code.gen-check`

#### Regression Testing
- [x] Related features still work
- [x] No new errors in logs
- [x] Performance not impacted

### Screenshots/Evidence
<!-- Add screenshots or logs showing the fix works -->

### Test Environment
- Platform: Rancher Desktop / k3d
- Kubernetes Version: v1.30.2
- OpenChoreo Version: main branch, commit abc123

### Additional Notes
<!-- Any additional observations -->
```

## Quick Reference

### Verification Checklist

```bash
# Pre-verification
[ ] Issue understood
[ ] Test plan created
[ ] Component identified

# Deployment
[ ] Changes deployed locally
[ ] Pods running successfully
[ ] No errors in logs

# Testing
[ ] Manual test scenarios passed
[ ] Automated tests passed
[ ] Regression tests passed
[ ] Performance acceptable

# Documentation
[ ] Verification report created
[ ] Screenshots/evidence captured
[ ] Ready for PR submission
```

### Common Verification Commands

```bash
# Check deployment
kubectl get pods -n openchoreo-control-plane

# View logs
kubectl logs -n openchoreo-control-plane deployment/<component> -f

# Run tests
make test

# Access UI
kubectl port-forward -n openchoreo-control-plane svc/openchoreo-backstage 7007:7007

# Test API
curl http://localhost:8080/api/v1/<endpoint>

# Check resources
kubectl get projects,components -A
```

## Success Criteria

- ✅ Issue is fully understood
- ✅ Fix verified manually
- ✅ Automated tests pass
- ✅ No regressions detected
- ✅ Performance acceptable
- ✅ Documentation complete
- ✅ Verification report created
- ✅ Ready for PR submission

## Related Skills

- `openchoreo-local-development` - Development workflow
- `openchoreo-build-and-deploy` - Building and deploying
- `openchoreo-rancher-setup` - Environment setup
