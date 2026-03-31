# Work Skill Analysis Prompt

## Task

You will receive source materials (documents, messages, emails, PR reviews, etc.) for **{name}**.
Extract their work capabilities and methods to build a Work Skill.

**Principle: Extract only work-related content, ignore small talk. Don't speculate — if there's evidence, write it; if not, mark "insufficient material".**

---

## Universal Extraction Dimensions (all roles)

### 1. Scope of Responsibility

Identify from materials:
- Systems / modules / products / business areas they own
- Documentation they maintain (API docs, wikis, runbooks, design docs...)
- Responsibility boundaries (what's theirs, what's not)
- Frequently mentioned project names, internal tools, domain terms

```
Output format:
Domain: [description]
Core systems: [list]
Maintained docs: [list]
Boundaries: [what they own / what they don't]
```

### 2. Workflow

Extract from task descriptions, meeting notes, PR patterns:
- How they handle incoming tasks (steps)
- How they write proposals / design docs (structure preferences)
- How they manage timelines and deadlines
- How they handle incidents / urgent issues
- How they run or participate in meetings

```
Output format:
Receiving tasks: [steps]
Writing proposals: [structure description]
Incident response: [process]
Meeting behavior: [description]
```

### 3. Output Format Preferences

- Tables / bullet points / diagrams / prose
- Conclusion-first vs narrative buildup
- Documentation detail level (minimal / moderate / comprehensive)
- Communication format (email style, Slack message style, PR description style)

```
Output format:
Documentation style: [description]
Detail level: [minimal / moderate / comprehensive]
Communication format: [description]
```

### 4. Experience & Knowledge Base

Explicit experience, lessons learned, technical opinions they've expressed (quote directly):

```
- "[quote or summary]"
- "[quote or summary]"
```

---

## Role-Specific Extraction

Based on {name}'s role, focus on the relevant dimensions:

---

### 🖥️ Backend / Server Engineer

**Technical Standards**:
- Tech stack (languages, frameworks, middleware, infra)
- Naming conventions (API paths, variable/function naming)
- API design (response structure, error codes, pagination, idempotency)
- Database preferences (ORM vs raw SQL, transaction boundaries, migration approach)
- Error handling style
- Testing philosophy (unit test coverage, integration tests, TDD)

**Code Review Focus**:
- Issues they repeatedly flag (N+1 queries, transactions, concurrency, error handling...)
- CR comment style (direct/diplomatic, blocking/suggestion/nit classification)

**Deployment & Ops**:
- Monitoring and observability priorities
- Incident debugging steps
- Release/deploy process and rollback strategy

---

### 🌐 Frontend Engineer

**Technical Standards**:
- Tech stack (framework, state management, styling approach, build tools)
- Component decomposition principles (when to split, when not to)
- Performance priorities (FCP, lazy loading, bundle size, Core Web Vitals...)
- API integration and error handling patterns
- Accessibility standards

**Engineering Practices**:
- Linting and formatting preferences (ESLint rules, Prettier config)
- Testing expectations (unit / integration / E2E / visual regression)
- CR focus (accessibility, responsiveness, cross-browser, performance)

---

### 🤖 ML / AI Engineer

**Research & Experimentation**:
- Problem framing approach (how they break down ML problems)
- Experiment design habits (baseline selection, ablation studies)
- Metrics preferences (offline vs online metrics, guardrail metrics)
- Preferred models / methods / frameworks

**Engineering & Production**:
- Training framework preferences
- Model serving / deployment pipeline
- Data processing standards
- Feature engineering approach

**Documentation & Communication**:
- Experiment report style (conclusion-heavy vs process-heavy)
- Papers / methods they frequently reference

---

### 📱 Product Manager / TPM

**Requirements Handling**:
- PRD structure and detail level
- User story / acceptance criteria definition style
- How they align with engineering (review process, spec iteration)

**Decision Framework**:
- Prioritization method (RICE / MoSCoW / ICE / custom)
- Data-driven vs intuition ratio
- How they handle conflicting requirements
- Stakeholder management approach

**Deliverables**:
- Document types they produce (PRD, RFC, one-pager, competitive analysis)
- Prototyping tool preferences
- Analytics / instrumentation involvement

---

### 🎨 Designer (Product / UX / Visual)

**Design Standards**:
- Design system / component library used
- Annotation and handoff specifications
- Pixel-perfect expectations
- Accessibility and responsive design approach

**Workflow**:
- From requirements to design explorations to final deliverable
- Design review / critique participation
- How they handle implementation fidelity issues
- Feedback and iteration style

---

### 📊 Data Analyst / Data Scientist

**Analysis Methods**:
- Preferred frameworks (funnel analysis, cohort analysis, A/B testing, causal inference...)
- SQL style (concise / heavily commented / CTE-heavy)
- Visualization preferences (chart types, tools)

**Reporting Style**:
- Conclusions vs raw data ratio
- How rigorously they enforce "data-driven" standards
- How they handle data anomalies / definition disputes

---

### 🔧 DevOps / SRE / Platform Engineer

**Infrastructure Standards**:
- IaC tools and patterns (Terraform, Pulumi, CDK)
- CI/CD pipeline design preferences
- Monitoring and alerting philosophy
- Incident management approach (runbooks, on-call, postmortems)

**Operational Practices**:
- Change management process
- Capacity planning approach
- Security and compliance awareness
- Documentation and runbook standards

---

## Output Requirements

- Language: match user's language
- Dimensions with no information: mark `(Insufficient material — recommend adding relevant documents)`
- Conclusions with source evidence: quote original text in quotation marks
- Output will be used directly to generate work.md — must be specific and actionable, avoid "might", "tends to", "possibly"
