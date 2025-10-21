# Terraform Module Design & Development - Copilot Workspace

This repository contains **AI workspace instructions and prompts** to assist with **Terraform module design, development, and best practices** across multiple cloud providers.

## 🎯 Purpose

Provide intelligent, context-aware assistance for Terraform module creation by combining:
- **Azure Verified Modules (AVM)** specifications and requirements
- **HashiCorp official module design principles**
- **Multi-cloud best practices** (Azure, AWS, GCP, and more)
- **Terraform MCP Server integration** for live documentation

## 📂 Repository Structure

```
copilot-instructions/
├── .github/
│   └── copilot-instructions.md    # Persistent AI context (always active)
├── .prompts/
│   ├── design-new-module.prompt   # Design modules from scratch
│   ├── review-module.prompt       # Comprehensive module reviews
│   ├── lookup-requirement.prompt  # Find specific requirements
│   ├── compare-approaches.prompt  # Evaluate design alternatives
│   ├── generate-docs.prompt       # Create documentation
│   ├── README.md                  # Prompt usage guide
│   └── QUICK-REFERENCE.md         # Cheat sheet
└── README.md                      # This file
```

## 🚀 Quick Start

### 1. Clone This Repository

```bash
git clone https://github.com/<your-org>/copilot-instructions.git
cd copilot-instructions
```

### 2. Open in VS Code

The `.github/copilot-instructions.md` file automatically provides context to GitHub Copilot in ALL conversations.

### 3. Start Using!

#### For General Questions
Just ask! The workspace instructions are always active:

```
"What are the scoping principles for Terraform modules?"
"How should I structure a resource module for AWS Lambda?"
"Explain the difference between resource and pattern modules"
```

#### For Specific Tasks
Use prompt files in `.prompts/` directory:

1. Open the relevant `.prompt` file
2. Read the instructions
3. Fill in your specific details
4. Select all (Cmd/Ctrl + A)
5. Send to Copilot Chat

## 📚 What You Get

### Persistent Context (Always Active)

The `.github/copilot-instructions.md` file provides:

✅ **Multi-Cloud Support** - Azure, AWS, GCP, and other providers
✅ **Dual Frameworks** - HashiCorp + Azure Verified Modules principles
✅ **Module Scoping** - Encapsulation, Privileges, Volatility dimensions
✅ **Classifications** - Resource, Pattern, and Utility module types
✅ **MVP Philosophy** - 80% use case targeting, sensible defaults
✅ **Workflow Integration** - VCS-driven, API-driven, CLI-driven workflows
✅ **Consumption Models** - Service Catalog vs Infrastructure Franchise
✅ **Terraform MCP Server** - Live provider/module documentation lookup

### Task-Specific Prompts

| Prompt File | Use Case | What It Does |
|-------------|----------|--------------|
| `design-new-module.prompt` | Starting new module | Guides through scoping, MVP definition, structure design |
| `review-module.prompt` | Module compliance review | Checks against HashiCorp + AVM requirements |
| `lookup-requirement.prompt` | Find specific requirement | Searches and explains AVM/HashiCorp requirements |
| `compare-approaches.prompt` | Design decision | Evaluates alternatives against best practices |
| `generate-docs.prompt` | Documentation creation | Creates comprehensive README, examples, changelog |

## 🎓 Key Concepts

### Module Scoping (HashiCorp)

Every module design should consider three dimensions:

1. **Encapsulation** - What infrastructure is deployed together?
2. **Privileges** - Who has permission to deploy this?
3. **Volatility** - How often does this infrastructure change?

**Rule**: If a module's purpose is hard to explain, it's too complex!

### Module Classifications

| Type | Definition | Naming (HashiCorp) | Naming (AVM) |
|------|------------|-------------------|--------------|
| **Resource** | Single primary resource | `terraform-<provider>-<resource>` | `avm-res-<provider>-<resource>` |
| **Pattern** | Multiple resources/solution | `terraform-<provider>-<pattern>` | `avm-ptn-<pattern>` |
| **Utility** | Functions/helpers | `terraform-<provider>-<utility>` | `avm-utl-<utility>` |

### MVP Approach (HashiCorp)

✅ **DO**:
- Target 80% of use cases
- Keep initial versions simple
- Expose only most commonly modified variables
- Maximize outputs (even if not immediately needed)
- Provide sensible, secure defaults

❌ **DON'T**:
- Code for edge cases in MVP
- Create multi-purpose modules
- Expose every possible parameter
- Use complex conditionals initially

### Terraform MCP Server Integration

Your workspace has access to the **Terraform MCP Server** for real-time documentation:

**Use it for**:
- Provider resource/data source schemas
- Module registry searches
- Policy examples
- Latest API documentation

**Example queries**:
- "Show me the latest arguments for `azurerm_storage_account`"
- "Find modules for Kubernetes on AWS"
- "Search for AWS compliance policies"

## 🔧 Configuration

### Terraform MCP Server Setup

Ensure your MCP server is configured in VS Code settings:

```json
{
  "mcp.servers": {
    "terraform": {
      "command": "terraform-mcp-server",
      "args": []
    }
  }
}
```

### Workspace Settings

Add this repository to your VS Code workspace for automatic context loading:

```json
{
  "folders": [
    { "path": "/path/to/copilot-instructions" },
    { "path": "/path/to/your/terraform/project" }
  ]
}
```

