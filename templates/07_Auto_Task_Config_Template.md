# Auto Task Configuration - [PROJECT_TITLE]

<!-- ==============================================================================
     INSTRUCTION: SPEC CODING TEMPLATE - AUTO TASK CONFIGURATION
     ==============================================================================
     This template defines automated task execution configuration for
     infrastructure and platform engineering projects. It provides a structured
     YAML configuration that drives task automation, branching strategy,
     commit conventions, status tracking, and validation gates.

     HOW TO USE THIS TEMPLATE:
     1. Replace all [PLACEHOLDER] markers with project-specific content.
     2. Copy the YAML configuration block into a file named
        AUTO_TASK_CONFIG.yaml in your project root or .spec/ directory.
     3. Follow the inline comments (<!-- INSTRUCTION: ... -->) for guidance.
     4. Validate your configuration using the field reference (Section 3).
     5. Pair this configuration with a Task List document (Template 04).
     ============================================================================ -->

**Document ID**: AUTOCONFIG_[PROJECT_NAME]
<!-- INSTRUCTION: Short, uppercase, underscore-separated. Example: AUTOCONFIG_ALB_FARGATE_JAVA_SERVICE -->

**Issue**: #[ISSUE_NUMBER] - [Issue Title]
<!-- INSTRUCTION: Reference GitHub/Jira issue. Example: #1880 - ALB + Fargate Infrastructure -->

**Created**: [DATE]
<!-- INSTRUCTION: Date first created. Example: 2025-01-15 -->

**Branch**: [BRANCH_NAME]
<!-- INSTRUCTION: Must match branch.name in YAML config. Example: feature/1880-alb-fargate-java-service -->

**Status**: Draft
<!-- INSTRUCTION: Lifecycle: Draft -> Review -> Approved -> Active -> Complete -->

---

## Table of Contents

