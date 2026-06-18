# Spec Coding Templates - Reference Guide & Usage Tutorial

> A structured specification system for infrastructure and platform engineering projects.

**Version**: 1.3 | **Last Updated**: 2026-06-02 | **Templates**: 9 documents, ~7,750 lines | **Skills**: 3 loadable skills

---

## 1. Spec Coding Methodology Overview

### What is Spec Coding?

Spec Coding is a development practice where **structured specifications drive the coding process**. Instead of jumping directly into implementation, engineers first define the problem, requirements, design, and task breakdown in a series of interconnected documents.

**Core Principle**: Think before you code, document before you build.

### Why Spec Coding?

| Benefit | Description |
|---------|-------------|
| **Reduces Rework** | Design mistakes caught on paper cost 10x less than code mistakes |
| **Improves Alignment** | Cross-team reviews on specs prevent miscommunication |
| **Creates Audit Trail** | Every decision is documented with rationale |
| **Enables Automation** | Structured specs can drive automated task execution |
| **Accelerates Onboarding** | New engineers understand the system through specs, not code archaeology |

### When to Use Spec Coding

- Infrastructure and platform engineering projects
- Multi-team features requiring coordination
- Systems with compliance or audit requirements
- Projects where the cost of failure is high
- Any project spanning more than one sprint

---

## 2. The Four-Document Pipeline (四文档流水线)

Spec Coding follows a sequential pipeline where each document builds on the previous one:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────────┐
│                 │     │                 │     │                 │     │                 │     │                     │
│  1. GAP Analysis │────>│ 2. Requirements │────>│       3. FIP   │────>│   4. Task List  │────>│ 5. AUTO_TASK_CONFIG │
│                 │     │                 │     │ (Feature Impl.  │     │                 │     │ (Execution Spec)    │
│ "Where are we?" │     │ "What to build?"│     │     Plan)       │     │ "How to execute?"│     │ "Automation rules"  │
│                 │     │                 │     │                 │     │                 │     │                     │
│ "Where to go?"  │     │ "How to verify?"│     │ "How to design?"│     │ "In what order?" │     │ "How to automate?"  │
│                 │     │                 │     │                 │     │                 │     │                     │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────────┘
        |                       |                       |                       |                            |
        v                       v                       v                       v                            v
  Baseline current        Define acceptance        Architecture with       Phased tasks with          Config YAML for
  state and identify      criteria, NFRs,          Mermaid diagrams,       dependencies,              automated task
  gaps to close           infra requirements       risk assessment         effort estimates            execution
                                                            │
                                                            ▼
                                                 ┌─────────────────────┐
                                                 │  🔄 Review Gate     │
                                                 │  (after REQ & FIP)  │
                                                 └─────────────────────┘
                                                            │
                                                            ▼
                                                 Cross-context validation
                                                 before implementation
```

### Pipeline Flow

| Stage | Document | Input | Output | Reviewers | Review Gate |
|-------|----------|-------|--------|-----------|-------------|
| 1 | GAP Analysis | Business problem, current infrastructure | Gap list, investment justification | Tech Lead, Product Owner | - |
| 2 | Requirements | GAP Analysis findings | FR/NFR/infra/integration requirements | Tech Lead, QA, Security | **🔄 Cross-context review** |
| 3 | FIP | Requirements spec | Architecture design, implementation plan, risk assessment | Architects, Senior Engineers | **🔄 Cross-context review** |
| 4 | Task List | FIP implementation plan | Phased tasks with dependencies, effort estimates | Engineering Team | - |
| 5 | AUTO_TASK_CONFIG | Task List spec | YAML configuration for automated execution | DevOps, SRE | - |

**🔄 Cross-Context Review Gate**: After Requirements and FIP completion, conduct a structured review across different contexts (technical, security, operations, business) to validate the specification before implementation begins. This review ensures:
- Requirements align with business objectives and user needs
- Architecture decisions are sound and complete
- Security and operational concerns are addressed
- Implementation plan is feasible and properly estimated
- Cross-team dependencies are identified and managed

---

## 3. Supporting Specification Documents (辅助规范文档)

In addition to the four-document pipeline, four supporting documents provide cross-cutting rules and automation:

| Document | Role | Applied When |
|----------|------|-------------|
| **Naming Rules** (05) | Standardize resource naming across all environments | Referenced during any resource creation |
| **Failure Patterns** (06) | Document known failure modes and recovery procedures | Referenced during FIP risk assessment |
| **AUTO_TASK_CONFIG** (07) | Specification schema for automated task execution with validation gates | Applied after Task List is finalized |
| **Infra Dependencies** (08) | Define environment promotion, component dependencies, blockers | Referenced during FIP architecture design |
| **Bug Analysis Report** (09) | Structured investigation and documentation of bugs with fix strategy | Applied when bugs are discovered in any phase |

```
                         ┌──────────────────────────┐
                         │   Supporting Documents    │
                         │                           │
                         │  ┌─────────────────────┐ │
                         │  │ 05: Naming Rules     │ │──── Applied to ALL resources
                         │  └─────────────────────┘ │
                         │  ┌─────────────────────┐ │
                         │  │ 06: Failure Patterns │ │──── Referenced in FIP Section 5
                         │  └─────────────────────┘ │
                         │  ┌─────────────────────┐ │
                         │  │ 07: AUTO_TASK_CONFIG │ │──── Specifies execution of Task List
                         │  └─────────────────────┘ │
                         │  ┌─────────────────────┐ │
                         │  │ 08: Infra Dependencies│ │──── Constrains FIP Section 1
                         │  └─────────────────────┘ │
                         └──────────────────────────┘
