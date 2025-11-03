# Technical Workshops

## 🎯 Purpose

This document provides hands-on exercises for module design and testing implementation. These workshops are designed to be interactive, with participants building real examples during the session.

---

## 🏗️ Workshop 1: Module Design & Refactoring

### **Duration**: 2 hours

### **Objective**
Take an existing module and refactor it using HashiCorp's scoping principles and AVM/best practice patterns.

---

### **Pre-Workshop Setup**

**Participants need:**
- Terraform v1.7+ installed
- VS Code or preferred editor
- Access to a module from their codebase
- Whiteboard or Miro board

**Facilitator prepares:**
- Example module to analyze (if they don't have one)
- HashiCorp scoping framework printed/displayed
- Module evaluation checklist

---

### **Part 1: Module Evaluation (45 minutes)**

#### **Step 1: Inventory Current Module (10 min)**

Pick a module and document:

```
Module Name: _______________________
Primary Resource(s): _______________________
Secondary Resources: _______________________
Number of Input Variables: _______
Number of Outputs: _______
Lines of Code: _______
Test Coverage: _______%
Documentation Quality (1-10): _______
```

#### **Step 2: Scoping Analysis (20 min)**

**Framework: Three Dimensions of Scoping**

Use this worksheet:

**1. Encapsulation (Deploy Together)**
```
Question: Are all resources in this module always deployed together?

Resources in module:
- Resource 1: _______________________
- Resource 2: _______________________
- Resource 3: _______________________

Can you deploy Resource 1 without Resource 2?  ☐ Yes ☐ No
Can you deploy Resource 2 without Resource 3?  ☐ Yes ☐ No

Conclusion:
☐ Well-scoped (resources always deployed together)
☐ Over-scoped (too much in one module)
☐ Under-scoped (too granular, should include more)
```

**2. Privileges (Security Boundaries)**
```
Question: Does this module cross team/permission boundaries?

Who can create Resource 1? Team/Role: _______________________
Who can create Resource 2? Team/Role: _______________________
Who can create Resource 3? Team/Role: _______________________

Are these the same team/role?  ☐ Yes ☐ No

If No, list different privilege boundaries:
- _______________________
- _______________________

Conclusion:
☐ Respects boundaries (same privilege level)
☐ Violates boundaries (should be split)
```

**3. Volatility (Change Frequency)**
```
Question: Do all resources change at the same frequency?

Resource 1 changes: ☐ Rarely ☐ Monthly ☐ Weekly ☐ Daily
Resource 2 changes: ☐ Rarely ☐ Monthly ☐ Weekly ☐ Daily
Resource 3 changes: ☐ Rarely ☐ Monthly ☐ Weekly ☐ Daily

Are these similar?  ☐ Yes ☐ No

Example: Database (rarely changes) + App servers (daily changes) = BAD

Conclusion:
☐ Similar volatility (good)
☐ Mixed volatility (consider splitting)
```

#### **Step 3: MVP Assessment (15 min)**

**Questions:**

1. **Does this module cover 80% of use cases?**
   ```
   Common use cases:
   1. _______________________
   2. _______________________
   3. _______________________
   
   Edge cases this module handles:
   - _______________________
   - _______________________
   
   Assessment:
   ☐ Focuses on common cases (good)
   ☐ Tries to handle too many edge cases (over-engineered)
   ```

2. **Are required inputs truly required?**
   ```
   Required inputs:
   - _______________________  Truly required? ☐ Yes ☐ No (has default)
   - _______________________  Truly required? ☐ Yes ☐ No (has default)
   - _______________________  Truly required? ☐ Yes ☐ No (has default)
   
   Could any have sensible defaults?
   ```

3. **Are outputs comprehensive?**
   ```
   Current outputs: _______
   
   Do outputs include:
   ☐ Primary resource ID
   ☐ Primary resource ARN/URI
   ☐ All useful attributes for chaining
   ☐ Connection strings/endpoints
   
   Missing outputs that should be added:
   - _______________________
   - _______________________
   ```

---

### **Part 2: Redesign Workshop (45 minutes)**

#### **Step 1: Identify Issues (10 min)**

Group discussion:
- What scoping issues did we find?
- What MVP violations exist?
- What would an ideal design look like?

#### **Step 2: Propose Refactoring (20 min)**

**Template:**

```
Current Design:
┌──────────────────────────────────────┐
│  Module: current-module               │
│  - Resource A                         │
│  - Resource B                         │
│  - Resource C                         │
│  Issues:                              │
│  - Crosses privilege boundary         │
│  - Mixes static and dynamic resources│
│  - Too many edge cases                │
└──────────────────────────────────────┘

Proposed Design:
┌──────────────────────┐  ┌──────────────────────┐
│  Module: static-infra │  │  Module: dynamic-app  │
│  - Resource A (DB)    │  │  - Resource B (App)   │
│  Changes: Rarely      │  │  Changes: Daily       │
│  Team: Data team      │  │  Team: App team       │
└──────────────────────┘  └──────────────────────┘
       │                            │
       └────────────┬───────────────┘
                    │
            ┌───────▼───────┐
            │  Compose with │
            │  root module  │
            └───────────────┘
```

#### **Step 3: Live Refactoring (15 min)**

Facilitator live-codes the refactoring:

1. Split module into separate files
2. Define clear interfaces (variables/outputs)
3. Update examples
4. Add basic tests

---

### **Part 3: Documentation Review (30 minutes)**

#### **README.md Checklist**

Review against this checklist:

```markdown
# Module: <name>

## Overview
☐ Clear description of what this deploys
☐ When you should/shouldn't use this
☐ Link to design decisions

## Features
☐ List of capabilities
☐ What's included
☐ What's NOT included

## Prerequisites
☐ Terraform version
☐ Provider versions
☐ Required permissions
☐ Existing resources needed

## Usage

### Basic Example
☐ Minimum viable configuration
☐ Copy-paste ready
☐ Working (tested)

### Common Scenarios
☐ Scenario 1: [Name]
☐ Scenario 2: [Name]
☐ Scenario 3: [Name]

### Advanced Example
☐ All features enabled
☐ Complex configuration

## Requirements
☐ Table of versions (auto-generated OK)

## Inputs
☐ Table of variables (auto-generated OK)
☐ Required vs optional clearly marked

## Outputs
☐ Table of outputs (auto-generated OK)

## Module Dependencies
☐ List external modules/resources

## Testing
☐ How to run tests
☐ Expected results

## Contributing
☐ Link to contribution guide

## License & Support
☐ License information
☐ Support contacts
```

#### **Documentation Exercise**

Participants update their module's README to include missing sections.

---

## 🧪 Workshop 2: Testing Implementation

### **Duration**: 2 hours

### **Objective**
Write unit and integration tests for a Terraform module using the built-in testing framework.

---

### **Pre-Workshop Setup**

**Participants need:**
- Terraform v1.7+ installed
- A module to test (can use example from Workshop 1)
- Cloud credentials (for integration tests) OR mock-only approach

**Facilitator prepares:**
- Example module with tests (working)
- Testing framework cheat sheet
- Test templates

---

### **Part 1: Testing Fundamentals (30 minutes)**

#### **Concept Overview (10 min)**

**Facilitator explains:**

```
Testing Hierarchy:
1. Validation (terraform validate, preconditions)
2. Unit Tests (plan-only, mocked providers, fast)
3. Integration Tests (apply, real resources, slow)
4. Compliance Tests (policies, security scanning)
```

**Live Demo**: Show running each type

#### **Test File Structure (10 min)**

**Facilitator explains:**

```
module/
├── main.tf
├── variables.tf
├── outputs.tf
├── tests/
│   ├── unit/
│   │   ├── naming.tftest.hcl
│   │   ├── tagging.tftest.hcl
│   │   └── conditional.tftest.hcl
│   ├── integration/
│   │   └── deployment.tftest.hcl
│   └── setup/
│       ├── main.tf        # Helper module
│       └── outputs.tf
```

#### **Test Syntax (10 min)**

**Facilitator live-codes a simple test:**

```hcl
# tests/unit/example.tftest.hcl

mock_provider "aws" {}

variables {
  bucket_name = "test-bucket"
}

run "validate_bucket_name" {
  command = plan

  assert {
    condition     = aws_s3_bucket.main.bucket == "test-bucket"
    error_message = "Bucket name doesn't match"
  }
}
```

**Run it live**: `terraform test tests/unit/example.tftest.hcl`

---

### **Part 2: Hands-On Unit Testing (45 minutes)**

#### **Exercise 1: Variable Validation Test (15 min)**

**Task**: Write a test that validates variable constraints

```hcl
# variables.tf
variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod"
  }
}
```

**Test to write:**
```hcl
# tests/unit/variables.tftest.hcl

mock_provider "aws" {}

run "valid_environment" {
  command = plan
  
  variables {
    environment = "dev"
  }
  
  # Should succeed
}

run "invalid_environment" {
  command = plan
  
  variables {
    environment = "invalid"
  }
  
  expect_failures = [
    var.environment
  ]
}
```

**Participants**: Write similar tests for their module's variables

#### **Exercise 2: Conditional Logic Test (15 min)**

**Task**: Test resources created based on conditions

```hcl
# main.tf
resource "aws_s3_bucket_versioning" "main" {
  count  = var.enable_versioning ? 1 : 0
  bucket = aws_s3_bucket.main.id
  
  versioning_configuration {
    status = "Enabled"
  }
}
```

**Test to write:**
```hcl
# tests/unit/conditionals.tftest.hcl

mock_provider "aws" {}

run "versioning_enabled" {
  command = plan
  
  variables {
    enable_versioning = true
  }
  
  assert {
    condition     = length(aws_s3_bucket_versioning.main) == 1
    error_message = "Versioning should be created"
  }
}

run "versioning_disabled" {
  command = plan
  
  variables {
    enable_versioning = false
  }
  
  assert {
    condition     = length(aws_s3_bucket_versioning.main) == 0
    error_message = "Versioning should not be created"
  }
}
```

**Participants**: Write tests for their conditional resources

#### **Exercise 3: Tagging Test (15 min)**

**Task**: Validate that tags are applied correctly

**Test template:**
```hcl
# tests/unit/tagging.tftest.hcl

mock_provider "aws" {}

variables {
  common_tags = {
    Project    = "TestProject"
    CostCenter = "Engineering"
  }
}

run "validate_tag_merging" {
  command = plan
  
  assert {
    condition     = aws_s3_bucket.main.tags["Project"] == "TestProject"
    error_message = "Project tag not set"
  }
  
  assert {
    condition     = aws_s3_bucket.main.tags["ManagedBy"] == "Terraform"
    error_message = "ManagedBy tag not set"
  }
}
```

**Participants**: Write tagging tests for their modules

---

### **Part 3: Integration Testing (30 minutes)**

#### **Exercise 4: Basic Deployment Test (15 min)**

**Facilitator demonstrates:**

```hcl
# tests/integration/deployment.tftest.hcl

provider "aws" {
  region = "us-east-1"
}

variables {
  bucket_name = "integration-test-${uuid()}"
}

run "deploy_and_validate" {
  command = apply  # Creates real resources
  
  assert {
    condition     = aws_s3_bucket.main.bucket != ""
    error_message = "Bucket not created"
  }
  
  assert {
    condition     = can(regex("^arn:aws:s3", aws_s3_bucket.main.arn))
    error_message = "Invalid ARN format"
  }
}

# Terraform auto-destroys after test
```

**Run live with participants**

#### **Exercise 5: Multi-Step Test with Helper (15 min)**

**Facilitator demonstrates:**

```hcl
# tests/setup/main.tf
resource "aws_vpc" "test" {
  cidr_block = "10.0.0.0/16"
}

output "vpc_id" {
  value = aws_vpc.test.id
}
```

```hcl
# tests/integration/with_vpc.tftest.hcl

provider "aws" {
  region = "us-east-1"
}

run "setup_vpc" {
  module {
    source = "./tests/setup"
  }
}

run "deploy_into_vpc" {
  command = apply
  
  variables {
    vpc_id = run.setup_vpc.vpc_id
  }
  
  assert {
    condition     = aws_instance.main.vpc_id == run.setup_vpc.vpc_id
    error_message = "Instance not in correct VPC"
  }
}
```

---

### **Part 4: CI/CD Integration (15 minutes)**

#### **GitHub Actions Example**

**Facilitator shows:**

```yaml
# .github/workflows/test.yml
name: Terraform Tests

on:
  pull_request:
  push:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.7.0
      
      - name: Run Unit Tests
        run: terraform test -filter="tests/unit/*.tftest.hcl"
  
  integration-tests:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.7.0
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1
      
      - name: Run Integration Tests
        run: terraform test -filter="tests/integration/*.tftest.hcl"
```

**Participants**: Adapt for their CI/CD system

---

## 📊 Workshop Deliverables

### **By End of Workshops, Participants Have:**

**From Workshop 1:**
- [ ] Evaluated at least one module using scoping framework
- [ ] Identified 2-3 improvements for that module
- [ ] Updated module README with missing sections
- [ ] Created refactoring plan (if needed)

**From Workshop 2:**
- [ ] Written 3+ unit tests for their module
- [ ] Written 1+ integration test
- [ ] Run tests locally successfully
- [ ] Created CI/CD workflow for automated testing

---

## 💡 Facilitation Tips

### **Keep It Interactive**
- Avoid long lectures
- Live-code examples
- Have participants code along
- Pair programming for struggling participants

### **Use Real Modules**
- Work with their actual code when possible
- Real examples are more compelling
- Issues are more relevant

### **Time Management**
- Use a timer for exercises
- Have "solution" code ready if they get stuck
- Skip advanced topics if running short

### **Create Safe Environment**
- "No stupid questions"
- Normalize not knowing things
- Celebrate attempts, not just success

---

**Next Steps:**
- [Workshop Agenda](./workshop-agenda.md)
- [Success Criteria](./success-criteria.md)
