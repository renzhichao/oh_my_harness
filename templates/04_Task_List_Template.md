# [PROJECT_TITLE] Task List

<!-- ==============================================================================
     INSTRUCTION: SPEC CODING TEMPLATE - TASK LIST
     ==============================================================================
     Structured task list for infrastructure and platform engineering projects.
     Tracks work across phases with granular task detail, acceptance criteria,
     dependencies, and risk mitigation.

     HOW TO USE:
     1. Replace all [PLACEHOLDER] markers with project-specific content.
     2. Follow inline (<!-- INSTRUCTION: ... -->) comments for guidance.
     3. Keep status/priority emoji system consistent:
        Status:  COMPLETED | IN_PROGRESS | PENDING | BLOCKED | CANCELLED
        Priority: HIGH | MEDIUM | LOW
     4. Every task MUST have: Status, Priority, Estimated Time, Description,
        Acceptance Criteria, Related Files, Commit Message, Dependencies.
     5. Number tasks sequentially within each phase (e.g., 1.1, 1.2, 2.1).
     ============================================================================ -->

## Project Information
<!-- INSTRUCTION: Fill in all fields before work begins. This section provides
     essential context for anyone reading the task list. -->

- **Project Name**: [PROJECT_TITLE]
<!-- INSTRUCTION: Full project name as it appears in the issue tracker.
     Example: MalBank Mobile App - iOS E2E Automation Framework (Week 1) -->
- **Branch**: [BRANCH_NAME]
<!-- INSTRUCTION: Feature branch convention: type/issue-number-short-description
     Example: feature/1110-week1-ios-ui-e2e-automation
     Example: bugfix/2132-dev-postgresql-nlb-routing-fix -->
- **Issue**: #[ISSUE_NUMBER] - [Issue Title]
<!-- INSTRUCTION: GitHub/Jira issue number and title.
     Example: #1110 - E2E Automation Framework for iOS Mobile App -->
- **Objective**: [One-line objective]
<!-- INSTRUCTION: Single sentence describing the end goal. The "north star"
     that every task contributes toward.
     Example: Implement Detox-based E2E testing framework with 3 critical user journeys -->
- **Priority**: High priority tasks executed first
<!-- INSTRUCTION: State the execution priority rule. -->
- **Reference Documents**:
<!-- INSTRUCTION: List supporting documents with relative paths. Remove types
     that do not exist for this project. -->
  - DIP: `path/to/DIP.md`
<!-- INSTRUCTION: Path to Design Implementation Plan, if one exists. -->
  - Requirements: `path/to/req.md`
<!-- INSTRUCTION: Path to requirements document or FIP. -->
  - Gap Analysis: `path/to/GAP.md`
<!-- INSTRUCTION: Path to GAP analysis document, if created. -->
  - Review Report: `path/to/REVIEW.md`
<!-- INSTRUCTION: Path to any review report that informed the task list. -->

---

## Execution Rules
<!-- INSTRUCTION: Define 5-7 execution rules governing task processing.
     Each rule should be specific and actionable. -->

1. Execute tasks by priority (HIGH > MEDIUM > LOW) and dependency order
<!-- INSTRUCTION: Priority takes precedence, but dependencies may force a
     specific sequence within the same priority level. -->
2. All phases must complete in order: Phase 0 -> Phase 1 -> Phase 2 -> Phase 3 -> Phase 4
<!-- INSTRUCTION: Adjust phase list to match your project structure. Not all
     projects need 5 phases. Enforce no overlap unless explicitly noted. -->
3. Each task requires validation against acceptance criteria before marking as COMPLETED
<!-- INSTRUCTION: Define validation method: automated test pass, peer review,
     lead approval, or CI/CD pipeline green. -->
4. Commit code after each task completion using conventional commit format
<!-- INSTRUCTION: See the Commit Message field in each task for format. -->
5. Mark task as BLOCKED immediately if encountering unresolvable issues and escalate within 4 hours
<!-- INSTRUCTION: Include escalation timeframe. Typical paths: team lead,
     engineering manager, Slack channel. -->