```

---

## 4. Template Index (模板索引)

| # | Template | File | ~Lines | Purpose |
|---|----------|------|--------|---------|
| 01 | GAP Analysis | `templates/01_GAP_Analysis_Template.md` | ~1,073 | Baseline current state, identify gaps, justify investment |
| 02 | Requirements | `templates/02_Requirements_Template.md` | ~1,022 | Define functional, non-functional, infra, and integration requirements |
| 03 | Feature Implementation Plan | `templates/03_FIP_Template.md` | ~990 | Detailed technical design with architecture diagrams and implementation plan |
| 04 | Task List | `templates/04_Task_List_Template.md` | ~927 | Break implementation into phased tasks with granularity thresholds, complexity ratings, and effort estimates |
| 05 | Naming Rules | `templates/05_Naming_Rules_Template.md` | ~685 | Standardize resource naming conventions across all environments |
| 06 | Failure Patterns | `templates/06_Failure_Patterns_Template.md` | ~650 | Document known failure modes, root causes, and recovery procedures |
| 07 | AUTO_TASK_CONFIG | `templates/07_Auto_Task_Config_Template.md` | ~550 | YAML configuration for automated task execution with validation gates |
| 08 | Infra/DevOps Dependencies | `templates/08_Infra_DevOps_Dependency_Rules_Template.md` | ~654 | Environment promotion rules, component dependencies, deployment blockers |
| 09 | Bug Analysis Report | `templates/09_Bug_Analysis_Report_Template.md` | ~750 | Structured bug analysis with root cause investigation, solution comparison, and fix strategy |

**Total**: ~7,750 lines of structured specification templates.

---

## 5. Quick Start Guide (快速开始)

### Step 1: Copy Templates to Your Project

```bash
# Copy the entire templates directory into your project
cp -r spec-coding-templates/templates/ /path/to/your/project/docs/specs/

# Or copy individual templates as needed
cp spec-coding-templates/templates/01_GAP_Analysis_Template.md \
   /path/to/your/project/docs/GAP_MyProject.md
```

### Step 2: Follow the Pipeline

```
1. START ──→ Fill GAP Analysis (Template 01)
   │             Assess current state, identify gaps
   │
2. NEXT ────→ Write Requirements (Template 02)
   │             Convert gaps into measurable requirements
   │
3. REVIEW ──→ 🔄 Cross-Context Review Gate
   │             Validate Requirements across technical, security,
   │             operational, and business contexts before proceeding
   │
4. THEN ────→ Create FIP (Template 03)
   │             Design architecture, plan implementation
   │
5. REVIEW ──→ 🔄 Cross-Context Review Gate
   │             Validate FIP architecture, risk assessment,
   │             and implementation plan before execution
   │
6. FINALLY ─→ Build Task List (Template 04)
   │             Break into phased, trackable tasks
   │
7. CONFIGURE → Setup AUTO_TASK_CONFIG (Template 07)
                Specify execution rules, validation gates,
                and automation parameters for tasks
   │
8. EXECUTE ──→ Run automated task execution
                Based on AUTO_TASK_CONFIG rules (requires
                compatible runtime: Claude Code Agent Tool,
                custom orchestrator, or Sub-Agent system)

