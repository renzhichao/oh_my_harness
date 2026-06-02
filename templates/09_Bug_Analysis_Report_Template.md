# Bug Analysis Report (BAR) — [ISSUE_NUMBER]

**Issue**: #[ISSUE_NUMBER] — [ISSUE_TITLE]
<!-- INSTRUCTION: Replace with GitHub/Jira issue number and title. Example: #614 — Epic详情页 Blueprint Run按钮点击无响应 -->

**Priority**: [Critical | High | Medium | Low]
<!-- INSTRUCTION: Assess bug severity based on impact and urgency. -->

**Status**: [OPEN | IN_PROGRESS | RESOLVED | SUPERSEDED]
<!-- INSTRUCTION: Track bug lifecycle status. -->

**Branch**: `[BRANCH_NAME]`
<!-- INSTRUCTION: Feature branch for the fix. Example: fix/614-import-and-run-module-not-found -->

**Report Date**: [DATE]
<!-- INSTRUCTION: Date when bug was reported. Example: 2026-05-12 -->

**Analysis Date**: [DATE]
<!-- INSTRUCTION: Date when analysis was completed. Example: 2026-05-12 -->

---

## Table of Contents

- [Section 1: Bug Description](#section-1-bug-description)
- [Section 2: Root Cause Analysis](#section-2-root-cause-analysis)
- [Section 3: Impact Assessment](#section-3-impact-assessment)
- [Section 4: Solution Analysis](#section-4-solution-analysis)
- [Section 5: Recommended Fix Strategy](#section-5-recommended-fix-strategy)
- [Section 6: Related Files](#section-6-related-files)
- [Section 7: Test Plan](#section-7-test-plan)
- [Section 8: Time Estimates](#section-8-time-estimates)
- [Section 9: Related Issues & References](#section-9-related-issues--references)

---

## Section 1: Bug Description

### 1.1 Current Behavior (当前行为)

<!-- INSTRUCTION: Describe what is currently happening. Include:
     - Observable symptoms
     - Error messages or stack traces
     - User-visible impact
     - Conditions that trigger the bug
     Be specific and include evidence (logs, screenshots, metrics) -->

**Symptoms**:
- [ ] Symptom 1 — [description with evidence]
- [ ] Symptom 2 — [description with evidence]
- [ ] Symptom 3 — [description with evidence]

**Trigger Conditions**:
```
[Trigger sequence or conditions]
Example:
User clicks "Run" button on Epic detail page
    → POST /dashboard/api/epics/import-and-run
    → Backend imports module
    → ModuleNotFoundError: No module named 'kevin.utils'
    → HTTP 500 error returned to user
```

**Error Evidence**:
```json
[Error log or stack trace]
Example: {"detail": "No module named 'kevin.utils'"}
```

### 1.2 Expected Behavior (期望行为)

<!-- INSTRUCTION: Describe what SHOULD happen. Include:
     - Correct behavior from user perspective
     - Expected system responses
     - Expected state changes -->

**Expected Flow**:
```
[Expected behavior sequence]
Example:
User clicks "Run" button
    → Blueprint validation succeeds
    → Planning agent initiates
    → Task breakdown completes
    → User sees "Planning started" confirmation
```

**Expected System Response**:
- [ ] Response 1 — [description]
- [ ] Response 2 — [description]

### 1.3 Scope & Severity

<!-- INSTRUCTION: Assess the scope and severity of this bug -->

**Scope**:
- **Affected Components**: [list affected system components]
- **Affected Users**: [describe user impact scope]
- **Affected Environments**: [dev/test/staging/prod]

**Severity Assessment**:
| Dimension | Rating | Rationale |
|-----------|--------|-----------|
| User Impact | [Critical/High/Medium/Low] | [impact on users] |
| Data Risk | [Critical/High/Medium/Low] | [potential data loss/corruption] |
| Service Availability | [Critical/High/Medium/Low] | [uptime impact] |
| Business Impact | [Critical/High/Medium/Low] | [business consequences] |

---

## Section 2: Root Cause Analysis (根因分析)

### 2.1 Fault Isolation

<!-- INSTRUCTION: Systematically trace the fault through system layers.
     Identify where the expected behavior diverges from actual. -->

**Data Flow Tracking**:
```
[Data flow diagram showing where the fault occurs]
Example:
GitHub labeled event (action: labeled)
    │
    ▼
githubWebhook.ts → normalizeForPlanning()
    │ adaptIssueEdited() → action !== 'edited' → return null
    │ adaptIssueCreated() → action !== 'opened' → return null
    │ normalized_events = [] → return 200 ok
    │
    ✗ labeled action 不产生任何 normalized event
    │
    （但 scratchpad 实际被创建了 — 说明存在其他触发路径）
    │
    ▼ 触发源不明（可能是手动 dispatch 或其他 webhook delivery）
```

**Fault Location**:
- **File**: [file path]
- **Function/Method**: [function name]
- **Line Numbers**: [line range]
- **Module/Component**: [component name]

### 2.2 Root Cause Categories

<!-- INSTRUCTION: Identify one or more root cause categories. Multiple categories
     may apply for complex bugs with multiple contributing factors. -->

| # | Root Cause Category | Probability | Evidence |
|---|-------------------|-------------|----------|
| 1 | [Category 1] | [High/Medium/Low] | [supporting evidence] |
| 2 | [Category 2] | [High/Medium/Low] | [supporting evidence] |
| 3 | [Category 3] | [High/Medium/Low] | [supporting evidence] |

**Common Root Cause Categories**:
- **Logic Error**: Code logic does not match requirements
- **Missing Implementation**: Required functionality not implemented
- **Integration Failure**: Component integration point failure
- **Configuration Error**: Missing or incorrect configuration
- **Race Condition**: Timing-dependent concurrent execution issue
- **Resource Exhaustion**: Insufficient resources (memory, disk, connections)
- **Dependency Issue**: Third-party library or service failure
- **Data Corruption**: Invalid or corrupted data state
- **Permission/Access**: Authorization or authentication failure

### 2.3 Detailed Root Cause Analysis

<!-- INSTRUCTION: For each identified root cause, provide detailed analysis -->

#### Root Cause #1: [Title]

**Description**: [detailed explanation of the root cause]

**Evidence**:
```yaml
Evidence_Item_1: [description]
  File: [file path]
  Line: [line number]
  Code: [code snippet]
  Behavior: [what happens]

Evidence_Item_2: [description]
  ...
```

**Why This Causes the Bug**:
```
[Explanation of the causal chain]
Example:
The slim bundle build script manually lists files to include.
kevin/utils.py is not in the list.
When the endpoint tries to import kevin.utils, the module doesn't exist.
This triggers ModuleNotFoundError at runtime.
```

**Verification Method**: [how to verify this root cause]

#### Root Cause #2: [Title] (if applicable)

<!-- INSTRUCTION: Add additional root causes as needed -->

### 2.4 Contributing Factors

<!-- INSTRUCTION: Identify any contributing factors that made the bug more
     likely or severe -->

| Factor | Description | Impact |
|--------|-------------|--------|
| [Factor 1] | [description] | [how it contributed] |
| [Factor 2] | [description] | [how it contributed] |

---

## Section 3: Impact Assessment

### 3.1 Technical Impact

**Performance Impact**:
- [ ] Latency increase: [description]
- [ ] Throughput reduction: [description]
- [ ] Resource consumption: [description]

**Reliability Impact**:
- [ ] Error rate increase: [description]
- [ ] Service availability: [description]
- [ ] Data consistency: [description]

**Security Impact**:
- [ ] Vulnerability exposure: [description]
- [ ] Compliance impact: [description]
- [ ] Authorization bypass: [description]

### 3.2 Business Impact

**User Experience**:
- [ ] Feature unavailable: [description]
- [ ] User workflow disrupted: [description]
- [ ] Data loss: [description]

**Operational Impact**:
- [ ] Increased support load: [description]
- [ ] Manual intervention required: [description]
- [ ] Increased monitoring/alerts: [description]

### 3.3 Affected Metrics

<!-- INSTRUCTION: List any metrics that would be affected by this bug -->

| Metric | Current State | Expected State | Delta |
|--------|---------------|----------------|-------|
| [Metric 1] | [value] | [value] | [impact] |
| [Metric 2] | [value] | [value] | [impact] |

---

## Section 4: Solution Analysis (解决方案分析)

### 4.1 Solution Options

<!-- INSTRUCTION: Present multiple solution approaches with pros/cons.
     Recommend one option with rationale. -->

#### Option A: [Solution Title] (Recommended)

**Description**: [detailed description of the solution]

**Implementation Approach**:
```yaml
Changes:
  - File: [file path]
    Change: [description of modification]
  - File: [file path]
    Change: [description of modification]
```

**Pros**:
- [ ] [Advantage 1]
- [ ] [Advantage 2]

**Cons**:
- [ ] [Disadvantage 1]
- [ ] [Disadvantage 2]

**Risk Assessment**:
- **Implementation Risk**: [Low/Medium/High] — [description]
- **Regression Risk**: [Low/Medium/High] — [description]
- **Deployment Risk**: [Low/Medium/High] — [description]

#### Option B: [Alternative Solution]

**Description**: [detailed description]

**Implementation Approach**:
```yaml
Changes:
  - File: [file path]
    Change: [description]
```

**Pros**:
- [ ] [Advantage 1]

**Cons**:
- [ ] [Disadvantage 1]

**Risk Assessment**:
- **Implementation Risk**: [Low/Medium/High] — [description]
- **Regression Risk**: [Low/Medium/High] — [description]

#### Option C: [Another Alternative]

<!-- INSTRUCTION: Add additional options as needed -->

### 4.2 Solution Comparison Matrix

| Criterion | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| **Implementation Complexity** | [Low/Medium/High] | [Low/Medium/High] | [Low/Medium/High] |
| **Time to Implement** | [estimate] | [estimate] | [estimate] |
| **Risk Level** | [Low/Medium/High] | [Low/Medium/High] | [Low/Medium/High] |
| **Long-term Sustainability** | [rating] | [rating] | [rating] |
| **Side Effects** | [description] | [description] | [description] |

---

## Section 5: Recommended Fix Strategy

### 5.1 Phased Implementation Plan

<!-- INSTRUCTION: For complex fixes, break down into phases -->

**Phase 1 — [Phase Name] (Priority: High)**
- **Timeline**: [duration]
- **Objective**: [goal]
- **Tasks**:
  - [ ] Task 1.1 — [description]
  - [ ] Task 1.2 — [description]

**Phase 2 — [Phase Name] (Priority: Medium)**
- **Timeline**: [duration]
- **Objective**: [goal]
- **Tasks**:
  - [ ] Task 2.1 — [description]

**Phase 3 — [Long-term Improvements] (Priority: Low)**
- **Timeline**: [duration]
- **Objective**: [goal]

### 5.2 Rollout Strategy

**Deployment Approach**:
- [ ] **Canary**: Deploy to subset of traffic/users first
- [ ] **Blue-Green**: Switch traffic between old and new versions
- [ ] **Feature Flag**: Use feature flags to enable/disable
- [ ] **Direct Cutover**: Replace old version immediately

**Validation Steps**:
1. [Validation step 1]
2. [Validation step 2]
3. [Validation step 3]

**Rollback Plan**:
- **Trigger**: [conditions that trigger rollback]
- **Procedure**: [rollback steps]
- **Validation**: [how to verify rollback succeeded]

---

## Section 6: Related Files

### 6.1 Files Requiring Changes

<!-- INSTRUCTION: List all files that need to be modified -->

| File | Change Type | Description | Estimated Lines |
|------|-------------|-------------|-----------------|
| `[file path]` | [Modify/Add/Delete] | [change description] | [LOC] |
| `[file path]` | [Modify/Add/Delete] | [change description] | [LOC] |

### 6.2 Files for Reference/Investigation

<!-- INSTRUCTION: List related files that help understand the bug -->

| File | Purpose |
|------|---------|
| `[file path]` | [why this file is relevant] |
| `[file path]` | [why this file is relevant] |

---

## Section 7: Test Plan

### 7.1 Reproduction Steps

<!-- INSTRUCTION: Provide clear steps to reproduce the bug -->

**Preconditions**:
1. [Condition 1]
2. [Condition 2]

**Steps to Reproduce**:
1. [Step 1]
2. [Step 2]
3. [Step 3]
4. [Step 4]

**Expected Result**: [what should happen]
**Actual Result**: [what actually happens]

### 7.2 Verification Tests

<!-- INSTRUCTION: Define tests to verify the fix -->

**Unit Tests**:
- [ ] Test Case 1 — [description]
  - File: [test file path]
  - Assert: [what is verified]

**Integration Tests**:
- [ ] Test Case 2 — [description]
  - Scenario: [test scenario]
  - Assert: [what is verified]

**Regression Tests**:
- [ ] Test Case 3 — [description]
  - Purpose: [ensure fix doesn't break existing functionality]

### 7.3 Manual Validation

<!-- INSTRUCTION: Manual verification steps before deployment -->

**Pre-deployment Checklist**:
- [ ] [Check 1]
- [ ] [Check 2]
- [ ] [Check 3]

**Post-deployment Validation**:
- [ ] [Validation 1]
- [ ] [Validation 2]
- [ ] [Validation 3]

---

## Section 8: Time Estimates (估时)

<!-- INSTRUCTION: Estimate effort for each phase/task -->

| Phase/Task | Estimated Time | Owner | Notes |
|-----------|----------------|-------|-------|
| **Phase 1** | | | |
| Task 1.1 | [duration] | [assignee] | [notes] |
| Task 1.2 | [duration] | [assignee] | [notes] |
| **Phase 2** | | | |
| Task 2.1 | [duration] | [assignee] | [notes] |
| **Testing** | | | |
| Unit tests | [duration] | [assignee] | |
| Integration tests | [duration] | [assignee] | |
| **Deployment** | | | |
| Pre-dep validation | [duration] | [assignee] | |
| Deployment execution | [duration] | [assignee] | |
| **Total** | **[total time]** | | |

**Risk Buffer**: [% additional time for unexpected issues]

---

## Section 9: Related Issues & References

### 9.1 Related Issues

<!-- INSTRUCTION: Link to related issues, PRs, or bugs -->

| Type | ID | Title | Relationship |
|------|----|----|-------------|
| Issue | #[number] | [title] | [how it's related] |
| PR | #[number] | [title] | [how it's related] |
| BAR | #[number] | [title] | [similar bug pattern] |

### 9.2 Commits

<!-- INSTRUCTION: Reference relevant commits -->

- **[Commit SHA]** — [commit message] — [why relevant]
- **[Commit SHA]** — [commit message] — [why relevant]

### 9.3 References

<!-- INSTRUCTION: Link to documentation, discussions, or references -->

- [Documentation/Resource 1] — [why relevant]
- [Documentation/Resource 2] — [why relevant]

### 9.4 Lessons Learned

<!-- INSTRUCTION: Capture lessons to prevent similar bugs -->

**Prevention Measures**:
- [ ] Measure 1 — [how to prevent similar bugs]
- [ ] Measure 2 — [how to prevent similar bugs]

**Process Improvements**:
- [ ] Improvement 1 — [process or tool change]
- [ ] Improvement 2 — [process or tool change]

---

## Appendix

### A. Troubleshooting Guide

| Symptom | Likely Cause | Verification Method |
|---------|--------------|---------------------|
| [Symptom 1] | [Cause] | [How to verify] |
| [Symptom 2] | [Cause] | [How to verify] |

### B. Alternative Analysis Paths

<!-- INSTRUCTION: Document alternative analysis approaches tried -->

**Attempted Path 1 — [Description]**:
- **Approach**: [what was tried]
- **Result**: [what was found]
- **Conclusion**: [why this path was abandoned or pursued]

**Attempted Path 2 — [Description]**:
- **Approach**: [what was tried]
- **Result**: [what was found]
- **Conclusion**: [why this path was abandoned or pursued]

---

**BAR Author**: [Author Name]
**Version**: [1.0]
**Last Updated**: [Date]

<!-- INSTRUCTION: Increment version for significant updates:
     Major (2.0): Complete re-analysis or new root cause
     Minor (1.1): Additional information or clarification
     Patch (1.0.1): Documentation corrections -->