6. All tasks must include passing tests (unit, integration, or E2E as appropriate) before COMPLETED
<!-- INSTRUCTION: Remove this rule if automated testing is not required. -->
7. Deploy to environments in order: dev -> test -> staging -> prod
<!-- INSTRUCTION: Remove for single-environment projects. NEVER skip environments. -->

---

## Task Status Legend
<!-- INSTRUCTION: Use these status markers consistently. Update task status
     in real-time as work progresses. -->

- COMPLETED: Completed and validated
<!-- INSTRUCTION: Mark COMPLETED only when ALL acceptance criteria are met AND
     the commit has been pushed. -->
- IN_PROGRESS: Currently executing
<!-- INSTRUCTION: Mark IN_PROGRESS when beginning active work. Only ONE task
     should be IN_PROGRESS at a time per person. -->
- PENDING: Waiting to execute
<!-- INSTRUCTION: Default status for all new tasks. -->
- BLOCKED: Blocked, requires manual intervention
<!-- INSTRUCTION: Include a "Blocker" description explaining what is needed. -->
- CANCELLED: Cancelled or deprioritized
<!-- INSTRUCTION: Do NOT delete cancelled tasks; keep for historical record. -->

## Priority Legend
<!-- INSTRUCTION: Priority determines execution order within a phase. All HIGH
     tasks should be attempted before any MEDIUM tasks. -->

- HIGH: High priority, affects core functionality
<!-- INSTRUCTION: Critical path tasks, blockers, or core deliverables.
     Example: Infrastructure setup, security config, core API. -->
- MEDIUM: Medium priority, important but not urgent
<!-- INSTRUCTION: Completeness tasks not on critical path.
     Example: Supporting utilities, documentation. -->
- LOW: Low priority, optimization and enhancement features
<!-- INSTRUCTION: Nice-to-have, deferrable without impacting delivery.
     Example: Performance optimization, cosmetic improvements. -->

---

<!-- INSTRUCTION: PHASE STRUCTURE GUIDANCE
     Phase 0: Pre-Development Setup (environment, tooling, documentation)
     Phase 1: Core Implementation (primary feature or infrastructure build)
     Phase 2: Multi-Environment Deployment (dev/test/staging/prod rollout)
     Phase 3: Integration (cross-component integration and E2E testing)
     Phase 4: Monitoring and Documentation (observability, runbooks, handover)
     Add or remove phases based on project needs. Each phase should have:
     1. Descriptive phase title  2. Sub-sections by week/day  3. Standard task format  4. End-of-phase checkpoint -->

---

## Phase 0: Pre-Development Setup

<!-- INSTRUCTION: Preparatory work before any feature or infrastructure code.
     Typical tasks: environment provisioning, tool installation, dependency
     setup, documentation review, baseline configuration. All Phase 0 tasks
     should be COMPLETED before Phase 1 begins. -->

### Task 0.1: [Environment Setup Task Title]
**Status**: PENDING
<!-- INSTRUCTION: Use the emoji matching current state. For new tasks use PENDING.
     Update this field whenever task status changes. -->
**Priority**: HIGH
<!-- INSTRUCTION: Setup tasks are typically HIGH because they block all subsequent
     work. Use the red circle emoji for HIGH. -->
**Estimated Time**: [X hours]
<!-- INSTRUCTION: Realistic estimate with buffer. Round up to nearest half-hour.
     Example: "1.5 hours", "2 hours", "1 day". -->
**Description**: [What this task does and why it is needed]
<!-- INSTRUCTION: Write 2-3 sentences: what it accomplishes, why it is needed
     (context), and any important constraints. Be specific enough for someone
     unfamiliar with the project. -->

