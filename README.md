# Spec Coding Templates - Reference Guide & Usage Tutorial

> A structured specification system for infrastructure and platform engineering projects.

**Version**: 1.0 | **Last Updated**: 2025-04-10 | **Templates**: 8 documents, ~6,500 lines

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
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │     │                 │
│  1. GAP Analysis │────>│ 2. Requirements │────>│       3. FIP   │────>│   4. Task List  │
│                 │     │                 │     │ (Feature Impl.  │     │                 │
│ "Where are we?" │     │ "What to build?"│     │     Plan)       │     │ "How to execute?"│
│                 │     │                 │     │                 │     │                 │
│ "Where to go?"  │     │ "How to verify?"│     │ "How to design?"│     │ "In what order?" │
│                 │     │                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
        |                       |                       |                       |
        v                       v                       v                       v
  Baseline current        Define acceptance        Architecture with       Phased tasks with
  state and identify      criteria, NFRs,          Mermaid diagrams,       dependencies,
  gaps to close           infra requirements       risk assessment         effort estimates
```

### Pipeline Flow

| Stage | Document | Input | Output | Reviewers |
|-------|----------|-------|--------|-----------|
| 1 | GAP Analysis | Business problem, current infrastructure | Gap list, investment justification | Tech Lead, Product Owner |
| 2 | Requirements | GAP Analysis findings | FR/NFR/infra/integration requirements | Tech Lead, QA, Security |
| 3 | FIP | Requirements spec | Architecture design, implementation plan, risk assessment | Architects, Senior Engineers |
| 4 | Task List | FIP implementation plan | Phased tasks with dependencies, effort estimates | Engineering Team |

---

## 3. Supporting Specification Documents (辅助规范文档)

In addition to the four-document pipeline, four supporting documents provide cross-cutting rules and automation:

| Document | Role | Applied When |
|----------|------|-------------|
| **Naming Rules** (05) | Standardize resource naming across all environments | Referenced during any resource creation |
| **Failure Patterns** (06) | Document known failure modes and recovery procedures | Referenced during FIP risk assessment |
| **AUTO_TASK_CONFIG** (07) | Configure automated task execution with validation gates | Applied after Task List is finalized |
| **Infra Dependencies** (08) | Define environment promotion, component dependencies, blockers | Referenced during FIP architecture design |

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
                         │  │ 07: AUTO_TASK_CONFIG │ │──── Drives execution of Task List
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
| 04 | Task List | `templates/04_Task_List_Template.md` | ~843 | Break implementation into phased tasks with dependencies and effort estimates |
| 05 | Naming Rules | `templates/05_Naming_Rules_Template.md` | ~685 | Standardize resource naming conventions across all environments |
| 06 | Failure Patterns | `templates/06_Failure_Patterns_Template.md` | ~650 | Document known failure modes, root causes, and recovery procedures |
| 07 | AUTO_TASK_CONFIG | `templates/07_Auto_Task_Config_Template.md` | ~550 | YAML configuration for automated task execution with validation gates |
| 08 | Infra/DevOps Dependencies | `templates/08_Infra_DevOps_Dependency_Rules_Template.md` | ~654 | Environment promotion rules, component dependencies, deployment blockers |

**Total**: ~6,500 lines of structured specification templates.

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
3. THEN ────→ Create FIP (Template 03)
   │             Design architecture, plan implementation
   │
4. FINALLY ─→ Build Task List (Template 04)
   │             Break into phased, trackable tasks
   │
5. THROUGHOUT→ Apply Naming Rules (Template 05)
                Reference Failure Patterns (Template 06)
                Configure AUTO_TASK_CONFIG (Template 07)
                Enforce Infra Dependencies (Template 08)
```

### Step 3: Fill and Review

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

### Scenario B: Bug Fix or Small Enhancement (Abbreviated Pipeline)

**Context**: Fix NLB routing issue in the dev PostgreSQL cluster.