## 📖 Usage Examples

### Example 1: Design a New AWS Lambda Module

1. Open `.prompts/design-new-module.prompt`
2. Fill in:
   - **Cloud Provider**: AWS
   - **Primary Resource**: Lambda Function
   - **Module Type**: Resource
   - **Primary Use Cases**: Serverless APIs, event processing, scheduled tasks
3. Send to Copilot → Get complete module design

### Example 2: Review Existing Azure Storage Module

1. Open `.prompts/review-module.prompt`
2. Provide module path or code
3. Send to Copilot → Get compliance report with:
   - Scoping evaluation
   - AVM requirement checks
   - HashiCorp best practice validation
   - Actionable recommendations

### Example 3: Compare Design Approaches

1. Open `.prompts/compare-approaches.prompt`
2. Describe 2+ alternatives
3. Send to Copilot → Get analysis of:
   - Trade-offs
   - Compliance with requirements
   - Long-term maintenance implications
   - Recommended approach

## 🌐 Multi-Cloud Support

This workspace is designed to be **cloud-agnostic** while still providing cloud-specific guidance:

### For Azure
- Applies AVM requirements (TFNFR, TFFR, SNFR, SFR, RMFR, RMNFR, PMFR, PMNFR)
- Checks for standard interfaces (RBAC, locks, tags, diagnostic settings, Private Endpoints)
- References Azure Verified Modules documentation
- Validates WAF alignment

### For AWS/GCP/Other Providers
- Applies HashiCorp module design principles
- Uses Terraform MCP Server for provider-specific documentation
- Follows standard module structure and naming
- Emphasizes Well-Architected Framework principles

## 📋 Requirements Hierarchy

### HashiCorp (All Providers)
- Module creation workflow (scoping, MVP, nesting)
- Naming conventions
- Consumption models (Service Catalog, Infrastructure Franchise)
- Terraform workflows (VCS-driven, API-driven, CLI-driven)

### AVM (Azure Modules)
- **TFNFR** - Terraform Non-Functional Requirements
- **TFFR** - Terraform Functional Requirements
- **SNFR** - Shared Non-Functional Requirements
- **SFR** - Shared Functional Requirements
- **RMFR/RMNFR** - Resource Module requirements
- **PMFR/PMNFR** - Pattern Module requirements

## 🤝 Contributing

### Adding New Prompts

Create new `.prompt` files for additional workflows:

```markdown
# Prompt Title

Brief description of what this prompt does.

## Instructions

Step-by-step guide for using this prompt.

## Input Required

What information the user should provide.

---

## Your Request

[Placeholder for user input]
```

### Updating Instructions

Edit `.github/copilot-instructions.md` to add:
- New design principles
- Additional cloud provider guidance
- Custom organizational standards
- Specific compliance requirements

### Submitting Changes

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 🔗 References

### HashiCorp Official Documentation
- [Module Creation Patterns](https://developer.hashicorp.com/terraform/tutorials/modules/pattern-module-creation)
- [Module Overview](https://developer.hashicorp.com/terraform/tutorials/modules/module)
- [Consumption Models](https://developer.hashicorp.com/validated-designs/terraform-operating-guides-adoption/consumption-models)
- [Terraform Workflows](https://developer.hashicorp.com/validated-designs/terraform-operating-guides-adoption/terraform-workflows)

### Azure Verified Modules
- [AVM Specifications](https://azure.github.io/Azure-Verified-Modules/)
- [Terraform Requirements](https://azure.github.io/Azure-Verified-Modules/specs/tf/)
- [Module Classifications](https://azure.github.io/Azure-Verified-Modules/specs/shared/module-classifications/)

## 💡 Tips & Best Practices

### Module Design
1. **Start with scoping** - Apply encapsulation, privileges, volatility
2. **Think MVP first** - 80% use cases, no edge cases
3. **Maximize outputs** - Even if not immediately needed
4. **Document thoroughly** - README with examples
5. **Test extensively** - Unit, integration, example tests

### Workspace Usage
1. **Ask specific questions** - Better context = better answers
2. **Use prompts for complex tasks** - Structured workflows yield better results
3. **Reference requirements** - Cite TFNFR, HashiCorp principles
4. **Leverage MCP Server** - Get latest documentation
5. **Iterate and refine** - Start simple, enhance over time

### Collaboration
1. **Share prompts** - Help team with reusable templates
2. **Document decisions** - Maintain decision log
3. **Version modules** - Semantic versioning (x.y.z)
4. **Review before release** - Use review prompt
5. **Maintain changelog** - Track changes per version

## 🆘 Support

### Questions?
- Check `.prompts/README.md` for prompt usage guide
- Review `.prompts/QUICK-REFERENCE.md` for cheat sheet
- Ask Copilot using workspace context

### Issues?
- Verify Terraform MCP Server is running
- Check `.github/copilot-instructions.md` is present
- Ensure VS Code Copilot is enabled
- Review MCP server configuration

## 📜 License

[Specify your license here]

## 🙏 Acknowledgments

- **HashiCorp** for Terraform and official module design principles
- **Microsoft Azure** for Azure Verified Modules specifications
- **Terraform Community** for best practices and patterns

---

**Ready to build better Terraform modules?** Start asking questions or use the prompts! 🚀
