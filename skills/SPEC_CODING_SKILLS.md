# Spec Coding Skills for AI Coding Agents

> A comprehensive skill system for Claude Code and other AI Coding Agents to systematically apply Spec Coding templates.

**Version**: 1.0 | **Last Updated**: 2026-06-02

---

## Overview

The Spec Coding Skills system enables AI Coding Agents to:
- Guide users through structured specification development
- Apply appropriate templates based on project context
- Conduct systematic reviews and validation
- Configure and execute automated task workflows
- Perform comprehensive bug analysis

## Skill Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Spec Coding Skills System                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐     ┌─────────────────┐               │
│  │ Core Workflow   │────→│ Template Skills │               │
│  │    Skills       │     │                 │               │
│  └─────────────────┘     └─────────────────┘               │
│           │                       │                          │
│           ▼                       ▼                          │
│  ┌─────────────────┐     ┌─────────────────┐               │
│  │ Review &        │     │ Supporting      │               │
│  │ Validation      │────→│ Skills          │               │
│  └─────────────────┘     └─────────────────┘               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Skill Index

| Skill ID | Skill Name | Purpose | Template |
|----------|------------|---------|----------|
| `spec:init` | Initialize Spec Coding | Start new Spec Coding workflow | All |
| `spec:gap` | GAP Analysis | Assess current state and identify gaps | Template 01 |
| `spec:req` | Requirements | Define functional and non-functional requirements | Template 02 |
| `spec:fip` | Feature Implementation Plan | Design architecture and implementation | Template 03 |
| `spec:task` | Task List | Break down into phased tasks | Template 04 |
| `spec:review` | Cross-Context Review | Validate specifications across contexts | N/A |
| `spec:auto` | Auto Task Configuration | Configure automated execution | Template 07 |
| `spec:bug` | Bug Analysis Report | Systematic bug investigation | Template 09 |
| `spec:workflow` | Full Pipeline | Execute complete Spec Coding workflow | All |

---

## Skill 1: spec:init

**Purpose**: Initialize a new Spec Coding project and determine the appropriate workflow.

**Trigger**:
- User asks to start a new infrastructure/platform project
- User mentions "spec coding", "structured specification", or "template-driven development"

**Execution Steps**:

1. **Project Context Gathering**
   - Ask: "What is the project name or issue number?"
   - Ask: "What type of work is this?" (New service, enhancement, bugfix, migration)
   - Ask: "What is the primary goal?"
   - Ask: "Any existing documentation or issues to reference?"

2. **Workflow Selection**
   ```python
   if work_type == "bugfix":
       workflow = "bug_analysis_workflow"
   elif work_type in ["small_enhancement", "quick_fix"]:
       workflow = "abbreviated_pipeline"
   elif work_type in ["new_service", "major_feature"]:
       workflow = "full_pipeline"
   else:
       workflow = "adaptive_pipeline"
   ```

3. **Directory Setup**
   ```bash
   # Create specs directory
   mkdir -p docs/specs
   
   # Copy selected templates
   cp templates/01_GAP_Analysis_Template.md docs/specs/GAP_{project_name}.md
   # ... based on workflow
   ```

4. **Initial Configuration**
   - Create project manifest: `docs/specs/.spec-coding-config.yaml`
   - Set up document lifecycle tracking
   - Initialize review checklist

**Output**: Project structure initialized with appropriate templates

**Validation**:
- [ ] Specs directory created
- [ ] Templates copied and renamed
- [ ] Config file created
- [ ] User确认 workflow 符合预期

---

## Skill 2: spec:gap

**Purpose**: Conduct GAP Analysis to baseline current state and identify improvement areas.

**Trigger**:
- After `spec:init` for full pipeline projects
- User asks "what are we missing?", "current state assessment"
- Starting new infrastructure initiative

**Execution Steps**:

1. **Context Assessment**
   ```
   Current Infrastructure:
   - What services exist?
   - What platforms/technologies?
   - What are the pain points?
   - What constraints exist?
   ```