```
(No GAP Analysis needed - problem is well-defined)

Light Requirements ──→ Targeted FIP ──→ Focused Task List

Requirements: Only FR sections for the routing fix
FIP: Only affected component design
Task List: 3-5 tasks in a single phase
```

**Templates Used**: 02 (partial) + 03 (partial) + 04 (partial)

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

### Scenario D: Naming Convention Rollout (Standalone)

**Context**: Standardize naming for all Lambda functions and DynamoDB tables.

```
Naming Rules Template ──→ CI/CD Validation Script ──→ Migration Plan
      │
      └── Define patterns for all resource types
          Create validation script (Section 6)
          Plan migration for existing resources (Section 7)
```

**Templates Used**: 05 only (with Section 6 validation and Section 7 migration)

### Scenario E: Multi-Environment Deployment Automation

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
07: AUTO_TASK_CONFIG ──► Drives execution of Template 04
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
│   └── 08_Infra_DevOps_Dependency_Rules_Template.md ← Supporting: Dependencies
└── examples/
    ├── README.md                                  ← Example walkthrough
    ├── filled-gap-analysis.md                     ← Filled GAP Analysis
    ├── filled-requirements.md                     ← Filled Requirements
    ├── filled-fip.md                              ← Filled FIP
    └── filled-task-list.md                        ← Filled Task List
```

---

## 10. Best Practices (最佳实践)

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

## 11. Template Detail Reference

### Template 01: GAP Analysis (~1,073 lines)
- **Sections**: Overview, Current State Assessment, Gap Analysis Matrix, Strategic Roadmap, Risk Assessment, Success Metrics
- **Key Deliverable**: Justification for investment with quantified gaps

### Template 02: Requirements (~1,022 lines)
- **Sections**: Executive Summary, Functional Requirements (FR-X.X.X), Non-Functional Requirements, Infrastructure Requirements, Integration Requirements, Deployment Requirements, Testing Requirements, Acceptance Criteria
- **Key Deliverable**: Measurable requirements with acceptance criteria checkboxes

### Template 03: Feature Implementation Plan (~990 lines)
- **Sections**: Architecture Design (Mermaid), Detailed Design, Security Design, Performance Design, Risk Assessment, Implementation Plan, Testing Strategy, Monitoring, Rollout Plan
- **Key Deliverable**: Complete technical design ready for implementation

### Template 04: Task List (~843 lines)
- **Sections**: Phase breakdown, Task tables with dependencies, Effort estimation, Critical path, Acceptance criteria per phase
- **Key Deliverable**: Actionable task list with clear ownership and ordering

### Template 05: Naming Rules (~685 lines)
- **Sections**: General Principles, Resource Naming Rules (10 resource types), Environment Suffix Rules, Tagging Standards, Validation Script, Migration Guide
- **Key Deliverable**: Enforceable naming standards with CI/CD validation

### Template 06: Failure Patterns (~650 lines)
- **Sections**: Failure Categories, Pattern Library, Root Cause Analysis Templates, Recovery Procedures, Prevention Checklist
- **Key Deliverable**: Playbook for known failure modes

### Template 07: AUTO_TASK_CONFIG (~550 lines)
- **Sections**: Configuration Template (YAML), Field Reference, Usage Examples, Integration with Task List, Customization Guide, Troubleshooting
- **Key Deliverable**: YAML configuration driving automated task execution

### Template 08: Infra/DevOps Dependencies (~654 lines)
- **Sections**: Environment Dependency Rules, Component Dependencies, Cross-Service Rules, Deployment Blockers, Naming & Tagging Dependencies, Validation Checklists, Common Patterns
- **Key Deliverable**: Deployment safety rules preventing out-of-order or unsafe changes

---

## Footer

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | 2025-04-10 |
| **Maintainer** | Platform Engineering Team |
| **License** | Internal Use |

---

*Part of the Spec Coding Templates collection. For questions or improvements, contact the platform engineering team.*
