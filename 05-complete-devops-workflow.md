# Complete DevOps Workflow

[← Back to Guide Overview](./README.md)

---

## Table of Contents

- [Overview: Local Dev → PR → Test → Publish → Consume](#overview-local-dev--pr--test--publish--consume)
- [Phase 1: Local Development](#phase-1-local-development)
- [Phase 2: Pull Request](#phase-2-pull-request)
- [Phase 3: Automatic Publishing](#phase-3-automatic-publishing)
- [Phase 4: Terraform Cloud Testing](#phase-4-terraform-cloud-testing)
- [Phase 5: Module Consumption](#phase-5-module-consumption)
- [Version Management Strategies](#version-management-strategies)
- [Complete Workflow Diagram](#complete-workflow-diagram)
- [Cost and Time Breakdown](#cost-and-time-breakdown)

---

## Overview: Local Dev → PR → Test → Publish → Consume

```
[Developer] → [Feature Branch] → [Pull Request] → [CI Validation] → [Merge] → [Auto-Publish] → [TFC Tests] → [Consumers]
```

This workflow ensures quality at every stage while maintaining velocity.

---

## Phase 1: Local Development

```bash
# 1. Clone module repository
git clone https://github.com/my-org/terraform-azurerm-storage.git
cd terraform-azurerm-storage

# 2. Create feature branch
git checkout -b feature/add-private-endpoint

# 3. Install pre-commit hooks
pre-commit install

# 4. Make changes
# Edit main.tf, variables.tf, outputs.tf

# 5. Update tests
# Edit tests/unit-tests.tftest.hcl
# Edit tests/integration-tests.tftest.hcl

# 6. Run unit tests locally (fast feedback)
terraform init
terraform test -filter=tests/unit-tests.tftest.hcl

# 7. Run integration tests (optional, if credentials available)
export ARM_SUBSCRIPTION_ID="..."
terraform test -filter=tests/integration-tests.tftest.hcl

# 8. Stage and commit (pre-commit hooks run automatically)
git add .
git commit -m "feat: add private endpoint support"
# Pre-commit runs:
# - terraform fmt (auto-fixes formatting)
# - terraform validate (checks syntax)
# - tflint (checks best practices)
# - terraform-docs (updates README)
# - tfsec/trivy (security scan)
# - terraform test (unit tests only)

# 9. Push to remote
git push origin feature/add-private-endpoint
```

**Key Points**:
- Pre-commit hooks provide immediate feedback
- Unit tests run locally (fast, free)
- Integration tests optional (slow, costs money)
- All formatting/docs auto-fixed before commit

---

## Phase 2: Pull Request

```bash
# 1. Create PR
gh pr create --title "Add private endpoint support" --body "Adds ability to deploy private endpoints"

# 2. Add semantic version label
# Go to GitHub PR page → Labels → Select "semver:minor"
# (New feature = minor version bump)

# 3. CI/CD validation runs automatically
# GitHub Actions workflow: module_validate.yml
# Jobs:
# - Terraform format check
# - Terraform validate
# - TFLint
# - tfsec/trivy security scan
# - terraform-docs (updates README)
# - Unit tests (terraform test -filter=tests/unit-tests.tftest.hcl)
# - Integration tests (if cloud credentials configured)

# 4. Review check results
# Green checkmarks = all validations passed
# Red X = something failed, fix and push again

# 5. Code review
# Team members review code, suggest changes
# Make changes, commit, push (CI re-runs automatically)

# 6. Approval
# Reviewer approves PR

# 7. Merge
gh pr merge --merge
```

**Semantic Version Labels**:
- `semver:major` - Breaking changes (1.0.0 → 2.0.0)
- `semver:minor` - New features (1.0.0 → 1.1.0)
- `semver:patch` - Bug fixes (1.0.0 → 1.0.1)

---

## Phase 3: Automatic Publishing

```bash
# GitHub Actions workflow: pr_merge.yml
# Triggered on: PR merged to main

# Workflow steps:
# 1. Detect semantic version label (semver:major/minor/patch)
# 2. Call Python script: get_module_version.py
#    - Queries TFC API for current module version
#    - Calculates new version based on label
#    - Example: 1.0.0 + semver:minor = 1.1.0
# 3. Call Python script: publish_module_version.py
#    - Publishes new version to TFC Private Module Registry
#    - Uses VCS commit SHA as version reference
# 4. TFC automatically:
#    - Downloads code from Git
#    - Runs terraform init
#    - Runs terraform test (all tests)
#    - Publishes version if tests pass
#    - Marks version as "failed" if tests fail
```

**Configuration required** (GitHub repository secrets/variables):
```yaml
# Variables
TFE_ORG: "my-terraform-org"
TFE_MODULE: "storage-account"
TFE_PROVIDER: "azurerm"

# Secrets
TFE_TOKEN: "xxxxxxxxxx"  # TFC API token with module:write permissions
```

**What happens automatically**:
1. PR merged → workflow triggers
2. Version calculated (e.g., 1.0.0 → 1.1.0)
3. Module published to PMR
4. Git tag created (v1.1.0)
5. Tests run in TFC
6. Version marked as ready (or failed)

---

## Phase 4: Terraform Cloud Testing

**TFC automatically runs tests when module published**:

```
Module published → TFC triggers test workflow
↓
1. terraform init
2. terraform test -filter=tests/unit-tests.tftest.hcl  (fast)
3. terraform test -filter=tests/integration-tests.tftest.hcl  (if credentials configured)
4. Results displayed in TFC UI:
   - ✅ All tests passed → Version marked as "ready"
   - ❌ Tests failed → Version marked as "failed", not recommended for use
```

**Test results visible**:
- TFC UI: Navigate to Registry → Module → Version → Tests
- Shows pass/fail for each test
- Shows detailed error messages for failures

**Configuration required** (TFC workspace):
```hcl
# For integration tests to run in TFC:
# Workspace → Variables → Add:
ARM_CLIENT_ID = "..."          # Azure service principal
ARM_CLIENT_SECRET = "..."      # (sensitive)
ARM_TENANT_ID = "..."
ARM_SUBSCRIPTION_ID = "..."
```

**Benefits of TFC testing**:
- Consistent environment (not developer's laptop)
- Automated on every publish
- Visible to entire team
- Failed versions clearly marked

---

## Phase 5: Module Consumption

**Application team consumes published module**:

```hcl
# project-a/main.tf
module "storage_account" {
  source  = "app.terraform.io/my-org/storage-account/azurerm"
  version = "1.1.0"  # Pin to specific version
  
  name                     = var.storage_account_name
  resource_group_name      = var.resource_group_name
  location                 = var.location
  enable_private_endpoint  = true
  subnet_id                = var.subnet_id
  
  tags = {
    Environment = "Production"
    ManagedBy   = "Terraform"
  }
}

output "storage_account_id" {
  value = module.storage_account.id
}
```

**Consumption workflow**:
```bash
# 1. Initialize (downloads module from PMR)
terraform init

# 2. Plan
terraform plan

# 3. Apply
terraform apply

# 4. Later: Upgrade to new version
# Edit main.tf: version = "1.1.0" -> version = "1.2.0"
terraform init -upgrade
terraform plan   # Review changes
terraform apply
```

---

## Version Management Strategies

### Strategy 1: Strict Pinning (Recommended for Production)

```hcl
module "storage" {
  source  = "app.terraform.io/my-org/storage/azurerm"
  version = "1.1.0"  # Exact version
}
```

**Pros**:
- Predictable - no surprises
- Explicit control over upgrades
- Easy to audit ("Which version are we on?")

**Cons**:
- Manual updates required
- Miss bug fixes automatically

**Use for**: Production environments

---

### Strategy 2: Patch Updates Allowed

```hcl
module "storage" {
  source  = "app.terraform.io/my-org/storage/azurerm"
  version = "~> 1.1.0"  # 1.1.x (patch updates only)
}
```

**Pros**:
- Automatic bug fixes
- No breaking changes

**Cons**:
- Slight unpredictability
- Need to test before production

**Use for**: Development/staging environments

---

### Strategy 3: Minor Updates Allowed (Not Recommended for Production)

```hcl
module "storage" {
  source  = "app.terraform.io/my-org/storage/azurerm"
  version = "~> 1.0"  # 1.x (minor + patch updates)
}
```

**Pros**:
- Get new features automatically

**Cons**:
- Unexpected behavior changes
- Higher risk

**Use for**: Sandbox/experimental environments only

---

## Complete Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│ Local Development                                                    │
├─────────────────────────────────────────────────────────────────────┤
│ 1. Developer creates feature branch                                  │
│ 2. Makes code changes (main.tf, variables.tf, tests/)               │
│ 3. Runs local unit tests (terraform test)                           │
│ 4. Commits (pre-commit hooks run: fmt, validate, lint, test)        │
│ 5. Pushes to GitHub                                                  │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│ Pull Request Validation (CI)                                         │
├─────────────────────────────────────────────────────────────────────┤
│ GitHub Actions: module_validate.yml                                  │
│ ✅ terraform fmt -check                                              │
│ ✅ terraform validate                                                │
│ ✅ tflint                                                            │
│ ✅ tfsec/trivy                                                       │
│ ✅ terraform-docs (updates README)                                   │
│ ✅ Unit tests (terraform test -filter=unit)                         │
│ ✅ Integration tests (optional, if credentials configured)          │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│ Code Review                                                          │
├─────────────────────────────────────────────────────────────────────┤
│ 1. Team reviews code                                                 │
│ 2. Suggests changes                                                  │
│ 3. Developer addresses feedback                                      │
│ 4. Approval granted                                                  │
│ 5. Developer adds semantic version label (semver:major/minor/patch) │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│ Merge & Automatic Publishing                                         │
├─────────────────────────────────────────────────────────────────────┤
│ GitHub Actions: pr_merge.yml                                         │
│ 1. Detects merge to main                                             │
│ 2. Reads semver label from PR                                        │
│ 3. Queries TFC for current version                                   │
│ 4. Calculates new version (e.g., 1.0.0 + minor = 1.1.0)            │
│ 5. Publishes to TFC Private Module Registry                         │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│ TFC Automatic Testing                                                │
├─────────────────────────────────────────────────────────────────────┤
│ 1. TFC downloads module from Git                                     │
│ 2. Runs terraform init                                               │
│ 3. Runs all tests (unit + integration)                              │
│ 4. Publishes version if tests pass                                   │
│ 5. Marks version as "failed" if tests fail                          │
└────────────────────────┬────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│ Module Available for Consumption                                     │
├─────────────────────────────────────────────────────────────────────┤
│ App teams can now use the new version:                               │
│ module "storage" {                                                    │
│   source  = "app.terraform.io/my-org/storage/azurerm"               │
│   version = "1.1.0"                                                  │
│ }                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Cost and Time Breakdown

### Cost Breakdown

| Phase | Cost | Duration |
|-------|------|----------|
| Local development | $0 (unit tests only) | 5-30 min |
| PR validation (CI) | $0 (GitHub Actions free tier) | 2-5 min |
| Integration tests in CI | $$ (cloud resources) | 5-10 min |
| TFC automatic testing | $$$ (cloud resources) | 5-15 min |
| Module consumption | $$$$ (actual infrastructure) | Varies |

**Cost optimization tips**:
1. Run unit tests (command=plan) frequently - they're free
2. Run integration tests (command=apply) less frequently - they cost money
3. Use smallest/cheapest SKUs for testing (e.g., Basic instead of Premium)
4. Clean up test resources automatically (terraform test does this)
5. Use dedicated test subscriptions with spending alerts

---

### Time Breakdown

**End-to-end timeline for new module version**:

```
Feature development: 2-8 hours (coding + testing)
PR creation: 5 minutes
CI validation: 5 minutes
Code review: 30 minutes - 2 days (depends on team)
Merge: 1 minute
Auto-publish: 2 minutes
TFC testing: 10 minutes
──────────────────────────────────────────────────
Total: 3 hours - 3 days (mostly waiting for review)
```

**Rapid iteration for bug fixes**:

```
Identify bug: 10 minutes
Create hotfix branch: 1 minute
Fix code: 10 minutes
Local test: 5 minutes
Commit + push: 2 minutes
Create PR with semver:patch: 2 minutes
CI validation: 5 minutes
Fast-track approval: 5 minutes
Merge: 1 minute
Auto-publish: 2 minutes
TFC testing: 10 minutes
──────────────────────────────────────────────────
Total: 53 minutes (can be faster with emergency process)
```

---

## Best Practices

### ✅ DO:
- Use pre-commit hooks for fast feedback
- Run unit tests frequently
- Pin module versions in production
- Use semantic versioning labels
- Automate publishing via CI/CD
- Test in TFC before marking ready
- Document breaking changes

### ❌ DON'T:
- Skip testing before merge
- Manually upload modules
- Use latest/unpinned versions in prod
- Bypass code review for changes
- Forget to add semver labels
- Skip integration tests completely

---

## Next Steps

- **Learn about module structure**: Read [Module Structure Anti-Patterns](./01-module-structure.md)
- **Understand testing deeply**: See [Testing Strategy Deep Dive](./03-testing-strategy.md)
- **Handle failures properly**: Check [Rollback and State Management](./04-rollback-state-management.md)

---

[← Previous: Rollback & State](./04-rollback-state-management.md) | [Back to Guide Overview](./README.md)

**Document Version**: 2.0  
**Last Updated**: November 2025