2. **Template Filling Guide**
   Walk through each section of Template 01:
   - **Section 1: Overview** → Define project scope and stakeholders
   - **Section 2: Current State** → Document existing infrastructure
   - **Section 3: Gap Analysis** → Identify and prioritize gaps
   - **Section 4: Strategic Roadmap** → Plan closure approach
   - **Section 5: Risk Assessment** → Identify risks and mitigations
   - **Section 6: Success Metrics** → Define measurable outcomes

3. **Gap Identification Framework**
   ```
   For each potential gap:
   - Description: What is missing?
   - Impact: Why does it matter?
   - Priority: 🔴 CRITICAL | 🟡 HIGH | 🟢 MEDIUM
   - Evidence: Data supporting the gap
   - Business Case: ROI justification
   ```

4. **Quality Checks**
   - Are gaps specific and measurable?
   - Is each gap supported by evidence?
   - Are priorities justified?
   - Is the business case clear?

**Output**: Completed GAP Analysis document with prioritized gaps

**Validation**:
- [ ] All template sections filled
- [ ] Gaps have priority ratings
- [ ] Business justification included
- [ ] Stakeholders identified

---

## Skill 3: spec:req

**Purpose**: Convert GAP Analysis findings into structured requirements.

**Trigger**:
- After GAP Analysis completion
- User asks to "define requirements", "specify what to build"
- Requirements gathering phase

**Prerequisites**:
- GAP Analysis completed and approved
- Stakeholder input available

**Execution Steps**:

1. **Gap to Requirements Mapping**
   ```
   For each GAP Analysis gap:
   → Generate corresponding Functional Requirement (FR-X.X.X)
   → Derive Non-Functional Requirements (NFR)
   → Identify Infrastructure Requirements
   → Specify Integration Requirements
   ```

2. **Template Filling Guide**
   Walk through Template 02 sections:
   
   - **Functional Requirements (FR-X.X.X)**
     ```
     Format:
     FR-X.X.X: [Requirement title]
     Description: [detailed requirement]
     Acceptance Criteria:
       - [ ] Criterion 1
       - [ ] Criterion 2
     Priority: 🔴🟡🟢
     Source: GAP-XX
     ```
   
   - **Non-Functional Requirements**
     - Performance (latency, throughput)
     - Scalability (concurrent users, data growth)
     - Availability (uptime, SLA)
     - Security (encryption, auth, compliance)
     - Maintainability (monitoring, logging)
   
   - **Infrastructure Requirements**
     - Cloud provider/AWS services
     - Networking (VPC, subnets, routing)
     - Compute (EC2, ECS, Lambda)
     - Storage (S3, EBS, RDS)
   
   - **Integration Requirements**
     - External APIs and services
     - Internal service dependencies
     - Data integration patterns
   
   - **Deployment Requirements**
     - Environments (dev, test, staging, prod)
     - Deployment strategy
     - Rollback procedures
   
   - **Testing Requirements**
     - Unit test coverage
     - Integration test scenarios
     - Performance test benchmarks
     - Security test requirements

3. **Requirements Quality Checks**
   - Are requirements unambiguous?
   - Is each requirement testable?
   - Are acceptance criteria complete?
   - Are requirements traceable to gaps?
   - Are NFRs measurable?

**Output**: Completed Requirements document with FR-X.X.X IDs

**Validation**:
- [ ] All gaps have corresponding requirements
- [ ] Each requirement has acceptance criteria
- [ ] FR-X.X.X IDs assigned
- [ ] NFRs are measurable
- [ ] Integration requirements specified
- [ ] Ready for review gate

**Next Step**: `spec:review` (Cross-Context Review)

---

## Skill 4: spec:fip

**Purpose**: Design architecture and create detailed implementation plan.

**Trigger**:
- After Requirements approval and review
- User asks for "architecture design", "implementation plan"
- Architecture phase

**Prerequisites**:
- Requirements document approved
- Cross-context review completed
- Architecture constraints identified

**Execution Steps**:

1. **Requirements to Architecture Mapping**
   ```
   For each requirement cluster:
   → Design component/service
   → Define interfaces
   → Specify data flow
   → Identify dependencies
   ```