**Acceptance Criteria**:
- [ ] [Criterion 1: a specific, testable condition]
- [ ] [Criterion 2: a specific, testable condition]
- [ ] [Criterion 3: a specific, testable condition]
<!-- INSTRUCTION: Acceptance criteria define "done". Each must be:
     - Testable: verifiable with a command, test, or manual check
     - Specific: no ambiguity about what "done" looks like
     - Complete: together they fully define task completion
     Good: "Terraform plan shows 0 changes after apply"
     Bad:  "Terraform is configured"
     Use checkboxes (- [ ]). Mark as [x] when criterion is met. -->

**Related Files**:
- `[path/to/file.ext]` (CREATE | UPDATE | REFERENCE)
<!-- INSTRUCTION: List every file created, modified, or referenced.
     Tags: CREATE (new), UPDATE (existing), REFERENCE (inform only).
     Example:
     - `infrastructure/terraform/main.tf` (CREATE)
     - `infrastructure/terraform/variables.tf` (UPDATE) -->

**Commit Message**: `chore: [descriptive message in conventional commit format]`
<!-- INSTRUCTION: Conventional Commits: type(scope): description
     Types: feat (feature), fix (bug), chore (tooling), docs (documentation),
            test (tests), refactor (restructure), ci (CI/CD), deploy (release)
     Examples: feat(alb): add ALB for java service
               fix(nlb): correct dev cluster routing
               chore(terraform): add backend config -->

**Dependencies**: None
<!-- INSTRUCTION: List task numbers that must complete first. Use "None" if none.
     Hard dep: "Task X.Y" (cannot start until complete)
     Soft dep: "Task X.Y (soft)" (can start but may need rework)
     Example: "Task 0.1 (WireMock must be running)" -->

---

### Task 0.2: [Tooling Configuration Task Title]
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: [Detailed description of tooling or configuration setup]

**Acceptance Criteria**:
- [ ] [Specific verifiable criterion]
- [ ] [Specific verifiable criterion]
- [ ] [Specific verifiable criterion]

**Related Files**:
- `[path/to/config/file]` (CREATE | UPDATE)

**Commit Message**: `chore: [conventional commit message]`

**Dependencies**: Task 0.1
<!-- INSTRUCTION: Task 0.1 must be COMPLETED before this task begins. Always
     verify the referenced task is actually complete before starting. -->

---

### Task 0.3: [Documentation or Guidelines Task Title]
**Status**: PENDING
**Priority**: MEDIUM
**Estimated Time**: [X hours]
**Description**: [Detailed description of documentation or guidelines to create]

**Acceptance Criteria**:
- [ ] [Document created at specified path]
- [ ] [Covers required topics]
- [ ] [Reviewed by team lead]

**Related Files**:
- `[path/to/document.md]` (CREATE)

**Commit Message**: `docs: [conventional commit message]`

**Dependencies**: None
<!-- INSTRUCTION: Documentation tasks often have no code dependencies and can
     run in parallel with other setup tasks. Use "None" to indicate parallelizable. -->

---

<!-- INSTRUCTION: END-OF-PHASE CHECKPOINT
     Include after each phase to validate deliverables before proceeding.
     Contains: validation criteria, risk review questions, decision points. -->

### Phase 0 Checkpoint

**Validation Criteria**:
- [ ] All Phase 0 tasks COMPLETED
- [ ] Development environment operational
- [ ] Reference documents reviewed and understood

**Risk Review Questions**:
1. Are all environments accessible? (YES/NO)
2. Are all dependencies installed? (YES/NO)
3. Can we proceed to Phase 1? (YES/NO)

**Decision Point**: If any answer is NO, resolve blockers before proceeding.

---

## Phase 1: Infrastructure Foundation

<!-- INSTRUCTION: Core implementation phase. Typically the largest with the
     most tasks. Organize into weekly (or daily) sub-groups. -->

### Week 1: [Module Group Name]
<!-- INSTRUCTION: Group tasks by logical module. Name should describe what
     tasks accomplish together.
     Examples: "Core Infrastructure Provisioning", "Networking and Security",
     "Database and Storage Setup", "ALB Configuration" -->