9. THROUGHOUT→ Apply Naming Rules (Template 05)
                Reference Failure Patterns (Template 06)
                Enforce Infra Dependencies (Template 08)
```

### Step 3: Fill and Review

**🔄 Cross-Context Review Process**: After completing Requirements (Template 02) and FIP (Template 03), conduct a structured review before proceeding to implementation. This review should include:

- **Technical Review**: Validate architecture decisions, technology choices, and implementation feasibility
- **Security Review**: Assess security implications, identify vulnerabilities, verify compliance requirements
- **Operational Review**: Evaluate deployability, monitoring, maintenance, and operational concerns
- **Business Review**: Confirm alignment with business objectives, user needs, and cost expectations

**Review Deliverables**:
- Review checklist with findings and action items
- Updated specification documents addressing review feedback
- Approval/sign-off from required stakeholders before proceeding

Each template follows the same pattern:
1. Replace `[PLACEHOLDER]` markers with project-specific content
2. Read and follow `<!-- INSTRUCTION: ... -->` comments, then remove them
3. Use consistent severity ratings: `🔴 CRITICAL`, `🟡 HIGH`, `🟢 MEDIUM`
4. Track status with: `✅` (done), `❌` (pending), `🔄` (in progress), `⏳` (blocked)

---

## 6. Template Usage Guide (模板使用指南)

### Placeholder Markers

All templates use `[PLACEHOLDER]` markers for project-specific content:

```markdown
<!-- Before filling -->
**Issue**: #[ISSUE_NUMBER] - [Issue Title]
**Branch**: [BRANCH_NAME]
**Author**: [AUTHOR_NAME]

<!-- After filling -->
**Issue**: #1880 - ALB + Fargate Infrastructure for Java Container Deployment
**Branch**: feature/1880-alb-fargate-java-card-service
**Author**: arthurren
```

### Instruction Comments

Templates embed guidance as HTML comments. Read them, follow the instructions, then remove:

```markdown
<!-- INSTRUCTION: Provide 3-5 bullet points describing the gap.
     Each bullet should state what is missing and why it matters.
     Remove this comment after filling. -->
```

### Severity Rating System

All templates use a consistent severity system:

| Rating | Meaning | Action Required |
|--------|---------|-----------------|
| 🔴 CRITICAL | Blocks delivery | Must resolve before proceeding |
| 🟡 HIGH | Required but not immediately blocking | Plan to address |
| 🟢 MEDIUM | Important but deferrable | Handle incrementally |

### Status Markers

| Marker | Meaning |
|--------|---------|
| ✅ | Completed / Passed |
| ❌ | Pending / Not Done |
| 🔄 | In Progress |
| ⏳ | Blocked / Waiting |
| 🚫 | Cancelled / Removed |

### File Naming Convention for Filled Templates

```
{document_type}_{project_name}.md

Examples:
GAP_ALB_Fargate_java_service.md
REQ_1880_alb_fargate_java_service.md
FIP_1928_pipeline_improvement.md
TASK_LIST_1880_alb_fargate_java_service.md
NAMING_card_service_v2.md
```

---

## 7. Typical Usage Scenarios (典型使用场景)

### Scenario A: New Infrastructure Service (Full Pipeline)

**Context**: Deploy a new Java microservice on ECS Fargate with ALB, API Gateway, and VPC Link.

```
GAP Analysis ──→ Requirements ──→ FIP ──→ Task List ──→ AUTO_TASK_CONFIG
      │                                  │
      │                                  ├── Naming Rules (for all resources)
      │                                  ├── Failure Patterns (risk assessment)
      │                                  └── Infra Dependencies (component ordering)
      │
      └── Documents current infrastructure gaps:
          "No container orchestration platform exists"
          "No standardized deployment pipeline for Java services"
```

**Templates Used**: 01 + 02 + 03 + 04 + 05 + 06 + 07 + 08 (all templates)

### Scenario B: Bug Investigation & Fix (Bug Analysis Report)

**Context**: Production bug where Epic status is stuck at "New" despite planning agent creating scratchpad.

```
Bug Report ──→ Bug Analysis Report (BAR) ──→ Targeted Fix ──→ Validation

BAR Template 09:
  - Bug description with current vs expected behavior
  - Root cause analysis (multi-layered with probability ranking)
  - Solution comparison (Options A/B/C with pros/cons)
  - Phased fix strategy with risk assessment
  - Test plan and time estimates