2. **Template Filling Guide**
   Walk through Template 03 sections:
   
   - **Section 1: Architecture Design**
     ```
     Use Mermaid diagrams:
     - System Context Diagram
     - Container Diagram
     - Component Diagram
     - Sequence Diagrams for key flows
     - Deployment Diagram
     ```
   
   - **Section 2: Detailed Design**
     - Component specifications
     - API contracts
     - Data models
     - Configuration schemas
   
   - **Section 3: Security Design**
     - Authentication/authorization
     - Encryption at rest and in transit
     - Network security (VPC, security groups)
     - Secrets management
     - Compliance requirements
   
   - **Section 4: Performance Design**
     - Capacity planning
     - Caching strategy
     - Load balancing
     - Auto-scaling rules
     - Performance targets
   
   - **Section 5: Risk Assessment**
     - Reference Template 06 (Failure Patterns)
     - Identify potential failure modes
     - Design mitigations
     - Define recovery procedures
   
   - **Section 6: Implementation Plan**
     - Phase breakdown
     - Component ordering (reference Template 08)
     - Dependency sequencing
   
   - **Section 7: Testing Strategy**
     - Test pyramid (unit/integration/e2e)
     - Test data strategy
     - Performance testing approach
   
   - **Section 8: Monitoring**
     - Metrics to collect
     - Alerting thresholds
     - Dashboards
   
   - **Section 9: Rollout Plan**
     - Deployment strategy (blue/green, canary)
     - Validation steps
     - Rollback triggers

3. **Architecture Review Checklist**
   - [ ] Are all requirements addressed?
   - [ ] Are interfaces well-defined?
   - [ ] Is security designed in?
   - [ ] Are failure modes handled?
   - [ ] Is the design implementable?
   - [ ] Are dependencies manageable?

**Output**: Completed FIP document with architecture diagrams

**Validation**:
- [ ] All Mermaid diagrams render correctly
- [ ] Requirements traceability matrix complete
- [ ] Security design included
- [ ] Risk assessment with mitigations
- [ ] Testing strategy defined
- [ ] Ready for review gate

**Next Step**: `spec:review` (Cross-Context Review)

---

## Skill 5: spec:task

**Purpose**: Break down implementation into phased, trackable tasks.

**Trigger**:
- After FIP approval
- User asks for "task breakdown", "implementation tasks"
- Task planning phase

**Prerequisites**:
- FIP document approved
- Cross-context review completed
- Implementation plan clear

**Execution Steps**:

1. **FIP to Task Decomposition**
   ```
   For each FIP implementation phase:
   → Identify work items
   → Apply granularity thresholds
   → Estimate effort and complexity
   → Define dependencies
   ```

2. **Granularity Enforcement**
   ```python
   def check_task_granularity(task):
       issues = []
       if task.estimated_loc > 500:
           issues.append("LOC exceeds 500, decompose")
       if task.estimated_hours > 4:
           issues.append("Effort exceeds 4 hours, decompose")
       if task.complexity == "HIGH" and not task.justified:
           issues.append("HIGH complexity, decompose unless justified")
       return issues
   ```

3. **Template Filling Guide**
   Walk through Template 04 sections:
   
   - **Phase Breakdown**
     ```
     Phase 1: Foundation
       - Infrastructure setup
       - Core components
     
     Phase 2: Features
       - Feature implementation
       - Integration
     
     Phase 3: Validation
       - Testing
       - Documentation
     ```
   
   - **Task Definition Template**
     ```
     TASK-XXX: [Task title]
     Status: ❌ PENDING
     Priority: HIGH/MEDIUM/LOW
     Estimated Time: [hours]
     Estimated LOC: [lines]
     Complexity: LOW/MEDIUM/HIGH
     
     Description: [detailed description]
     
     Acceptance Criteria:
       - [ ] AC1: [verifiable outcome]
       - [ ] AC2: [verifiable outcome]
     
     Related Files:
       - [file1] — [change description]
       - [file2] — [change description]
     
     Commit Message:
       [type](TASK-XXX): [short description]
     
     Dependencies:
       - TASK-YYY (must complete first)
     ```

4. **Task Quality Checks**
   - Does each task have clear AC?
   - Are estimates realistic?
   - Are dependencies correct?
   - Is granularity enforced?
   - Are related files specified?

**Output**: Completed Task List with phased tasks

