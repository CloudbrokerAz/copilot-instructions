# Security & Local Development

## 🎯 Purpose

This document addresses the common security concern that leads to "no local Terraform execution" policies. It provides alternatives that improve both security AND developer productivity, challenging the false trade-off between security and developer experience.

---

## 🔒 Understanding the Security Concern

### **The Common Policy**

Many organizations implement:
> "Developers may not run Terraform on their local machines. All Terraform execution must occur in CI/CD pipelines."

### **The Stated Reason**

- **Fear of API key leakage** - Developers might accidentally commit credentials to Git
- **Lack of auditability** - No logs of what was executed locally
- **Inconsistent execution** - Different versions/configurations on different machines
- **Shadow IT prevention** - Ensure all infrastructure goes through approved process

### **The Unintended Consequences**

- ⏱️ **Slow feedback loops** - 10-15 minute pipeline runs for simple validation
- 🐛 **Difficult debugging** - Can't reproduce issues locally
- 😤 **Developer frustration** - Waiting for pipelines kills flow state
- 🔄 **More commits** - "Fix typo," "Fix another typo," "Actually fix it this time"
- 🚫 **No testing** - Can't run unit tests locally without cloud access

---

## 🗣️ Discussion Guide: Understanding the Policy

### **Root Cause Questions**

**1. "Walk me through the security concern that led to the no-local-execution policy."**

*Why this matters*: Identify root concern to propose targeted mitigations rather than blanket restrictions.

**Follow-up probes:**
- "Was there a specific incident that triggered this?"
- "Is this a compliance requirement or internal policy?"
- "Who mandated this policy?"
- "When was it last reviewed?"

**2. "Has the no-local-execution policy successfully prevented API key leakage?"**

*Why this matters*: Challenge the assumption. If incidents still occur, policy isn't working. Opens discussion on better solutions.

**Follow-up probes:**
- "Have there been any credential leakage incidents since this policy?"
- "Are developers finding workarounds?"
- "What unintended consequences have emerged?"

**3. "What's the security team's core concern—accidental commits, developer machines, or something else?"**

*Why this matters*: Different concerns need different solutions.

| Concern | Better Solution |
|---------|-----------------|
| **Accidental commits** | Git hooks, pre-commit scanning, `.gitignore`, encrypted state |
| **Developer machines** | Endpoint security, dynamic credentials, short TTLs |
| **Auditability** | Centralized logging, remote execution, Terraform Cloud |
| **Inconsistent execution** | Docker/containers, version pinning, remote execution |

**4. "How are cloud credentials managed today for pipeline execution?"**

*Why this matters*: If pipelines have credentials, the security boundary isn't "local vs pipeline"—it's credential lifecycle management.

**Key insight**: If pipelines have long-lived credentials in environment variables, that's arguably WORSE than local development with short-lived tokens.

---

## ✅ Better Security Alternatives

### **Solution 1: Dynamic Short-Lived Credentials**

**How It Works:**
- Credentials are generated on-demand from Vault/OIDC/Workload Identity
- Expire in 15 minutes to 1 hour
- Scoped to specific resources/actions
- Automatically rotated

**Implementation Options:**

#### **Option A: HashiCorp Vault Dynamic Secrets**

```hcl
# Developer's Terraform configuration
provider "aws" {
  region = "us-east-1"
  # No hardcoded credentials!
}

# In terraform.tf or environment
terraform {
  backend "remote" {
    organization = "my-org"
    workspaces {
      name = "dev-workspace"
    }
  }
}

# Vault provides short-lived AWS credentials automatically
data "vault_aws_access_credentials" "creds" {
  backend = "aws"
  role    = "developer"
  ttl     = "15m"  # 15-minute expiration
}

provider "aws" {
  access_key = data.vault_aws_access_credentials.creds.access_key
  secret_key = data.vault_aws_access_credentials.creds.secret_key
  token      = data.vault_aws_access_credentials.creds.security_token
}
```

**Benefits:**
- ✅ No long-lived credentials to leak
- ✅ Credentials expire quickly
- ✅ Scoped permissions per role
- ✅ Full audit trail in Vault
- ✅ Works locally and in CI/CD

#### **Option B: OIDC/Workload Identity**

**For AWS:**
```hcl
# Assume role via OIDC (no credentials!)
provider "aws" {
  assume_role_with_web_identity {
    role_arn                = "arn:aws:iam::123456789:role/terraform-dev"
    web_identity_token_file = "/var/run/secrets/token"
    session_name            = "terraform-local"
  }
}
```

