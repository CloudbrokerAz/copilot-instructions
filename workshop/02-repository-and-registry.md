# Repository Structure & Module Registry

## 🎯 Purpose

This document provides discussion frameworks for addressing mono-repository challenges and making the case for Terraform Private Module Registry (PMR). It covers repository organization strategies, module versioning, and the benefits of purpose-built tooling.

---

## 📦 The Mono-Repo Problem

### **Common Mono-Repo Pattern**

Many organizations start with:

```
terraform-mono-repo/
├── modules/
│   ├── networking/
│   ├── compute/
│   ├── database/
│   └── ...
├── environments/
│   ├── dev/
│   ├── staging/
│   └── production/
├── pipelines/
│   ├── module-validation/
│   └── deployment/
└── docs/
```

### **Why It Seems Like a Good Idea**

- **Simplicity**: Everything in one place
- **Discoverability**: One repo to search
- **Coordination**: Atomic commits across modules
- **Consistency**: Shared pipelines and standards

### **Why It Breaks Down at Scale**

1. **No True Versioning** - Git tags apply to entire repo, not individual modules
2. **Blast Radius** - One bad commit can affect everything
3. **Coupling** - Impossible to have different teams on different versions
4. **Slow CI/CD** - Every change triggers validation of everything
5. **Merge Conflicts** - Multiple teams editing same repo
6. **No Ownership Boundaries** - Unclear who owns what

---

## 🗣️ Discussion Guide: Repository Strategy

### **Understanding Current Pain Points**

**Questions to Ask:**

**1. "How do you handle module versioning in the mono-repo? Can consumers pin to specific versions?"**

*Why this matters*: Mono-repos make versioning difficult. This drives module coupling, breaking changes propagating uncontrolled, and inability to have stable/dev versions simultaneously.

**Follow-up probes:**
- "If I want to use version 1.2.0 of the networking module, how do I specify that?"
- "Can two applications use different versions of the same module?"
- "How do you communicate breaking changes to module consumers?"

**2. "When multiple teams need different versions of the same module, how do you handle that?"**

*Why this matters*: Tests whether they've hit the ceiling of mono-repo limitations. Highlights need for proper module registry.

**Common answers and responses:**
- "We force everyone to upgrade" → What if upgrade breaks their app?
- "We duplicate the module with v1/v2 directories" → How many copies exist?
- "We maintain separate branches" → How do you merge improvements?

**3. "How often do breaking changes in one module unexpectedly affect other teams?"**

*Why this matters*: Quantifies the blast radius of changes in mono-repo. Builds business case for module isolation.

**Capture:**
- Frequency (per month/quarter)
- Example incidents
- Time to detect and fix
- Teams impacted

**4. "How do you discover what modules exist and which ones you should use?"**

*Why this matters*: Without a registry, discoverability is poor. Teams may duplicate work or use wrong modules.

**Follow-up probes:**
- "Is there a catalog or documentation?"
- "How does a new team member find the right module?"
- "Are there examples of how to use each module?"
- "How do you know which modules are supported vs experimental?"

---

## 🏗️ Alternative Repository Strategies

### **Option 1: Multi-Repo with Module Registry**

**Structure:**
```
terraform-module-networking/          (Individual repo)
├── main.tf
├── variables.tf
├── outputs.tf
├── README.md
├── examples/
└── tests/

terraform-module-compute/             (Individual repo)
├── main.tf
├── ...

terraform-module-database/            (Individual repo)
├── main.tf
├── ...
```

