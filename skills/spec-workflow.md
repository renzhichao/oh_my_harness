---
name: spec:workflow
description: Execute complete Spec Coding workflow from initialization to execution
parameters:
  project_name:
    type: string
    required: true
    description: Project name or identifier
  work_type:
    type: string
    required: false
    description: Type of work (new_service, enhancement, bugfix, migration)
  issue_number:
    type: string
    required: false
    description: GitHub/Jira issue number
---

# Spec Coding Full Workflow

You are orchestrating a complete Spec Coding workflow for the project **{{project_name}}**.

## Workflow Phases

### Phase 1: Initialization
1. Create `docs/specs/` directory structure
2. Copy appropriate templates based on work type
3. Initialize configuration tracking

### Phase 2: Specification Development
Execute sequentially based on work type:

**Full Pipeline** (new_service, major_feature):
- GAP Analysis → Requirements → **Review** → FIP → **Review** → Task List → Auto Config

**Abbreviated Pipeline** (small_enhancement):
- Requirements → FIP → Task List → Auto Config

**Bug Analysis** (bugfix):
- Bug Analysis Report only

### Phase 3: Execution
1. Configure AUTO_TASK_CONFIG
2. Execute tasks per configuration
3. Track status continuously

## Quality Gates

At each review gate, validate:
- [ ] Technical feasibility
- [ ] Security considerations
- [ ] Operational viability
- [ ] Business alignment

## Output

Generate:
- Completed specification documents
- Review findings report
- Task execution configuration
- Progress tracking setup

Begin by asking the user for project context if not provided.