**Validation**:
- [ ] All tasks have AC checkboxes
- [ ] Granularity thresholds enforced
- [ ] Dependencies validated
- [ ] Estimates provided
- [ ] Commit messages defined
- [ ] Related files listed

**Next Step**: `spec:auto` (Configure automation)

---

## Skill 6: spec:review

**Purpose**: Conduct cross-context review to validate specifications.

**Trigger**:
- After Requirements completion (before FIP)
- After FIP completion (before Task List)
- User asks for "review", "validation", "stakeholder sign-off"

**Prerequisites**:
- Specification document completed
- Reviewers identified

**Execution Steps**:

1. **Review Context Setup**
   ```
   Document Type: [REQ | FIP]
   Review Contexts:
   - Technical: Architecture, feasibility, implementation
   - Security: Vulnerabilities, compliance, authorization
   - Operational: Deployability, monitoring, maintenance
   - Business: Alignment, cost, timeline
   ```

2. **Review Checklist Generation**
   
   **For Requirements (REQ)**:
   ```yaml
   Technical Review:
     - [ ] Requirements are technically feasible
     - [ ] Technology choices are justified
     - [ ] Performance requirements are realistic
     - [ ] Scalability is addressed
   
   Security Review:
     - [ ] Security requirements specified
     - [ ] Data protection included
     - [ ] Compliance requirements met
   
   Operational Review:
     - [ ] Deployability considered
     - [ ] Monitoring defined
     - [ ] Maintenance addressed
   
   Business Review:
     - [ ] Aligns with business goals
     - [ ] User needs reflected
     - [ ] Cost acceptable
     - [ ] Timeline reasonable
   ```
   
   **For FIP**:
   ```yaml
   Technical Review:
     - [ ] Architecture is sound
     - [ ] Components well-defined
     - [ ] Interfaces specified
     - [ ] Data flow clear
   
   Security Review:
     - [ ] Security designed in
     - [ ] Threats addressed
     - [ ] Encryption specified
     - [ ] Auth/authorization clear
   
   Operational Review:
     - [ ] Deployment strategy viable
     - [ ] Monitoring comprehensive
     - [ ] Rollback defined
     - [ ] Runbooks included
   
   Business Review:
     - [ ] Meets requirements
     - [ ] Risk acceptable
     - [ ] Implementation feasible
     - [ ] Dependencies manageable
   ```

3. **Review Facilitation**
   - Present review checklist
   - Collect feedback from each context
   - Identify action items
   - Track resolution

4. **Review Outcomes**
   ```
   Outcomes:
   - APPROVED: Proceed to next phase
   - APPROVED_WITH_CHANGES: Address specific items first
   - NEEDS_REVISION: Significant rework required
   - REJECTED: Fundamental issues, restart phase
   
   Action Items:
   - [ ] Item 1 — [owner] — [due date]
   - [ ] Item 2 — [owner] — [due date]
   ```

**Output**: Review report with findings and approval decision

**Validation**:
- [ ] All contexts reviewed
- [ ] Findings documented
- [ ] Action items tracked
- [ ] Decision recorded
- [ ] Sign-off obtained (if approved)

---

## Skill 7: spec:auto

**Purpose**: Configure AUTO_TASK_CONFIG for automated task execution.

**Trigger**:
- After Task List completion
- User asks to "configure automation", "setup task execution"
- Ready to implement

**Prerequisites**:
- Task List finalized
- Execution environment prepared
- Agent runtime available (Claude Code Agent Tool, etc.)

**Execution Steps**:

1. **Task List Analysis**
   ```
   Parse Task List:
   → Extract all tasks with IDs, dependencies
   → Build dependency graph
   → Identify parallelizable tasks
   → Estimate execution phases
   ```