```

**Templates Used**: 09 (BAR) + optionally 02/03/04 for complex fixes requiring redesign

**When to use BAR**:
- Bug investigation requires systematic root cause analysis
- Multiple solution approaches need comparison
- Fix has significant risk and needs phased rollout
- Bug reveals architectural or process issues worth documenting

### Scenario C: Quick Fix (Abbreviated Pipeline)

**Context**: Fix NLB routing issue in the dev PostgreSQL cluster (well-understood problem).

```
(No GAP Analysis or BAR needed - problem is well-defined)

Light Requirements ──→ Targeted FIP ──→ Focused Task List

Requirements: Only FR sections for the routing fix
FIP: Only affected component design
Task List: 3-5 tasks in a single phase
```

**Templates Used**: 02 (partial) + 03 (partial) + 04 (partial)

### Scenario D: Security Hardening (Cross-Cutting)

### Scenario C: Security Hardening (Cross-Cutting)

**Context**: Implement mTLS between all microservices.

```
GAP Analysis ──→ Security-Focused Requirements ──→ FIP (Security Design emphasis)
      │                                                   │
      └── Assess current security posture                  └── Reference Failure Patterns
          "No encryption between services"                     (Template 06) for threat model
          "No certificate rotation policy"
```

**Templates Used**: 01 + 02 + 03 + 06

### Scenario E: Naming Convention Rollout (Standalone)

**Context**: Standardize naming for all Lambda functions and DynamoDB tables.

```
Naming Rules Template ──→ CI/CD Validation Script ──→ Migration Plan
      │
      └── Define patterns for all resource types
          Create validation script (Section 6)
          Plan migration for existing resources (Section 7)
```

**Templates Used**: 05 only (with Section 6 validation and Section 7 migration)

### Scenario F: Multi-Environment Deployment Automation

**Context**: Automate deployment of a Fargate service across dev, staging, prod.

```
Infra Dependencies ──→ AUTO_TASK_CONFIG
      │                     │
      └── Define env         └── Configure automated execution
          promotion order        with validation gates per env
          and blockers
```

**Templates Used**: 08 + 07

---

## 8. Cross-Template References (模板间引用关系)

Templates reference each other throughout the pipeline:

```
01: GAP Analysis
 └──► 02: Requirements (gaps become requirements)
      └──► 03: FIP (requirements drive architecture)
           ├──► 04: Task List (design breaks into tasks)
           ├──► 06: Failure Patterns (risk assessment)
           ├──► 08: Infra Dependencies (component ordering)
           └──► 05: Naming Rules (resource naming)

04: Task List
 └──► 07: AUTO_TASK_CONFIG (tasks drive automation)

05: Naming Rules ──► Applied across ALL templates
06: Failure Patterns ──► Referenced in FIP Section 5 (Risk Assessment)
07: AUTO_TASK_CONFIG ──► Specifies execution contract for Template 04
08: Infra Dependencies ──► Constrains FIP Section 1 (Architecture Design)
```

### Cross-Reference Checklist

When writing a FIP (Template 03), verify you have referenced:

- [ ] Requirements document for traceability (REQ- IDs)
- [ ] Naming Rules for all new resources (Template 05)
- [ ] Failure Patterns in risk assessment (Template 06)
- [ ] Infra Dependencies for component ordering (Template 08)
- [ ] Task List for implementation phases (Template 04)
- [ ] AUTO_TASK_CONFIG for automation setup (Template 07)

---

## 9. Directory Structure (目录结构)

```
spec-coding-templates/
├── README.md                                      ← You are here
├── templates/
│   ├── 01_GAP_Analysis_Template.md                ← Pipeline Stage 1
│   ├── 02_Requirements_Template.md                ← Pipeline Stage 2
│   ├── 03_FIP_Template.md                         ← Pipeline Stage 3
│   ├── 04_Task_List_Template.md                   ← Pipeline Stage 4
│   ├── 05_Naming_Rules_Template.md                ← Supporting: Naming
│   ├── 06_Failure_Patterns_Template.md            ← Supporting: Failures
│   ├── 07_Auto_Task_Config_Template.md            ← Supporting: Automation
│   ├── 08_Infra_DevOps_Dependency_Rules_Template.md ← Supporting: Dependencies
│   └── 09_Bug_Analysis_Report_Template.md         ← Supporting: Bug Investigation
├── skills/                                         ← AI Coding Agent Skills
│   ├── README.md                                  ← Skills documentation
│   ├── SPEC_CODING_SKILLS.md                      ← Complete skill reference
│   ├── spec-workflow.md                           ← Full workflow skill
│   ├── spec-review.md                             ← Cross-context review skill
│   └── spec-bug.md                                ← Bug analysis skill
├── cn/                                            ← Chinese translation
│   ├── README.md
│   ├── templates/                                 ← Chinese templates
│   └── examples/                                  ← Chinese examples
└── examples/
    ├── README.md                                  ← Example walkthrough
    ├── filled-gap-analysis.md                     ← Filled GAP Analysis
    ├── filled-requirements.md                     ← Filled Requirements
    ├── filled-fip.md                              ← Filled FIP
    └── filled-task-list.md                        ← Filled Task List
