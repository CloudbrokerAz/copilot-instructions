# Current State Assessment

## 🎯 Purpose

This document provides structured questions to understand the organization's existing Terraform practices, identify pain points, and establish a baseline for measuring improvement. Use this as the foundation for all workshop discussions.

## 📊 Assessment Framework

### **Goals of This Assessment**

1. **Understand the developer experience** - What's their daily workflow?
2. **Identify organizational constraints** - What policies/mandates exist and why?
3. **Quantify pain points** - How much time/effort is wasted?
4. **Map stakeholders** - Who are the decision-makers and influencers?
5. **Establish success metrics** - What would "better" look like?

### **Assessment Approach**

- **Open-ended questions** - Let them tell their story
- **Avoid judgment** - Past decisions made sense in their context
- **Seek specifics** - "Walk me through..." gets real details
- **Quantify where possible** - Time, frequency, cost, satisfaction scores
- **Document everything** - You'll reference this throughout the workshop

---

## 🔍 Developer Experience

### **Daily Workflow Questions**

**1. "Walk me through how a developer makes a change to a module today—from idea to production."**

*Why this matters*: Reveals the actual developer experience, pain points, and where process improvements would have the most impact. You want to understand:
- How many steps are involved?
- How many teams need to approve/review?
- Where do developers get blocked?
- What's the total time from commit to deployed?

**2. "What percentage of a developer's time is spent waiting for pipelines vs actual coding?"**

*Why this matters*: Quantifies productivity impact. Slow feedback loops = frustrated developers, slower innovation, higher costs.
- Target metric to capture: "X hours per week waiting"
- Calculate annual cost: Hours × hourly rate × number of developers

**3. "How long does it take to test a one-line change in a module?"**

*Why this matters*: Highlights pipeline tax. Should be seconds (local), not minutes/hours (pipeline).
- Best-in-class: < 30 seconds (local validation + unit tests)
- Typical: 5-15 minutes (CI/CD pipeline)
- Their current state: ???

**4. "When something goes wrong, how does a developer troubleshoot it?"**

*Why this matters*: Reveals debugging capabilities and support burden.
- Can they reproduce locally?
- Do they have access to logs?
- How often do they need to involve senior engineers or other teams?

**5. "How would you rate developer satisfaction with the current Terraform workflow on a scale of 1-10?"**

*Why this matters*: Low satisfaction = attrition risk, slower delivery, lower code quality. Builds urgency.
- Get specific: "What would make it an 8 or 9?"
- Document their pain points in their own words

---

## 🏗️ Architecture & Tooling

### **Module Organization Questions**

**1. "How do you discover what modules exist and which ones you should use?"**

*Why this matters*: Without a registry, discoverability is poor. Teams may duplicate work or use wrong modules.
- Do they have a catalog/wiki?
- Is it up to date?
- Are there examples?
- Can you search?

**2. "How do you handle module versioning? Can consumers pin to specific versions?"**

*Why this matters*: Mono-repos make versioning difficult. This drives module coupling, breaking changes propagating uncontrolled, and inability to have stable/dev versions simultaneously.
- Are versions tracked via Git tags/branches?
- Can different teams use different versions?
- How do you communicate breaking changes?

**3. "When multiple teams need different versions of the same module, how do you handle that?"**

*Why this matters*: Tests whether they've hit the ceiling of mono-repo limitations. Highlights need for proper module registry.
- Do teams maintain forks?
- Do you create new modules with version suffixes (module-v1, module-v2)?
- Do you force everyone to upgrade together?

**4. "How often do breaking changes in one module unexpectedly affect other teams?"**

*Why this matters*: Quantifies the blast radius of changes in mono-repo. Builds business case for module isolation.
- Frequency: Per month? Per quarter?
- Impact: Minor fix or production incident?
- Resolution time: Hours? Days?

---

## 🔒 Security & Compliance

### **Security Policy Questions**

**1. "Walk me through the security concern that led to the no-local-execution policy."**

*Why this matters*: Identify root concern to propose targeted mitigations rather than blanket restrictions.
- Was there a specific incident?
- Is this a compliance requirement?
- Who mandated this policy?
- Has it been reviewed recently?

