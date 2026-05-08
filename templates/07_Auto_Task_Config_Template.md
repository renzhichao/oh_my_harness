# Auto Task Configuration - [PROJECT_TITLE]

<!-- ==============================================================================
     INSTRUCTION: SPEC CODING TEMPLATE - AUTO TASK CONFIGURATION (v2)
     ==============================================================================
     This template defines automated task execution configuration for
     infrastructure and platform engineering projects, using a Sub-Agent
     architecture. Each task runs in an isolated Sub-Agent context, enabling
     parallel execution, clean context separation, and resumable workflows.

     HOW TO USE THIS TEMPLATE:
     1. Replace all [PLACEHOLDER] markers with project-specific content.
     2. Copy the YAML configuration block into AUTO_TASK_CONFIG.yaml.
     3. Follow the inline comments (INSTRUCTION: markers) for guidance.
     4. Validate your configuration using the field reference (Section 4).
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

- [Section 1: Design Philosophy](#section-1-design-philosophy)
- [Section 2: Architecture Overview](#section-2-architecture-overview)
- [Section 3: Task Execution Lifecycle](#section-3-task-execution-lifecycle)
- [Section 4: Configuration Template](#section-4-configuration-template)
- [Section 5: Configuration Fields Reference](#section-5-configuration-fields-reference)
- [Section 6: Task Execution Rules](#section-6-task-execution-rules)
- [Section 7: Task List Integration](#section-7-task-list-integration)
- [Section 8: Orchestrator Execution Templates](#section-8-orchestrator-execution-templates)
- [Section 9: Parallel Execution Strategy](#section-9-parallel-execution-strategy)
- [Section 10: Exception Handling](#section-10-exception-handling)
- [Section 11: Customization Guide](#section-11-customization-guide)
- [Section 12: Troubleshooting](#section-12-troubleshooting)

---

## Section 1: Design Philosophy

### 1.1 Core Principles

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Sub-Agent Task Execution Principles               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🔒 Native Context Isolation                                       │
│     ├── Each task executed by an independent Sub-Agent              │
│     ├── Sub-Agent has its own context window, naturally isolated    │
│     ├── Orchestrator main context stays clean                       │
│     └── Isolation guaranteed by architecture, not discipline        │
│                                                                     │
│  🎯 Single Responsibility                                           │
│     ├── Each Sub-Agent completes exactly one well-defined goal      │
│     ├── Task granularity is controllable and traceable              │
│     └── Completion criteria are explicit (Acceptance Criteria)      │
│                                                                     │
│  📝 State Persistence                                               │
│     ├── Task states recorded in the Task List document              │
│     ├── Every state change is immediately persisted                 │
│     └── Supports resumption after interruption                      │
│                                                                     │
│  ⚡ Parallel Execution                                              │
│     ├── Dependency-free tasks dispatched to multiple Sub-Agents     │
│     ├── Orchestrator handles dependency ordering and concurrency    │
│     └── Coordination via Task List and file system                  │
│                                                                     │
│  🔄 Idempotent Execution                                            │
│     ├── Same task can be safely re-executed                         │
│     ├── Pre-checks ensure execution conditions are met              │
│     └── Failed tasks can resume from checkpoint                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 v1 to v2 Key Changes

| Dimension | v1 (Manual Isolation) | v2 (Sub-Agent Isolation) |
|-----------|----------------------|--------------------------|
| Context isolation | Manual `/clear`, new sessions | Sub-Agent native isolation |
| Isolation reliability | Depends on human discipline | Architecture-level guarantee |
| Parallel capability | Not supported (single session serial) | Native support (multi-Agent parallel) |
| Orchestrator context | Polluted by task details | Clean, dispatch only |
| Inter-task communication | Via files / Task List | Via files / Task List (unchanged) |

### 1.3 When to Use This Template

- Multi-step infrastructure deployments with sequential or parallel phases
- Environment promotion workflows (dev, test, staging, prod)
- Projects where task execution consistency and auditability are required
- Multi-module parallel development to reduce total time
- Batch test generation (parallelizable, independent)

---

## Section 2: Architecture Overview

### 2.1 Two-Layer Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Two-Layer Execution Architecture               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    Orchestrator (Main Agent)                 │   │
│   │                                                             │   │
│   │  Responsibilities:                                          │   │
│   │  ├── Read Task List, parse task dependencies                │   │
│   │  ├── Select next executable task(s)                         │   │
│   │  ├── Construct Sub-Agent Prompts                            │   │
│   │  ├── Dispatch Sub-Agents to execute tasks                   │   │
│   │  ├── Collect Sub-Agent execution results                    │   │
│   │  ├── Update Task List status                                │   │
│   │  └── Handle exceptions and retries                          │   │
│   │                                                             │   │
│   │  Context: Clean, only dispatch state                        │   │
│   └──────────┬──────────────┬──────────────┬────────────────────┘   │
│              │              │              │                         │
│              ▼              ▼              ▼                         │
│   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│   │  Sub-Agent 1 │ │  Sub-Agent 2 │ │  Sub-Agent 3 │              │
│   │  (Task A)    │ │  (Task B)    │ │  (Task C)    │              │
│   │              │ │              │ │              │              │
│   │ Independent  │ │ Independent  │ │ Independent  │              │
│   │ context      │ │ context      │ │ context      │              │
│   │ Destroyed    │ │ Destroyed    │ │ Destroyed    │              │
│   │ after done   │ │ after done   │ │ after done   │              │
│   └──────────────┘ └──────────────┘ └──────────────┘              │
│                                                                     │
│   Communication: Task List document + File system + Git repository │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Orchestrator vs Sub-Agent Responsibility Matrix

| Responsibility | Orchestrator | Sub-Agent |
|---------------|-------------|-----------|
| Read Task List | YES | NO (task details injected via Prompt) |
| Dependency check | YES | NO (Orchestrator guarantees deps met) |
| State management | YES | NO (reports result, Orchestrator updates) |
| Code writing | NO | YES |
| Test execution | NO | YES |
| Git commit | NO (or on behalf of Sub-Agent) | YES |
| AC verification | Receives result | YES (executes and reports) |
| Exception retry | YES | NO (reports failure, Orchestrator decides) |

---

## Section 3: Task Execution Lifecycle

### 3.1 Lifecycle Diagram

```
┌──────────┐
│  PENDING │ ◄──────────────────────────────────────────┐
└────┬─────┘                                            │
     │ [Orchestrator selects task]                      │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│DEPENDENCY│────►│ 1. Check dependencies COMPLETED│      │
│  CHECK   │     │ 2. Check prerequisite files    │      │
└────┬─────┘     └──────────────────────────────┘       │
     │ [Dependencies met]                                │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│ PREPARE  │────►│ 1. Update Task List to RUNNING│      │
│   AGENT  │     │ 2. Construct Sub-Agent Prompt │       │
└────┬─────┘     │ 3. Inject task details         │       │
     │           └──────────────────────────────┘       │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│  SPAWN   │────►│ 1. Invoke Agent Tool           │      │
│SUB-AGENT │     │ 2. Sub-Agent runs in isolation │  [FAILED]
└────┬─────┘     │ 3. Await execution result      │      │
     │           └──────────────────────────────┘       │
     │ [Result returned]                                 │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│  RESULT  │────►│ 1. Parse Sub-Agent result     ├──────┘
│  PROCESS │     │ 2. Verify AC completion       │
└────┬─────┘     │ 3. Check deliverables          │
     │ [Passed]  └──────────────────────────────┘
     ▼
┌──────────┐     ┌──────────────────────────────┐
│ FINALIZE │────►│ 1. Git add & commit           │
│          │     │ 2. Update Task List status    │
└────┬─────┘     │ 3. Record execution summary   │
     │           └──────────────────────────────┘
     ▼
┌──────────┐     ┌──────────────────────────────┐
│  NEXT    │────►│ 1. Select next executable task│
│  TASK    │     │ 2. Dispatch parallel if able  │
└────┬─────┘     │ 3. End if no more tasks       │
     │           └──────────────────────────────┘
     ▼
┌──────────┐
│COMPLETED │
└──────────┘
```

### 3.2 Phase Descriptions

| Phase | Orchestrator Action | Sub-Agent Action |
|-------|---------------------|------------------|
| PENDING | Awaiting selection | - |
| DEPENDENCY_CHECK | Verify prerequisites | - |
| PREPARE_AGENT | Construct Prompt | - |
| SPAWN_SUB_AGENT | Invoke Agent Tool | Execute in isolated context |
| RESULT_PROCESS | Parse result, verify AC | - |
| FINALIZE | Git commit, update docs | - |
| NEXT_TASK | Select and dispatch | - |
| COMPLETED | - | - |

---

## Section 4: Configuration Template

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
  # INSTRUCTION: Max concurrent Sub-Agents. Use 1 for sequential. Range: 1-5.
  stop_on_failure: true | false
  auto_commit: true | false
  commit_message_prefix: "[prefix]"

sub_agent:
  # INSTRUCTION: Sub-Agent dispatch configuration.
  default_mode: foreground | background | worktree
  # INSTRUCTION: foreground=wait for result, background=fire and continue, worktree=file isolation.
  subagent_type: general-purpose | [custom-type]
  # INSTRUCTION: Optional. Matches available agent types in your tool.
  max_retries: 2
  # INSTRUCTION: Max retry attempts per failed task. Range: 0-3.
  retry_backoff: [30, 60]
  # INSTRUCTION: Seconds to wait between retries. Length must match max_retries.

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

## Section 5: Configuration Fields Reference

<!-- INSTRUCTION: Use this table to validate your YAML configuration.
     Check the "Required" column to identify mandatory fields. -->

### 5.1 Execution Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `execution.mode` | string | Yes | `auto` | Automation level: `auto`, `semi-auto`, `manual` |
| `execution.max_parallel_tasks` | integer | Yes | `3` | Max concurrent Sub-Agents. Use `1` for sequential |
| `execution.stop_on_failure` | boolean | Yes | `true` | Halt all execution when any task fails |
| `execution.auto_commit` | boolean | No | `true` | Commit automatically after each task |
| `execution.commit_message_prefix` | string | No | `"[automate]"` | Prefix for auto-generated commit messages |

### 5.2 Sub-Agent Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `sub_agent.default_mode` | string | No | `foreground` | Default dispatch mode: `foreground`, `background`, `worktree` |
| `sub_agent.subagent_type` | string | No | `general-purpose` | Agent type for specialized tasks |
| `sub_agent.max_retries` | integer | No | `2` | Max retry attempts per failed task |
| `sub_agent.retry_backoff` | integer[] | No | `[30, 60]` | Seconds to wait between retries |

### 5.3 Branch & Commit Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `branch.name` | string | Yes | - | Feature branch name for automated work |
| `branch.create_if_missing` | boolean | No | `true` | Create the branch if it does not exist |
| `branch.base_branch` | string | Yes | `"main"` | Base branch to branch from and merge into |
| `commit.auto_stage` | boolean | No | `true` | Automatically stage all changed files |
| `commit.conventional_commits` | boolean | No | `true` | Enforce Conventional Commits format |
| `commit.message_template` | string | No | `"[type]([scope]): [description]"` | Template with tokens: `[type]`, `[scope]`, `[description]`, `[task_id]` |

### 5.4 Tracking & Validation Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
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

## Section 6: Task Execution Rules

### 6.1 Execution Order Strategy

```yaml
execution_order:
  strategy: "dependency_first"
  # INSTRUCTION: Available strategies:
  #   dependency_first: Execute all prerequisite tasks first
  #   phase_sequential: Tasks within a phase can parallelize, across phases must be sequential
  #   priority_based: Higher priority tasks dispatched first

  parallel:
    enabled: true
    # INSTRUCTION: Enable parallel dispatch to multiple Sub-Agents.
    max_concurrent: [number]
    # INSTRUCTION: Max concurrent Sub-Agents. Should match execution.max_parallel_tasks.
    strategy: "dependency_level"
    # INSTRUCTION: dependency_level = tasks at the same dependency level can run in parallel.
```

### 6.2 Task Selection Algorithm

```python
def get_next_tasks(task_list: List[Task], max_concurrent: int = 3) -> List[Task]:
    """
    Get the next batch of executable tasks (supports parallel dispatch).

    Selection rules:
    1. Status is PENDING
    2. All dependencies have status COMPLETED
    3. Sort by task ID (stable ordering)
    4. Do not exceed max_concurrent concurrent tasks
    """
    ready_tasks = []

    for task in sorted(task_list, key=lambda t: t.id):
        if task.status != TaskStatus.PENDING:
            continue

        dependencies_met = all(
            dep.status == TaskStatus.COMPLETED
            for dep in task.dependencies
        )

        if not dependencies_met:
            continue

        ready_tasks.append(task)

        if len(ready_tasks) >= max_concurrent:
            break

    return ready_tasks
```

### 6.3 Sub-Agent Mode Selection Matrix

| Scenario | Recommended Mode | Reason |
|----------|-----------------|--------|
| Sequential tasks (A -> B -> C) | foreground | Later tasks depend on earlier results |
| Independent tasks (A, B, C unrelated) | background | Parallel execution saves time |
| May modify same files | worktree | File-system-level isolation prevents conflicts |
| Single simple task | foreground | No complex isolation needed |
| Batch test writing | background | Parallelizable, no inter-dependencies |

---

## Section 7: Task List Integration

<!-- INSTRUCTION: AUTO_TASK_CONFIG integrates with Task List (Template 04) via status_tracking.file. -->

### 7.1 Status Update Triggers

```yaml
task_list_update:
  update_triggers:
    - on_task_start       # Before Sub-Agent dispatch
    - on_task_complete    # After Sub-Agent returns success
    - on_task_failure     # After Sub-Agent returns failure

  fields_to_update:
    on_task_start:
      - status: "RUNNING"
      - start_time: "current timestamp"
      - agent_mode: "foreground | background | worktree"

    on_task_complete:
      - status: "COMPLETED"
      - end_time: "current timestamp"
      - commit_id: "Git Commit SHA"
      - commit_summary: "change summary"
      - acceptance_criteria: "mark each item complete"

    on_task_failure:
      - status: "FAILED"
      - failure_reason: "failure cause"
```

### 7.2 Task State Machine

```
                         ┌───────────┐
              ┌─────────►│ CANCELLED │
              │          └───────────┘
              │ [user cancel]
              │
         ┌────┴────┐    [dep not met]    ┌─────────┐
         │ PENDING │◄──────────────────│ BLOCKED │
         └────┬────┘                   └────▲────┘
              │                              │
              │ [dispatch Sub-Agent]         │ [dep incomplete]
              │                              │
              ▼                              │
         ┌─────────┐                         │
         │ RUNNING │─────────────────────────┘
         └────┬────┘
              │ [Sub-Agent returns result]
        ┌─────┴──────┐
        │            │
        ▼            ▼
   ┌──────────┐  ┌────────┐
   │COMPLETED │  │ FAILED │
   └──────────┘  └────┬───┘
                       │ [Orchestrator retries]
                       ▼
                  ┌─────────┐
                  │ PENDING │
                  └─────────┘
```

### 7.3 Sub-Agent Result Report Protocol

<!-- INSTRUCTION: Sub-Agents must return results in this fixed format. -->

```markdown
## Sub-Agent Execution Report

### Result
- **Status**: SUCCESS / FAILED
- **Failure Reason**: (fill only when FAILED)

### Changed Files
- <file1> — <change description>
- <file2> — <change description>

### Acceptance Criteria Verification
- [x] AC1: <description>
- [x] AC2: <description>
- [ ] AC3: <description> — <reason incomplete>

### Git Commit
- Commit ID: <SHA> (if Sub-Agent committed)
- OR: Not committed, Orchestrator to commit on behalf

### Notes
- <any issues requiring attention>
```

---

## Section 8: Orchestrator Execution Templates

### 8.1 Orchestrator Main Prompt

<!-- INSTRUCTION: Use this prompt template to instruct the Orchestrator agent.
     Replace [PLACEHOLDER] values with project-specific paths and settings. -->

```markdown
# Task Orchestration Directive (Orchestrator)

## Role
You are a Task Orchestrator. You dispatch Sub-Agents to execute tasks from the
Task List in order. You do NOT write code directly — all development work is
delegated through the Agent Tool.

## Execution Context
- **Task List Path**: [path/to/task-list.md]
- **Starting Task ID**: [TASK-001]
- **Max Concurrency**: [3]

## Execution Steps

### Step 1: Read Task List
Read the Task List document. Build the task dependency graph.

### Step 2: Dispatch Loop
Repeat until all tasks are complete:

1. **Select Tasks**: Call get_next_tasks() to find executable tasks
2. **Construct Prompts**: Generate Sub-Agent Prompt for each task (see 8.2)
3. **Dispatch**: Invoke Agent Tool:
   - Has downstream dependents → foreground (wait for completion)
   - No downstream dependents → background (parallel execution)
4. **Process Results**: Parse Sub-Agent execution report
5. **Update Status**: Update Task List
6. **Git Commit**: If Sub-Agent did not commit, commit on its behalf

### Step 3: Output Summary Report

## Constraints
1. Do NOT write code or modify files (Task List updates excepted)
2. All development work is done by Sub-Agents
3. Update Task List immediately after each Sub-Agent returns
4. On FAILED tasks, analyze cause and decide whether to retry
5. Do not exceed max_concurrent parallel Sub-Agents
```

### 8.2 Sub-Agent Task Prompt Template

<!-- INSTRUCTION: Use this template to generate the prompt for each Sub-Agent.
     Fill in the bracketed sections from the Task List for each task. -->

```markdown
# Task Execution Directive (Sub-Agent)

## Task Information
- **Task ID**: [TASK-XXX]
- **Task Description**: [full description from Task List]
- **Phase**: [Phase N]

## Prior Task Summary
<!-- INSTRUCTION: If this task has completed predecessors, include their summaries.
     If no predecessors, omit this section. -->
- [TASK-YYY]: [completion summary, e.g., "Created project directory structure, Commit: abc1234"]

## Acceptance Criteria
<!-- INSTRUCTION: Copy the AC list from the Task List for this task. -->
- [ ] [AC1 description]
- [ ] [AC2 description]

## Reference Documents
<!-- INSTRUCTION: Copy reference document paths from the Task List. -->
- [path/to/reference-doc]

## Execution Requirements

### Code Changes
1. Complete development work per the task description
2. Ensure all Acceptance Criteria are satisfied
3. Keep code style consistent with existing project style

### Verification
1. Check each Acceptance Criteria item
2. Run test cases (if available)
3. Verify deliverables exist and are correct

### Git Commit
1. Stage changed files with `git add`
2. Commit with format:
   ```
   [type]([task-id]): [short description]

   - [change 1]
   - [change 2]

   Task: [TASK-XXX]
   ```
3. Record the Commit ID

## Output Requirements
After completion, output a fixed-format execution report:

```
## Sub-Agent Execution Report

### Result
- **Status**: SUCCESS / FAILED
- **Failure Reason**: (FAILED only)

### Changed Files
- <file> — <change description>

### Acceptance Criteria Verification
- [x] AC1: <description>
- [ ] AC2: <description> — <reason>

### Git Commit
- Commit ID: <SHA>

### Notes
- <any issues>
```

## Constraints
1. Complete ONLY the specified task — do not perform other tasks
2. Do NOT modify files outside the task scope
3. If unable to continue, report FAILED with reason
```

### 8.3 Parallel Dispatch Template

<!-- INSTRUCTION: Use when multiple tasks at the same dependency level can run concurrently. -->

```markdown
# Parallel Task Dispatch Directive

## Parallelizable Tasks
The following tasks have no inter-dependencies and can be dispatched simultaneously:

### Sub-Agent 1: [TASK-XXX]
- **Task Description**: [description]
- **Acceptance Criteria**: [AC list]
- **Execution Mode**: background

### Sub-Agent 2: [TASK-YYY]
- **Task Description**: [description]
- **Acceptance Criteria**: [AC list]
- **Execution Mode**: background

## Dispatch Method
Issue both Agent tool calls in a single message (parallel tool calls):

Agent Call 1:
- description: "[TASK-XXX]: [short description]"
- prompt: <full TASK-XXX Prompt>
- run_in_background: true

Agent Call 2:
- description: "[TASK-YYY]: [short description]"
- prompt: <full TASK-YYY Prompt>
- run_in_background: true

## Collect Results
Wait for all background Sub-Agents to complete, then process results one by one.
```

### 8.4 Task Recovery Prompt

<!-- INSTRUCTION: Use when resuming a previously failed or interrupted task. -->

```markdown
# Task Recovery Directive

## Execution Context
- **Task List Path**: [path/to/task-list.md]
- **Recovery Task ID**: [TASK-XXX]

## Recovery Steps

### Step 1: Assess Current State
1. Read Task List for [TASK-XXX] current status
2. Check Git working directory status
3. Analyze completed vs. pending work

### Step 2: Construct Recovery Prompt
Generate a recovery Sub-Agent Prompt including:
1. Original task description and AC
2. Work already completed (from Git diff analysis)
3. Work remaining
4. Add "resume from checkpoint" directive

### Step 3: Dispatch Recovery Sub-Agent
Invoke Agent Tool to execute the recovery task.
```

---

## Section 9: Parallel Execution Strategy

### 9.1 Dependency Graph Analysis

```
<!-- INSTRUCTION: Replace with your actual task dependency graph.
     Identify which levels can be parallelized. -->

Example dependency graph:

    TASK-001 (PENDING)
        │
        ├── TASK-002 (depends on 001)
        │       │
        │       └── TASK-004 (depends on 002, 003)
        │
        └── TASK-003 (depends on 001)

Execution levels:
  Level 0: [TASK-001]              → Sequential
  Level 1: [TASK-002, TASK-003]    → Can parallelize
  Level 2: [TASK-004]              → Sequential
```

### 9.2 Parallel Scheduling Rules

| Rule | Description |
|------|-------------|
| Same level, no deps | Tasks at the same dependency level with no mutual dependencies can be dispatched in parallel |
| Cross level | Tasks at different levels must execute sequentially |
| Max concurrency | Parallel count must not exceed `max_concurrent` |
| Failure isolation | If one task fails, pause remaining same-level dispatches |

### 9.3 Parallel Safety Constraints

| Constraint | Description | Check Method |
|-----------|-------------|--------------|
| File conflicts | Parallel tasks should not modify the same files | Declare change scope in Prompt |
| Dependency completeness | Prerequisites must be COMPLETED | Task List status check |
| Resource limits | Concurrency must not exceed max_concurrent | Orchestrator count |
| Failure isolation | One failure must not affect others | Each Sub-Agent is independent |

---

## Section 10: Exception Handling

### 10.1 Exception Types and Strategies

| Exception Type | Description | Orchestrator Response |
|---------------|-------------|----------------------|
| Dependency not met | Dependency task not complete | Mark BLOCKED, retry later |
| Sub-Agent execution failed | Sub-Agent returns FAILED | Analyze cause, decide retry/skip/abort |
| Sub-Agent timeout | Sub-Agent long-running | Wait for timeout, mark FAILED |
| AC verification failed | Output doesn't meet requirements | Re-dispatch with fix instructions |
| Git commit failed | Commit conflict or permissions | Pause, wait for manual resolution |
| Task List update failed | File lock or format error | Retry or update manually |
| Parallel task conflict | Parallel tasks modified same files | Serialize retry or use worktree |

### 10.2 Failure Handling Flow

```
Sub-Agent FAILED
       │
       ▼
Orchestrator analyzes
Sub-Agent report
       │
  ┌────┴────────┬──────────────┬──────────────┐
  │             │              │              │
  ▼             ▼              ▼              ▼
Retryable    Fix needed     Needs manual   Should cancel
  │             │              │              │
  ▼             ▼              ▼              ▼
Reset to    Fix Prompt     Pause and      Mark as
PENDING,    then re-       notify user    CANCELLED
re-dispatch dispatch       │
                              ▼
                           Manual
                           intervention

Retry limit: max_retries per task (default: 2)
On limit exceeded: mark FAILED, pause downstream, request manual intervention
```

---

## Section 11: Customization Guide

<!-- INSTRUCTION: Use this section to extend the configuration beyond
     the default template. -->

### 11.1 Sub-Agent Type Configuration

```yaml
# INSTRUCTION: Configure Sub-Agent types for specialized tasks.
sub_agent_types:
  foreground:
    use_when: "Task has downstream dependents, requires sequential execution"
    tool_params:
      description: "Task short description"
      prompt: "Full task Prompt"

  background:
    use_when: "Task has no downstream dependents, can run in parallel"
    tool_params:
      description: "Task short description"
      prompt: "Full task Prompt"
      run_in_background: true

  worktree:
    use_when: "Task may conflict with other tasks on files, needs full isolation"
    tool_params:
      description: "Task short description"
      prompt: "Full task Prompt"
      isolation: "worktree"
```

### 11.2 Custom Blocking Conditions

```yaml
blocking_conditions:
  # INSTRUCTION: Each condition needs "condition" (string) and "action" (stop|skip|ask).
  # Evaluated in order; first match wins.

  - condition: "Database migration pending"
    action: "ask"
  - condition: "SSL certificate expiring within 7 days"
    action: "stop"
  - condition: "ECS service desired count mismatch"
    action: "ask"
```

### 11.3 Environment-Specific Overrides

```yaml
# INSTRUCTION: Base configuration applies to all environments.
execution:
  mode: semi-auto
  stop_on_failure: true

environments:
  order: ["dev", "staging", "prod"]
  deploy_sequence: true
  require_approval: ["staging", "prod"]

  overrides:
    dev:
      execution:
        mode: auto
      validation:
        post_task:
          - "curl -sf http://dev-internal.[DOMAIN]/health"
    staging:
      execution:
        mode: semi-auto
      validation:
        post_task:
          - "curl -sf https://staging.[DOMAIN]/health"
    prod:
      execution:
        mode: semi-auto
        max_parallel_tasks: 1
      validation:
        post_task:
          - "curl -sf https://www.[DOMAIN]/health"
```

---

## Section 12: Troubleshooting

<!-- INSTRUCTION: Each entry includes symptom, root cause, and resolution. -->

### 12.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Tasks not executing | `execution.mode` set to `manual` | Change to `auto` or `semi-auto` |
| Status not updating | `status_tracking.file` path incorrect | Verify path is relative to project root and file exists |
| Commits not created | `auto_commit` or `auto_stage` is `false` | Set both to `true` for full auto-commit |
| Branch not found | `branch.name` does not match existing branch | Set `create_if_missing` to `true` or verify name |
| Sub-Agent timeout | Task too complex or environment issue | Break into smaller tasks; check connectivity |
| Parallel task conflicts | Shared file modifications | Use `worktree` mode or reduce `max_parallel_tasks` |
| AC verification fails | Incomplete implementation | Re-dispatch with fix instructions from failure report |
| Retry limit exceeded | Persistent failure | Investigate root cause manually before re-dispatching |
| Context pollution | Sub-Agent leaking into Orchestrator | Verify using proper Sub-Agent dispatch, not inline execution |

### 12.2 Debug Mode

```yaml
# INSTRUCTION: Disable debug mode before merging to main branch.
debug:
  enabled: true
  log_file: "debug/auto_task_debug.log"
  verbosity: "detailed"
  # INSTRUCTION: Levels: "minimal" (errors), "standard" (+warnings), "detailed" (full trace).
  include_environment_variables: false
  # INSTRUCTION: WARNING: true exposes secrets in logs. Use ONLY for local debugging.
  dry_run: false
  # INSTRUCTION: When true, reports actions without executing. For config validation.
```

---

## Appendix

### A. Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────────┐
│              Sub-Agent Task Automation Quick Reference               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Orchestrator Start                                                 │
│     1. Read Task List                                               │
│     2. Build dependency graph                                       │
│     3. Select executable tasks                                      │
│     4. Construct Sub-Agent Prompt for each task                     │
│     5. Dispatch via Agent Tool                                      │
│                                                                     │
│  Parallel Execution                                                 │
│     1. Same-level, no-dep tasks → background mode                  │
│     2. Issue multiple Agent tool calls in one message               │
│     3. Wait for all background Sub-Agents to complete               │
│     4. Process results one by one                                   │
│                                                                     │
│  Task Complete                                                      │
│     1. Parse Sub-Agent execution report                             │
│     2. Verify Acceptance Criteria                                   │
│     3. Git commit (if Sub-Agent didn't commit)                     │
│     4. Update Task List status                                      │
│     5. Select next batch of tasks                                   │
│                                                                     │
│  Task Failed                                                        │
│     1. Analyze Sub-Agent failure report                             │
│     2. Decide: retry / fix-and-retry / manual intervention         │
│     3. Max retries ≤ max_retries (default: 2)                      │
│     4. On limit exceeded, mark FAILED and pause                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### B. Commit Message Convention

```
<type>(<task-id>): <short description>

<detailed description (optional)>

Task: <TASK-ID>
```

**Types**: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`

### C. Agent Tool Parameter Reference

```yaml
# Agent Tool dispatch modes
foreground:
  description: "Task short description"
  prompt: "<full Sub-Agent task Prompt>"

background:
  description: "Task short description"
  prompt: "<full Sub-Agent task Prompt>"
  run_in_background: true

worktree:
  description: "Task short description"
  prompt: "<full Sub-Agent task Prompt>"
  isolation: "worktree"

# Optional subagent_type values
available_types:
  general-purpose: "General tasks (default)"
  python-expert: "Python development"
  frontend-architect: "Frontend development"
  quality-engineer: "Testing and quality assurance"
  security-engineer: "Security-related tasks"
  refactoring-expert: "Code refactoring"
```

---

**Version**: 2.0
<!-- INSTRUCTION: Increment on significant changes. Semantic versioning:
     Major (3.0): Breaking field changes
     Minor (2.1): New optional fields
     Patch (2.0.1): Documentation corrections -->

**Last Updated**: [DATE]
<!-- INSTRUCTION: Update this date whenever the configuration is modified. -->