#### Task 1.1: [Core Infrastructure Task Title]
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: [Detailed description of the core infrastructure task]

**Acceptance Criteria**:
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]
- [ ] [Criterion 4]
- [ ] [Criterion 5]
<!-- INSTRUCTION: Infrastructure tasks typically have 3-7 criteria covering:
     resource creation, configuration, connectivity, security, verification. -->

**Related Files**:
- `[path/to/terraform/main.tf]` (CREATE)
- `[path/to/terraform/variables.tf]` (CREATE)
- `[path/to/terraform/environments/dev/terraform.tfvars]` (CREATE)

**Commit Message**: `feat: [conventional commit message]`

**Dependencies**: Phase 0 complete

**Implementation Notes**:
<!-- INSTRUCTION: Optional section for complex tasks. Include commands,
     config values, or code examples in fenced code blocks. -->
```bash
# Example implementation command
terraform init && terraform plan && terraform apply
```

---

#### Task 1.2: [Configuration and Integration Task Title]
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: [Detailed description of the configuration task]

**Acceptance Criteria**:
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

**Related Files**:
- `[path/to/config/file]` (UPDATE)

**Commit Message**: `feat: [conventional commit message]`

**Dependencies**: Task 1.1
<!-- INSTRUCTION: Core infrastructure must be provisioned before this
     configuration can be applied. Always respect dependency ordering. -->

---

### Week 2: [Module Group Name]
<!-- INSTRUCTION: Week 2 builds on Week 1. May include additional
     infrastructure, integration components, or deployment preparation. -->

#### Task 1.3: [Integration Task Title]
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: [Detailed description of the integration work]

**Acceptance Criteria**:
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]
- [ ] [Criterion 4]

**Related Files**:
- `[path/to/integration/file]` (CREATE)
- `[path/to/existing/file]` (UPDATE)

**Commit Message**: `feat: [conventional commit message]`

**Dependencies**: Task 1.2

---

#### Task 1.4: [Validation and Testing Task Title]
**Status**: PENDING
**Priority**: MEDIUM
**Estimated Time**: [X hours]
**Description**: [Detailed description of validation and testing work]

**Acceptance Criteria**:
- [ ] [Test passes with expected output]
- [ ] [Coverage meets threshold]
- [ ] [No regressions in existing tests]

**Related Files**:
- `[path/to/test/file]` (CREATE)

**Commit Message**: `test: [conventional commit message]`

**Dependencies**: Task 1.3

---

### Phase 1 Checkpoint

**Validation Criteria**:
- [ ] All Phase 1 infrastructure provisioned and operational
- [ ] Integration tests passing
- [ ] No critical security findings

**Risk Review Questions**:
1. Is the core infrastructure stable? (YES/NO)
2. Are all services communicating correctly? (YES/NO)
3. Can we proceed to multi-environment deployment? (YES/NO)

---

## Phase 2: Multi-Environment Deployment

<!-- INSTRUCTION: Deploy Phase 1 infrastructure across environments.
     Standard progression: dev -> test -> staging -> prod.
     Each environment has its own tasks with environment-specific criteria.
     Principles: dev first (validation), test (integration), staging (parity),
     prod (approval required). -->

### Week 3: Dev and Test Environments
<!-- INSTRUCTION: Lower-stakes environments. Deploy aggressively to catch
     configuration issues before staging/prod. -->

#### Task 2.1: Deploy to Development Environment
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
<!-- INSTRUCTION: Deployment tasks need environment-specific acceptance criteria.
     Common checks: terraform apply succeeds, health check returns 200,
     service reachable from expected networks, logs flowing, auto-scaling active. -->
**Description**: Deploy all Phase 1 infrastructure to the development environment

**Acceptance Criteria**:
- [ ] `terraform apply` completes with 0 errors
- [ ] Health check endpoint returns HTTP 200
- [ ] Service accessible from dev VPC
- [ ] CloudWatch logs flowing correctly
- [ ] No security group rule violations

