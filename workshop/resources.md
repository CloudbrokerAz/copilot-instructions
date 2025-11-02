# Resources

## 🎯 Purpose

This document provides curated links to official HashiCorp documentation, community resources, and additional learning materials for Terraform module design, testing, and best practices.

---

## 📚 Official HashiCorp Documentation

### **Terraform Testing**

| Resource | Description | When to Use |
|----------|-------------|-------------|
| [Terraform Tests Language](https://developer.hashicorp.com/terraform/language/tests) | Complete reference for test syntax, run blocks, assertions | Writing tests, test structure |
| [Testing Tutorial](https://developer.hashicorp.com/terraform/tutorials/configuration-language/test) | Step-by-step tutorial with examples | Learning testing from scratch |
| [Mocking Documentation](https://developer.hashicorp.com/terraform/language/tests/mocking) | Mock providers, resources, data sources | Unit tests without credentials |

### **Module Development**

| Resource | Description | When to Use |
|----------|-------------|-------------|
| [Module Creation Patterns](https://developer.hashicorp.com/terraform/tutorials/modules/pattern-module-creation) | Official guidance on module scoping, design | Designing new modules |
| [Module Overview](https://developer.hashicorp.com/terraform/tutorials/modules/module) | Fundamentals of module structure | Getting started with modules |
| [Standard Module Structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure) | File organization, naming conventions | Structuring module files |

### **Terraform Workflows & Adoption**

| Resource | Description | When to Use |
|----------|-------------|-------------|
| [Terraform Workflows](https://developer.hashicorp.com/validated-designs/terraform-operating-guides-adoption/terraform-workflows) | VCS-driven, API-driven, CLI-driven patterns | CI/CD integration decisions |
| [Consumption Models](https://developer.hashicorp.com/validated-designs/terraform-operating-guides-adoption/consumption-models) | Service Catalog vs Infrastructure Franchise | Organizational adoption strategy |

### **HCP Terraform / Terraform Enterprise**

| Resource | Description | When to Use |
|----------|-------------|-------------|
| [HCP Terraform Documentation](https://developer.hashicorp.com/terraform/cloud-docs) | Complete product documentation | Setting up HCP Terraform |
| [Private Module Registry](https://developer.hashicorp.com/terraform/cloud-docs/registry) | Publishing and consuming private modules | Module registry setup |
| [No-Code Provisioning](https://developer.hashicorp.com/terraform/cloud-docs/no-code-provisioning) | Self-service infrastructure | Service Catalog implementation |
| [Policy Enforcement](https://developer.hashicorp.com/terraform/cloud-docs/policy-enforcement) | Sentinel and OPA integration | Governance and compliance |

### **Vault Integration**

| Resource | Description | When to Use |
|----------|-------------|-------------|
| [Vault Provider Documentation](https://registry.terraform.io/providers/hashicorp/vault/latest/docs) | Terraform Vault provider reference | Retrieving secrets from Vault |
| [Vault AWS Secrets Engine](https://developer.hashicorp.com/vault/docs/secrets/aws) | Dynamic AWS credentials | Generating short-lived AWS keys |
| [Vault Azure Secrets Engine](https://developer.hashicorp.com/vault/docs/secrets/azure) | Dynamic Azure credentials | Generating short-lived Azure tokens |
| [Vault Database Secrets](https://developer.hashicorp.com/vault/docs/secrets/databases) | Dynamic database credentials | Rotating DB passwords |

---

## 🏆 Azure Verified Modules (AVM)

If working with Azure-specific modules:

| Resource | Description | When to Use |
|----------|-------------|-------------|
| [AVM Terraform Specifications](https://aka.ms/avm/tf/spec) | Official Terraform module requirements | Azure module development |
| [AVM Module Index](https://aka.ms/avm/index/tf) | Searchable list of published modules | Finding Azure modules |
| [AVM Contributing Guide](https://aka.ms/avm/contribute) | How to contribute to AVM | Publishing Azure modules |

---

## 🛠️ Tools & Utilities

### **Testing & Validation**

| Tool | Purpose | Link |
|------|---------|------|
| **terraform-docs** | Auto-generate README documentation | [github.com/terraform-docs/terraform-docs](https://github.com/terraform-docs/terraform-docs) |
| **tflint** | Linter for Terraform code | [github.com/terraform-linters/tflint](https://github.com/terraform-linters/tflint) |
| **checkov** | Security & compliance scanning | [checkov.io](https://www.checkov.io/) |
| **tfsec** | Security scanner for Terraform | [github.com/aquasecurity/tfsec](https://github.com/aquasecurity/tfsec) |
| **terrascan** | Policy-as-code scanning | [runterrascan.io](https://runterrascan.io/) |

### **Development Experience**

| Tool | Purpose | Link |
|------|---------|------|
| **pre-commit** | Git hooks framework | [pre-commit.com](https://pre-commit.com/) |
| **terraform-pre-commit** | Pre-commit hooks for Terraform | [github.com/antonbabenko/pre-commit-terraform](https://github.com/antonbabenko/pre-commit-terraform) |
| **GitLeaks** | Detect secrets in code | [github.com/gitleaks/gitleaks](https://github.com/gitleaks/gitleaks) |
| **TruffleHog** | Find high-entropy secrets | [github.com/trufflesecurity/trufflehog](https://github.com/trufflesecurity/trufflehog) |

### **CI/CD Integrations**

| Platform | Terraform Integration | Link |
|----------|----------------------|------|
| **GitHub Actions** | Setup Terraform action | [github.com/hashicorp/setup-terraform](https://github.com/hashicorp/setup-terraform) |
| **GitLab CI** | Terraform CI/CD templates | [docs.gitlab.com/ee/user/infrastructure/](https://docs.gitlab.com/ee/user/infrastructure/) |
| **Azure DevOps** | Terraform extension | [marketplace.visualstudio.com/terraform](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.custom-terraform-tasks) |
| **Jenkins** | Terraform plugin | [plugins.jenkins.io/terraform](https://plugins.jenkins.io/terraform/) |

---

## 📖 Learning Resources

### **Books**

| Title | Author | Focus | Link |
|-------|--------|-------|------|
| **Terraform: Up & Running** | Yevgeniy Brikman | Comprehensive Terraform guide | [terraformupandrunning.com](https://www.terraformupandrunning.com/) |
| **Terraform Best Practices** | HashiCorp | Official best practices | [hashicorp.com/resources](https://www.hashicorp.com/resources) |

### **Video Tutorials**

| Resource | Provider | Topics |
|----------|----------|--------|
| **HashiCorp Learn** | HashiCorp | Official tutorials, hands-on labs |
| **Terraform YouTube Channel** | HashiCorp | Weekly webinars, product updates |
| **HashiCorp User Groups** | Community | Real-world case studies |

### **Courses**

| Platform | Course | Level |
|----------|--------|-------|
| **HashiCorp Learn** | [Terraform Associate Certification](https://developer.hashicorp.com/terraform/tutorials/certification-003) | Beginner to Intermediate |
| **A Cloud Guru** | Terraform courses | All levels |
| **Udemy** | Terraform courses | All levels |

---

## 👥 Community Resources

### **HashiCorp Community**

| Resource | Description | Link |
|----------|-------------|------|
| **HashiCorp Discuss** | Official community forum | [discuss.hashicorp.com](https://discuss.hashicorp.com/) |
| **Terraform Registry** | Public module examples | [registry.terraform.io](https://registry.terraform.io/) |
| **HashiCorp User Groups** | Local meetups and events | [hashicorp.com/user-groups](https://www.hashicorp.com/community/user-groups) |

### **Public Module Repositories (Examples)**

Browse these for inspiration and patterns:

| Repository | Provider | Focus |
|------------|----------|-------|
| **terraform-aws-modules** | AWS Community | High-quality AWS modules |
| **terraform-google-modules** | Google | Google Cloud modules |
| **Azure Verified Modules** | Microsoft | Azure modules |

**GitHub Organizations**:
- [github.com/terraform-aws-modules](https://github.com/terraform-aws-modules)
- [github.com/terraform-google-modules](https://github.com/terraform-google-modules)
- [github.com/Azure/terraform-azurerm-modules](https://github.com/Azure)

---

## 🔍 Advanced Topics

### **Policy-as-Code**

| Technology | Purpose | Link |
|------------|---------|------|
| **Sentinel** | Policy language for HCP Terraform | [docs.hashicorp.com/sentinel](https://docs.hashicorp.com/sentinel) |
| **Open Policy Agent (OPA)** | Open-source policy engine | [openpolicyagent.org](https://www.openpolicyagent.org/) |

### **GitOps for Infrastructure**

| Tool | Description | Link |
|------|-------------|------|
| **Atlantis** | Terraform pull request automation | [runatlantis.io](https://www.runatlantis.io/) |
| **Terragrunt** | Terraform wrapper for DRY configurations | [terragrunt.gruntwork.io](https://terragrunt.gruntwork.io/) |
| **Spacelift** | Collaborative infrastructure platform | [spacelift.io](https://spacelift.io/) |

### **Cost Management**

| Tool | Purpose | Link |
|------|---------|------|
| **Infracost** | Cost estimates in pull requests | [infracost.io](https://www.infracost.io/) |
| **terraform-cost-estimation** | CLI cost estimation | [github.com/antonbabenko/terraform-cost-estimation](https://github.com/antonbabenko/terraform-cost-estimation) |

---

## 📊 Example Repositories

### **Testing Examples**

| Repository | Description |
|------------|-------------|
| [hashicorp-education/learn-terraform-test](https://github.com/hashicorp-education/learn-terraform-test) | Official testing tutorial repository |
| [Azure/terraform-azurerm-avm-template](https://github.com/Azure/terraform-azurerm-avm-template) | Azure Verified Module template with tests |

### **CI/CD Examples**

| Repository | CI/CD Platform | Description |
|------------|----------------|-------------|
| [hashicorp/terraform-github-actions](https://github.com/hashicorp/terraform-github-actions) | GitHub Actions | Official GitHub Actions examples |
| [hashicorp/terraform-gitlab-ci](https://gitlab.com/hashicorp-demo/terraform-examples) | GitLab CI | GitLab CI/CD templates |

---

## 🔐 Security Resources

### **Dynamic Credentials**

| Provider | Documentation | Description |
|----------|--------------|-------------|
| **AWS** | [OIDC Identity Provider](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html) | GitHub Actions to AWS without keys |
| **Azure** | [Workload Identity](https://learn.microsoft.com/azure/active-directory/workload-identities/) | Azure AD workload identity |
| **GCP** | [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) | GCP federation for CI/CD |

### **Secret Scanning**

| Tool | Purpose | Link |
|------|---------|------|
| **GitLeaks** | Scan repos for secrets | [github.com/gitleaks/gitleaks](https://github.com/gitleaks/gitleaks) |
| **TruffleHog** | Find credentials in git history | [github.com/trufflesecurity/trufflehog](https://github.com/trufflesecurity/trufflehog) |
| **GitHub Secret Scanning** | Built-in secret detection | [docs.github.com/code-security](https://docs.github.com/en/code-security/secret-scanning) |

---

## 📞 HashiCorp Support

### **How to Get Help**

| Channel | When to Use | Response Time |
|---------|-------------|---------------|
| **Community Discuss** | General questions, best practices | Community-driven |
| **GitHub Issues** | Bug reports, feature requests | Varies |
| **HashiCorp Support** | Customers with support contracts | Per SLA (typically 24-48 hours) |
| **Professional Services** | Implementation help, custom solutions | Contact sales |

### **Contact Information**

- **Sales**: [hashicorp.com/contact-sales](https://www.hashicorp.com/contact-sales)
- **Support Portal**: [support.hashicorp.com](https://support.hashicorp.com/)
- **Community**: [discuss.hashicorp.com](https://discuss.hashicorp.com/)

---

## 🗺️ Quick Reference Cheat Sheet

### **Most Important Links (Bookmark These)**

1. **Testing**: https://developer.hashicorp.com/terraform/language/tests
2. **Module Creation**: https://developer.hashicorp.com/terraform/tutorials/modules/pattern-module-creation
3. **HCP Terraform Docs**: https://developer.hashicorp.com/terraform/cloud-docs
4. **Terraform Registry**: https://registry.terraform.io/
5. **Community Forum**: https://discuss.hashicorp.com/

### **Quick Searches**

- **Find a provider**: https://registry.terraform.io/browse/providers
- **Find a module**: https://registry.terraform.io/browse/modules
- **Terraform changelog**: https://github.com/hashicorp/terraform/blob/main/CHANGELOG.md
- **Provider documentation**: `https://registry.terraform.io/providers/{namespace}/{name}/latest/docs`

---

## 📌 Workshop-Specific Resources

Materials created during this workshop:

- [Current State Assessment](./01-current-state-assessment.md)
- [Repository & Module Registry](./02-repository-and-registry.md)
- [Security & Local Development](./03-security-and-local-dev.md)
- [Testing Strategy](./04-testing-strategy.md)
- [Vault Integration](./05-vault-integration.md)
- [Business Case & Change Management](./06-business-case-and-change.md)
- [Technical Workshops](./07-technical-workshops.md)
- [Workshop Agenda](./workshop-agenda.md)
- [Key Messages](./key-messages.md)
- [Success Criteria](./success-criteria.md)

---

**Next Steps**: Bookmark the HashiCorp resources most relevant to your role and organization. Join the community forum to stay updated on best practices.