- [Section 1: Overview](#section-1-overview)
- [Section 2: Configuration Template](#section-2-configuration-template)
- [Section 3: Configuration Fields Reference](#section-3-configuration-fields-reference)
- [Section 4: Usage Examples](#section-4-usage-examples)
- [Section 5: Integration with Task List](#section-5-integration-with-task-list)
- [Section 6: Customization Guide](#section-6-customization-guide)
- [Section 7: Troubleshooting](#section-7-troubleshooting)

---

## Section 1: Overview

### 1.1 What is AUTO_TASK_CONFIG?

AUTO_TASK_CONFIG is a YAML-based configuration file that drives automated task
execution for infrastructure and platform engineering workflows. It defines how
tasks are executed, how branches and commits are managed, how status is tracked,
and what validation gates must pass before proceeding. It pairs with a Task List
document (Template 04) for full automation coverage.

### 1.2 When to Use This Template

- Multi-step infrastructure deployments with sequential or parallel phases
- Environment promotion workflows (dev, test, staging, prod)
- Batch provisioning tasks requiring validation gates
- Projects where task execution consistency and auditability are required

### 1.3 Integration with Task List Documents

<!-- INSTRUCTION: Ensure status_tracking.file points to the correct Task List document. -->

```
[AUTO_TASK_CONFIG.yaml] ----references----> [task-list.md]
        |                                        |
        | drives execution                       | defines tasks
        v                                        v
   [Automation Engine] ----updates-------> [task-list.md status]
```

---

## Section 2: Configuration Template

<!-- INSTRUCTION: Copy the YAML block below into AUTO_TASK_CONFIG.yaml.
     Replace all [placeholder] values with project-specific configuration.
     Remove inline # comments after configuration is finalized. -->

```yaml
# ==============================================================================
# AUTO_TASK_CONFIG - [Project Name]
# INSTRUCTION: Replace [Project Name] with your project identifier.
# ==============================================================================

execution:
  mode: auto | semi-auto | manual
  # INSTRUCTION: auto=full automation, semi-auto=pause at gates, manual=track only
  max_parallel_tasks: [number]
  # INSTRUCTION: Max concurrent tasks. Use 1 for sequential. Range: 1-5.
  stop_on_failure: true | false
  auto_commit: true | false
  commit_message_prefix: "[prefix]"

branch:
  name: "[branch-name]"
  create_if_missing: true | false
  base_branch: "main"

commit:
  auto_stage: true | false
  # INSTRUCTION: When true, stages all changed files automatically.
  conventional_commits: true | false
  message_template: "[type]([scope]): [description]"
  # INSTRUCTION: Tokens: [type], [scope], [description], [task_id]

status_tracking:
  file: "[path/to/task-list.md]"
  # INSTRUCTION: Path to Task List markdown (Template 04). Relative to project root.
  format: "emoji" | "text"
  # INSTRUCTION: emoji=[ ]/[x] checkboxes; text=TODO/DONE/BLOCKED/SKIPPED.
  update_frequency: "after_each_task" | "after_each_phase" | "manual"

progress_documentation:
  enabled: true | false
  file: "[path/to/progress.md]"
  include_timestamps: true | false

blocking_conditions:
  # INSTRUCTION: Conditions that halt or alter execution. Actions: stop, skip, ask.
  - condition: "Uncommitted changes in working directory"
    action: "stop"
  - condition: "Target environment unreachable"
    action: "stop"
  - condition: "Validation gate failure"
    action: "ask"
  - condition: "Branch divergence from base"
    action: "ask"

environments:
  order: ["dev", "test", "staging", "prod"]
  deploy_sequence: true | false
  # INSTRUCTION: When true, dev must succeed before test, test before staging, etc.
  require_approval: ["staging", "prod"]
  # INSTRUCTION: Environments where automation pauses for human confirmation.

validation:
  pre_task:
    - "[validation command]"
    # INSTRUCTION: Example: "terraform validate", "aws sts get-caller-identity"
  post_task:
    - "[validation command]"
    # INSTRUCTION: Example: "terraform plan -detailed-exitcode"
  pre_phase:
    - "[validation command]"
    # INSTRUCTION: Example: "terraform init -backend-config=env.hcl"
  post_phase:
    - "[validation command]"
    # INSTRUCTION: Example: "pytest tests/integration/"
```

---

## Section 3: Configuration Fields Reference

<!-- INSTRUCTION: Use this table to validate your YAML configuration.
     Check the "Required" column to identify mandatory fields. -->

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `execution.mode` | string | Yes | `auto` | Automation level: `auto`, `semi-auto`, `manual` |
| `execution.max_parallel_tasks` | integer | Yes | `3` | Max concurrent tasks. Use `1` for sequential |
| `execution.stop_on_failure` | boolean | Yes | `true` | Halt all execution when any task fails |
| `execution.auto_commit` | boolean | No | `true` | Commit automatically after each task |
| `execution.commit_message_prefix` | string | No | `"[automate]"` | Prefix for auto-generated commit messages |
| `branch.name` | string | Yes | - | Feature branch name for automated work |
| `branch.create_if_missing` | boolean | No | `true` | Create the branch if it does not exist |
| `branch.base_branch` | string | Yes | `"main"` | Base branch to branch from and merge into |
| `commit.auto_stage` | boolean | No | `true` | Automatically stage all changed files |
| `commit.conventional_commits` | boolean | No | `true` | Enforce Conventional Commits format |
| `commit.message_template` | string | No | `"[type]([scope]): [description]"` | Template with tokens: `[type]`, `[scope]`, `[description]`, `[task_id]` |
| `status_tracking.file` | string | Yes | - | Path to the Task List markdown file |
| `status_tracking.format` | string | No | `"emoji"` | Status display: `emoji` or `text` |
| `status_tracking.update_frequency` | string | No | `"after_each_task"` | Cadence: `after_each_task`, `after_each_phase`, `manual` |
| `progress_documentation.enabled` | boolean | No | `true` | Enable automatic progress logging |
| `progress_documentation.file` | string | No | `"progress.md"` | Path to the progress log file |
| `progress_documentation.include_timestamps` | boolean | No | `true` | Include ISO 8601 timestamps in entries |
| `blocking_conditions[].condition` | string | Yes | - | Description of the blocking condition |
| `blocking_conditions[].action` | string | Yes | `"stop"` | Action: `stop`, `skip`, or `ask` |
| `environments.order` | string[] | No | `["dev","test","staging","prod"]` | Ordered environment names |
| `environments.deploy_sequence` | boolean | No | `true` | Enforce sequential promotion |
| `environments.require_approval` | string[] | No | `["staging","prod"]` | Environments requiring manual approval |
| `validation.pre_task` | string[] | No | `[]` | Commands to run before each task |
| `validation.post_task` | string[] | No | `[]` | Commands to run after each task |
| `validation.pre_phase` | string[] | No | `[]` | Commands to run before each phase |
| `validation.post_phase` | string[] | No | `[]` | Commands to run after each phase |

---

## Section 4: Usage Examples

<!-- INSTRUCTION: Adapt these examples to your project. Each is a complete
     configuration that can be copied and modified. -->

### 4.1 Basic Example - Infrastructure Deployment

```yaml
# AUTO_TASK_CONFIG - Terraform VPC Deployment
# INSTRUCTION: Single-environment Terraform deployment with minimal validation.

execution:
  mode: auto
  max_parallel_tasks: 1
  stop_on_failure: true
  auto_commit: true
  commit_message_prefix: "[tf-deploy]"

branch:
  name: "feature/1920-vpc-network-setup"
  create_if_missing: true
  base_branch: "main"

commit:
  auto_stage: true
  conventional_commits: true
  message_template: "feat(network): [description] - refs #[task_id]"

status_tracking:
  file: "devops/docs/tasks/TASK_LIST_1920.md"
  format: "emoji"
  update_frequency: "after_each_task"

progress_documentation:
  enabled: true
  file: "devops/docs/progress/PROGRESS_1920.md"
  include_timestamps: true

blocking_conditions:
  - condition: "Uncommitted changes in working directory"
    action: "stop"
  - condition: "Terraform state lock detected"
    action: "stop"

environments:
  order: ["dev"]
  deploy_sequence: false
  require_approval: []

validation:
  pre_task:
    - "terraform fmt -check"
    - "terraform validate"
  post_task:
    - "terraform plan -detailed-exitcode"
  pre_phase:
    - "terraform init -backend-config=dev.hcl"
  post_phase:
    - "terraform output -json > devops/outputs/dev-vpc.json"
```

### 4.2 Advanced Example - Multi-Environment with Gates

```yaml
# AUTO_TASK_CONFIG - Java Fargate Service Multi-Env Deployment
# INSTRUCTION: Full environment promotion with approval gates and validation.

execution:
  mode: semi-auto
  max_parallel_tasks: 2
  stop_on_failure: true
  auto_commit: true
  commit_message_prefix: "[fargate-deploy]"

branch:
  name: "feature/1880-alb-fargate-java-service"
  create_if_missing: false
  base_branch: "main"

commit:
  auto_stage: true
  conventional_commits: true
  message_template: "[type]([scope]): [description]"

status_tracking:
  file: "devops/docs/tasks/TASK_LIST_1880.md"
  format: "emoji"
  update_frequency: "after_each_phase"

progress_documentation:
  enabled: true
  file: "devops/docs/progress/PROGRESS_1880.md"
  include_timestamps: true

blocking_conditions:
  - condition: "Uncommitted changes in working directory"
    action: "stop"
  - condition: "Target environment unreachable"
    action: "stop"
  - condition: "Validation gate failure"
    action: "ask"
  - condition: "Branch divergence from base"
    action: "ask"
  - condition: "ALB target group health check failing"
    action: "stop"

environments:
  order: ["dev", "test", "staging", "prod"]
  deploy_sequence: true
  require_approval: ["staging", "prod"]

validation:
  pre_task:
    - "terraform fmt -check"
    - "terraform validate"
    - "aws sts get-caller-identity"
  post_task:
    - "terraform plan -detailed-exitcode"
    - "aws elbv2 describe-target-health --target-group-arn $TG_ARN"
  pre_phase:
    - "terraform init -backend-config=env/$ENV.hcl"
  post_phase:
    - "pytest tests/integration/test_$ENV.py"
    - "terraform output -json > devops/outputs/$ENV-outputs.json"
```

### 4.3 Minimal Example - Single Task Automation

```yaml
# AUTO_TASK_CONFIG - Quick Lambda Deploy
# INSTRUCTION: Minimal config for small tasks. Only required fields populated.

execution:
  mode: auto
  max_parallel_tasks: 1
  stop_on_failure: true

branch:
  name: "bugfix/2105-lambda-timeout-fix"
  create_if_missing: true
  base_branch: "main"

status_tracking:
  file: "devops/docs/tasks/TASK_LIST_2105.md"
  format: "text"
  update_frequency: "after_each_task"
```

---

## Section 5: Integration with Task List

<!-- INSTRUCTION: AUTO_TASK_CONFIG integrates with Task List (Template 04) via status_tracking.file. -->

### 5.1 Reference Mechanism

The `status_tracking.file` field points to a Task List markdown file. The
automation engine reads task IDs/descriptions, executes tasks in phase order,
and updates status fields based on `update_frequency`.

### 5.2 Status Update Flow

```
                     AUTO_TASK_CONFIG.yaml
                            |
                 [Reads config settings]
                            |
                            v
   +--------------------------------------------------+
   |              AUTOMATION ENGINE                     |
   |  1. Read Task List file                            |
   |     v                                              |
   |  2. Parse tasks and phases                         |
   |     v                                              |
   |  3. For each task:                                 |
   |     +--> Run pre_task validation                   |
   |     +--> Execute task                              |
   |     +--> Run post_task validation                  |
   |     +--> Update status in Task List                |
   |     +--> Write progress log entry                  |
   |     +--> Commit changes (if auto_commit=true)      |
   |     v                                              |
   |  4. For each phase:                                |
   |     +--> Run pre_phase validation                  |
   |     +--> Execute all tasks in phase                |
   |     +--> Run post_phase validation                 |
   +--------------------------------------------------+
                   |                  |
                   v                  v
        [task-list.md]      [progress.md updated]
```

### 5.3 Progress Tracking Example

Task List before execution:

```markdown
### Phase 1: Network Infrastructure
- [ ] T1.1 Create VPC and subnets
- [ ] T1.2 Configure route tables
```

Task List after T1.1 completes:

```markdown
### Phase 1: Network Infrastructure
- [x] T1.1 Create VPC and subnets
- [ ] T1.2 Configure route tables
```

Progress log entry:

```
## 2025-01-15T09:32:14Z - T1.1 Create VPC and subnets
- Status: COMPLETED
- Duration: 47s
- Validation: pre_task passed, post_task passed
- Commit: feat(network): create VPC and subnets - refs T1.1
```

---

## Section 6: Customization Guide

<!-- INSTRUCTION: Use this section to extend the configuration beyond
     the default template. -->

### 6.1 Adding Custom Blocking Conditions

```yaml
blocking_conditions:
  # INSTRUCTION: Each condition needs "condition" (string) and "action" (stop|skip|ask).
  # Evaluated in order; first match wins.

  - condition: "Database migration pending"
    action: "ask"
    # INSTRUCTION: "ask" lets the operator decide whether to proceed.

  - condition: "SSL certificate expiring within 7 days"
    action: "stop"
    # INSTRUCTION: "stop" is safest for security-related conditions.

  - condition: "ECS service desired count mismatch"
    action: "ask"
```

### 6.2 Custom Validation Commands

Validation commands are shell commands that must return exit code 0. The
automation engine provides environment variables: `$ENV` (current environment),
`$PHASE` (current phase), `$TASK_ID` (current task identifier).

```yaml
validation:
  pre_task:
    # INSTRUCTION: Infrastructure validation before task execution.
    - "terraform fmt -check -recursive"
    - "tflint --init --config=.tflint.hcl"
  post_task:
    # INSTRUCTION: Deployment verification after task completion.
    - "curl -sf $HEALTH_CHECK_URL/health || exit 1"
  pre_phase:
    # INSTRUCTION: Phase-level environment preparation.
    - "aws ssm get-parameter --name /config/$ENV/api-endpoint"
  post_phase:
    # INSTRUCTION: Phase-level integration verification.
    - "pytest tests/smoke/ -m $ENV --tb=short -q"
```

### 6.3 Environment-Specific Overrides

```yaml
# INSTRUCTION: Base configuration applies to all environments.
execution:
  mode: semi-auto
  stop_on_failure: true

environments:
  order: ["dev", "staging", "prod"]
  deploy_sequence: true
  require_approval: ["staging", "prod"]

  # INSTRUCTION: Per-environment overrides merge shallowly with base config.
  overrides:
    dev:
      execution:
        mode: auto
        # INSTRUCTION: Dev runs fully automated.
      validation:
        post_task:
          - "curl -sf http://dev-internal.example.com/health"
    staging:
      execution:
        mode: semi-auto
      validation:
        post_task:
          - "curl -sf https://staging.example.com/health"
    prod:
      execution:
        mode: semi-auto
        max_parallel_tasks: 1
        # INSTRUCTION: Prod is strictly sequential.
      validation:
        post_task:
          - "curl -sf https://www.example.com/health"
          - "python3 scripts/verify_prod_readiness.py"
```

---

## Section 7: Troubleshooting

<!-- INSTRUCTION: Each entry includes symptom, root cause, and resolution. -->

### 7.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Tasks not executing | `execution.mode` set to `manual` | Change to `auto` or `semi-auto` |
| Status not updating | `status_tracking.file` path incorrect | Verify path is relative to project root and file exists |
| Commits not created | `auto_commit` or `auto_stage` is `false` | Set both to `true` for full auto-commit |
| Branch not found | `branch.name` does not match existing branch | Set `create_if_missing` to `true` or verify name |
| Validation always fails | Command returns non-zero exit code | Test validation commands manually in target environment |
| Execution stops unexpectedly | Blocking condition triggered | Review blocking_conditions and check progress log |
| Parallel tasks interfere | Shared state or file conflicts | Reduce `max_parallel_tasks` to `1` |
| Environment promotion blocked | Previous env failed validation | Check post_task validation output for preceding env |
| Commit message rejected | Template non-compliant with `conventional_commits` | Ensure template produces valid conventional format |
| Progress log not created | `progress_documentation.enabled` is `false` | Set to `true` and verify file path |

### 7.2 Debug Mode

```yaml
# INSTRUCTION: Disable debug mode before merging to main branch.
debug:
  enabled: true
  log_file: "debug/auto_task_debug.log"
  # INSTRUCTION: Path relative to project root. Directory must exist.
  verbosity: "detailed"
  # INSTRUCTION: Levels: "minimal" (errors), "standard" (+warnings), "detailed" (full trace).
  include_environment_variables: false
  # INSTRUCTION: WARNING: true exposes secrets in logs. Use ONLY for local debugging.
  dry_run: false
  # INSTRUCTION: When true, reports actions without executing. For config validation.
```

---

**Version**: 1.0
<!-- INSTRUCTION: Increment on significant changes. Semantic versioning:
     Major (2.0): Breaking field changes
     Minor (1.1): New optional fields
     Patch (1.0.1): Documentation corrections -->

**Last Updated**: [DATE]
<!-- INSTRUCTION: Update this date whenever the configuration is modified. -->