**Related Files**:
- `infrastructure/terraform/environments/dev/terraform.tfvars` (UPDATE)

**Commit Message**: `deploy: apply infrastructure to dev environment`

**Dependencies**: Phase 1 complete

**Verification**:
<!-- INSTRUCTION: Include copy-pasteable verification commands. -->
```bash
terraform output -state=environments/dev/terraform.tfstate
curl -s https://dev.[service-endpoint]/health | jq .
```

---

#### Task 2.2: Deploy to Test Environment
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: Deploy infrastructure to the test environment with integration validation

**Acceptance Criteria**:
- [ ] All dev acceptance criteria pass in test environment
- [ ] Integration tests pass against test endpoints
- [ ] Cross-service communication verified
- [ ] Test data loaded successfully

**Related Files**:
- `infrastructure/terraform/environments/test/terraform.tfvars` (UPDATE)

**Commit Message**: `deploy: apply infrastructure to test environment`

**Dependencies**: Task 2.1

---

### Week 4: Staging and Prod Environments
<!-- INSTRUCTION: Higher-stakes deployments. Include explicit approval gates
     and rollback procedures in acceptance criteria. -->

#### Task 2.3: Deploy to Staging Environment
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: Deploy to staging with production-parity validation

**Acceptance Criteria**:
- [ ] All test acceptance criteria pass in staging
- [ ] Production parity verified (instance types, scaling policies)
- [ ] Load test at 50% production traffic passes
- [ ] Rollback procedure tested and documented
- [ ] Security scan completed with no critical findings

**Related Files**:
- `infrastructure/terraform/environments/staging/terraform.tfvars` (UPDATE)

**Commit Message**: `deploy: apply infrastructure to staging environment`

**Dependencies**: Task 2.2
**Approval Required**: Team lead sign-off before proceeding
<!-- INSTRUCTION: Specify who must approve and through what channel
     (Slack approval, PR review, email confirmation). -->

---

#### Task 2.4: Deploy to Production Environment
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: Production deployment with monitoring validation

**Acceptance Criteria**:
- [ ] All staging acceptance criteria pass in production
- [ ] Smoke tests pass against production endpoints
- [ ] Monitoring dashboards showing healthy metrics
- [ ] Alert rules active and verified
- [ ] On-call engineer confirmed for post-deployment monitoring

**Related Files**:
- `infrastructure/terraform/environments/prod/terraform.tfvars` (UPDATE)

**Commit Message**: `deploy: apply infrastructure to production environment`

**Dependencies**: Task 2.3
**Approval Required**: Engineering manager sign-off required

**Rollback Plan**:
<!-- INSTRUCTION: Every production deployment MUST include a rollback plan.
     Include exact commands to revert to previous state. -->
```bash
# Rollback command
terraform apply -state=environments/prod/terraform.tfstate [previous-version]
```

---

### Phase 2 Checkpoint

**Validation Criteria**:
- [ ] Infrastructure deployed to all 4 environments
- [ ] All health checks passing
- [ ] No critical or high-severity security findings
- [ ] Monitoring dashboards operational

**Risk Review Questions**:
1. Are all environments healthy? (YES/NO)
2. Is the deployment repeatable and automated? (YES/NO)
3. Are rollback procedures documented and tested? (YES/NO)

---

## Phase 3: Integration

<!-- INSTRUCTION: Cross-component integration and end-to-end testing.
     Validates that infrastructure works correctly with other systems.
     Common tasks: API validation, cross-service communication, database
     connectivity, auth integration, third-party API, E2E user journeys. -->

### Week 5: Integration Testing

#### Task 3.1: End-to-End Integration Validation
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: Validate end-to-end data flow across all integrated components

**Acceptance Criteria**:
- [ ] Request flows through all components successfully
- [ ] Response times within acceptable thresholds
- [ ] Error handling works across service boundaries
- [ ] Data consistency verified at each integration point
- [ ] Idempotency verified for retry scenarios