```

---

## 10. AI Coding Agent Skills

### What Are Skills?

Skills are loadable instruction files that enable AI Coding Agents (like Claude Code) to systematically apply Spec Coding templates. Think of them as "executable workflows" that guide both the AI and the user through structured specification development.

### Available Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| **spec:workflow** | Execute complete Spec Coding workflow | Starting a new project, running full pipeline |
| **spec:review** | Conduct cross-context review | After Requirements or FIP completion |
| **spec:bug** | Systematic bug analysis | When bugs are reported, need root cause investigation |

### Installation

```bash
# Copy skills to Claude Code skills directory
cp -r skills/* ~/.claude/skills/

# Or link for easy updates
ln -s $(pwd)/skills/spec-*.md ~/.claude/skills/
```

### Usage Examples

```bash
# Start new project workflow
/spec:workflow project_name="MyService" work_type="new_service" issue_number="123"

# Conduct review
/spec:review document_type="REQ" document_path="docs/specs/REQ_MyService.md"

# Analyze bug
/spec:bug issue_number="456" issue_title="Auth_Failure" error_evidence="401 Unauthorized"
```

### Skill Parameters

**spec:workflow**:
- `project_name` (required): Project identifier
- `work_type` (optional): new_service, enhancement, bugfix, migration
- `issue_number` (optional): GitHub/Jira issue number

**spec:review**:
- `document_type` (required): REQ or FIP
- `document_path` (required): Path to specification document

**spec:bug**:
- `issue_number` (required): Bug issue number
- `issue_title` (required): Bug title
- `error_evidence` (optional): Error logs or evidence

### Skill Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Spec Coding Skills System                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  User Request ──→ Skill Loading ──→ Parameter Validation   │
│                                      ↓                      │
│                             Template Integration            │
│                                      ↓                      │
│                             Guided Execution                │
│                                      ↓                      │
│                             Quality Validation              │
│                                      ↓                      │
│                             Document Generation            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Skill Development

See `skills/README.md` for:
- Complete skill reference
- Extension development guide
- Quality assurance checklist
- Troubleshooting tips

---

## 11. Framework Comparison: oh_my_harness vs Mainstream AI Coding Frameworks

### Overview Comparison Table

| Dimension | oh_my_harness | GitHub Spec Kit | Superpowers | GSD | GStack |
|-----------|---------------|------------------|-------------|-----|--------|
| **GitHub Stars** | Open source | GitHub Official | 94,000 | 35,000 | 85,000+ |
| **Creator** | Arthur Ren (任志超) | GitHub | Community | Community | Gary Tan (YC CEO) |
| **Core Focus** | Spec Coding methodology templates | Spec-Driven development toolkit | TDD enforcement framework | Context rot prevention | Role-based AI team governance |
| **Language** | **Chinese + English** | English | English | English | English |
| **Primary Use** | Enterprise training, methodology implementation | GitHub integration, tool-agnostic | Test-driven development | Long session quality | Solo founders, AI team simulation |

### Core Features Comparison

#### oh_my_harness Unique Characteristics

**Methodology-Oriented**:
- **Original Spec Coding methodology** with four engineering principles: Spec First / Living Documents / Traceability / Validation
- **8 core templates** (~7,750 lines of structured specifications)
- **Full-lifecycle coverage**: Requirements → Design → Implementation → Testing → Deployment

**Enterprise-Grade Features**:
- **Chinese localization** for seamless adoption in Chinese teams
- **Financial industry validation** through real-world implementations (MUFG Bank, Guosen Securities)
- **Training-oriented design** specifically for AI coding education programs
- **Git-based version management** with specs evolving alongside code

**Template System**:
```
Requirements: requirement-spec + functional-impact
Design: api-spec + component-spec + data-model
Implementation: implementation-plan + tech-design
Testing: testing-strategy + regression-checklist
Deployment: deployment-plan + rollback-plan
```

#### Other Frameworks Characteristics

**GitHub Spec Kit**:
- Agent-agnostic (supports multiple AI tools)
- Automatic feature numbering system
- YAML validation workflows
- **Advantage**: GitHub official support, mature tool ecosystem

**Superpowers**:
- **TDD enforcement** as core discipline
- 6 major AI coding domains coverage
- Planning + Debugging + Verification capabilities
- Markdown-based documentation
- **Advantage**: Strong engineering discipline, test-driven approach

**GSD**:
- **Solves context rot problem** in long AI coding sessions
- 69 commands + 24 specialized agents
- Phase → Plan → Execute lifecycle
- Meta-prompting framework
- **Advantage**: Quality assurance for extended coding sessions

**GStack**:
- **Role-based governance** (CEO Reviews, Security Audits, Browser QA)
- 6 specialized skills
- 10K+ lines of code per week capacity
- AI team simulation for individuals
- **Advantage**: Complete AI team simulation for solo developers

### Differentiation Matrix

| Differentiation Dimension | oh_my_harness Unique Advantage |
|---------------------------|----------------------------------|
| **Methodology Depth** | **Original Spec Coding methodology**, not just a tool collection |
| **Language Localization** | **Native Chinese support** removes language barriers |
| **Industry Adaptation** | **Financial industry validation** (MUFG, Guosen Securities) |
| **Training Orientation** | Designed as **enterprise training curriculum**, not purely tool-focused |
| **Production Validation** | **mal.ai 10x productivity improvement** real-world results |
| **Enterprise Compliance** | Built-in **financial-grade compliance** (7-year audit logs, data security) |
| **Legacy System Support** | Provides **archaeology methodology** and reverse engineering support |

### Recommended Usage Scenarios

**Choose oh_my_harness when**:
- Financial, securities, or high-compliance industries
- Teams requiring Chinese documentation
- Enterprise AI coding training programs
- Legacy system standardization needs
- Need complete methodology over pure tools
- Spec-driven development with audit trails

**Choose other frameworks when**:
- **GitHub Spec Kit**: Already using GitHub toolchain, need official integration
- **Superpowers**: TDD practitioners, emphasizing test-driven development
- **GSD**: Long AI coding sessions, concerned about context rot
- **GStack**: Individual founders needing complete AI team simulation

### Core Value Proposition

**oh_my_harness fundamental value**:
1. **Methodological originality**: Created Spec Coding methodology from scratch, not a wrapper around tools
2. **Enterprise production validation**: Large-scale validation across financial (MUFG) and automotive (NIO) industries
3. **Chinese localization**: No language or cultural barriers for higher team adoption
4. **Compliance by design**: Financial-grade compliance requirements built-in from inception
5. **Training ecosystem**: Complete training curriculum with real-world case studies

**Essential difference from other frameworks**:
- **Other frameworks**: Tool-first, solving specific technical problems
- **oh_my_harness**: Methodology-first, solving engineering system challenges

---

*Sources:*
- *[GitHub Spec Kit - Official Repository](https://github.com/github/spec-kit)*
- *[Superpowers Skills Framework - Termdock](https://www.termdock.com/en/blog/superpowers-framework-agent-skills)*
- *[Superpowers, GSD, GStack Comparison - Medium](https://medium.com/@tentenco/superpowers-gsd-and-gstack-what-each-claude-code-framework-actually-constrains-12a1560960ad)*
- *[GSD Context Rot Prevention - The New Stack](https://thenewstack.io/beating-the-rot-and-getting-stuff-done/)*
- *[GStack by Gary Tan - GitHub](https://github.com/garrytan/gstack)*

---

## 12. Best Practices (最佳实践)

### Specification Writing

1. **Start with GAP Analysis** even for "obvious" projects -- the exercise often reveals hidden gaps
2. **Review templates with 2+ team members** before implementation begins
3. **Keep instruction comments** until the template is fully filled, then remove them
4. **Use consistent severity ratings** across all documents in a project
5. **Link every specification to a GitHub/Jira issue** for traceability

### Document Lifecycle

| Phase | Status | Meaning |
|-------|--------|---------|
| Draft | `Draft` | Initial creation, actively being written |
| Review | `Under Review` | Peer review phase |
| Approved | `Approved` | Design sign-off received |
| Building | `Implementation In Progress` | Engineering team executing |
| Done | `Complete` | Delivered and validated |

### Version Control

- Commit filled specifications alongside code changes
- Update specifications when implementation deviates from plan
- Use the `Review History` table at the end of each FIP to track changes
- Increment document version with each significant update

### Anti-Patterns to Avoid

| Anti-Pattern | Why It Fails | What to Do Instead |
|-------------|-------------|-------------------|
| Skipping GAP Analysis | Misses hidden requirements | Always baseline current state |
| Copy-pasting templates without reading instructions | Generic, unfilled specs | Read every INSTRUCTION comment |
| Writing specs after implementation | Documentation theater | Write specs before coding |
| Never updating specs after changes | Specs become misleading | Update when implementation deviates |
| One-person spec writing | Blind spots, bias | Minimum 2 reviewers |
| Over-engineering small tasks | Wastes time on ceremony | Use abbreviated pipeline (Scenario B) |

---

## 13. Template Detail Reference

### Template 01: GAP Analysis (~1,073 lines)
- **Sections**: Overview, Current State Assessment, Gap Analysis Matrix, Strategic Roadmap, Risk Assessment, Success Metrics
- **Key Deliverable**: Justification for investment with quantified gaps

### Template 02: Requirements (~1,022 lines)
- **Sections**: Executive Summary, Functional Requirements (FR-X.X.X), Non-Functional Requirements, Infrastructure Requirements, Integration Requirements, Deployment Requirements, Testing Requirements, Acceptance Criteria
- **Key Deliverable**: Measurable requirements with acceptance criteria checkboxes

### Template 03: Feature Implementation Plan (~990 lines)
- **Sections**: Architecture Design (Mermaid), Detailed Design, Security Design, Performance Design, Risk Assessment, Implementation Plan, Testing Strategy, Monitoring, Rollout Plan
- **Key Deliverable**: Complete technical design ready for implementation

### Template 04: Task List (~927 lines)
- **Sections**: Task Granularity Standards (LOC/effort/complexity thresholds), Phase breakdown, Task tables with dependencies, Effort and complexity estimation, Decomposition rules, Critical path, Acceptance criteria per phase
- **Key Deliverable**: Actionable task list with granularity enforcement — any task exceeding 500 LOC, 4 person-hours, or HIGH complexity must be decomposed into subtasks

### Template 05: Naming Rules (~685 lines)
- **Sections**: General Principles, Resource Naming Rules (10 resource types), Environment Suffix Rules, Tagging Standards, Validation Script, Migration Guide
- **Key Deliverable**: Enforceable naming standards with CI/CD validation

### Template 06: Failure Patterns (~650 lines)
- **Sections**: Failure Categories, Pattern Library, Root Cause Analysis Templates, Recovery Procedures, Prevention Checklist
- **Key Deliverable**: Playbook for known failure modes

### Template 07: AUTO_TASK_CONFIG (~550 lines)
- **Sections**: Configuration Template (YAML), Field Reference, Usage Examples, Integration with Task List, Customization Guide, Troubleshooting
- **Key Deliverable**: YAML configuration schema specifying automated task execution contracts (requires external runtime to execute)

### Template 08: Infra/DevOps Dependencies (~654 lines)
- **Sections**: Environment Dependency Rules, Component Dependencies, Cross-Service Rules, Deployment Blockers, Naming & Tagging Dependencies, Validation Checklists, Common Patterns
- **Key Deliverable**: Deployment safety rules preventing out-of-order or unsafe changes

### Template 09: Bug Analysis Report (~750 lines)
- **Sections**: Bug Description, Root Cause Analysis (multi-layered with probability ranking), Impact Assessment, Solution Analysis (Options A/B/C), Recommended Fix Strategy, Related Files, Test Plan, Time Estimates, Related Issues & References
- **Key Deliverable**: Comprehensive bug investigation document with systematic root cause analysis, solution comparison, and phased fix strategy

---

## Footer

| Field | Value |
|-------|-------|
| **Version** | 1.3 |
| **Last Updated** | 2026-06-18 |
| **Maintainer** | Platform Engineering Team |
| **License** | Internal Use |

---

*Part of the Spec Coding Templates collection. For questions or improvements, contact the platform engineering team.*
