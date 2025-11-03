# Workshop Directory - Quick Navigation

## 📂 Directory Structure

```
workshop/
├── README.md                              # Start here - Overview and usage guide
├── 01-current-state-assessment.md         # Discovery questions and baseline metrics
├── 02-repository-and-registry.md          # Mono-repo vs PMR discussion guide
├── 03-security-and-local-dev.md           # Security AND developer experience solutions
├── 04-testing-strategy.md                 # Unit, integration, and mocking guide
├── 05-vault-integration.md                # Dynamic secrets and auth patterns
├── 06-business-case-and-change.md         # ROI calculation and change management
├── 07-technical-workshops.md              # Hands-on exercises (module design, testing)
├── workshop-agenda.md                     # 2-day workshop schedule
├── key-messages.md                        # Talking points by audience
├── resources.md                           # HashiCorp docs, tools, community links
└── success-criteria.md                    # Measurable outcomes and decision framework
```

---

## 🎯 Who Should Use What

### **Workshop Facilitators**

**Essential reading**:
1. [README.md](./README.md) - Complete overview
2. [workshop-agenda.md](./workshop-agenda.md) - Schedule and flow
3. [key-messages.md](./key-messages.md) - How to frame discussions
4. All numbered guides (01-07) - Deep content knowledge

**Day-of materials**:
- [Current State Assessment](./01-current-state-assessment.md) - Questions to ask
- [Technical Workshops](./07-technical-workshops.md) - Hands-on exercises
- [Success Criteria](./success-criteria.md) - What defines success

---

### **Workshop Participants (Engineers)**

**Pre-workshop**:
- [README.md](./README.md) - What to expect

**During workshop**:
- Follow facilitator's guidance
- Focus on hands-on sections in [Technical Workshops](./07-technical-workshops.md)

**Post-workshop**:
- [Testing Strategy](./04-testing-strategy.md) - Reference for writing tests
- [Security & Local Dev](./03-security-and-local-dev.md) - Dynamic credentials patterns
- [Resources](./resources.md) - Learning materials

---

### **Leadership / Decision Makers**

**Essential reading**:
1. [README.md](./README.md) - Overview
2. [Business Case & Change Management](./06-business-case-and-change.md) - ROI and roadmap
3. [Key Messages](./key-messages.md) - Strategic framing
4. [Success Criteria](./success-criteria.md) - How we'll measure success

**Optional deep dives**:
- [Repository & Module Registry](./02-repository-and-registry.md) - PMR business case
- [Security & Local Dev](./03-security-and-local-dev.md) - Security improvements

---

### **Security Team**

**Essential reading**:
1. [Security & Local Dev](./03-security-and-local-dev.md) - Dynamic credentials, mocking, policies
2. [Vault Integration](./05-vault-integration.md) - Auth methods and secret engines
3. [Current State Assessment](./01-current-state-assessment.md) - Security questions

**Key sections**:
- Dynamic short-lived credentials (better than current!)
- Mocked provider testing (zero cloud access)
- Security comparison matrices

---

### **Platform / DevOps Team**

**Essential reading**:
1. All numbered guides (01-07) - Complete technical context
2. [Technical Workshops](./07-technical-workshops.md) - Implementation patterns
3. [Testing Strategy](./04-testing-strategy.md) - CI/CD integration

**Implementation guides**:
- [Repository & Module Registry](./02-repository-and-registry.md) - Migration strategy
- [Testing Strategy](./04-testing-strategy.md) - Test framework
- [Vault Integration](./05-vault-integration.md) - Vault setup

---

## 🚀 Workshop Flow

### **Day 1: Discovery & Vision**

| Time | Document | Purpose |
|------|----------|---------|
| **AM** | [01-current-state-assessment.md](./01-current-state-assessment.md) | Understand pain points |
| | [02-repository-and-registry.md](./02-repository-and-registry.md) | Discuss module management |
| **PM** | [03-security-and-local-dev.md](./03-security-and-local-dev.md) | Address security concerns |
| | [04-testing-strategy.md](./04-testing-strategy.md) | Introduce testing approach |
| | [05-vault-integration.md](./05-vault-integration.md) | Improve Vault patterns |