**Related Files**:
- `[path/to/integration/test]` (CREATE)

**Commit Message**: `test: add end-to-end integration validation`

**Dependencies**: Phase 2 complete

---

#### Task 3.2: Regression Test Suite Execution
**Status**: PENDING
**Priority**: MEDIUM
**Estimated Time**: [X hours]
**Description**: Run full regression test suite across all environments

**Acceptance Criteria**:
- [ ] All regression tests pass in dev
- [ ] All regression tests pass in test
- [ ] All regression tests pass in staging
- [ ] No performance degradation vs. baseline
- [ ] Test report generated and reviewed

**Related Files**:
- `[path/to/regression/tests]` (REFERENCE)

**Commit Message**: `test: execute full regression suite across environments`

**Dependencies**: Task 3.1

---

### Phase 3 Checkpoint

**Validation Criteria**:
- [ ] All integration tests passing
- [ ] End-to-end data flow validated
- [ ] Regression tests green across all environments

---

## Phase 4: Monitoring and Documentation

<!-- INSTRUCTION: Completes the project with observability and documentation.
     Produces: monitoring dashboards, alerts, runbooks, architecture docs,
     troubleshooting guides, project summary/handover document. -->

### Week 6: Monitoring and Documentation

#### Task 4.1: Monitoring Dashboard and Alert Setup
**Status**: PENDING
**Priority**: HIGH
**Estimated Time**: [X hours]
**Description**: Create monitoring dashboards and configure alerts for production

**Acceptance Criteria**:
- [ ] Dashboard created with key metrics (CPU, memory, request count, latency)
- [ ] Alerts configured for critical thresholds
- [ ] Alert routing to correct on-call channel
- [ ] Dashboard accessible to all team members
- [ ] Alert response procedures documented

**Related Files**:
- `[path/to/monitoring/dashboard.json]` (CREATE)
- `[path/to/monitoring/alerts.tf]` (CREATE)

**Commit Message**: `feat: add monitoring dashboards and alerts`

**Dependencies**: Phase 3 complete

---

#### Task 4.2: Operational Runbook and Documentation
**Status**: PENDING
**Priority**: MEDIUM
**Estimated Time**: [X hours]
**Description**: Create operational runbooks and architecture documentation

**Acceptance Criteria**:
- [ ] Deployment runbook created with step-by-step procedures
- [ ] Troubleshooting guide covers common failure scenarios
- [ ] Architecture diagram updated to reflect final state
- [ ] Operational contacts and escalation paths documented
- [ ] Document reviewed by team lead

**Related Files**:
- `[path/to/docs/runbook.md]` (CREATE)
- `[path/to/docs/architecture.md]` (UPDATE)

**Commit Message**: `docs: add operational runbook and architecture documentation`

**Dependencies**: Task 4.1

---

#### Task 4.3: Project Summary Report
**Status**: PENDING
**Priority**: LOW
**Estimated Time**: [X hours]
**Description**: Create comprehensive project summary with metrics and lessons learned

**Acceptance Criteria**:
- [ ] All deliverables documented
- [ ] Success metrics reported (compare target vs. actual)
- [ ] Lessons learned captured
- [ ] Remaining items identified for future work
- [ ] Report reviewed by project stakeholders

**Related Files**:
- `[path/to/docs/project-summary.md]` (CREATE)

**Commit Message**: `docs: add project summary report`

**Dependencies**: Task 4.2

---

### Phase 4 Checkpoint

**Validation Criteria**:
- [ ] Monitoring dashboards showing healthy metrics
- [ ] Alerts tested and routing correctly
- [ ] All documentation complete and reviewed
- [ ] Project summary delivered to stakeholders

---

## Summary Statistics

<!-- INSTRUCTION: High-level summary table. Update as tasks complete.
     "Actual Time" filled during execution. Status uses legend emojis. -->