2. **Configuration Generation**
   
   Fill Template 07 YAML:
   ```yaml
   execution:
     mode: auto | semi-auto | manual
     max_parallel_tasks: 3
     stop_on_failure: true
     auto_commit: true
   
   sub_agent:
     default_mode: foreground | background | worktree
     max_retries: 2
     retry_backoff: [30, 60]
   
   branch:
     name: "feature/{issue-number}-{short-description}"
     create_if_missing: true
     base_branch: "main"
   
   commit:
     staging_strategy: "task_scope"
     conventional_commits: true
   
   status_tracking:
     file: "docs/specs/TASK_LIST_{project}.md"
     update_frequency: "after_each_task"
   
   environments:
     order: ["dev", "test", "staging", "prod"]
     deploy_sequence: true
     require_approval: ["staging", "prod"]
   
   validation:
     post_task:
       - "pytest tests/"
       - "terraform validate"
   ```

3. **Execution Strategy**
   ```python
   # Determine parallelization opportunities
   dependency_levels = build_dependency_levels(task_list)
   
   for level in dependency_levels:
       if len(level.tasks) > 1:
           mode = "background"  # Parallel
       else:
           mode = "foreground"  # Sequential
   ```

4. **Validation Checks**
   - [ ] YAML syntax valid
   - [ ] Branch name format correct
   - [ ] File paths exist
   - [ ] Validation commands executable
   - [ ] Runtime requirements met

**Output**: AUTO_TASK_CONFIG.yaml ready for execution

**Validation**:
- [ ] Configuration file created
- [ ] YAML validates
- [ ] Task list reference correct
- [ ] Environment order defined
- [ ] Validation commands specified
- [ ] Ready to execute

**Next Step**: Execute automated task workflow (requires external runtime)

---

## Skill 8: spec:bug

**Purpose**: Conduct systematic bug analysis and develop fix strategy.

**Trigger**:
- Bug report created
- User asks for "bug analysis", "root cause", "investigation"
- Production issue identified

**Execution Steps**:

1. **Bug Context Collection**
   ```
   Gather:
   - Issue number and title
   - Error messages and stack traces
   - Reproduction steps
   - Impact assessment
   - Affected environments
   ```

2. **Root Cause Investigation**
   ```
   Investigation Framework:
   1. Fault Isolation
      - Trace data flow
      - Identify divergence point
      - Locate fault in code/system
   
   2. Root Cause Categories
      - Logic Error
      - Missing Implementation
      - Integration Failure
      - Configuration Error
      - Race Condition
      - Resource Exhaustion
      - Dependency Issue
      - Data Corruption
   
   3. Probability Ranking
      - High: Strong evidence
      - Medium: Some evidence
      - Low: Speculative
   ```

3. **Template Filling Guide**
   Walk through Template 09 sections:
   
   - **Section 1: Bug Description**
     - Current behavior (with evidence)
     - Expected behavior
     - Trigger conditions
     - Scope and severity
   
   - **Section 2: Root Cause Analysis**
     - Fault isolation
     - Root cause categories with probability
     - Detailed analysis for each cause
     - Contributing factors
   
   - **Section 3: Impact Assessment**
     - Technical impact
     - Business impact
     - Affected metrics
   
   - **Section 4: Solution Analysis**
     - Option A (recommended)
     - Option B (alternative)
     - Option C (another alternative)
     - Comparison matrix
   
   - **Section 5: Recommended Fix Strategy**
     - Phased implementation
     - Rollout strategy
   
   - **Section 6-9**: Related files, test plan, estimates, references

4. **Solution Evaluation**
   ```
   Criteria:
   - Implementation complexity
   - Time to implement
   - Risk level
   - Long-term sustainability
   - Side effects
   ```

**Output**: Completed Bug Analysis Report

**Validation**:
- [ ] Bug description with evidence
- [ ] Root cause identified
- [ ] Multiple solutions evaluated
- [ ] Fix strategy defined
- [ ] Test plan specified
- [ ] Time estimates provided

**Next Step**: Implement fix (may use abbreviated pipeline)

---

## Skill 9: spec:workflow

**Purpose**: Orchestrate complete Spec Coding workflow from start to execution.

**Trigger**:
- User asks to "run spec coding", "complete workflow"
- Starting major new initiative
- Full pipeline project

**Execution Flow**:

