# Workshop Agenda

## 🎯 Recommended 2-Day Workshop Structure

This agenda provides a suggested flow for a comprehensive Terraform modernization workshop. Adjust timing and topics based on your specific organizational needs.

---

## 📅 Day 1: Discovery & Vision

### **Goal**: Understand current state, align on problems, and build vision for future state

---

### **8:30 AM - 9:00 AM: Welcome & Introductions (30 min)**

**Format**: Plenary

**Activities**:
- Introductions (name, role, Terraform experience)
- Workshop objectives and agenda
- Ground rules (psychological safety, participation, etc.)
- Icebreaker: "What's your biggest Terraform pain point?"

**Facilitator prep**:
- Name tags
- Agenda printed
- Whiteboard/Miro ready

---

### **9:00 AM - 10:30 AM: Current State Assessment (90 min)**

**Format**: Facilitated discussion using [01-current-state-assessment.md](./01-current-state-assessment.md)

**Topics**:
- Developer workflow walkthrough (someone live demos)
- Pain points identification (capture on whiteboard)
- Quantify impact (time, cost, satisfaction)
- Organizational constraints

**Activities**:
- Live demo of current workflow
- Facilitated Q&A using assessment questions
- Stakeholder mapping exercise
- Pain point prioritization (dot voting)

**Deliverables**:
- Documented current state
- Prioritized pain points
- Baseline metrics

---

### **10:30 AM - 10:45 AM: Break**

---

### **10:45 AM - 12:00 PM: Repository & Module Registry Discussion (75 min)**

**Format**: Facilitated discussion using [02-repository-and-registry.md](./02-repository-and-registry.md)

**Topics**:
- Mono-repo challenges
- Module versioning issues
- Private Module Registry (PMR) overview
- PMR vs Artifactory comparison

**Activities**:
- Diagram current repository structure
- Discuss versioning pain points
- PMR demonstration (facilitator shows HCP Terraform UI)
- Cost/benefit discussion

**Deliverables**:
- Repository strategy options
- PMR business case framework
- Migration approach (if applicable)

---

### **12:00 PM - 1:00 PM: Lunch**

---

### **1:00 PM - 2:30 PM: Security & Local Development (90 min)**

**Format**: Facilitated discussion + demonstrations using [03-security-and-local-dev.md](./03-security-and-local-dev.md)

**Topics**:
- Understanding security concerns
- Dynamic credentials (Vault, OIDC)
- Mocked provider testing
- HCP Terraform remote execution

**Activities**:
- Security team presents their concerns (10 min)
- Facilitator demonstrates mocked unit tests (20 min)
- Facilitator demonstrates dynamic credentials (20 min)
- Group discussion on acceptable approaches (30 min)
- Draft policy update (10 min)

**Deliverables**:
- Security concern documentation
- Alternative approaches evaluated
- Proposed policy updates

---

### **2:30 PM - 2:45 PM: Break**

---

### **2:45 PM - 4:00 PM: Testing Strategy (75 min)**

**Format**: Presentation + demonstration using [04-testing-strategy.md](./04-testing-strategy.md)

**Topics**:
- Testing fundamentals (validation, unit, integration)
- Terraform Testing Framework overview
- Mocking capabilities
- Testing implementation roadmap

**Activities**:
- Facilitator live-codes a unit test (20 min)
- Facilitator live-codes an integration test (20 min)
- Group discussion: "What should we test first?" (20 min)
- Define phased testing roadmap (15 min)

**Deliverables**:
- Testing strategy document
- Phased implementation plan
- Initial test coverage targets

---

### **4:00 PM - 4:45 PM: Vault Integration (45 min)**

**Format**: Facilitated discussion using [05-vault-integration.md](./05-vault-integration.md)

**Topics**:
- Current Vault usage patterns
- Dynamic secrets vs static secrets
- Vault authentication methods
- Terraform + Vault best practices

**Activities**:
- Vault team presents current approach (10 min)
- Facilitator shows dynamic secret example (15 min)
- Joint discussion: Terraform + Vault collaboration (15 min)
- Define action items for improvement (5 min)

**Deliverables**:
- Vault integration improvement plan
- Joint responsibility matrix
- Shared documentation commitment

---

### **4:45 PM - 5:00 PM: Day 1 Wrap-Up (15 min)**

**Activities**:
- Recap of key findings
- Preview of Day 2 (hands-on)
- Homework (optional): Review documentation
- Q&A

---