| Phase | Tasks | Estimated Time | Actual Time | Status |
|-------|-------|----------------|-------------|--------|
| Phase 0: Pre-Development Setup | 3 | [X] hours | [X] hours | PENDING |
| Phase 1: Infrastructure Foundation | 4 | [X] hours | [X] hours | PENDING |
| Phase 2: Multi-Environment Deployment | 4 | [X] hours | [X] hours | PENDING |
| Phase 3: Integration | 2 | [X] hours | [X] hours | PENDING |
| Phase 4: Monitoring and Documentation | 3 | [X] hours | [X] hours | PENDING |
| **Total** | **16** | **[X] hours** | **[X] hours** | **PENDING** |

---

## Critical Path

<!-- INSTRUCTION: The longest chain of dependent tasks determining minimum
     project duration. If any critical path task is delayed, the project is
     delayed. To identify: 1) Map all dependencies 2) Find longest chain
     3) List in order. All critical path tasks should be HIGH priority. -->

1. Task 0.1: [Environment Setup] -> Task 0.2: [Tooling Configuration]
2. Task 0.2: [Tooling Configuration] -> Task 1.1: [Core Infrastructure]
3. Task 1.1: [Core Infrastructure] -> Task 1.2: [Configuration]
4. Task 1.2: [Configuration] -> Task 2.1: [Dev Deployment]
5. Task 2.1: [Dev Deployment] -> Task 2.2: [Test Deployment]
6. Task 2.2: [Test Deployment] -> Task 2.3: [Staging Deployment]
7. Task 2.3: [Staging Deployment] -> Task 2.4: [Prod Deployment]
8. Task 2.4: [Prod Deployment] -> Task 3.1: [Integration Validation]
9. Task 3.1: [Integration Validation] -> Task 4.1: [Monitoring Setup]
<!-- INSTRUCTION: Replace with your actual critical path. For parallel work
     streams, there may be multiple paths merging at integration points. -->

**Critical Path Duration**: [X] working days
<!-- INSTRUCTION: Sum critical path task estimates. Add 20% buffer for unknowns. -->

---

## Risk Mitigation

<!-- INSTRUCTION: Document risks with probability, impact, and mitigation.
     Review at each phase checkpoint. Categories: Technical (compatibility,
     performance), Process (dependencies, approvals), External (vendors).
     Each risk: description, probability, impact, mitigation, owner. -->

### High Risk Tasks
<!-- INSTRUCTION: Failure would significantly delay project or cause production
     issues. Must have explicit mitigation and fallback plans. -->

1. **Task [X.Y]: [Task Title]**
   - **Risk**: [Specific description of what could go wrong]
   - **Probability**: [HIGH/MEDIUM/LOW]
   - **Impact**: [HIGH/MEDIUM/LOW]
   - **Mitigation**: [Specific action to prevent or recover]
   - **Fallback Plan**: [What to do if the risk materializes]
   - **Owner**: [Person or role responsible]
<!-- INSTRUCTION: Example:
     1. **Task 1.1: Core Infrastructure Provisioning**
        - **Risk**: Terraform state lock prevents apply
        - **Probability**: MEDIUM
        - **Impact**: HIGH
        - **Mitigation**: DynamoDB state locking with timeout
        - **Fallback Plan**: Manual unlock procedure in runbook
        - **Owner**: DevOps Engineer -->

2. **Task [X.Y]: [Task Title]**
   - **Risk**: [Specific description]
   - **Probability**: [HIGH/MEDIUM/LOW]
   - **Impact**: [HIGH/MEDIUM/LOW]
   - **Mitigation**: [Specific action]
   - **Fallback Plan**: [What to do if risk materializes]
   - **Owner**: [Person or role]

3. **Task [X.Y]: [Production Deployment Task]**
   - **Risk**: [Specific production risk]
   - **Probability**: [HIGH/MEDIUM/LOW]
   - **Impact**: HIGH
   - **Mitigation**: [Specific action including rollback procedure]
   - **Fallback Plan**: [Rollback to previous known-good state]
   - **Owner**: [Person or role]
