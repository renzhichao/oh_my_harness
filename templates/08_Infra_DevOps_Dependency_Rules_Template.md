# Infrastructure & DevOps Dependency Rules - [PROJECT_TITLE]

<!-- ==============================================================================
     INSTRUCTION: SPEC CODING TEMPLATE - INFRASTRUCTURE & DEVOPS DEPENDENCY RULES
     ==============================================================================
     This template defines dependency rules for infrastructure and platform
     engineering projects. It covers environment promotion, component deployment
     ordering, cross-service dependencies, deployment blockers, naming/tagging
     conventions, validation checklists, and common operational patterns.

     HOW TO USE THIS TEMPLATE:
     1. Replace all [PLACEHOLDER] markers with project-specific content.
     2. Remove or adapt sections that do not apply to your project.
     3. Follow the inline comments (<!-- INSTRUCTION: ... -->) for guidance.
     4. Maintain the severity rating system consistently:
        - 🔴 HARD BLOCKER  = Deployment cannot proceed until resolved
        - 🟡 SOFT BLOCKER  = Deployment can proceed with documented exception
        - 🟢 ADVISORY      = Best practice; address in next iteration
     5. Use ASCII diagrams as-is where applicable; redraw only when topology
        changes require it.
     6. Keep the environment table, dependency matrix, and blocker tables in
        sync with the actual Terraform modules and CI/CD pipelines.
     ============================================================================ -->

**Document ID**: DEPS_[PROJECT_NAME]
<!-- INSTRUCTION: Use a short, uppercase, underscore-separated identifier.
     Example: DEPS_ALB_Fargate_java_service, DEPS_2132_postgresql_nlb_routing -->

**Issue**: #[ISSUE_NUMBER] - [Issue Title]
<!-- INSTRUCTION: Reference the GitHub/Jira issue number and its title.
     Example: #2132 - Dev PostgreSQL NLB Routing Fix -->

**Created**: [DATE]
<!-- INSTRUCTION: Use the date this document was first created.
     Example: 2025-03-20 -->

**Branch**: [BRANCH_NAME]
<!-- INSTRUCTION: The feature branch where implementation work will occur.
     Example: bugfix/2132-dev-postgresql-nlb-routing-fix -->

**Status**: Draft
<!-- INSTRUCTION: Track document lifecycle:
     - "Draft" (initial authoring)
     - "Under Review" (peer review)
     - "Approved" (ready for implementation)
     - "Implemented" (rules enforced in pipelines)
     - "Deprecated" (superseded by newer version) -->

---

## Table of Contents

