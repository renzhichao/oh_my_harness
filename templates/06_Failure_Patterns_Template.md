# Deployment Failure Patterns and Solutions - [PLATFORM/SERVICE]

<!-- ==============================================================================
     INSTRUCTION: SPEC CODING TEMPLATE - DEPLOYMENT FAILURE PATTERNS
     ==============================================================================
     This template captures deployment failure lessons for infrastructure and
     platform engineering teams deploying across multiple environments.

     HOW TO USE:
     1. Replace all [PLACEHOLDER] markers with project-specific content.
     2. Follow inline comments (<!-- INSTRUCTION: ... -->) for guidance.
     3. Run Pre-Deployment Checklist (Section 0) before each deployment.
     4. Add new patterns as discovered; update after each environment.

     NAMING CONVENTION: {platform}-deploy-failure-patterns.md
     Examples: fargate-deploy-failure-patterns.md, terraform-ec2-deploy-failure-patterns.md
     ============================================================================ -->

**Created**: [DATE]
<!-- INSTRUCTION: ISO 8601 format (YYYY-MM-DD). Example: 2025-12-18 -->

**Last Updated**: [DATE]
<!-- INSTRUCTION: Update every time a pattern is added or modified. -->

**Context**: [Task/Issue #] - [Brief description]
<!-- INSTRUCTION: Reference the task/issue that motivated this document.
     Example: Task 1843 - Unified DB Cluster Deployment -->

**Platform**: [AWS EC2/Terraform/ECS Fargate/Kubernetes/etc.]
<!-- INSTRUCTION: Deployment platform. Determines relevant failure categories. -->

**Region**: [Primary deployment region]
<!-- INSTRUCTION: AWS/cloud region. Example: me-central-1 -->

---

## Overview

<!-- INSTRUCTION: 2-4 sentences explaining why this document exists. Cover:
     what service/infrastructure is deployed, what types of failures are
     captured, and who should read it (DevOps, SRE, developers). -->

[Explain what service or infrastructure is being deployed, what types of
failures this document captures, and who should read it.]

**Key Insight**: [One-line takeaway about the most important lesson learned]
<!-- INSTRUCTION: The single most important lesson. Example: "Even with
     documented patterns, new environments encounter the same issues if
     pre-deployment validation is not performed." -->

---

## Table of Contents

1. [Pre-Deployment Checklist](#0-pre-deployment-checklist) ⭐ **START HERE**
<!-- INSTRUCTION: Section 0 must always be first, marked with ⭐. -->

2. [Category 1: [Name]](#1-category-1-name)
3. [Category 2: [Name]](#2-category-2-name)
4. [Category 3: [Name]](#3-category-3-name)
5. [Category 4: [Name]](#4-category-4-name)
<!-- INSTRUCTION: Common categories: IAM/Permissions, Network (NACL, SG, Routing),
     Container/Application, Load Balancer/Health Check, Secrets/Config,
     Infrastructure State (Terraform). Add/remove based on platform. -->

6. [Deployment Sequence & Checkpoints](#deployment-sequence--checkpoints)
7. [Quick Diagnosis Checklist](#quick-diagnosis-checklist)

---

## 0. Pre-Deployment Checklist

<!-- INSTRUCTION: The most critical section. Contains runnable bash scripts
     that validate the environment BEFORE deployment begins. Each check must:
     1. Have a clear title
     2. Include a complete, copy-pasteable bash script
     3. Show expected successful output
     4. Include remediation steps for failures

     Cover: connectivity, network/NACL/SG, images, secrets, load balancer.
     Order from fundamental (connectivity) to specific (application). -->

**⚠️ CRITICAL**: Run this checklist BEFORE deploying each environment to avoid common failures.

### 0.1 Infrastructure Validation

<!-- INSTRUCTION: Set environment variables that all subsequent checks use. -->

```bash
# Set environment variables
ENV="[environment]"  # Change to: dev, test, staging, prod
REGION="[region]"

echo "=== Pre-Deployment Checklist for $ENV environment ==="

# Verify AWS credentials and region
aws sts get-caller-identity --region $REGION --query '{Account:Account,Region:Region}'
```

**✅ Expected**: AWS account ID and region confirmed

### 0.2 Connectivity Check

<!-- INSTRUCTION: Validate that the deployment tool (SSM, SSH, kubectl)
     can reach target instances/nodes. -->

```bash
# Get all instances for the environment
INSTANCES=$([command to list instances for the environment])

echo "Instances found: $INSTANCES"

# Check connectivity for each instance
for instance in $INSTANCES; do
  STATUS=$([command to check instance connectivity])
  echo "Instance $instance Status: ${STATUS:-NOT_CONNECTED}"
done
```

**✅ Expected**: All instances should show `[Online/Ready/Connected]`

### 0.3 Network and Security Group Validation

<!-- INSTRUCTION: Most common failure category. Verify ports, NACL rules,
     and routing. Check ALL security groups involved, not just the primary
     one. A common mistake is missing that auxiliary components (ETCD,
     monitoring) use a different SG. -->

```bash
# Verify security group rules for required ports
SG_ID=$([command to find security group])
echo "Security Group: $SG_ID"

# Check required ports
REQUIRED_PORTS=([port1] [port2] [port3])
for port in "${REQUIRED_PORTS[@]}"; do
  RULE=$([command to check if port is open])
  if [ "$RULE" == "[]" ]; then
    echo "  ❌ Port $port: NOT CONFIGURED"
  else
    echo "  ✅ Port $port: Configured"
  fi
done
```

**✅ Expected**: All required ports should show `Configured`

<!-- INSTRUCTION: If components are in multiple security groups, include
     separate sub-checks for each SG explicitly. -->

### 0.4 Secrets and Configuration Check

<!-- INSTRUCTION: Verify all secrets/parameters exist to prevent mid-deployment
     crashes when a missing secret causes startup failure. -->

```bash
# Verify secrets/parameters exist
SECRET_NAME="[secret-name-pattern-${ENV}]"
[command to check secret existence] || echo "❌ Secret $SECRET_NAME NOT FOUND"
```

**✅ Expected**: All secrets/parameters should exist with valid ARNs

### 0.5 Image/Artifact Availability Check

<!-- INSTRUCTION: Verify Docker images or artifacts are accessible from the
     target environment. Reference the Verified Working Images table. -->

```bash
# Verify required images/artifacts are available
IMAGES=(
  "[image1:tag]"
  "[image2:tag]"
  "[image3:tag]"
)

for image in "${IMAGES[@]}"; do
  echo "Checking $image..."
  [command to verify image availability] && echo "  ✅ Available" || echo "  ❌ NOT AVAILABLE"
done
```

**✅ Expected**: All images/artifacts should show `Available`

### 0.6 Load Balancer/Target Group Check

<!-- INSTRUCTION: Verify target group exists. Targets may not be registered
     yet (that happens during deployment), but the group should exist. -->

```bash
# Check target group and registered targets
TG_ARN=$([command to find target group])

if [ -n "$TG_ARN" ]; then
  echo "Target Group: $TG_ARN"
  TARGETS=$([command to list registered targets])
  if [ -z "$TARGETS" ]; then
    echo "  ⚠️ No targets registered yet (expected before deployment)"
  else
    echo "  ✅ Existing targets: $TARGETS"
  fi
else
  echo "❌ No target group found for $ENV"
fi
```

**✅ Expected**: Target group exists (targets may be empty before deployment)

### 0.N Quick Fix Commands Reference

<!-- INSTRUCTION: One-liner fixes for the most common checklist failures.
     This is the cheat sheet engineers reach for during deployment.
     Keep commands copy-pasteable using variables from scripts above. -->

| Issue | Quick Fix Command |
|-------|-------------------|
| [Issue 1, e.g., Missing port] | `[exact command to fix]` |
| [Issue 2, e.g., NACL blocking] | `[exact command to fix]` |
| [Issue 3, e.g., SSM not connected] | `[exact command to fix]` |
| [Issue 4, e.g., Targets not registered] | `[exact command to fix]` |

---

## 1. [Category Name: e.g., IAM/Permissions Issues]

<!-- INSTRUCTION: Group patterns by root domain: IAM, Network, Container,
     Application, Load Balancer, etc. If the root cause involves the same
     AWS service/layer, keep in same category. Aim for 3-6 patterns per
     category. Split if exceeding 8 patterns. -->

### Problem 1.1: [Problem Title]

<!-- INSTRUCTION: Each pattern MUST follow this four-part structure:
     1. Symptom  - What the engineer sees (error message, behavior)
     2. Root Cause - WHY it happens (2-3 sentences)
     3. Solution - How to fix it (commands, code, config)
     4. Prevention - How to avoid it (actionable steps)

     This mirrors actual debugging: see error → understand why → fix now → prevent later.

     Name problems descriptively for searchability. Good: "SSM Agent Not
     Pre-installed on Amazon Linux 2023". Bad: "SSM Problem" or "Issue #1234". -->

**Symptom**:
<!-- INSTRUCTION: Copy the EXACT error message or output. Use code blocks.
     Include enough context for engineers to match by searching key phrases. -->

```
[Exact error message or output that indicates the problem]
```

**Root Cause**:
<!-- INSTRUCTION: Explain WHY in 2-4 sentences. Be specific about the mechanism.
     Example: "NACLs are stateless. If inbound rules only allow private subnet
     CIDRs, return traffic from NAT Gateway will be blocked." -->

[Explain the underlying technical reason this failure occurs.]

**Solution**:
<!-- INSTRUCTION: Exact commands or code to fix. Use bash/HCL/JSON blocks.
     Prefer the fastest fix for active incidents. Use "Option 1/2" for alternatives. -->

```bash
# [Comment explaining what this command does]
[fix command 1]

# [Comment explaining what this command does]
[fix command 2]
```

**Prevention**:
<!-- INSTRUCTION: 2-4 specific, actionable steps. Not generic advice.
     Good: "Always check AMI docs for pre-installed software"
     Bad: "Be careful with configuration" -->

- [Prevention step 1 - specific and actionable]
- [Prevention step 2 - specific and actionable]
- [Prevention step 3 - specific and actionable]

---

### Problem 1.2: [Next Problem in Category]

**Symptom**:
```
[Exact error message or observed behavior]
```

**Root Cause**:
[2-3 sentences explaining the technical mechanism behind the failure]

**Solution**:
```bash
# [Command or code to fix the issue]
[fix command]
```

<!-- INSTRUCTION: Use appropriate code block language: bash for CLI, hcl for
     Terraform, json for task definitions/IAM policies, yaml for K8s/CI. -->

OR for infrastructure code fixes:
```hcl
# Terraform fix example
resource "aws_[resource_type]" "[name]" {
  [attribute] = "[correct_value]"
}
```

**Prevention**:
- [Prevention step 1]
- [Prevention step 2]

---

## 2. [Category Name: e.g., Network Issues]

<!-- INSTRUCTION: Network issues deserve detailed coverage. Common patterns:
     NACL rule ordering, NACL blocking NAT return traffic, SG missing ports,
     SG attached to wrong instance, subnet routing, DNS failures.
     For NACL issues, always show problematic AND corrected configurations. -->

### Problem 2.1: NACL Rule Order Breaking Environment Isolation

<!-- INSTRUCTION: Complete example pattern showing expected quality level. -->

**Symptom**:
```
# Expected: Connection to another environment should fail
$ timeout 3 bash -c "echo > /dev/tcp/[other-env-ip]/[port]"
# Actual: Connection SUCCESS (isolation broken!)
```

**Root Cause**:
NACL rules are evaluated in rule number order, and the first matching rule wins. If an allow-all rule (0.0.0.0/0) is placed BEFORE deny rules for other environments, the deny rules are never evaluated for cross-environment traffic.

**Problematic Configuration**:
<!-- INSTRUCTION: Showing broken config alongside the fix lets engineers
     compare their current state and immediately recognize the issue. -->
```
Rule 50:  allow 0.0.0.0/0     ← Matches FIRST, allows ALL traffic
Rule 200: deny 10.0.20.0/24   ← Never evaluated for 10.0.20.x traffic
```

**Corrected Configuration**:
```
Rule 100: allow [own-subnet-CIDR]   (own environment)
Rule 200: deny [other-env-CIDR-1]   (evaluated BEFORE allow-all)
Rule 201: deny [other-env-CIDR-2]
Rule 500: allow 0.0.0.0/0           (NAT GW return, evaluated AFTER deny)
Rule 32767: deny 0.0.0.0/0          (default deny)
```

**Solution**:
```bash
# Delete the low-numbered allow rule
aws ec2 delete-network-acl-entry \
  --network-acl-id [NACL_ID] \
  --rule-number 50 \
  --ingress

# Add it after deny rules (e.g., rule 500)
aws ec2 create-network-acl-entry \
  --network-acl-id [NACL_ID] \
  --rule-number 500 \
  --protocol -1 \
  --rule-action allow \
  --ingress \
  --cidr-block 0.0.0.0/0
```

**Prevention**:
- Always use rule numbers > 400 for the allow 0.0.0.0/0 rule
- Test environment isolation after any NACL changes
- Include isolation tests in deployment validation checklist

---

### Problem 2.2: [Additional Network Pattern]

<!-- INSTRUCTION: Follow Symptom → Root Cause → Solution → Prevention. -->

[Pattern content following the standard structure]

---

## 3. [Category Name: e.g., Container/Application Issues]

<!-- INSTRUCTION: Show EXACT error messages for searchability. Focus on:
     image pull failures, OOMKilled/resource limits, startup crashes,
     missing env vars, volume mount issues, health check config. -->

### Problem 3.1: [Container/Application Problem]

**Symptom**:
```
[Exact error from container runtime, orchestrator, or application logs]
```

**Root Cause**:
[Explain why the container or application fails]

**Solution**:
```bash
# [Fix commands]
```

**Prevention**:
- [Prevention step 1]
- [Prevention step 2]

---

## 4. [Category Name: e.g., Load Balancer/Health Check Issues]

<!-- INSTRUCTION: Often the final deployment hurdle. Service may be running
     correctly internally but unreachable from outside. Common patterns:
     health check failures (wrong port/path), targets not registered,
     SGs blocking health check traffic, listener misconfiguration. -->

### Problem 4.1: [Load Balancer/Health Check Problem]

**Symptom**:
```
[Error showing unhealthy targets or failed health checks]
```

**Root Cause**:
[Explain why health checks fail or targets are unreachable]

**Solution**:
```bash
# [Fix commands]
```

**Prevention**:
- [Prevention step 1]
- [Prevention step 2]

---

## Deployment Sequence & Checkpoints

<!-- INSTRUCTION: Recommended deployment order with explicit checkpoints.
     Many failures occur when phases are executed out of order. Each phase
     needs: 1) Clear name, 2) Specific actions, 3) CHECKPOINT with
     verification command, 4) Expected results.

     The checkpoint is a STOP point. Do not proceed until it passes.
     This prevents cascading failures across phases.

     Phase structures by platform:
     EC2/VM: Pre-validation → Foundation services → Primary app → Secondary → LB
     Container: Pre-validation → Infrastructure → IAM → Container → LB -->

### Recommended Deployment Order

```
Phase 1: Pre-Deployment Validation (Section 0)
├── ✅ 0.1 Infrastructure Validation
├── ✅ 0.2 Connectivity Check
├── ✅ 0.3 Network and Security Group Validation
├── ✅ 0.4 Secrets and Configuration Check
└── ✅ 0.5 Image/Artifact Availability Check

Phase 2: [Component 1] Deployment
├── Deploy [component name]
├── ⏸️ CHECKPOINT: Verify [component] health
│   └── [verification command]
└── Expected: [what success looks like]

Phase 3: [Component 2] Deployment
├── Deploy [component name]
├── ⏸️ CHECKPOINT: Verify [component] health
│   └── [verification command]
└── Expected: [what success looks like]

Phase 4: [Component N] Deployment
├── Deploy [component name]
├── Register targets with load balancer
├── ⏸️ CHECKPOINT: Verify load balancer health
│   └── [verification command]
└── Expected: All targets showing healthy

Phase 5: Post-Deployment Validation
├── Test connection via [load balancer/service endpoint]
├── Verify logs streaming to [CloudWatch/logging system]
└── Document deployment results in [tracking system]
```

<!-- INSTRUCTION: ASCII tree characters: ├── branch, └── last item,
     ⏸️ checkpoint, ✅ must-pass. Keep readable; group if >6 items per phase. -->

### Failure Recovery Flowchart

<!-- INSTRUCTION: Decision tree for top 3-5 failure scenarios. Format:
     [Symptom]?
     ├── Check [diagnostic] → Fix: [solution]
     ├── Check [diagnostic] → Fix: [solution]
     └── [Last resort action]
     Focus on most frequent or hardest-to-diagnose failures. -->

```
[Primary service] not connecting?
├── Check [diagnostic 1, e.g., NACL rules] → Fix: [command]
├── Check [diagnostic 2, e.g., security group ports] → Fix: [command]
└── [Escalation: e.g., reboot instances and re-check]

[Component A] cluster not forming?
├── Check [diagnostic 1] → Fix: [solution]
├── Check [diagnostic 2] → Fix: [solution]
└── [Last resort: e.g., redeploy with fresh data volumes]

Health checks failing?
├── Check [service is running] → Fix: [restart command]
├── Check [port is accessible] → Fix: [security group command]
└── [Manual registration or restart]
```

---

## Quick Diagnosis Checklist

<!-- INSTRUCTION: Linear step-by-step diagnostic for when the engineer
     does not know which pattern applies. This is the "first responder" guide.
     Order from most common/easiest to least common/most invasive.
     Each step: 1 diagnostic command + what to look for. -->

When [common failure, e.g., "SSM connection fails"] or deployment does not work, check in this order:

### Step 1: [First Diagnostic - Most Common Check]
<!-- INSTRUCTION: Highest-probability, easiest check. EC2: instance status.
     Container: task/container status. -->

```bash
[diagnostic command to check the most likely cause]
```

### Step 2: [Second Diagnostic - IAM/Permissions]
```bash
[diagnostic command for IAM or permissions check]
```

### Step 3: [Third Diagnostic - Network]
```bash
[diagnostic command for network/connectivity check]
```

### Step 4: [Fourth Diagnostic - Logs]
```bash
[command to retrieve application or system logs]
```

### Step 5: [Last Resort]
<!-- INSTRUCTION: Invasive action for stubborn issues: reboot, recreate,
     force-unlock. Always note required wait time. -->

```bash
[last resort command, e.g., reboot instances]
# Wait [X] minutes for [service] to reinitialize
```

---

## Summary of Key Lessons

<!-- INSTRUCTION: Quick-scan reference mapping each issue to its fix.
     Use same language as pattern titles for easy lookup.
     Bold the most frequently encountered issues. -->

| Issue | Quick Fix |
|-------|-----------|
| [Issue from Pattern 1.1] | [One-line fix summary] |
| [Issue from Pattern 1.2] | [One-line fix summary] |
| [Issue from Pattern 2.1] | [One-line fix summary] |
| [Issue from Pattern 3.1] | [One-line fix summary] |
| [Issue from Pattern 4.1] | [One-line fix summary] |

---

## Verified Working Images/Versions

<!-- INSTRUCTION: Document EXACT versions verified to work. Include a
     "DO NOT USE" list for images that look correct but fail. This prevents
     different team members independently discovering the same broken images. -->

| Component | Image/Version | Verified Date | Notes |
|-----------|-------------|---------------|-------|
| [Component 1] | `[exact image:tag]` | [Date] | [Caveats] |
| [Component 2] | `[exact image:tag]` | [Date] | [Caveats] |
| [Component 3] | `[exact image:tag]` | [Date] | [Caveats] |

**⚠️ DO NOT USE**:
- `[image:tag]` - [Reason: does not exist, broken, incompatible]
- `[image:tag]` - [Reason]

---

## Environment Deployment Summary

<!-- INSTRUCTION: Track deployment status across environments. Update after
     each deployment. Use ✅ Completed, ⏳ Pending, or ❌ Blocked. -->

| Environment | [Key Config] | [Special Note] | Deployed |
|-------------|-------------|----------------|----------|
| Dev | [Value] | [Note] | ✅/⏳/❌ |
| Test | [Value] | [Note] | ✅/⏳/❌ |
| Staging | [Value] | [Note] | ✅/⏳/❌ |
| Prod | [Value] | [Note] | ✅/⏳/❌ |

---

## Related Documentation

<!-- INSTRUCTION: External docs, runbooks, and guides relevant to the
     failures documented here. Include AWS/cloud docs and internal links. -->

- [Link to relevant AWS/cloud provider documentation]
- [Link to internal runbook or operations guide]
- [Link to related architecture documentation]

---

## Document History

<!-- INSTRUCTION: Chronological change log. Helps readers know when patterns
     were discovered and if the document covers their deployment scenario.
     Use bold for new sections and important updates. -->

| Date | Changes |
|------|---------|
| [Date] | Initial document with [environment] deployment lessons |
| [Date] | Added [category] issues from [environment] deployment |
| [Date] | **[Environment] Update**: [summary of new patterns] |

---

*Document created based on [Task/Issue reference] deployment experience*

<!-- ==============================================================================
     INSTRUCTION: MAINTENANCE GUIDELINES
     ==============================================================================
     1. UPDATE FREQUENCY: After every environment deployment. Add new patterns,
        update solutions if they change, update Environment Deployment Summary.
     2. PATTERN QUALITY: Exact error (searchable) → Clear cause (educational)
        → Copy-pasteable fix (actionable) → Prevention (preventive).
     3. CHECKLIST ACCURACY: Test Section 0 before each deployment. If a check
        fails unexpectedly, update the check or add a new pattern.
     4. VERSION TRACKING: Update Verified Working Images when versions change.
     5. KNOWLEDGE SHARING: Reference in deployment PRs, onboarding, post-mortems.
     ============================================================================ -->