<!-- INSTRUCTION: Production deployment risks are always HIGH impact. Include
     the exact rollback command or procedure as part of mitigation. -->

### Medium Risk Tasks
<!-- INSTRUCTION: Concerning but manageable. May cause delays but unlikely
     to cause project failure or production incidents. -->

1. **Task [X.Y]: [Task Title]**
   - **Risk**: [Specific description]
   - **Mitigation**: [Specific action]
   - **Owner**: [Person or role]

2. **Task [X.Y]: [Task Title]**
   - **Risk**: [Specific description]
   - **Mitigation**: [Specific action]
   - **Owner**: [Person or role]

3. **Task [X.Y]: [Documentation Task]**
   - **Risk**: Documentation may become outdated during active development
   - **Mitigation**: Schedule documentation updates at each phase checkpoint
   - **Owner**: [Person or role]
<!-- INSTRUCTION: Documentation drift is a common medium risk. Consider
     including it as standard for all projects with significant docs. -->

---

## Automatic Execution Statistics

<!-- INSTRUCTION: Track aggregate statistics. Update when task status changes.
     Provides a quick snapshot of overall project health. -->

- **Total Tasks**: [X]
<!-- INSTRUCTION: Count all tasks across all phases. -->
- **Completed**: [X]
<!-- INSTRUCTION: Count COMPLETED tasks. -->
- **In Progress**: [X]
<!-- INSTRUCTION: Count IN_PROGRESS tasks. Should be 0 or 1. -->
- **Pending**: [X]
<!-- INSTRUCTION: Count PENDING tasks. -->
- **Blocked**: [X]
<!-- INSTRUCTION: Count BLOCKED tasks. If > 0, list blockers above. -->
- **Cancelled**: [X]
<!-- INSTRUCTION: Count CANCELLED tasks. -->
- **Estimated Total Time**: [X] hours ([X] working days)
<!-- INSTRUCTION: Sum of all estimates. 8 productive hours/day. -->
- **Actual Elapsed Time**: [X] hours ([X] working days)
<!-- INSTRUCTION: Track from project start. Compare against estimated total. -->
- **Current Progress**: [X]%
<!-- INSTRUCTION: (Completed / Total) * 100. For accuracy, weight by time:
     (Sum of completed estimates / Total estimates) * 100 -->

---

## Quick Reference Commands

<!-- INSTRUCTION: Common commands organized by category. Must be copy-pasteable
     or have clearly marked placeholders. Categories: daily workflow, debugging,
     git workflow, monitoring. -->

### Daily Workflow
```bash
# Morning startup
cd [project/directory]
# Build / apply infrastructure
[build-or-apply-command]
# Run tests
[test-command]
# Check health
[health-check-command]
```

### Debugging
```bash
# Clean build
[clean-and-rebuild-command]
# Verbose output
[verbose-test-or-apply-command]
# Manual verification
[verification-command]
```

### Git Workflow
```bash
# After each task
git add [specific-files]
git commit -m "type: conventional commit message"
git push origin [BRANCH_NAME]
```
<!-- INSTRUCTION: Replace [BRANCH_NAME] with actual branch from Project
     Information section for team consistency. -->

---

*Last Updated: [DATE]*
<!-- INSTRUCTION: Update every modification. ISO 8601: YYYY-MM-DD. -->
*Project Status: [STATUS]*
<!-- INSTRUCTION: PENDING | IN_PROGRESS | BLOCKED | COMPLETE. Include emoji. -->
*Automatic Execution Mode: Enabled*
<!-- INSTRUCTION: Change to "Disabled" if not used with automated tools. -->
*Next Milestone: [Next phase or checkpoint]*
<!-- INSTRUCTION: Next upcoming milestone. Example: "Phase 1: Infrastructure Foundation" -->