**Benefits:**
- ✅ Independent versioning (semantic versioning per module)
- ✅ Clear ownership (one team per repo)
- ✅ Isolated changes (changes to networking don't affect compute)
- ✅ Faster CI/CD (only test what changed)
- ✅ Flexible consumption (pin versions independently)

**Challenges:**
- ⚠️ More repos to manage
- ⚠️ Cross-module changes require coordination
- ⚠️ Need a registry to discover modules

---

### **Option 2: Module Monorepo with Registry**

**Structure:**
```
terraform-modules/                    (Mono-repo)
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   └── tests/
│   ├── compute/
│   └── database/
└── .github/
    └── workflows/
        ├── networking-ci.yml         (Conditional: only runs if modules/networking/* changed)
        ├── compute-ci.yml
        └── database-ci.yml
```

**Benefits:**
- ✅ Simpler than multi-repo (one clone)
- ✅ Can do cross-module atomic commits
- ✅ Shared CI/CD tooling
- ✅ Easier discoverability in one repo

**Challenges:**
- ⚠️ Still need registry for versioning
- ⚠️ Requires smart CI/CD (conditional execution)
- ⚠️ Can still have merge conflicts

**Key Point**: Even with mono-repo, you **must** use a registry to publish versioned modules.

---

## 🗄️ Private Module Registry (PMR) Deep Dive

### **What is PMR?**

**Terraform Private Module Registry** is a purpose-built service (part of HCP Terraform/Terraform Enterprise) for managing internal Terraform modules.

**Core Capabilities:**
- 📦 Semantic versioning per module
- 📚 Integrated documentation (auto-generated from README)
- 🔍 Search and discovery UI
- 🔗 Dependency tracking (knows who uses what)
- 🏷️ Lifecycle management (latest, beta, deprecated tags)
- 🧪 Test integration (run tests before publishing)
- 🚫 No-code provisioning (generate UI from variables)
- 🔒 Access control (who can publish, who can consume)

---

## 🗣️ Discussion Guide: PMR vs Artifactory

### **Understanding the Artifactory Decision**

**Questions to Ask:**

**1. "What specific requirements led to choosing Artifactory over Terraform's Private Module Registry?"**

*Why this matters*: Understand if it was a deliberate decision or lack of awareness. Helps frame PMR benefits in their context.

**Common reasons and responses:**

| Reason | Response |
|--------|----------|
| "We already have Artifactory for everything" | PMR is purpose-built, not generic artifact storage. Different needs. |
| "PMR costs extra money" | True, but calculate dev time saved. PMR ROI is typically < 6 months. |
| "We didn't know PMR existed" | Let's evaluate it now against current pain points. |
| "Corporate standard is Artifactory" | Is there an exception process for specialized tools? |

**2. "How does your team consume modules from Artifactory today? What's the developer workflow?"**

*Why this matters*: Reveals friction points that PMR would solve (versioning, documentation, dependency tracking).

**Walk through their current process:**
1. How do they find the module?
2. How do they know which version to use?
3. Where is the documentation?
4. How do they reference it in their Terraform code?
5. How do they know about updates?

**3. "Can you search and browse available modules with documentation and examples?"**

*Why this matters*: PMR provides built-in documentation, version history, and usage examples. Artifactory doesn't.

**PMR provides:**
- Web UI for browsing modules
- README auto-rendering
- Version history with changelogs
- Usage examples
- Input/output documentation
- Dependency graph

**Artifactory provides:**
- File storage
- Version storage
- Access control
- That's it.

**4. "How do you track which modules are deprecated or which versions are stable vs experimental?"**

*Why this matters*: PMR has built-in lifecycle management and version tags.

**PMR capabilities:**
- Mark versions as `latest`, `beta`, `deprecated`
- Users can filter by stability
- Automated notifications for deprecated versions

**Artifactory:**
- Manual tagging in filenames or metadata
- No built-in lifecycle management

**5. "When a module has a security vulnerability, how do you identify and notify all consumers?"**

*Why this matters*: PMR tracks dependencies and consumers. Artifactory requires manual tracking.

**Scenario:**
> "Your networking module has a security issue. You need to notify all teams using it and ensure they upgrade. How do you do that today?"

**With Artifactory:**
- Manual search through codebases
- Hope people read Slack announcements
- No way to enforce upgrades

**With PMR + Policy:**
- Identify all consumers (PMR knows)
- Send targeted notifications
- Use Sentinel/OPA to block deployments with vulnerable versions

---

## 📊 PMR vs Artifactory Comparison

| Capability | Artifactory | Terraform PMR | Winner |
|------------|-------------|---------------|--------|
| **Generic artifact storage** | ✅ Yes | ❌ No | Artifactory |
| **Semantic versioning** | ⚠️ Manual | ✅ Built-in | **PMR** |
| **Module documentation** | ❌ None | ✅ Auto-rendered | **PMR** |
| **Search & discovery** | ⚠️ File search | ✅ Module registry UI | **PMR** |
| **Dependency tracking** | ❌ No | ✅ Yes | **PMR** |
| **Version constraints** | ⚠️ Manual | ✅ Built-in | **PMR** |
| **No-code provisioning** | ❌ No | ✅ Yes | **PMR** |
| **Test integration** | ❌ No | ✅ Yes | **PMR** |
| **Policy enforcement** | ❌ No | ✅ Yes (Sentinel/OPA) | **PMR** |
| **Usage analytics** | ⚠️ Download counts | ✅ Full tracking | **PMR** |

---

## 💰 Building the Business Case for PMR

### **Cost Analysis Framework**

**Current State Costs (with Artifactory):**

1. **Module Discovery Time**
   - Average time to find right module: ___ minutes
   - Frequency: ___ times per week
   - Developers affected: ___ people
   - **Annual cost**: `minutes × frequency × 52 weeks × developers × hourly rate`

2. **Breaking Change Incidents**
   - Average incidents per quarter: ___
   - Average resolution time: ___ hours
   - Teams impacted per incident: ___
   - **Annual cost**: `incidents × 4 × hours × teams × hourly rate`

3. **Duplicate Module Creation**
   - Modules duplicated because couldn't find existing: ___
   - Time to create duplicate: ___ hours
   - **Annual cost**: `duplicates × hours × hourly rate`

4. **Manual Version Management**
   - Time spent managing versions manually: ___ hours/month
   - **Annual cost**: `hours × 12 × hourly rate`

**Example Calculation:**

```
Current State Annual Cost:
- Module discovery: 30 min × 3 times/week × 52 weeks × 10 devs × $100/hr = $78,000
- Breaking changes: 2 incidents × 4 quarters × 8 hours × 3 teams × $100/hr = $19,200
- Duplicate creation: 3 duplicates × 40 hours × $100/hr = $12,000
- Manual versioning: 10 hours/month × 12 × $100/hr = $12,000
----------------------------------------
TOTAL: $121,200/year
```

**Future State Costs (with PMR):**

```
PMR Cost:
- HCP Terraform Team Edition: ~$20/user/month × 10 users × 12 months = $2,400/year
OR
- Terraform Enterprise: ~$50k-100k/year (for larger orgs)

Time Savings:
- Module discovery: 5 minutes (6x faster) = $65,000 saved
- Breaking changes: 50% reduction = $9,600 saved
- Duplicate creation: 80% reduction = $9,600 saved
- Manual versioning: 90% reduction = $10,800 saved
----------------------------------------
TOTAL SAVINGS: $95,000/year
NET ROI: $92,600/year (using HCP Terraform)
Payback period: < 2 weeks
```

---

## 🚀 Migration Strategy

### **Phased Approach (Recommended)**

**Phase 1: Pilot (1-2 months)**
- Select 2-3 high-value modules
- Publish to PMR
- 1-2 teams consume from PMR
- Keep Artifactory for everything else
- **Goal**: Prove value, identify issues

**Phase 2: New Modules First (2-3 months)**
- All new modules go to PMR
- Existing modules stay in Artifactory
- Document PMR publishing process
- Train module authors
- **Goal**: Establish new pattern

**Phase 3: Migrate High-Usage Modules (3-6 months)**
- Identify top 10 most-used modules
- Migrate to PMR one at a time
- Coordinate with consumers
- Update documentation
- **Goal**: Maximize PMR value

**Phase 4: Long Tail Migration (6-12 months)**
- Migrate remaining active modules
- Deprecate unused modules
- Decommission Artifactory for Terraform
- **Goal**: Complete transition

---

## 🗣️ Discussion Guide: Getting Buy-In

### **Questions to Build Consensus**

**1. "If module discovery and versioning were simpler, how much time would that save per week?"**

*Get specific answers from different personas:*
- Module authors: "I spend X hours managing versions"
- Module consumers: "I spend Y hours finding the right module"
- Operations: "I spend Z hours troubleshooting version conflicts"

**2. "Would self-service infrastructure with built-in guardrails reduce your team's support burden?"**

*Paint the picture:*
- Developers can browse modules in a UI
- Documentation is always up-to-date
- Examples are embedded
- No need to ask "which module should I use?"

**3. "How would you prioritize: faster development vs maintaining current tooling?"**

*Frame as strategic decision:*
- Option A: Keep Artifactory, continue current pain
- Option B: Add PMR, accelerate development
- "What would leadership choose?"

**4. "Can we run a pilot with one team and measure outcomes?"**

*Reduce risk:*
- 4-week pilot
- 1 team, 2-3 modules
- Measure time savings
- Gather feedback
- Decision point: Continue or stop

---

## 📋 Recommendation Template

Use this template to formally recommend PMR:

---

### **Recommendation: Adopt Terraform Private Module Registry**

**Problem Statement:**
Our current Terraform module management approach using Artifactory creates friction in:
- Module versioning and dependency management
- Module discovery and documentation
- Breaking change prevention
- Developer productivity

**Proposed Solution:**
Adopt Terraform Private Module Registry (PMR) via HCP Terraform/Terraform Enterprise.

**Benefits:**
- **Faster Development**: Reduce module discovery time by 80% (from 30 min to 5 min)
- **Fewer Incidents**: Reduce breaking changes by 50% through proper versioning
- **Better DX**: Self-service module consumption with documentation and examples
- **Improved Security**: Track dependencies, enforce policies, identify vulnerabilities

**Costs:**
- HCP Terraform: $X/year (or Terraform Enterprise: $Y/year)
- Implementation time: Z hours (pilot phase)
- Migration effort: W hours (spread over 12 months)

**ROI:**
- Annual savings: $95,000 (time savings + incident reduction)
- Payback period: < 3 months
- 5-year NPV: $XXX,000

**Risks & Mitigations:**
- **Risk**: Team resistance to change → **Mitigation**: Pilot with volunteers, show value
- **Risk**: Integration complexity → **Mitigation**: HashiCorp Professional Services support
- **Risk**: Cost approval → **Mitigation**: Executive sponsor, clear ROI

**Recommendation:**
Approve 4-week pilot with Team X, Modules Y & Z. Decision point after pilot to proceed or stop.

---

## 💡 Key Messages to Emphasize

1. **Purpose-Built Tools Win** - Artifactory is great for artifacts, PMR is great for Terraform modules. Different tools for different jobs.

2. **Versioning Is Not Optional** - Without proper versioning, you can't have stable production and active development simultaneously.

3. **Developer Experience Matters** - Happy developers = faster delivery = competitive advantage.

4. **This Is About Velocity** - PMR isn't just "nice to have," it's a productivity multiplier.

5. **Start Small, Prove Value** - Don't need to migrate everything day 1. Pilot, learn, scale.

---

**Next Steps:**
- [Security & Local Development](./03-security-and-local-dev.md)
- [Testing Strategy](./04-testing-strategy.md)