**For Azure:**
```hcl
# Managed Identity / Workload Identity
provider "azurerm" {
  use_oidc = true
  # Token comes from Azure CLI or environment
}
```

**For GCP:**
```hcl
# Workload Identity Federation
provider "google" {
  # Credentials come from gcloud CLI or Workload Identity
}
```

**Benefits:**
- ✅ No credentials at all—just trust federation
- ✅ Cannot be leaked (no secret to leak)
- ✅ Scoped to specific identity
- ✅ Auditabl in cloud provider logs

---

### **Solution 2: Terraform Testing with Mocked Providers**

**The Breakthrough: Terraform v1.7+ Mocking**

Developers can run **unit tests locally with NO cloud credentials**:

```hcl
# tests/unit/validation.tftest.hcl

# Mock the entire provider—no credentials needed!
mock_provider "aws" {}

variables {
  bucket_name = "test-bucket"
  region      = "us-east-1"
}

run "validate_bucket_naming" {
  command = plan  # Plan-only, no resources created

  assert {
    condition     = aws_s3_bucket.main.bucket == "test-bucket"
    error_message = "Bucket name doesn't match input"
  }

  assert {
    condition     = can(regex("^[a-z0-9-]+$", aws_s3_bucket.main.bucket))
    error_message = "Bucket name contains invalid characters"
  }
}
```

**Running the test:**
```bash
$ terraform test
# No AWS credentials required!
# No AWS resources created!
# Fast feedback (< 5 seconds)
```

**What Gets Tested Without Credentials:**
- ✅ Variable validation
- ✅ Resource configuration logic
- ✅ Conditional expressions
- ✅ Output values
- ✅ Module composition
- ✅ Naming conventions
- ✅ Tagging logic

**What Doesn't Get Tested:**
- ❌ Actual cloud API behavior (needs integration tests in CI/CD)

**Security Benefits:**
- ✅ Developers can test locally with zero cloud access
- ✅ No credentials to leak
- ✅ Fast feedback loop restored
- ✅ Integration tests still run in controlled environment

---

### **Solution 3: HCP Terraform Remote Execution + CLI Workflow**

**How It Works:**
- Developers run `terraform plan` locally
- Execution happens remotely in HCP Terraform
- Output streams back to developer
- No credentials on local machine

**Developer Experience:**

```bash
$ cd my-terraform-project

$ terraform init
# Configures remote backend

$ terraform plan
# Plan runs remotely in HCP Terraform
# Developer sees output in real-time
# No cloud credentials on local machine!

$ terraform apply
# Apply also runs remotely
# Requires approval
```

**Configuration:**

```hcl
# terraform.tf
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "dev-workspace"
    }
  }
}
```

**Security Benefits:**
- ✅ Credentials stay in HCP Terraform (never on local machine)
- ✅ Full audit trail of all runs
- ✅ Policy enforcement (Sentinel/OPA)
- ✅ Remote state management
- ✅ Fast feedback (streaming output)

**Developer Benefits:**
- ✅ Feels like local execution
- ✅ Can run `terraform plan` anytime
- ✅ Can troubleshoot with real output
- ✅ No pipeline waiting

---

### **Solution 4: Git Hooks & Pre-Commit Scanning**

**Prevent Accidental Commits:**

```bash
# .git/hooks/pre-commit

#!/bin/bash

# Scan for potential secrets
if grep -r "aws_access_key_id\|aws_secret_access_key\|password.*=" . --exclude-dir=.git; then
  echo "❌ ERROR: Potential secrets detected!"
  echo "Remove credentials before committing."
  exit 1
fi

# Scan with dedicated tools
gitleaks detect --source . || exit 1
trufflehog filesystem . || exit 1

# Validate Terraform
terraform fmt -check -recursive || exit 1
terraform validate || exit 1

echo "✅ Pre-commit checks passed"
```

**Tools to Use:**
- **GitLeaks** - Detects secrets in code
- **TruffleHog** - Finds high-entropy strings (keys)
- **pre-commit framework** - Manages hooks across team
- **Checkov** - Security/compliance scanning

