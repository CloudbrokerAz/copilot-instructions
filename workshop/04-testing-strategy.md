# Testing Strategy

## 🎯 Purpose

This document provides comprehensive guidance on implementing Terraform testing practices, including unit tests, integration tests, and mocking. It addresses the common confusion around "what does testing Terraform mean?" and provides concrete implementation patterns.

---

## 🧪 Testing Fundamentals

### **The Confusion: What Is "Testing Terraform"?**

Many teams say they "test Terraform" but mean different things:

| What They Say | What They Mean | Is This Testing? |
|---------------|----------------|------------------|
| "We test Terraform" | `terraform validate` | ⚠️ Syntax validation, not testing |
| "We test Terraform" | Manual inspection of `plan` output | ⚠️ Manual review, not automated |
| "We test Terraform" | Deploy to dev environment | ⚠️ Manual integration test, not automated |
| "We test Terraform" | `checkov` / `tfsec` | ✅ Security/compliance testing |
| "We test Terraform" | Automated tests with assertions | ✅✅ Real testing |

### **Proper Testing Hierarchy**

```
┌─────────────────────────────────────────────────────────────┐
│                     Validation (Pre-Test)                     │
│  terraform validate, fmt, precondition, postcondition         │
│  Fast: < 1 second | Cost: $0 | Confidence: Low               │
└─────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Unit Tests (Mocked)                        │
│  Test logic without deploying resources                       │
│  Fast: < 10 seconds | Cost: $0 | Confidence: Medium          │
└─────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              Integration Tests (Real Resources)               │
│  Deploy resources, validate behavior, destroy                 │
│  Slow: 5-15 minutes | Cost: $$ | Confidence: High            │
└─────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 Contract/Compliance Tests                     │
│  Ensure outputs match expectations, policy compliance         │
│  Fast: < 1 minute | Cost: $ | Confidence: High               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🗣️ Discussion Guide: Current Testing Understanding

### **Discovery Questions**

**1. "What does 'testing Terraform' mean to your team today?"**

*Why this matters*: Reveals understanding gap. Many teams conflate validation with testing.

**Listen for:**
- Do they mention automated tests?
- Do they talk about assertions?
- Do they differentiate unit vs integration?

**2. "How do you validate a module works before releasing it to other teams?"**

*Why this matters*: Shows current quality assurance process (or lack thereof).

**Follow-up probes:**
- "Is this manual or automated?"
- "How long does it take?"
- "What breaks most often?"
- "How do you prevent regressions?"

**3. "When a module changes, how do you know it didn't break existing consumers?"**

*Why this matters*: Highlights need for regression testing. Mono-repo makes this harder.

**Common answers:**
- "We don't" → Opportunity for automated tests
- "We ask teams to test" → Manual, slow, unreliable
- "We have a test environment" → Good start, can be automated

**4. "Have you experienced cases where a module change broke production? What would have prevented it?"**

*Why this matters*: Real pain points build urgency for proper testing.

**Capture:**
- Incident details
- Root cause
- Time to detect
- Time to fix
- Cost/impact

**5. "What's stopping you from implementing tests today?"**

*Why this matters*: Reveals blockers—knowledge gap, tooling, pipeline restrictions, or cost concerns.

| Blocker | Solution |
|---------|----------|
| "We don't know how" | **This workshop!** |
| "No time" | Show ROI—tests save time long-term |
| "Too expensive" | Unit tests cost $0 (mocked) |
| "Can't run locally" | Mocking enables local testing |
| "Pipelines too slow" | Unit tests are fast (< 10 sec) |

---

## 📚 Terraform Testing Framework Overview

### **What Changed in Terraform v1.6+**

Before v1.6, testing Terraform required external tools (Terratest, kitchen-terraform, etc.).

**Terraform v1.6+** introduced built-in testing:
- `.tftest.hcl` test files
- `run` blocks for test scenarios
- `assert` blocks for validation
- `command = plan` or `apply`
- Helper modules for test fixtures

**Terraform v1.7+** added mocking:
- `mock_provider` blocks
- `override_resource`, `override_data`, `override_module`
- No credentials needed for unit tests!

---

## 🧪 Unit Testing with Mocked Providers

### **Concept: Test Logic Without Cloud Resources**

**What can be tested without deploying anything:**
- ✅ Variable validation
- ✅ Resource configuration logic
- ✅ Conditional expressions (`count`, `for_each`)
- ✅ Output values
- ✅ Module composition
- ✅ Naming conventions
- ✅ Tagging logic
- ✅ Data transformations

**What cannot be tested:**
- ❌ Actual cloud API behavior
- ❌ Resource creation success
- ❌ Cross-resource dependencies (real)
- ❌ Provider-specific quirks

### **Example: S3 Bucket Module**

**Module Structure:**
```
terraform-aws-s3-bucket/
├── main.tf
├── variables.tf
├── outputs.tf
├── tests/
│   ├── unit/
│   │   ├── naming.tftest.hcl
│   │   ├── tagging.tftest.hcl
│   │   └── versioning.tftest.hcl
│   └── integration/
│       └── deployment.tftest.hcl
```

**Module Code (`main.tf`):**
```hcl
resource "aws_s3_bucket" "main" {
  bucket = var.bucket_name

  tags = merge(
    var.common_tags,
    {
      Name        = var.bucket_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  )
}

resource "aws_s3_bucket_versioning" "main" {
  count  = var.enable_versioning ? 1 : 0
  bucket = aws_s3_bucket.main.id

  versioning_configuration {
    status = "Enabled"
  }
}

output "bucket_id" {
  value = aws_s3_bucket.main.id
}

output "bucket_arn" {
  value = aws_s3_bucket.main.arn
}
```

**Unit Test (`tests/unit/naming.tftest.hcl`):**
```hcl
# Mock the AWS provider—no credentials needed!
mock_provider "aws" {}

variables {
  bucket_name  = "my-test-bucket"
  environment  = "dev"
  common_tags  = {
    Project = "TestProject"
  }
  enable_versioning = true
}

run "validate_bucket_name" {
  command = plan  # Plan-only, no resources created

  assert {
    condition     = aws_s3_bucket.main.bucket == "my-test-bucket"
    error_message = "Bucket name doesn't match input variable"
  }
}

run "validate_naming_convention" {
  command = plan

  # Test different bucket name patterns
  variables {
    bucket_name = "invalid_bucket_name_with_underscores"
  }

  # This should still create the resource, but we can check properties
  assert {
    condition     = length(regexall("_", aws_s3_bucket.main.bucket)) > 0
    error_message = "Bucket name validation is not working"
  }
}

run "validate_lowercase_enforcement" {
  command = plan

  variables {
    bucket_name = "MyBucketWithUpperCase"
  }

  # Check if module handles case conversion
  assert {
    condition     = aws_s3_bucket.main.bucket == lower("MyBucketWithUpperCase")
    error_message = "Bucket names should be converted to lowercase"
  }
}
```

**Unit Test for Tagging (`tests/unit/tagging.tftest.hcl`):**
```hcl
mock_provider "aws" {}

variables {
  bucket_name  = "test-bucket"
  environment  = "production"
  common_tags  = {
    Project     = "MyProject"
    CostCenter  = "Engineering"
  }
  enable_versioning = false
}

run "validate_tag_merging" {
  command = plan

  assert {
    condition     = aws_s3_bucket.main.tags["Project"] == "MyProject"
    error_message = "Common tags not applied correctly"
  }

  assert {
    condition     = aws_s3_bucket.main.tags["Environment"] == "production"
    error_message = "Environment tag not set"
  }

  assert {
    condition     = aws_s3_bucket.main.tags["ManagedBy"] == "Terraform"
    error_message = "ManagedBy tag not set"
  }
}

run "validate_required_tags" {
  command = plan

  # Test without common_tags
  variables {
    common_tags = {}
  }

  assert {
    condition     = aws_s3_bucket.main.tags["Environment"] != null
    error_message = "Environment tag is required"
  }
}
```

**Unit Test for Conditional Resources (`tests/unit/versioning.tftest.hcl`):**
```hcl
mock_provider "aws" {}

variables {
  bucket_name       = "test-bucket"
  environment       = "dev"
  common_tags       = {}
  enable_versioning = true
}

run "versioning_enabled" {
  command = plan

  assert {
    condition     = length(aws_s3_bucket_versioning.main) == 1
    error_message = "Versioning should be enabled"
  }
}

run "versioning_disabled" {
  command = plan

  variables {
    enable_versioning = false
  }

  assert {
    condition     = length(aws_s3_bucket_versioning.main) == 0
    error_message = "Versioning should be disabled"
  }
}
```

### **Running Unit Tests**

```bash
$ cd terraform-aws-s3-bucket

$ terraform test -filter=tests/unit/*.tftest.hcl
# Runs all unit tests in tests/unit/

# Output:
tests/unit/naming.tftest.hcl... pass
tests/unit/tagging.tftest.hcl... pass
tests/unit/versioning.tftest.hcl... pass

Success! 3 passed, 0 failed.
```

**Time:** < 5 seconds  
**Cost:** $0  
**Credentials:** None needed  

---

## 🔗 Integration Testing with Real Resources

### **Concept: Deploy Real Infrastructure, Validate, Destroy**

**What gets tested:**
- ✅ Resources deploy successfully
- ✅ Cloud API behavior
- ✅ Cross-resource dependencies
- ✅ IAM policies work
- ✅ Network connectivity
- ✅ End-to-end scenarios

**Integration Test (`tests/integration/deployment.tftest.hcl`):**
```hcl
# Real provider—requires credentials
provider "aws" {
  region = "us-east-1"
}

variables {
  bucket_name       = "integration-test-bucket-${uuid()}"  # Unique name
  environment       = "test"
  common_tags       = {
    Test = "Integration"
  }
  enable_versioning = true
}

run "deploy_and_validate" {
  command = apply  # Actually creates resources

  assert {
    condition     = aws_s3_bucket.main.bucket != ""
    error_message = "Bucket was not created"
  }

  assert {
    condition     = aws_s3_bucket.main.arn != ""
    error_message = "Bucket ARN is empty"
  }

  # Validate versioning was enabled
  assert {
    condition     = length(aws_s3_bucket_versioning.main) == 1
    error_message = "Versioning not enabled"
  }
}

# Terraform automatically destroys resources after test completes
```

### **Integration Test with Helper Modules**

Complex modules often need supporting infrastructure (VPCs, IAM roles, etc.).

**Structure:**
```
tests/
├── integration/
│   └── deployment.tftest.hcl
└── setup/
    ├── main.tf          # Creates VPC, subnets, etc.
    ├── outputs.tf       # Exports IDs for use in tests
    └── variables.tf
```

**Helper Module (`tests/setup/main.tf`):**
```hcl
# Creates test VPC and subnets
resource "aws_vpc" "test" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "test-vpc"
  }
}

resource "aws_subnet" "test" {
  vpc_id     = aws_vpc.test.id
  cidr_block = "10.0.1.0/24"
  
  tags = {
    Name = "test-subnet"
  }
}

output "vpc_id" {
  value = aws_vpc.test.id
}

output "subnet_id" {
  value = aws_subnet.test.id
}
```

**Integration Test Using Helper (`tests/integration/with_vpc.tftest.hcl`):**
```hcl
provider "aws" {
  region = "us-east-1"
}

# First, create test infrastructure
run "setup" {
  module {
    source = "./tests/setup"
  }
}

# Then test the module with that infrastructure
run "deploy_into_vpc" {
  command = apply

  variables {
    vpc_id    = run.setup.vpc_id
    subnet_id = run.setup.subnet_id
  }

  assert {
    condition     = aws_instance.main.vpc_id == run.setup.vpc_id
    error_message = "Instance not in correct VPC"
  }
}

# Terraform destroys resources in reverse order: main resources, then setup
```

### **Running Integration Tests**

```bash
$ terraform test -filter=tests/integration/*.tftest.hcl

# Output:
tests/integration/deployment.tftest.hcl... pass
tests/integration/with_vpc.tftest.hcl... pass

Success! 2 passed, 0 failed.

# Resources were created and destroyed automatically
```

**Time:** 5-15 minutes (depends on resources)  
**Cost:** $ (ephemeral resources, minutes of runtime)  
**Credentials:** Required (AWS, Azure, GCP)  

---

## 🎭 Advanced: Mocking for Complex Scenarios

### **Mocking Data Sources**

Sometimes you need data from existing resources without actually querying them:

```hcl
# tests/unit/with_mocked_data.tftest.hcl

mock_provider "aws" {
  # Mock data source responses
  mock_data "aws_ami" {
    defaults = {
      id    = "ami-12345678"
      name  = "test-ami"
      owners = ["amazon"]
    }
  }
}

run "test_with_mocked_ami" {
  command = plan

  # Module uses data.aws_ami.ubuntu
  # Mock provider returns fake data
  
  assert {
    condition     = aws_instance.main.ami == "ami-12345678"
    error_message = "AMI not set correctly"
  }
}
```

### **Overriding Specific Resources**

Test edge cases without creating real resources:

```hcl
# tests/unit/failure_scenarios.tftest.hcl

mock_provider "aws" {}

override_resource {
  target = aws_s3_bucket.main
  
  # Simulate a bucket that already exists
  values = {
    id     = "existing-bucket"
    bucket = "existing-bucket"
    arn    = "arn:aws:s3:::existing-bucket"
  }
}

run "handle_existing_bucket" {
  command = plan

  # Module should handle existing bucket gracefully
  # Test error handling logic
}
```

### **Mock Files for Shared Data**

Create reusable mock data:

**`tests/mocks/common.tfmock.hcl`:**
```hcl
mock_provider "aws" {
  alias = "common"
  
  mock_data "aws_caller_identity" {
    defaults = {
      account_id = "123456789012"
      arn        = "arn:aws:iam::123456789012:user/test"
      user_id    = "AIDAI123456789012"
    }
  }
  
  mock_data "aws_region" {
    defaults = {
      name = "us-east-1"
    }
  }
}
```

**Use in tests:**
```hcl
# tests/unit/my_test.tftest.hcl

mock_provider "aws" {
  source = "./mocks/common.tfmock.hcl"
}

run "test_with_shared_mocks" {
  command = plan
  
  # Uses mock data from common.tfmock.hcl
}
```

---

## 📊 Testing Strategy by Module Type

### **Resource Modules (Single Resource)**

Focus on:
- ✅ Input variable validation
- ✅ Naming conventions
- ✅ Tagging/labeling
- ✅ Conditional features
- ✅ Output correctness

**Test Mix:**
- 80% unit tests (mocked)
- 20% integration tests (basic deployment)

### **Pattern Modules (Multi-Resource Architecture)**

Focus on:
- ✅ Resource composition
- ✅ Cross-resource dependencies
- ✅ End-to-end functionality
- ✅ Failure scenarios

**Test Mix:**
- 50% unit tests (component logic)
- 50% integration tests (full deployment)

### **Utility Modules (No Resources)**

Focus on:
- ✅ Data transformations
- ✅ Calculations
- ✅ Output formatting

**Test Mix:**
- 100% unit tests (no resources to deploy)

---

## 🗣️ Discussion Guide: Testing Implementation

### **Building a Testing Roadmap**

**Phase 1: Validation (Week 1-2)**
- Add `terraform validate` to CI/CD
- Add variable `validation` blocks
- Add `precondition` and `postcondition` checks

**Questions:**
- "Which modules have the most variable validation errors?"
- "Can we add validation rules to prevent common mistakes?"

**Phase 2: Unit Tests with Mocking (Month 1-2)**
- Install Terraform v1.7+
- Write unit tests for 5 core modules
- Enable local testing for developers
- Add pre-commit hooks

**Questions:**
- "Which 5 modules are most critical to test?"
- "Who will write the initial test examples?"
- "How do we share test patterns across the team?"

**Phase 3: Integration Tests in CI/CD (Month 2-3)**
- Create ephemeral test accounts/subscriptions
- Write integration tests for core modules
- Automate test execution in pipelines
- Set up test result reporting

**Questions:**
- "Do we have isolated test environments?"
- "How do we handle test resource cleanup?"
- "What's our budget for test infrastructure?"

**Phase 4: Automated Regression Testing (Month 3-6)**
- Run tests on every module change
- Block releases if tests fail
- Track test coverage metrics
- Continuous improvement

**Questions:**
- "What's our target test coverage (50%? 80%?)?"
- "How do we handle flaky tests?"
- "Who monitors test health?"

---

## 💰 ROI of Testing

### **Cost of NOT Testing**

```
Assumptions:
- 2 breaking changes per quarter from module updates
- 8 hours to detect and fix each incident
- 3 teams impacted per incident
- $100/hour loaded cost

Annual cost:
2 incidents × 4 quarters × 8 hours × 3 teams × $100/hr = $19,200/year
```

### **Cost of Testing**

```
Setup:
- Training: 16 hours × $100/hr = $1,600
- Initial test writing: 40 hours × $100/hr = $4,000
- CI/CD setup: 8 hours × $100/hr = $800
Total setup: $6,400

Ongoing:
- Test maintenance: 4 hours/month × $100/hr × 12 = $4,800/year
- Test infrastructure: $500/year (ephemeral resources)
Total ongoing: $5,300/year

Year 1 total: $11,700
Years 2+: $5,300/year

Break-even: 7 months
Annual savings (years 2+): $13,900/year
```

**Plus intangible benefits:**
- Confidence in changes
- Faster development (no fear of breaking things)
- Better module quality
- Easier onboarding (tests as examples)

---

## 📋 Testing Checklist

By the end of testing implementation, each module should have:

- [ ] **Validation**
  - [ ] Variable validation blocks
  - [ ] Preconditions on resources
  - [ ] Postconditions on outputs
  - [ ] `terraform validate` passes

- [ ] **Unit Tests**
  - [ ] Mocked provider tests
  - [ ] Variable validation tests
  - [ ] Tagging/naming tests
  - [ ] Conditional logic tests
  - [ ] Output value tests
  - [ ] Run time: < 10 seconds
  - [ ] Coverage: 80%+ of logic paths

- [ ] **Integration Tests**
  - [ ] Basic deployment test
  - [ ] Common scenarios (2-3)
  - [ ] Cleanup/destroy test
  - [ ] Run time: < 15 minutes
  - [ ] Cost: < $5 per run

- [ ] **Documentation**
  - [ ] How to run tests locally
  - [ ] How to write new tests
  - [ ] Test coverage report
  - [ ] Known limitations

- [ ] **CI/CD Integration**
  - [ ] Unit tests run on every PR
  - [ ] Integration tests run on merge
  - [ ] Test failures block release
  - [ ] Results reported to PR

---

## 💡 Key Messages to Emphasize

1. **Testing Is Not Optional** - Would you deploy application code without tests?

2. **Start Small** - Don't need 100% coverage day 1. Start with critical modules.

3. **Unit Tests Are Free** - Mocked providers cost $0 to run, unlimited times.

4. **Fast Feedback Wins** - Unit tests give answers in seconds, not minutes.

5. **Tests Are Documentation** - Good tests show how to use the module.

6. **Terraform Has Built-In Testing** - No excuse not to use it.

---

**Next Steps:**
- [Vault Integration](./05-vault-integration.md)
- [Technical Workshops](./07-technical-workshops.md)