### **Day 2: Hands-On & Planning**

| Time | Document | Purpose |
|------|----------|---------|
| **AM** | [07-technical-workshops.md](./07-technical-workshops.md) | Module design exercise |
| | [07-technical-workshops.md](./07-technical-workshops.md) | Testing implementation |
| **PM** | [06-business-case-and-change.md](./06-business-case-and-change.md) | Build ROI case |
| | [success-criteria.md](./success-criteria.md) | Define pilot |
| | | Wrap-up and next steps |

---

## 📊 Document Sizes

For planning printing/reading time:

| Document | Pages (approx) | Read Time |
|----------|----------------|-----------|
| README.md | 5 | 15 min |
| 01-current-state-assessment.md | 15 | 30 min |
| 02-repository-and-registry.md | 18 | 40 min |
| 03-security-and-local-dev.md | 22 | 45 min |
| 04-testing-strategy.md | 25 | 50 min |
| 05-vault-integration.md | 20 | 40 min |
| 06-business-case-and-change.md | 18 | 35 min |
| 07-technical-workshops.md | 20 | 40 min (+ exercises) |
| workshop-agenda.md | 12 | 25 min |
| key-messages.md | 10 | 20 min |
| resources.md | 8 | 15 min |
| success-criteria.md | 12 | 25 min |
| **TOTAL** | ~185 pages | ~6 hours reading |

**Recommendation**: Facilitators should read all; participants receive relevant sections only.

---

## 🔍 Finding Specific Topics

### **Testing**
- Fundamentals → [04-testing-strategy.md](./04-testing-strategy.md)
- Hands-on exercises → [07-technical-workshops.md](./07-technical-workshops.md#workshop-2-testing-implementation)
- CI/CD integration → [04-testing-strategy.md](./04-testing-strategy.md) + [resources.md](./resources.md)

### **Security**
- Local development → [03-security-and-local-dev.md](./03-security-and-local-dev.md)
- Dynamic credentials → [03-security-and-local-dev.md](./03-security-and-local-dev.md) + [05-vault-integration.md](./05-vault-integration.md)
- Mocking → [04-testing-strategy.md](./04-testing-strategy.md)

### **Module Design**
- Scoping principles → [07-technical-workshops.md](./07-technical-workshops.md#workshop-1-module-design--refactoring)
- Repository strategy → [02-repository-and-registry.md](./02-repository-and-registry.md)
- Best practices → [resources.md](./resources.md) (HashiCorp links)

### **Business Case**
- ROI calculation → [06-business-case-and-change.md](./06-business-case-and-change.md)
- Stakeholder management → [06-business-case-and-change.md](./06-business-case-and-change.md)
- Executive summary → [06-business-case-and-change.md](./06-business-case-and-change.md)

### **Vault**
- Integration patterns → [05-vault-integration.md](./05-vault-integration.md)
- Authentication → [05-vault-integration.md](./05-vault-integration.md)
- Dynamic secrets → [05-vault-integration.md](./05-vault-integration.md)

---

## 💡 Tips for Navigation

### **In VS Code**
- Use Cmd/Ctrl+P → type filename to jump quickly
- Use outline view (Cmd/Ctrl+Shift+O) to navigate within files
- Search across files (Cmd/Ctrl+Shift+F) for specific topics

### **In GitHub**
- Use the "Go to file" feature (press `t` in repo)
- Click through links in README files
- Use GitHub's file search

### **Printing**
- Print README.md first (overview)
- Print only relevant sections for specific audiences
- Consider printing workshop-agenda.md for all participants

---

## 📞 Questions?

If you can't find what you're looking for:

1. **Start with** [README.md](./README.md) - Comprehensive overview
2. **Check** [resources.md](./resources.md) - External documentation links
3. **Review** [workshop-agenda.md](./workshop-agenda.md) - See where topics are covered
4. **Use search** - Search for keywords across all files

---

## 🎉 Ready to Start?

**Facilitators**: Begin with [README.md](./README.md)  
**Participants**: Wait for facilitator guidance  
**Leadership**: Jump to [Business Case & Change Management](./06-business-case-and-change.md)

Good luck with your Terraform transformation! 🚀