- [Section 1: Overview](#section-1-overview)
- [Section 2: Environment Dependency Rules](#section-2-environment-dependency-rules)
- [Section 3: Infrastructure Component Dependencies](#section-3-infrastructure-component-dependencies)
- [Section 4: Cross-Service Dependency Rules](#section-4-cross-service-dependency-rules)
- [Section 5: Deployment Blocking Rules](#section-5-deployment-blocking-rules)
- [Section 6: Naming & Tagging Dependencies](#section-6-naming--tagging-dependencies)
- [Section 7: Validation Checklists](#section-7-validation-checklists)
- [Section 8: Common Patterns](#section-8-common-patterns)

---

## Section 1: Overview

### Purpose

This document defines the dependency rules governing infrastructure provisioning,
service deployment, and environment promotion for [PROJECT_NAME]. Dependency
rules ensure that resources are created in the correct order, services communicate
through approved paths, and deployments are blocked when prerequisite conditions
are not met.

<!-- INSTRUCTION: Tailor the purpose statement to the specific project scope.
     If the project spans multiple teams, mention the teams involved.
     Example: "governing infrastructure provisioning for the Java Fargate
     platform across the backend and platform engineering teams." -->

### Scope

| Aspect | Coverage |
|--------|----------|
| **Project** | [PROJECT_NAME] |
| **Teams** | [TEAM_1], [TEAM_2] |
| **Environments** | dev, test, staging, prod |
| **IaC Tool** | [Terraform / CloudFormation / Pulumi] |
| **CI/CD Platform** | [GitHub Actions / GitLab CI / Jenkins] |
| **Cloud Provider** | [AWS / Azure / GCP] |

<!-- INSTRUCTION: Fill in every row. Add or remove rows to match the
     technology stack. If the project uses multiple cloud providers,
     list them all and note primary vs secondary. -->

### Related Documents

| Document | ID | Relationship |
|----------|----|--------------|
| [GAP Analysis] | GAP_[PROJECT_NAME] | Baseline and gap identification |
| [Requirements Spec] | REQ_[PROJECT_NAME] | Functional and non-functional requirements |
| [Architecture Design] | ADR_[PROJECT_NAME] | Architecture decision records |

<!-- INSTRUCTION: Link to other spec-coding-template documents that
     relate to this project. Remove the table if no related documents exist. -->

---

## Section 2: Environment Dependency Rules

### 2.1 Environment Promotion Order

Infrastructure changes must follow the promotion path below. Skipping
environments is prohibited unless an explicit exception is approved by
[ROLE_NAME].

<!-- INSTRUCTION: Adjust the promotion path to match the actual
     environment pipeline. Add or remove environments as needed.
     Some projects include a "sandbox" or "qa" environment. -->

```
                     +-------+
                     |  dev  |   <-- All changes start here
                     +---+---+
                         |
                         v
                     +-------+
                     | test  |   <-- Automated integration tests
                     +---+---+
                         |
                         v
                     +---------+
                     | staging |   <-- Pre-production parity check
                     +----+----+
                          |
                          v
                     +-------+
                     | prod  |   <-- Production release
                     +-------+
```

**Promotion Rules:**

1. No change may be promoted to `staging` without a passing test run in `test`.
2. No change may be promoted to `prod` without a successful deployment to `staging`.
3. Hotfixes follow the path `dev -> staging -> prod` but require a PIR (Post-Incident Review) within [SLA_HOURS] hours.
<!-- INSTRUCTION: Adjust hotfix rules. Some organizations allow
     hotfixes to go directly to staging, others require a full path.
     Specify the SLA for post-incident review. -->

### 2.2 Environment Isolation Rules

<!-- INSTRUCTION: These rules define how environments must remain
     isolated from each other. Add rules specific to your
     organization's compliance requirements. -->

1. **Network Isolation**: Each environment must reside in its own VPC. No VPC peering between non-production and production environments.
2. **IAM Boundary**: IAM roles must include environment-scoped boundaries. A role created for `dev` must not be assumable in `prod`.
3. **State Isolation**: Terraform state files must be stored in environment-specific backends: `[TF_STATE_BUCKET]/[PROJECT]/[ENVIRONMENT]/terraform.tfstate`.
4. **Secret Isolation**: Secrets must be stored in environment-specific vaults or parameter store paths. No shared secrets across environments.
5. **DNS Isolation**: Each environment uses a dedicated hosted zone or subdomain: `[ENV].[DOMAIN]`.
6. **Logging Isolation**: CloudWatch log groups and S3 access logs must be scoped per environment. Centralized logging is permitted only via read-only cross-account roles.

### 2.3 Per-Environment Configuration

| Env | VPC CIDR | Subnets | ACM Certificate | Cluster Name | State Bucket |
|-----|----------|---------|-----------------|--------------|--------------|
| dev | [DEV_CIDR] | [DEV_SUBNETS] | [DEV_CERT_ARN] | [DEV_CLUSTER] | [DEV_STATE_BUCKET] |
| test | [TEST_CIDR] | [TEST_SUBNETS] | [TEST_CERT_ARN] | [TEST_CLUSTER] | [TEST_STATE_BUCKET] |
| staging | [STAGING_CIDR] | [STAGING_SUBNETS] | [STAGING_CERT_ARN] | [STAGING_CLUSTER] | [STAGING_STATE_BUCKET] |
| prod | [PROD_CIDR] | [PROD_SUBNETS] | [PROD_CERT_ARN] | [PROD_CLUSTER] | [PROD_STATE_BUCKET] |

<!-- INSTRUCTION: Fill in all values. Use exact ARNs, CIDR blocks,
     and bucket names from the Terraform variables. If subnets are
     split by tier (public/private/database), use comma-separated
     lists or reference a separate subnet table. -->

---

## Section 3: Infrastructure Component Dependencies

### 3.1 Dependency Graph

The following diagram illustrates the order in which infrastructure components
must be created or updated. Components at the top must exist before components
below them can be provisioned.

<!-- INSTRUCTION: Redraw this graph to match the actual infrastructure
     topology. The example below shows a common AWS ECS Fargate setup.
     Adjust node names and arrows as needed for your architecture. -->

```
Phase 1 (Foundation)         Phase 2 (Compute)          Phase 3 (Routing)
+-----------------+          +-----------------+        +------------------+
|      VPC        |--------->|   ECS Cluster   |------->|   ALB / NLB      |
+-----------------+          +-----------------+        +------------------+
         |                            |                          |
         v                            v                          v
+-----------------+          +-----------------+        +------------------+
| Security Groups |--------->|  Fargate Service|------->| API Gateway      |
+-----------------+          +-----------------+        +------------------+
         |                            |                          |
         v                            v                          v
+-----------------+          +-----------------+        +------------------+
|   IAM Roles     |--------->|   ECR Repo      |------->| VPC Link / Route |
+-----------------+          +-----------------+        +------------------+
```

### 3.2 Component Dependency Matrix

| Component | Depends On | Blocks | TF Module Path |
|-----------|-----------|--------|----------------|
| VPC | N/A | Security Groups, Subnets, NAT GW | `[TF_MODULE_PATH]/vpc` |
| Security Groups | VPC | ALB, ECS Service, RDS | `[TF_MODULE_PATH]/security-groups` |
| IAM Roles | N/A | ECS Task Execution, CI/CD | `[TF_MODULE_PATH]/iam` |
| ECS Cluster | VPC, Security Groups, IAM Roles | Fargate Service | `[TF_MODULE_PATH]/ecs-cluster` |
| ALB / NLB | VPC, Security Groups | Target Groups, Listener Rules | `[TF_MODULE_PATH]/alb` |
| Target Groups | ALB / NLB | Listener Rules | `[TF_MODULE_PATH]/target-groups` |
| Fargate Service | ECS Cluster, ALB, IAM Roles | API Gateway VPC Link | `[TF_MODULE_PATH]/fargate` |
| ECR Repository | IAM Roles | Fargate Service (image pull) | `[TF_MODULE_PATH]/ecr` |
| API Gateway | VPC Link | Routes, Integrations | `[TF_MODULE_PATH]/api-gateway` |
| VPC Link | NLB, VPC | API Gateway Private Integration | `[TF_MODULE_PATH]/vpc-link` |
| ACM Certificate | Route53 (DNS validation) | ALB HTTPS Listener | `[TF_MODULE_PATH]/acm` |
| Route53 Records | ALB / NLB / API Gateway | DNS Resolution | `[TF_MODULE_PATH]/route53` |

<!-- INSTRUCTION: This table must stay in sync with the Terraform
     module structure. Add or remove rows to match the actual
     components. The "Blocks" column lists downstream components
     that cannot be created until this component is ready. -->

### 3.3 Terraform Module Dependencies

<!-- INSTRUCTION: Replace the tree below with the actual module
     structure from the project's Terraform codebase. Use the
     directory layout as the source of truth. -->

```
[PROJECT_ROOT]/
+-- environments/
|   +-- dev/
|   |   +-- main.tf          <-- Root module (dev)
|   |   +-- variables.tf
|   |   +-- terraform.tfvars
|   +-- staging/
|   +-- prod/
+-- modules/
|   +-- vpc/                  <-- Phase 1
|   +-- security-groups/      <-- Phase 1
|   +-- iam/                  <-- Phase 1
|   +-- ecs-cluster/          <-- Phase 2
|   +-- alb/                  <-- Phase 2
|   +-- fargate-service/      <-- Phase 2
|   +-- ecr/                  <-- Phase 2
|   +-- api-gateway/          <-- Phase 3
|   +-- vpc-link/             <-- Phase 3
|   +-- route53/              <-- Phase 3
|   +-- acm/                  <-- Phase 3
+-- shared/
    +-- backend.tf
    +-- providers.tf
```

### 3.4 Resource Creation Order

<!-- INSTRUCTION: The phases below must match the dependency graph
     in Section 3.1. Each phase lists resources that can be created
     in parallel within that phase. -->

**Phase 1 -- Foundation (no dependencies)**

1. VPC, subnets, route tables, internet gateway, NAT gateway
2. Security groups (ALB, ECS, database)
3. IAM roles and policies (task execution, task role, CI/CD)
4. ACM certificate (DNS-validated)
5. ECR repository

**Phase 2 -- Compute Layer (depends on Phase 1)**

6. ECS cluster
7. CloudWatch log groups
8. ALB / NLB with target groups
9. Fargate service definition and task definition
10. ALB listener rules and health checks

**Phase 3 -- Routing and DNS (depends on Phase 2)**

11. VPC Link (API Gateway private integration)
12. API Gateway REST API, resources, methods
13. API Gateway VPC link integration
14. Route53 DNS records (alias to ALB / API Gateway)

**Phase 4 -- Validation (depends on Phase 3)**

15. End-to-end health check validation
16. DNS resolution verification
17. TLS certificate validation
18. Smoke test suite execution

---

## Section 4: Cross-Service Dependency Rules

### 4.1 Shared Infrastructure Rules

<!-- INSTRUCTION: Shared infrastructure refers to resources that
     multiple services depend on, such as VPCs, logging buckets,
     or KMS keys. Define ownership and change-approval rules. -->

| Shared Resource | Owner | Change Approval | Impact Scope |
|-----------------|-------|-----------------|--------------|
| VPC | [TEAM_OWNER] | [APPROVAL_PROCESS] | All services in environment |
| KMS Key | [TEAM_OWNER] | [APPROVAL_PROCESS] | Encryption-at-rest for all services |
| CloudWatch Log Groups | [TEAM_OWNER] | [APPROVAL_PROCESS] | Logging for all services |
| ECR Registry | [TEAM_OWNER] | [APPROVAL_PROCESS] | Image storage for all container services |
| Route53 Hosted Zone | [TEAM_OWNER] | [APPROVAL_PROCESS] | DNS for all services |

**Rules:**

1. Changes to shared infrastructure require a minimum of [APPROVER_COUNT] approvals.
2. Shared resource changes must be deployed during the designated maintenance window: [MAINTENANCE_WINDOW].
3. Rollback plans must be documented before any shared resource change is applied.
<!-- INSTRUCTION: Specify the number of required approvals and the
     maintenance window. If the organization does not have a fixed
     maintenance window, state "Change freeze periods only" or
     "Any time with approval." -->

### 4.2 Service-to-Service Communication Rules

```
                   +-------------------+
                   |   API Gateway     |
                   +--------+----------+
                            |
             +--------------+--------------+
             |                             |
     +-------+--------+          +---------+------+
     | Service A      |          | Service B      |
     | (Fargate)      |          | (Fargate)      |
     +-------+--------+          +---------+------+
             |                             |
             v                             v
     +-------+--------+          +---------+------+
     | Database A     |          | Database B     |
     +----------------+          +----------------+
```

<!-- INSTRUCTION: Redraw to match the actual service communication
     topology. If services communicate via EventBridge, SQS, or
     SNS, include those components in the diagram. -->

| Rule ID | Description | Enforcement |
|---------|-------------|-------------|
| COM-001 | Services must communicate via API Gateway or internal ALB. Direct task-to-task communication is prohibited. | Security Group rules |
| COM-002 | Service mesh communication must use mTLS. | [SERVICE_MESH_CONFIG] |
| COM-003 | No service may connect to another service's database directly. | IAM policy boundary |
| COM-004 | Cross-account communication requires VPC peering or PrivateLink with explicit approval. | Network ACL |

### 4.3 Database Access Rules

| Service | Database | Access Level | Connection Method | TF Module |
|---------|----------|--------------|-------------------|-----------|
| [SERVICE_A] | [DB_A_NAME] | Read-Write | ECS Task IAM Auth | `[TF_MODULE_PATH]/db-a` |
| [SERVICE_B] | [DB_B_NAME] | Read-Only | ECS Task IAM Auth | `[TF_MODULE_PATH]/db-b` |
| [SERVICE_C] | [DB_A_NAME] | Read-Only | Secret-based Auth | `[TF_MODULE_PATH]/db-a-ro` |

<!-- INSTRUCTION: Every service that accesses a database must be
     listed. If a service does not access any database, omit it
     from this table. Specify the authentication method used
     (IAM auth, secret-based, etc.). -->

**Rules:**

1. Each service must use its own IAM role or secret for database access. Shared credentials are prohibited.
2. Read-only access must be the default. Write access requires explicit justification in the Terraform module.
3. Database credentials must rotate every [ROTATION_DAYS] days via Secrets Manager rotation.
4. No service may have superuser or administrative access to any database in `staging` or `prod`.

---

## Section 5: Deployment Blocking Rules

### 5.1 Hard Blockers

<!-- INSTRUCTION: Hard blockers prevent deployment entirely. They
     represent conditions where proceeding would cause data loss,
     security exposure, or service outage. -->

| Blocker ID | Condition | Detection | Resolution |
|------------|-----------|-----------|------------|
| HB-001 | Terraform plan shows resource destruction of production data stores | `terraform plan` output | Manually review plan; add `lifecycle prevent_destroy` |
| HB-002 | Security group allows 0.0.0.0/0 ingress on non-HTTP ports | `tfsec` / `checkov` scan | Restrict CIDR to known ranges |
| HB-003 | IAM policy grants `*:*` on `*` resource | `tfsec` / IAM Access Analyzer | Scope policy to least privilege |
| HB-004 | ACM certificate is expired or pending validation | `aws acm describe-certificate` | Re-issue or re-validate certificate |
| HB-005 | ECS task definition uses privileged container in prod | `checkov` scan | Remove `privileged = true` |
| HB-006 | Database migration contains irreversible data loss (DROP TABLE) | Migration review script | Add reversible migration; peer review |
| HB-007 | [CUSTOM_BLOCKER] | [DETECTION_METHOD] | [RESOLUTION_STEPS] |

<!-- INSTRUCTION: Add project-specific hard blockers. Keep the
     detection method concrete (tool name, command, or check).
     The resolution must be an actionable step, not a vague
     instruction. -->

### 5.2 Soft Blockers

<!-- INSTRUCTION: Soft blockers generate warnings but do not stop
     deployment. They must be acknowledged and tracked for
     remediation. -->

| Blocker ID | Condition | Detection | Resolution | Default Action |
|------------|-----------|-----------|------------|----------------|
| SB-001 | Terraform plan shows > [THRESHOLD] resource changes | `terraform plan` summary | Review and confirm | Warn + require ack |
| SB-002 | Container image tag is `latest` instead of SHA or version | CI pipeline check | Pin image to digest or version tag | Warn |
| SB-003 | No smoke tests defined for new resource | Test framework | Add smoke test to pipeline | Warn |
| SB-004 | Resource tagging is incomplete (missing required tags) | `tfsec` / custom script | Add missing tags | Warn + ticket |
| SB-005 | Log retention period is not set or exceeds 365 days | `checkov` scan | Set appropriate retention | Warn |
| SB-006 | [CUSTOM_SOFT_BLOCKER] | [DETECTION_METHOD] | [RESOLUTION_STEPS] | [DEFAULT_ACTION] |

### 5.3 Environment-Specific Blockers

| Environment | Additional Blockers | Rationale |
|-------------|---------------------|-----------|
| dev | None | Development environment allows faster iteration |
| test | All hard blockers + SB-001, SB-002 | Baseline quality gate |
| staging | All hard blockers + all soft blockers | Must mirror production readiness |
| prod | All hard blockers + all soft blockers + manual approval from [ROLE_NAME] | Maximum safety for production |

<!-- INSTRUCTION: Adjust the blocker escalation per environment.
     Some organizations require change advisory board (CAB)
     approval for production. Document that requirement here
     if applicable. -->

---

## Section 6: Naming & Tagging Dependencies

### 6.1 Resource Naming Dependencies

<!-- INSTRUCTION: Resource names often encode environment, service,
     and region. If a downstream resource derives its name from an
     upstream resource, document that dependency here. -->

| Resource Type | Naming Pattern | Depends On | Example |
|---------------|---------------|------------|---------|
| VPC | `[project]-[env]-vpc` | Project name, Environment | `myapp-dev-vpc` |
| Subnet | `[project]-[env]-[tier]-subnet-[az]` | VPC name | `myapp-dev-private-subnet-a` |
| Security Group | `[project]-[env]-[service]-sg` | VPC, Service name | `myapp-dev-api-sg` |
| IAM Role | `[project]-[env]-[service]-[purpose]-role` | Service name | `myapp-dev-api-exec-role` |
| ECS Cluster | `[project]-[env]-cluster` | Project name, Environment | `myapp-dev-cluster` |
| ALB | `[project]-[env]-[service]-alb` | VPC, Security Group | `myapp-dev-api-alb` |
| Target Group | `[project]-[env]-[service]-tg` | ALB, VPC | `myapp-dev-api-tg` |
| ECR Repo | `[project]/[service]` | Service name | `myapp/api-service` |
| S3 Bucket | `[project]-[env]-[purpose]-[account-id]` | Account ID | `myapp-dev-logs-123456789` |
| CloudWatch Log Group | `/aws/ecs/[project]-[env]-[service]` | ECS Cluster | `/aws/ecs/myapp-dev-api` |

**Rules:**

1. Resource names must not exceed [AWS_NAMING_LIMIT] characters.
2. Names must use lowercase letters, numbers, and hyphens only. Underscores are permitted in IAM role names.
3. Hardcoded resource names are prohibited. All names must be derived from variables in Terraform.

### 6.2 Required Tags per Resource Type

| Tag Key | Required On | Format | Example | Enforced By |
|---------|-------------|--------|---------|-------------|
| `Environment` | All resources | `dev`, `test`, `staging`, `prod` | `prod` | `tfsec` |
| `Project` | All resources | Project short name | `myapp` | `tfsec` |
| `Service` | Compute, DB, ALB | Service short name | `api-service` | `tfsec` |
| `Owner` | All resources | Team or individual email | `platform@example.com` | `tfsec` |
| `CostCenter` | All resources | Cost center code | `CC-1234` | `tfsec` |
| `ManagedBy` | All resources | `terraform` | `terraform` | `tfsec` |
| `Repository` | All resources | Repository URL | `github.com/org/repo` | `tfsec` |
| `DataClassification` | Storage, DB | `public`, `internal`, `confidential` | `confidential` | `checkov` |

<!-- INSTRUCTION: Extend this table with organization-specific
     tags. If using AWS Tag Policies or Service Control Policies
     for enforcement, note that in the "Enforced By" column. -->

### 6.3 Tag-Based Access Control Rules

| Rule | Tag Condition | Effect | Scope |
|------|--------------|--------|-------|
| TBA-001 | `Environment = prod` AND caller not in [PROD_ACCESS_ROLE] | Deny modify actions | All prod resources |
| TBA-002 | `DataClassification = confidential` AND action is `s3:GetObject` | Allow only via approved roles | S3 buckets |
| TBA-003 | `ManagedBy != terraform` | Deny all actions (manual changes) | All resources |
| TBA-004 | `Environment` tag missing | Deny resource creation | All resources |

<!-- INSTRUCTION: Map these rules to actual IAM policies or
     Service Control Policies. If the organization uses ABAC
     (Attribute-Based Access Control), document the tag schema
     and policy bindings here. -->

---

## Section 7: Validation Checklists

### 7.1 Pre-Deployment Checklist

<!-- INSTRUCTION: Every checkbox must be verified before a
     deployment is initiated. Add project-specific checks.
     These checks should be automatable in CI/CD where possible. -->

- [ ] Terraform `plan` produces no errors and has been reviewed by [MIN_REVIEWERS] team members
- [ ] All hard blockers (Section 5.1) are resolved or documented with exception
- [ ] All soft blockers (Section 5.2) are acknowledged
- [ ] Container image is built, scanned, and pushed to ECR with a versioned tag (not `latest`)
- [ ] Database migration scripts have been tested in a lower environment
- [ ] Security group rules have been reviewed; no overly permissive ingress rules
- [ ] IAM policies follow least privilege; no `*:*` on `*` resource
- [ ] ACM certificate is issued and validated for the target domain
- [ ] Route53 DNS records are prepared (create or update)
- [ ] Rollback plan is documented and tested
- [ ] Deployment is scheduled within the approved maintenance window
- [ ] Stakeholders have been notified of the deployment window

### 7.2 Post-Deployment Validation

- [ ] Terraform `apply` completed without errors
- [ ] ECS tasks are running and have passed health checks
- [ ] ALB target group shows healthy targets
- [ ] API Gateway returns expected response on `/health` endpoint
- [ ] DNS resolution returns the correct IP address or alias
- [ ] TLS certificate is valid and serves the correct domain
- [ ] CloudWatch logs are being written to the correct log group
- [ ] Alerts and alarms are configured and functional
- [ ] Smoke test suite passes with [MIN_PASS_RATE]% success rate
- [ ] No unexpected resource changes appear in subsequent `terraform plan`

### 7.3 Environment Readiness Checklist

<!-- INSTRUCTION: Mark each environment as ready only after all
     checks pass. This checklist should be completed once per
     environment during initial setup and re-verified after
     any shared infrastructure change. -->

| Check | dev | test | staging | prod |
|-------|-----|------|---------|------|
| VPC and subnets provisioned | [ ] | [ ] | [ ] | [ ] |
| Security groups created | [ ] | [ ] | [ ] | [ ] |
| IAM roles and policies created | [ ] | [ ] | [ ] | [ ] |
| ECS cluster is active | [ ] | [ ] | [ ] | [ ] |
| ECR repository exists | [ ] | [ ] | [ ] | [ ] |
| ALB / NLB is provisioned | [ ] | [ ] | [ ] | [ ] |
| ACM certificate is valid | [ ] | [ ] | [ ] | [ ] |
| Route53 hosted zone configured | [ ] | [ ] | [ ] | [ ] |
| CloudWatch log groups created | [ ] | [ ] | [ ] | [ ] |
| Secrets stored in Secrets Manager | [ ] | [ ] | [ ] | [ ] |
| Terraform state backend configured | [ ] | [ ] | [ ] | [ ] |

### 7.4 Rollback Validation

<!-- INSTRUCTION: These checks verify that a rollback has been
     completed successfully and no residual state remains from
     the failed deployment. -->

- [ ] Previous Terraform state has been restored from backup
- [ ] ECS tasks are running the previous container image version
- [ ] ALB target group points to the previous task set
- [ ] DNS records reflect the pre-deployment state (if changed)
- [ ] Database migration has been rolled back (if applicable)
- [ ] CloudWatch alarms have returned to normal state
- [ ] Incident ticket has been created with root cause summary
- [ ] Post-mortem meeting is scheduled within [POSTMORTEM_SLA] hours

---

## Section 8: Common Patterns

### 8.1 New Service Onboarding Pattern

<!-- INSTRUCTION: This pattern describes the step-by-step process
     for adding a new service to the infrastructure. Each step
     must be completed before the next begins. -->

**Prerequisites**: Service has a defined Dockerfile and CI pipeline.

**Steps:**

1. **Create Terraform module**: `[TF_MODULE_PATH]/[NEW_SERVICE]/`
   - Add `main.tf`, `variables.tf`, `outputs.tf`
   - Define ECS task definition, service, and target group
   - Reference shared VPC, security groups, and IAM roles

2. **Register service in environment modules**: Update `environments/[ENV]/main.tf`
   - Add module block for `[NEW_SERVICE]`
   - Pass environment-specific variables

3. **Create ECR repository**: Add to shared ECR module or create standalone module.
   - Set lifecycle policy to retain last [IMAGE_RETENTION_COUNT] images

4. **Configure ALB listener rule**: Add routing rule to forward traffic to the new target group.
   - Path pattern: `[SERVICE_PATH]`

5. **Create API Gateway integration** (if applicable): Add VPC link integration for private API exposure.

6. **Add monitoring**: Create CloudWatch alarms for CPU, memory, and task count.

7. **Update documentation**: Add service entry to the dependency matrix (Section 3.2) and database access table (Section 4.3).

8. **Deploy through environments**: Follow promotion order in Section 2.1.

### 8.2 Database Migration Pattern

**Prerequisites**: Migration script is reviewed and tested locally.

**Steps:**

1. **Create reversible migration**: Every `UP` migration must have a corresponding `DOWN` migration.
2. **Test in dev**: Run migration against dev database. Verify schema changes and data integrity.
3. **Run in test**: Execute as part of CI pipeline. Verify application compatibility.
4. **Stage in staging**: Run against a copy of production data. Validate performance impact.
5. **Apply to prod**: Execute during maintenance window with rollback plan active.
6. **Verify**: Run data integrity checks. Monitor application error rates for [MONITORING_WINDOW] minutes.
7. **Complete**: Remove rollback flag. Update database access table (Section 4.3) if permissions changed.

<!-- INSTRUCTION: Specify the migration tool used (Flyway, Liquibase,
     Alembic, etc.) and any project-specific conventions such as
     naming patterns for migration files. -->

### 8.3 Multi-Region Pattern

<!-- INSTRUCTION: This pattern applies only to projects requiring
     multi-region deployment. Remove this section if the project
     is single-region. -->

**Prerequisites**: Primary region is fully deployed. DR region is identified.

**Steps:**

1. **Replicate foundational infrastructure**: Deploy VPC, subnets, security groups, and IAM roles in the DR region using the same Terraform modules with region-specific variables.
2. **Configure cross-region resources**: Set up S3 cross-region replication, DynamoDB global tables, or RDS read replicas as needed.
3. **Deploy compute layer**: Deploy ECS cluster and Fargate services in the DR region.
4. **Configure DNS failover**: Create Route53 health checks and failover routing policies.
   - Primary: `[REGION_PRIMARY]`
   - Secondary: `[REGION_DR]`
5. **Test failover**: Simulate primary region failure and verify automatic DNS failover.
6. **Document RTO/RPO**: Record achieved Recovery Time Objective and Recovery Point Objective.
   - RTO target: [RTO_TARGET]
   - RPO target: [RPO_TARGET]
7. **Schedule regular DR testing**: Run failover drill every [DR_DRILL_FREQUENCY] months.

---

## Footer

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | [DATE] |
| **Author** | [AUTHOR_NAME] |
| **Approved By** | [APPROVER_NAME] |
| **Next Review Date** | [REVIEW_DATE] |

<!-- INSTRUCTION: Update the version number with each significant
     change. Use semantic versioning (MAJOR.MINOR). The next
     review date should be no more than 6 months from the last
     update for actively maintained projects. -->