```
┌─────────────────────────────────────────────────────────────┐
│                  Spec Coding Full Workflow                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. spec:init                                              │
│     ├─ Gather project context                               │
│     ├─ Select workflow (full/abbreviated/bug)              │
│     └─ Setup directory structure                           │
│            │                                                │
│            ▼                                                │
│  2. spec:gap (if full pipeline)                            │
│     ├─ Assess current state                                │
│     ├─ Identify gaps                                       │
│     └─ Document findings                                   │
│            │                                                │
│            ▼                                                │
│  3. spec:req                                              │
│     ├─ Convert gaps to requirements                        │
│     ├─ Define FR/NFR/infra/integration                     │
│     └─ Create REQ document                                 │
│            │                                                │
│            ├──────────────────┐                            │
│            ▼                  ▼                            │
│  4. spec:review          │                            │
│     ├─ Cross-context      │                            │
│     ├─ Validate           │                            │
│     └─ Approve/Revise     │                            │
│            │                │                            │
│            └──────────────────┘                            │
│            │                                                │
│            ▼                                                │
│  5. spec:fip                                              │
│     ├─ Design architecture                                 │
│     ├─ Create Mermaid diagrams                             │
│     ├─ Specify security/performance                        │
│     └─ Create FIP document                                  │
│            │                                                │
│            ├──────────────────┐                            │
│            ▼                  ▼                            │
│  6. spec:review          │                            │
│     ├─ Cross-context      │                            │
│     ├─ Validate           │                            │
│     └─ Approve/Revise     │                            │
│            │                │                            │
│            └──────────────────┘                            │
│            │                                                │
│            ▼                                                │
│  7. spec:task                                             │
│     ├─ Decompose into phases                              │
│     ├─ Apply granularity thresholds                        │
│     ├─ Define tasks with AC                                │
│     └─ Create Task List                                    │
│            │                                                │
│            ▼                                                │
│  8. spec:auto                                             │
│     ├─ Configure AUTO_TASK_CONFIG                          │
│     ├─ Define execution rules                             │
│     └─ Prepare for automation                             │
│            │                                                │
│            ▼                                                │
│  9. Execute Automation                                    │
│     ├─ Run tasks per config                               │
│     ├─ Update status continuously                         │
│     └─ Validate completion                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Workflow Configuration**:

```yaml
workflow:
  type: full_pipeline | abbreviated_pipeline | bug_analysis
  
  full_pipeline:
    steps: [init, gap, req, review, fip, review, task, auto]
    review_gates: [after_req, after_fip]
  
  abbreviated_pipeline:
    steps: [init, req, fip, task, auto]
    review_gates: []
  
  bug_analysis:
    steps: [init, bug]
    review_gates: []
```

**Quality Gates**:
- Each step has validation criteria
- Review gates must pass before proceeding
- Document lifecycle tracked throughout
- Approval required at critical points

---

## Integration with Claude Code

### Usage Examples

```bash
# Start new project
/spec:init

# Run full workflow
/spec:workflow

# Execute specific phase
/spec:gap
/spec:req
/spec:fip
/spec:task

# Conduct review
/spec:review

# Configure automation
/spec:auto

# Analyze bug
/spec:bug
```

### Skill Parameters

Skills accept context parameters:

```yaml
spec:init:
  parameters:
    project_name: string
    work_type: enum[new_service, enhancement, bugfix, migration]
    issue_number: string
    templates: list[template_id]

spec:review:
  parameters:
    document_type: enum[REQ, FIP]
    contexts: list[technical, security, operational, business]
    reviewers: list[string]

spec:auto:
  parameters:
    execution_mode: enum[auto, semi-auto, manual]
    max_parallel: integer
    environments: list[string]
```

---

## Implementation Notes

### For AI Agent Developers

**State Management**:
- Track current workflow phase
- Maintain document lifecycle status
- Store review findings
- Monitor execution progress

**Template Integration**:
- Load templates from `/templates/` directory
- Parse INSTRUCTION comments
- Guide user through filling
- Validate completion

**Quality Enforcement**:
- Apply granularity thresholds for tasks
- Validate FR-X.X.X numbering
- Check Mermaid diagram syntax
- Verify YAML configuration

**Review Coordination**:
- Generate review checklists
- Collect feedback by context
- Track action items
- Facilitate approval

---

**Version**: 1.0
**Maintainer**: Spec Coding Templates Team
**License**: Proprietary — Personal non-commercial use
