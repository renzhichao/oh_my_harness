---
name: spec:bug
description: Conduct systematic bug analysis with root cause investigation
parameters:
  issue_number:
    type: string
    required: true
    description: Bug issue number
  issue_title:
    type: string
    required: true
    description: Bug title
  error_evidence:
    type: string
    required: false
    description: Error logs, stack traces, or evidence
---

# Bug Analysis Report Generation

You are conducting a **systematic bug analysis** for issue **#{{issue_number}}** — **{{issue_title}}**.

## Analysis Framework

### 1. Bug Description
Document:
- Current behavior (with evidence)
- Expected behavior
- Trigger conditions
- Scope and severity assessment

### 2. Root Cause Analysis
Apply systematic investigation:

**Fault Isolation**:
- Trace data flow to identify divergence point
- Locate fault in code/system
- Map failure propagation

**Root Cause Categories** (rank by probability):
- Logic Error
- Missing Implementation  
- Integration Failure
- Configuration Error
- Race Condition
- Resource Exhaustion
- Dependency Issue
- Data Corruption

**For each potential cause**:
- Provide evidence
- Explain causal chain
- Assign probability (High/Medium/Low)

### 3. Impact Assessment
Evaluate:
- Technical impact (performance, reliability, security)
- Business impact (users, operations)
- Affected metrics

### 4. Solution Analysis
Develop multiple approaches:

**Option A** (recommended):
- Description
- Implementation approach
- Pros/Cons
- Risk assessment

**Option B** (alternative):
- Description
- Implementation approach
- Pros/Cons
- Risk assessment

**Option C** (another alternative):
- Description
- Implementation approach
- Pros/Cons
- Risk assessment

### 5. Recommended Fix Strategy
- Phased implementation plan
- Rollout strategy
- Validation steps
- Rollback plan

### 6. Test Plan & Estimates
- Reproduction steps
- Verification tests
- Time estimates

## Output

Generate Bug Analysis Report following Template 09 structure:
- File: `docs/specs/BAR_{{issue_number}}_{{issue_title}}.md`
- Complete all sections
- Use severity ratings: 🔴 CRITICAL, 🟡 HIGH, 🟢 MEDIUM
- Include evidence and justification

Begin by gathering bug context from the user if error_evidence not provided.