## 📅 Day 2: Hands-On & Planning

### **Goal**: Build practical skills, create working examples, and develop actionable roadmap

---

### **8:30 AM - 9:00 AM: Day 1 Recap & Day 2 Overview (30 min)**

**Format**: Plenary

**Activities**:
- Review Day 1 findings
- Address questions from overnight reflection
- Set expectations for hands-on work
- Form working groups (if needed)

---

### **9:00 AM - 11:00 AM: Module Design Workshop (120 min)**

**Format**: Hands-on workshop using [07-technical-workshops.md](./07-technical-workshops.md#workshop-1-module-design--refactoring)

**Structure**:
- **Part 1**: Module evaluation (45 min)
  - Scoping analysis
  - MVP assessment
  - Documentation review

- **Part 2**: Redesign workshop (45 min)
  - Identify issues
  - Propose refactoring
  - Live refactoring demo

- **Part 3**: Documentation update (30 min)
  - README checklist
  - Update one module's docs

**Participants work on**:
- Their own modules (preferred)
- Example modules provided by facilitator

**Deliverables**:
- Evaluated module with findings
- Refactoring recommendations
- Updated documentation for at least one module

---

### **11:00 AM - 11:15 AM: Break**

---

### **11:15 AM - 1:15 PM: Testing Implementation Workshop (120 min)**

**Format**: Hands-on workshop using [07-technical-workshops.md](./07-technical-workshops.md#workshop-2-testing-implementation)

**Structure**:
- **Part 1**: Testing fundamentals (30 min)
  - Concept overview
  - Test file structure
  - Test syntax demo

- **Part 2**: Unit testing (45 min)
  - Variable validation tests
  - Conditional logic tests
  - Tagging tests

- **Part 3**: Integration testing (30 min)
  - Basic deployment test
  - Multi-step test with helper

- **Part 4**: CI/CD integration (15 min)
  - GitHub Actions example
  - Adapt to their CI/CD

**Participants build**:
- Unit tests for their modules
- Integration test (if time/credentials allow)
- CI/CD workflow

**Deliverables**:
- Working test suite (minimum 3 unit tests)
- Test execution demonstrated
- CI/CD workflow draft

---

### **1:15 PM - 2:15 PM: Lunch**

---

### **2:15 PM - 3:30 PM: Business Case & Change Management (75 min)**

**Format**: Workshop using [06-business-case-and-change.md](./06-business-case-and-change.md)

**Activities**:
- **Calculate current state costs** (20 min)
  - Use actual numbers from Day 1
  - Developer time, incidents, etc.
  
- **Calculate future state benefits** (20 min)
  - Time savings
  - Incident reduction
  - Productivity gains

- **Define investment required** (15 min)
  - Tooling costs
  - Implementation time
  - Training
  - Professional Services (if needed)

- **ROI calculation** (10 min)
  - Payback period
  - 3-year NPV

- **Stakeholder mapping** (10 min)
  - Identify decision makers
  - Champions and blockers
  - Engagement strategy

**Deliverables**:
- Business case with real numbers
- Executive summary draft
- Stakeholder engagement plan

---

### **3:30 PM - 3:45 PM: Break**

---

### **3:45 PM - 4:45 PM: Roadmap & Pilot Definition (60 min)**

**Format**: Collaborative planning

**Activities**:
- **Define pilot scope** (20 min)
  - Which team?
  - Which modules?
  - Timeline (4-8 weeks)
  - Success criteria

- **Phase planning** (20 min)
  - Phase 1: Pilot
  - Phase 2: Early adopters
  - Phase 3: Org-wide rollout
  - Phase 4: Optimization

- **Assign responsibilities** (15 min)
  - Champions identified
  - Module owners
  - Support structure
  - Escalation paths

- **Schedule follow-ups** (5 min)
  - Pilot kickoff date
  - Weekly check-ins
  - Mid-point review
  - Go/No-Go decision meeting

**Deliverables**:
- Pilot definition document
- Phased roadmap
- RACI matrix
- Meeting cadence established

---

### **4:45 PM - 5:30 PM: Wrap-Up & Next Steps (45 min)**

**Format**: Plenary

**Activities**:
- **Recap workshop outcomes** (10 min)
  - What we learned
  - What we built
  - What we decided

- **Review deliverables** (10 min)
  - Current state assessment
  - Technical recommendations
  - Business case
  - Pilot plan

- **Commitments** (10 min)
  - Leadership: Budget approval timeline
  - Security: Policy review timeline
  - Champions: Next actions
  - Facilitator: Follow-up support

- **Feedback** (10 min)
  - Workshop retrospective
  - What worked well
  - What to improve

- **Closing remarks** (5 min)
  - Thank participants
  - Celebrate progress
  - Excitement for pilot!

---

## 🎨 Alternative Workshop Formats

### **Half-Day Workshop (4 hours)**

For organizations with limited time:

**Hour 1**: Current state assessment + pain point prioritization  
**Hour 2**: Deep dive on #1 pain point (testing, PMR, or security)  
**Hour 3**: Hands-on workshop (module design OR testing)  
**Hour 4**: Next steps and pilot definition

**Trade-offs**: Less comprehensive, focus on highest priority

---

### **Virtual Workshop (2 half-days)**

For distributed teams:

**Day 1 (Morning)**: Discovery + discussions (3 hours)  
- Current state
- Repository/PMR
- Security/local dev

**Day 1 (Afternoon)**: Async work  
- Participants review materials
- Complete pre-work for Day 2

**Day 2 (Morning)**: Hands-on + planning (3 hours)  
- Testing workshop (live-coded together)
- Business case
- Pilot planning

**Trade-offs**: Requires more self-directed work, less interactive

---

### **Extended Workshop (3 days)**

For complex organizations or deep transformation:

**Day 1**: Discovery (same as above)  
**Day 2**: Technical deep dives (expanded workshops)  
**Day 3**: Implementation planning + pilot kickoff

**Additional topics on Day 3**:
- Advanced testing scenarios
- Policy-as-code (Sentinel/OPA)
- No-code modules
- Cost optimization strategies
- Migrating existing modules

---

## 📋 Facilitator Checklist

### **2 Weeks Before**

- [ ] Participants confirmed (names, roles, email addresses)
- [ ] Venue/Zoom room booked
- [ ] Agenda shared with participants
- [ ] Pre-workshop survey sent (current state questions)
- [ ] Materials prepared (slides, handouts, examples)
- [ ] Technical setup tested (HCP Terraform demo, test environment)

### **1 Week Before**

- [ ] Pre-work assigned (if any)
- [ ] Stakeholder 1:1s completed (key decision makers)
- [ ] Example modules selected (from their codebase if possible)
- [ ] Participant questions addressed
- [ ] Catering/logistics confirmed

### **1 Day Before**

- [ ] Test all demos (ensure they work!)
- [ ] Print materials if needed
- [ ] Charge laptop/clicker
- [ ] Backup plan for tech failures
- [ ] Final participant reminder email

### **Day Of**

- [ ] Arrive early, test AV
- [ ] Whiteboard markers, sticky notes ready
- [ ] Name tags set up
- [ ] Welcome slide displayed
- [ ] Snacks/drinks available

### **After Workshop**

- [ ] Send thank you email
- [ ] Share workshop deliverables (docs, business case, roadmap)
- [ ] Schedule follow-up meetings
- [ ] Send feedback survey
- [ ] Update materials based on feedback

---

## 🎤 Facilitator Tips

### **Setting the Tone**

- **Start with gratitude**: Thank them for their time and openness
- **Normalize challenges**: "Every organization struggles with this"
- **Focus on learning**: "We're here to discover, not criticize"
- **Encourage participation**: "Your experience matters"

### **Keeping Energy High**

- **Break every 90 minutes** minimum
- **Mix formats**: Discussion, demo, hands-on, group work
- **Use humor** (appropriately)
- **Celebrate wins**: When they complete an exercise, acknowledge it
- **Move around**: Don't stand in one spot

### **Handling Difficult Moments**

- **Skeptics**: Acknowledge their concerns, ask clarifying questions
- **Quiet participants**: Direct questions, small group work
- **Overly talkative**: "Let's hear from others," timebox discussions
- **Technical issues**: Have backup plans, don't troubleshoot live for too long
- **Scope creep**: "Let's parking lot that," stay on agenda

### **Making It Actionable**

- **Concrete examples**: Use their actual modules/code
- **Realistic timelines**: Don't promise overnight transformation
- **Clear ownership**: Every action item has a name
- **Follow-up plan**: Schedule next meetings before leaving

---

**Related Resources**:
- [Current State Assessment Questions](./01-current-state-assessment.md)
- [Technical Workshop Exercises](./07-technical-workshops.md)
- [Success Criteria](./success-criteria.md)