**2. "Has the no-local-execution policy successfully prevented API key leakage?"**

*Why this matters*: Challenge the assumption. If incidents still occur, policy isn't working. Opens discussion on better solutions.
- Any incidents since policy enacted?
- Any workarounds developers are using?
- Any unintended consequences?

**3. "What's the security team's core concern—accidental commits, developer machines, or something else?"**

*Why this matters*: Different concerns need different solutions.
- Accidental commits → Git hooks, pre-commit scanning
- Developer machines → Endpoint security, dynamic credentials
- Auditability → Centralized logging, remote execution

**4. "How are cloud credentials managed today for pipeline execution?"**

*Why this matters*: If pipelines have credentials, the security boundary isn't "local vs pipeline"—it's credential lifecycle management.
- Long-lived keys? (❌ Bad)
- Short-lived tokens? (✅ Better)
- Dynamic credentials from Vault? (✅✅ Best)

---

## 🧪 Testing & Quality

### **Current Testing Practices**

**1. "What does 'testing Terraform' mean to your team today?"**

*Why this matters*: Reveals understanding gap. Many teams conflate validation with testing.
- Do they mean `terraform validate`?
- Manual inspection of plans?
- Deploying to dev environment?
- Automated tests?

**2. "How do you validate a module works before releasing it to other teams?"**

*Why this matters*: Shows current quality assurance process (or lack thereof).
- Is there a release process?
- Who approves releases?
- How do you test it works?
- How do you prevent regressions?

**3. "When a module changes, how do you know it didn't break existing consumers?"**

*Why this matters*: Highlights need for regression testing. Mono-repo makes this harder.
- Do you have automated checks?
- Do consumers report breakage after the fact?
- How long does it take to identify a breaking change?

**4. "Have you experienced cases where a module change broke production? What would have prevented it?"**

*Why this matters*: Real pain points build urgency for proper testing.
- Get the story—what happened?
- What was the impact? (Downtime, cost, customer impact)
- How was it fixed?
- Has it happened more than once?

**5. "What's stopping you from implementing tests today?"**

*Why this matters*: Reveals blockers—knowledge gap, tooling, pipeline restrictions, or cost concerns.
- "We don't know how" → Education need
- "No time" → Prioritization problem
- "Too expensive" → Cost/benefit discussion needed
- "Can't run locally" → Mocking solution

---

## 🔐 Vault Integration

### **Secret Management Questions**

**1. "How do Terraform modules currently retrieve secrets from Vault?"**

*Why this matters*: Reveals whether they're using Vault properly (dynamic secrets, short TTLs) or as a static secret store.
- Do they use `vault_generic_secret`? (Static)
- Do they use dynamic secret engines? (Dynamic - better)
- Are secrets injected as environment variables?

**2. "What's the relationship between the Vault team and the Terraform team?"**

*Why this matters*: Organizational silos often create friction. Shared goals and collaboration reduce friction.
- Do they talk regularly?
- Is there shared documentation?
- Do they have joint design reviews?
- Who owns Vault policies for Terraform?

**3. "Do modules use Vault's dynamic secret engines (AWS, Azure, database credentials) or static secrets?"**

*Why this matters*: Dynamic secrets are the right pattern. Static secrets from Vault defeats the purpose.
- Dynamic = Vault generates short-lived credentials on-demand
- Static = Vault stores pre-created credentials

**4. "How does Vault authentication work for Terraform runs in your pipelines?"**

*Why this matters*: Checks if they're using secure patterns (AppRole, JWT, Kubernetes auth) vs tokens in environment variables.
- AppRole with short TTL? (Good)
- JWT/OIDC? (Better)
- Token in environment variable? (❌ Bad)

**5. "Have there been cases where Terraform couldn't access secrets it needed from Vault? What happened?"**

*Why this matters*: Identifies integration pain points.
- Token expiration issues?
- Policy misconfiguration?
- Network connectivity?
- How was it resolved?

---

## 📊 Metrics to Capture

During the assessment, try to quantify:

| Metric | Current State | Target State | Notes |
|--------|---------------|--------------|-------|
| Time to test a change | ___ minutes | < 1 minute (local) | Pipeline wait time |
| Module release frequency | ___ per month | 2-4x increase | How often modules updated |
| Breaking change incidents | ___ per quarter | < 1 per quarter | Unintended breakage |
| Developer satisfaction | ___ / 10 | 8+ / 10 | Anonymous survey |
| Time to onboard new dev | ___ days | < 1 day | Module discovery + setup |
| Vault integration issues | ___ per month | < 1 per month | Authentication/access problems |
| Test coverage | ___% | 80%+ | Modules with automated tests |
| Module discoverability | ___ minutes | < 5 minutes | Time to find right module |

---

## 🗺️ Stakeholder Mapping

### **Key People to Identify**

**1. "Who are the key decision-makers for changing Terraform practices?"**

Create a stakeholder map:

| Name | Role | Influence Level | Support Level | Key Concerns |
|------|------|-----------------|---------------|--------------|
| | | High/Med/Low | Champion/Neutral/Blocker | |
| | | | | |

**2. "What metrics matter most to leadership: speed to market, security, cost, or reliability?"**

*Why this matters*: Frame recommendations in terms of leadership priorities.
- Speed → Show how local dev + testing accelerates delivery
- Security → Show how dynamic credentials improve security
- Cost → Show ROI calculation (dev time saved)
- Reliability → Show how tests prevent incidents

**3. "What's the renewal/contract status with your current integration partner?"**

*Why this matters*: Timing matters. Renewal periods are decision windows.
- Contract ending soon → Opportunity to change
- Just renewed → Need internal change first
- No contract → Flexibility

**4. "Who would champion this modernization effort and have time to lead it?"**

*Why this matters*: Change needs a leader with authority and capacity.
- Identify potential champions in the room
- Understand their bandwidth
- Get their commitment

---

## 🎯 Vision & Success Definition

### **Future State Questions**

**1. "What would success look like 6 months from now?"**

*Why this matters*: Define shared vision. Makes abstract recommendations concrete.

Get specific commitments:
- "Developers can test modules locally in under 1 minute"
- "Zero unintended breaking changes from module updates"
- "All modules have automated test suites"
- "New developers can find and use modules in under 30 minutes"
- "Security team approves new local development workflow"

**2. "If you could wave a magic wand and fix one thing, what would it be?"**

*Why this matters*: Reveals top pain point. Start there.

**3. "What constraints are non-negotiable?"**

*Why this matters*: Understand hard boundaries vs soft preferences.
- Regulatory compliance requirements
- Corporate policies that won't change
- Budget limitations
- Resource constraints

---

## 📝 Assessment Output

By the end of this assessment, you should have:

- ✅ **Current state documentation** - Detailed workflow, architecture, constraints
- ✅ **Pain point inventory** - Quantified problems with time/cost estimates
- ✅ **Stakeholder map** - Decision-makers, champions, blockers
- ✅ **Success definition** - Shared vision of future state
- ✅ **Baseline metrics** - Numbers to measure improvement against
- ✅ **Prioritized opportunities** - Top 2-3 areas for improvement

This becomes the foundation for all subsequent workshop discussions and recommendations.

---

## 💡 Facilitation Tips

- **Start broad, then narrow** - Begin with "tell me about your workflow," then drill into specifics
- **Use silence** - After asking a question, wait. Let them think and elaborate.
- **Follow the energy** - If they're passionate about a topic, explore it deeper
- **Avoid leading questions** - Don't telegraph the "right" answer
- **Document verbatim** - Capture their exact words for pain points (useful for buy-in later)
- **Thank them for honesty** - Create safe space for sharing problems
- **Summarize back** - "So what I'm hearing is..." (confirms understanding)

---

**Next**: Once you have this assessment complete, move on to specific topic deep-dives:
- [Repository & Module Registry](./02-repository-and-registry.md)
- [Security & Local Development](./03-security-and-local-dev.md)
- [Testing Strategy](./04-testing-strategy.md)