**Benefits:**
- ✅ Prevents commits with secrets
- ✅ Validates Terraform before push
- ✅ Automatic enforcement (can't bypass)
- ✅ Fast feedback (seconds)

---

## 📊 Security Comparison Matrix

| Approach | Security Level | Developer Experience | Auditability | Cost |
|----------|----------------|---------------------|--------------|------|
| **No local execution (current)** | ⚠️ Medium | ❌ Poor | ✅ Good | $ |
| **Long-lived credentials** | ❌ Poor | ⚠️ Medium | ⚠️ Medium | $ |
| **Dynamic credentials (Vault)** | ✅ Excellent | ✅ Excellent | ✅ Excellent | $$ |
| **OIDC/Workload Identity** | ✅✅ Best | ✅ Excellent | ✅ Excellent | $ |
| **Mocked unit tests** | ✅ Excellent | ✅✅ Best | N/A (no cloud) | $ |
| **HCP Terraform remote exec** | ✅ Excellent | ✅ Very Good | ✅✅ Best | $$ |
| **Git hooks + scanning** | ✅ Good | ✅ Good | ⚠️ Medium | $ |

**Key Insight**: Better security AND better developer experience are possible simultaneously.

---

## 🗣️ Discussion Guide: Proposing Alternatives

### **Building Consensus**

**1. "If developers could test locally with fake data (mocks) and only real deployments needed credentials, would that address security concerns?"**

*Why this matters*: Separates unit testing (no risk) from integration testing (controlled risk).

**Framework to present:**
- **Unit tests**: Mocked providers, no credentials, run locally
- **Integration tests**: Real providers, credentials in CI/CD only
- **Developer flow**: Fast local validation → PR → Automated integration tests

**2. "Would time-limited credentials (expire in 15 minutes) from Vault satisfy the security team?"**

*Why this matters*: Dynamic credentials are objectively more secure than long-lived credentials.

**Points to make:**
- Credentials expire automatically
- Cannot be reused if leaked
- Scoped to specific permissions
- Full audit trail
- Industry standard (AWS IAM roles, Azure Managed Identity, GCP Workload Identity all use this pattern)

**3. "What if we enabled local execution for `terraform plan` only, with remote-only `apply`? Would that be acceptable?"**

*Why this matters*: Most debugging happens with `plan`. Separating read-only from write operations reduces risk.

**Proposal:**
- Local: `terraform init`, `terraform validate`, `terraform plan`, `terraform test`
- Remote only: `terraform apply`
- Credentials: Short-lived, read-only for plan

**4. "Can we pilot a 'secure local dev' approach with one team and measure outcomes?"**

*Why this matters*: Reduces risk. Prove it works before org-wide rollout.

**Pilot Proposal:**
- Duration: 4-6 weeks
- Team: 5-10 developers
- Scope: Unit tests with mocking + dynamic credentials for plan
- Metrics: Time saved, security incidents, satisfaction
- Decision point: Continue or revert

---

## 🎯 Recommended Architecture

### **Hybrid Approach (Best of All Worlds)**

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Workstation                      │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  terraform test (mocked)    ← NO CREDENTIALS NEEDED          │
│  terraform validate         ← NO CREDENTIALS NEEDED          │
│  terraform fmt              ← NO CREDENTIALS NEEDED          │
│                                                               │
│  terraform plan             ← SHORT-LIVED CREDENTIALS         │
│    ├─ Option A: Vault dynamic secrets (15 min TTL)           │
│    ├─ Option B: OIDC token exchange                          │
│    └─ Option C: HCP Terraform remote execution               │
│                                                               │
│  terraform apply            ← BLOCKED LOCALLY                 │
│    └─ Must run in CI/CD                                      │
│                                                               │
│  Pre-commit hooks           ← AUTOMATIC SECRET SCANNING       │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      CI/CD Pipeline                           │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  terraform test (integration)  ← EPHEMERAL TEST ENVIRONMENT  │
│  terraform plan                ← FULL CREDENTIALS            │
│  terraform apply               ← AFTER APPROVAL              │
│                                                               │
│  Credentials: Dynamic (Vault/OIDC, 1-hour TTL)               │
│  Audit: Full logs to SIEM                                    │
│  Policy: Sentinel/OPA enforcement                            │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### **What This Achieves**

| Requirement | Solution | Result |
|-------------|----------|--------|
| **Fast feedback** | Mocked unit tests + local plan | < 1 minute validation |
| **No credential leakage** | Dynamic short-lived credentials | Nothing to leak |
| **Auditability** | Centralized logging | Full visibility |
| **Controlled applies** | CI/CD only | Change management enforced |
| **Developer satisfaction** | Fast local testing | Happy developers |
| **Security posture** | Better than current state | Win-win |

---

## 💰 ROI Calculation

### **Current State (No Local Execution)**

```
Assumptions:
- 10 developers
- 5 pipeline runs per developer per day
- 10 minutes average pipeline wait time
- $100/hour loaded developer cost

Daily cost:
10 devs × 5 runs × 10 min × $100/hr ÷ 60 min = $83/day

Annual cost:
$83/day × 250 work days = $20,833/year
```

### **Future State (Secure Local Development)**

```
Assumptions:
- 80% of validations run locally (< 1 min)
- 20% still run in pipeline (integration tests)
- Short-lived credentials: $5,000 setup + $2,000/year (Vault)

Daily cost:
10 devs × 1 pipeline run × 10 min × $100/hr ÷ 60 min = $17/day

Annual cost:
$17/day × 250 work days = $4,167/year
Setup: $5,000 (year 1 only)
Vault: $2,000/year

Year 1 total: $11,167
Years 2+: $6,167/year

Annual savings: $14,666 (year 1), $20,833 (years 2+)
Payback period: 4 months
```

**Plus intangible benefits:**
- Higher developer satisfaction
- Faster time to market
- Fewer "debugging via pipeline" commits
- Better code quality (can test before committing)

---

## 🛡️ Addressing Security Team Concerns

### **Concern: "Developers will leak credentials"**

**Response:**
- With dynamic credentials (15 min TTL), leaked credentials are useless after 15 minutes
- With OIDC, there are no credentials to leak
- With mocked tests, no credentials needed for 80% of testing
- Git hooks prevent accidental commits
- Current pipeline credentials are arguably higher risk (longer-lived, broader access)

### **Concern: "We won't have audit logs"**

**Response:**
- Vault logs all credential requests
- Cloud provider logs all API calls (CloudTrail, Azure Monitor, GCP Cloud Audit)
- HCP Terraform logs all runs
- Actually BETTER auditability than current state

### **Concern: "Developers will bypass the pipeline"**

**Response:**
- `terraform apply` still requires pipeline (enforce with policy)
- Local `plan` is read-only—no infrastructure changes
- Pull request process still required
- Automated tests still run in CI/CD

### **Concern: "This is too complex to implement"**

**Response:**
- Start with mocked unit tests (zero setup, immediate value)
- Add dynamic credentials for specific teams (pilot)
- Phase rollout over 3-6 months
- HashiCorp Professional Services can help

---

## 📋 Recommended Policy Update

### **Current Policy**

> "Developers may not run Terraform on their local machines."

### **Proposed Policy**

> **Terraform Local Development Standard**
>
> Developers MAY run the following Terraform commands locally:
> - `terraform init`, `terraform validate`, `terraform fmt`
> - `terraform test` (unit tests with mocked providers)
> - `terraform plan` (with short-lived credentials from Vault/OIDC)
>
> Developers MUST NOT run the following commands locally:
> - `terraform apply` (must run in approved CI/CD pipelines)
>
> **Credential Management:**
> - All credentials must be dynamically generated with TTL < 1 hour
> - No long-lived credentials permitted on developer machines
> - All credential requests logged to centralized SIEM
>
> **Pre-Commit Requirements:**
> - Git hooks must scan for secrets before commit
> - All Terraform must pass validation before push
> - Unit tests must pass locally before creating PR
>
> **CI/CD Requirements:**
> - Integration tests run automatically on PR
> - `terraform apply` requires approval
> - All applies logged and audited
> - Policy-as-code (Sentinel/OPA) enforces guardrails

---

## 💡 Key Messages to Emphasize

1. **Security and Developer Experience Are Not Trade-Offs** - Modern tools enable both.

2. **Current Policy Is Security Theater** - If pipelines have credentials, you haven't eliminated risk, just shifted it.

3. **Dynamic Credentials Are More Secure** - Short-lived, scoped tokens beat long-lived keys.

4. **Mocking Changes Everything** - 80% of testing can happen with zero credentials.

5. **Industry Standard** - AWS, Azure, GCP all recommend dynamic credentials and workload identity.

6. **Pilot First, Prove Value** - Don't need to change everything day 1.

---

**Next Steps:**
- [Testing Strategy](./04-testing-strategy.md)
- [Vault Integration](./05-vault-integration.md)
