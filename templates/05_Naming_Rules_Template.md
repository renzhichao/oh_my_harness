# Naming Convention Rules - [PROJECT_TITLE]

<!-- ==============================================================================
     INSTRUCTION: SPEC CODING TEMPLATE - NAMING RULES
     ==============================================================================
     This template defines naming convention standards for infrastructure and
     platform engineering projects. It covers AWS resources, database objects,
     Terraform definitions, and tagging policies. Consistent naming reduces
     operational confusion, speeds up troubleshooting, and enforces governance.

     HOW TO USE THIS TEMPLATE:
     1. Replace all [PLACEHOLDER] markers with project-specific content.
     2. Remove resource sections that do not apply to your project.
     3. Follow the inline comments (<!-- INSTRUCTION: ... -->) for guidance.
     4. Enforce rules via CI/CD validation scripts (see Section 6).
     5. Update this document when new resource types are introduced.
     ============================================================================ -->

**Document ID**: NAMING_[PROJECT_NAME]
<!-- INSTRUCTION: Use a short, uppercase, underscore-separated identifier.
     Example: NAMING_ALB_Fargate, NAMING_payment_service_v2 -->

**Issue**: #[ISSUE_NUMBER] - [Issue Title]
<!-- INSTRUCTION: Reference the GitHub/Jira issue number and its title.
     Example: #2132 - Dev PostgreSQL NLB Routing Fix -->

**Created**: [DATE]
<!-- INSTRUCTION: Use the date this document was first created.
     Example: 2025-03-10 -->

**Branch**: [BRANCH_NAME]
<!-- INSTRUCTION: The feature or bugfix branch where implementation work occurs.
     Example: bugfix/2132-dev-postgresql-nlb-routing-fix -->

**Status**: Draft
<!-- INSTRUCTION: Track document lifecycle:
     - "Draft" (initial writing)
     - "Under Review" (team feedback)
     - "Approved" (ready for enforcement)
     - "Enforced" (CI/CD validation active)
     - "Revised" (updated after feedback or new resources) -->

---

## Table of Contents

- [1. Overview](#1-overview)
- [2. General Principles](#2-general-principles)
- [3. Resource Naming Rules](#3-resource-naming-rules)
- [4. Environment Suffix Rules](#4-environment-suffix-rules)
- [5. Tagging Standards](#5-tagging-standards)
- [6. Validation](#6-validation)
- [7. Migration Guide](#7-migration-guide)

---

## 1. Overview

### 1.1 Purpose

<!-- INSTRUCTION: Explain why naming conventions matter for this specific project.
     Reference operational pain points that inconsistent naming has caused,
     or compliance requirements that demand standardization. -->

This document establishes mandatory naming conventions for all infrastructure resources, code artifacts, and configuration objects created as part of **[PROJECT_TITLE]**. The goals are:

- **Discoverability** -- Engineers can locate any resource by name without opening the console.
- **Traceability** -- Every resource name links back to a service, environment, and cost center.
- **Consistency** -- Automated pipelines can parse and validate resource names programmatically.
- **Compliance** -- Names meet organizational governance and audit requirements defined in [POLICY_REFERENCE].

<!-- INSTRUCTION: Replace [POLICY_REFERENCE] with the internal policy document
     or compliance framework that mandates naming standards. If none exists,
     remove the compliance bullet or replace it with "internal best practices". -->

### 1.2 Scope

<!-- INSTRUCTION: List the resource categories this document governs.
     Add or remove categories based on the technology stack of the project. -->

These rules apply to the following resource types across all environments:

| Category | Resources |
|----------|-----------|
| Compute | Lambda Functions, ECS Services, EC2 Instances |
| Networking | API Gateway, VPCs, Subnets, Security Groups, NLB/ALB |
| Storage | S3 Buckets, DynamoDB Tables, EFS, EBS Volumes |
| Database | RDS Instances, Aurora Clusters, Table Columns, Indexes |
| IAM | Roles, Policies, Instance Profiles |
| Observability | CloudWatch Log Groups, Alarms, Dashboards |
| Configuration | SSM Parameters, Secrets Manager Secrets |
| Infrastructure as Code | Terraform Resources, Modules, State Files |
| CI/CD | CodePipeline Pipelines, CodeBuild Projects, Jenkins Jobs |

<!-- INSTRUCTION: Adjust the table above to match your project's actual
     resource inventory. Remove rows for unused categories and add rows
     for any missing ones (e.g., Kafka topics, Redis clusters, WAF rules). -->

---

## 2. General Principles

<!-- INSTRUCTION: These principles apply universally. Do not remove them.
     You may add project-specific principles but the four listed here
     must remain as the foundation. -->

### 2.1 Consistency Principle

All resources of the same type must follow identical naming patterns across every environment. A Lambda function in `dev` must use the same structural pattern as its counterpart in `prod`.

**Rule**: If two engineers name the same conceptual resource independently, the resulting names must be identical.

### 2.2 Descriptiveness Principle

A resource name must communicate its **purpose**, **owning service**, and **environment** without requiring the reader to consult additional documentation.

**Rule**: A new team member must identify what a resource does from its name alone.

### 2.3 Brevity Principle

Names must be as short as possible while remaining descriptive. Avoid redundant prefixes or suffixes that duplicate information already conveyed by the resource type or tagging.

**Rule**: Maximum name length must not exceed the shortest platform limit among target resources (e.g., 64 characters for Lambda, 63 for S3).

<!-- INSTRUCTION: Update the character limit if your project targets a platform
     with a stricter constraint. DynamoDB tables allow 255, Security Groups
     allow 255, IAM role names allow 64, S3 bucket names allow 63. -->

### 2.4 Environment Awareness Principle

Every deployable resource must encode its target environment in the name. This prevents accidental cross-environment operations and makes resource ownership immediately visible.

**Rule**: No resource name is valid without an environment indicator.

---

## 3. Resource Naming Rules

<!-- INSTRUCTION: Each subsection defines the naming pattern for a specific
     resource type. Remove sections for resources you do not use. Add new
     sections following the same format (pattern, rationale, examples table). -->

### 3.1 Lambda Functions

**Pattern**:
```
{stage}-{abbreviated-service}-{function-name}
```

<!-- INSTRUCTION: Define what "abbreviated-service" means for your project.
     It should be a 2-5 character short code. Example: "card" for card-service,
     "auth" for authentication-service, "pmt" for payment-service.
     Keep a service abbreviation registry in your team wiki. -->

**Rationale**: The leading stage segment groups functions by environment in sorted views and CloudWatch log paths.

**Separator**: Hyphen (`-`) | **Character Limit**: 64 characters

| Resource | Pattern | Example |
|----------|---------|---------|
| Card processing | `{stage}-card-process` | `dev-card-process` |
| Auth validation | `{stage}-auth-validate-token` | `staging-auth-validate-token` |
| Notification sender | `{stage}-notif-send-email` | `prod-notif-send-email` |
| Report generator | `{stage}-rpt-generate-monthly` | `test-rpt-generate-monthly` |
| Data sync handler | `{stage}-sync-[ENTITY]` | `dev-sync-customer` |

### 3.2 API Gateway Routes

**Pattern**:
```
/{stage}/{service}/{version}/{resource}[/{resource-id}[/{sub-resource}]]
```

<!-- INSTRUCTION: Adjust the route pattern to match your API design standard.
     Some teams prefer /api/{service}/{version}/{resource} without the stage
     in the path (using stage variables instead). Choose one convention and
     document it here. -->

**Rationale**: URL-based versioning allows simultaneous deployment of multiple API versions and clear routing for consumers.

| Resource | Pattern | Example |
|----------|---------|---------|
| List accounts | `GET /{stage}/card/v1/accounts` | `GET /prod/card/v1/accounts` |
| Get account by ID | `GET /{stage}/card/v1/accounts/{id}` | `GET /prod/card/v1/accounts/acc-12345` |
| Create transaction | `POST /{stage}/pmt/v1/transactions` | `POST /dev/pmt/v1/transactions` |
| Health check | `GET /{stage}/{service}/v1/health` | `GET /staging/auth/v1/health` |

### 3.3 DynamoDB Tables

**Pattern**:
```
malbank-{service}-{entity}-{environment}
```

<!-- INSTRUCTION: Replace "malbank" with your organization's prefix.
     The prefix ensures global uniqueness and groups tables visually
     in the AWS console. Some teams use the company name or project code. -->

**Rationale**: DynamoDB table names are global within an AWS account. The organization prefix prevents collision with tables from other projects.

**Separator**: Hyphen (`-`) | **Character Limit**: 255 characters

| Resource | Pattern | Example |
|----------|---------|---------|
| Customer profiles | `malbank-{service}-customer-{env}` | `malbank-card-customer-dev` |
| Transaction records | `malbank-pmt-transaction-{env}` | `malbank-pmt-transaction-prod` |
| Session store | `malbank-auth-session-{env}` | `malbank-auth-session-staging` |
| Audit log | `malbank-{service}-audit-{env}` | `malbank-pmt-audit-test` |
| Configuration data | `malbank-{service}-config-{env}` | `malbank-core-config-dev` |

### 3.4 Database Columns and Attributes

<!-- INSTRUCTION: These rules apply to relational databases (PostgreSQL, MySQL)
     and NoSQL attribute names (DynamoDB, MongoDB). Adjust the casing rules
     if your ORM or framework enforces a different convention. -->

**Column/Attribute Casing**: `camelCase` for table columns and DynamoDB attributes.
**Index Casing**: `PascalCase` for secondary indexes and constraint names.

**Rationale**: `camelCase` aligns with the JavaScript/TypeScript ecosystem and DynamoDB single-table design. `PascalCase` for indexes visually distinguishes access patterns from data attributes.

| Resource | Convention | Example |
|----------|------------|---------|
| Table column | `camelCase` | `customerId`, `createdAt`, `accountBalance` |
| Primary key attribute | `camelCase` | `pk`, `sk`, `userId` |
| GSI name | `PascalCase` | `GSI_ByCustomer`, `GSI_ByStatus` |
| LSI name | `PascalCase` | `LSI_CreatedAt` |
| Foreign key column | `camelCase` with `Id` suffix | `accountId`, `transactionId` |
| Boolean column | `camelCase` with `is/has` prefix | `isActive`, `hasVerified` |
| Timestamp column | `camelCase` with `At` suffix | `createdAt`, `updatedAt`, `deletedAt` |

<!-- INSTRUCTION: If your project uses PostgreSQL with snake_case for columns,
     replace the rules above and update the examples table accordingly.
     Document the rationale for any deviation from camelCase. -->

### 3.5 S3 Buckets

**Pattern**:
```
{organization}-{purpose}-{environment}-{region-code}
```

<!-- INSTRUCTION: S3 bucket names must be globally unique across all AWS accounts.
     Use a short region code: "us1" for us-east-1, "ap1" for ap-southeast-1. -->

**Rationale**: Global uniqueness requirement combined with environment and region identification for multi-region deployments.

**Separator**: Hyphen (`-`) | **Character Limit**: 63 characters
**Restrictions**: Lowercase letters, numbers, and hyphens only. No underscores or uppercase.

| Resource | Pattern | Example |
|----------|---------|---------|
| Application logs | `malbank-app-logs-{env}-{region}` | `malbank-app-logs-prod-us1` |
| Document uploads | `malbank-docs-upload-{env}-{region}` | `malbank-docs-upload-dev-us1` |
| Terraform state | `malbank-tfstate-{env}-{region}` | `malbank-tfstate-prod-us1` |
| Data lake raw | `malbank-datalake-raw-{env}-{region}` | `malbank-datalake-raw-staging-us1` |
| Artifact storage | `malbank-artifacts-{env}-{region}` | `malbank-artifacts-test-us1` |

### 3.6 IAM Roles and Policies

**Role Pattern**: `{organization}-{service}-{purpose}-role-{environment}`
**Policy Pattern**: `{organization}-{service}-{purpose}-policy-{environment}`

<!-- INSTRUCTION: The trailing "-role" / "-policy" suffix distinguishes roles
     from policies in logs and trust relationship documents. -->

**Separator**: Hyphen (`-`) | **Character Limit**: 64 characters

| Resource | Pattern | Example |
|----------|---------|---------|
| Lambda execution role | `malbank-{service}-execution-role-{env}` | `malbank-card-execution-role-dev` |
| ECS task role | `malbank-{service}-task-role-{env}` | `malbank-pmt-task-role-prod` |
| S3 read policy | `malbank-{service}-s3-read-policy-{env}` | `malbank-rpt-s3-read-policy-staging` |
| Cross-account role | `malbank-{service}-xaccount-role-{env}` | `malbank-core-xaccount-role-prod` |
| CI/CD deployment role | `malbank-deploy-pipeline-role-{env}` | `malbank-deploy-pipeline-role-dev` |

### 3.7 Security Groups

**Pattern**: `{organization}-{service}-{purpose}-sg-{environment}`

<!-- INSTRUCTION: Including the environment suffix maintains consistency
     with other naming patterns and aids log analysis, even though SGs
     are VPC-scoped and technically already environment-isolated. -->

**Separator**: Hyphen (`-`) | **Character Limit**: 255 characters

| Resource | Pattern | Example |
|----------|---------|---------|
| ALB security group | `malbank-{service}-alb-sg-{env}` | `malbank-card-alb-sg-prod` |
| Database security group | `malbank-{service}-db-sg-{env}` | `malbank-auth-db-sg-dev` |
| Lambda security group | `malbank-{service}-lambda-sg-{env}` | `malbank-pmt-lambda-sg-staging` |
| Egress security group | `malbank-{service}-egress-sg-{env}` | `malbank-core-egress-sg-prod` |
| Internal service SG | `malbank-{service}-internal-sg-{env}` | `malbank-rpt-internal-sg-test` |

### 3.8 CloudWatch Log Groups

**Pattern**: `/{organization}/{service}/{resource-type}/{resource-name}`

<!-- INSTRUCTION: Use forward slashes to create a hierarchical log structure.
     This allows retention policies at the /{organization}/{service} level
     to cascade down to all child log groups. -->

**Separator**: Forward slash (`/`) for hierarchy, hyphen (`-`) within segments

| Resource | Pattern | Example |
|----------|---------|---------|
| Lambda logs | `/malbank/{service}/lambda/{function-name}` | `/malbank/card/lambda/dev-card-process` |
| ECS task logs | `/malbank/{service}/ecs/{task-family}` | `/malbank/pmt/ecs/payment-processor` |
| API Gateway logs | `/malbank/{service}/apigw/{api-name}` | `/malbank/auth/apigw/auth-api-prod` |
| Application logs | `/malbank/{service}/app/{component}` | `/malbank/core/app/user-service` |

### 3.9 SSM Parameters

**Pattern**: `/{organization}/{environment}/{service}/{category}/{parameter-name}`

<!-- INSTRUCTION: Common categories: config, secret, feature-flag, endpoint, arn.
     Use SecureString type for any parameter containing credentials or
     sensitive configuration values. -->

**Separator**: Forward slash (`/`) for hierarchy, hyphen (`-`) within segments

| Resource | Pattern | Example |
|----------|---------|---------|
| Database endpoint | `/malbank/{env}/{service}/endpoint/db` | `/malbank/dev/auth/endpoint/db` |
| API key (secret) | `/malbank/{env}/{service}/secret/api-key` | `/malbank/prod/pmt/secret/api-key` |
| Feature flag | `/malbank/{env}/{service}/feature-flag/{flag}` | `/malbank/staging/card/feature-flag/new-ui` |
| Lambda ARN | `/malbank/{env}/{service}/arn/{function}` | `/malbank/dev/pmt/arn/process-payment` |
| Config value | `/malbank/{env}/{service}/config/{key}` | `/malbank/prod/core/config/max-retries` |

### 3.10 Terraform Resources

**Resource Block Pattern**: `resource "aws_{type}" "{environment}_{service}_{name}" { ... }`
**Module Pattern**: `module "{environment}_{service}_{purpose}" { ... }`

<!-- INSTRUCTION: Terraform resource addresses are internal to the state and
     do not affect the actual AWS resource name (set via the "name" argument).
     Consistent internal naming makes the Terraform code navigable. -->

**Separator**: Underscore (`_`) following Terraform convention

| Resource | Pattern | Example |
|----------|---------|---------|
| Lambda function | `resource "aws_lambda_function" "dev_card_process"` | -- |
| S3 bucket | `resource "aws_s3_bucket" "prod_docs_upload"` | -- |
| IAM role | `resource "aws_iam_role" "staging_auth_execution"` | -- |
| Security group | `resource "aws_security_group" "dev_card_alb"` | -- |
| Module call | `module "prod_card_alb"` | -- |

<!-- INSTRUCTION: The "name" or "name_prefix" argument inside each resource
     block must follow the AWS resource patterns in Sections 3.1-3.9.
     The Terraform internal name is independent of the AWS resource name. -->

---

## 4. Environment Suffix Rules

<!-- INSTRUCTION: Every deployable resource must encode its target environment.
     Add or remove environments based on your project's deployment pipeline. -->

### 4.1 Environment Mapping

| Environment | Suffix | Short Code | Example (Lambda) | Example (S3 Bucket) |
|-------------|--------|------------|-------------------|----------------------|
| Development | `dev` | `d` | `dev-card-process` | `malbank-app-logs-dev-us1` |
| Testing | `test` | `t` | `test-card-process` | `malbank-app-logs-test-us1` |
| Staging | `staging` | `s` | `staging-card-process` | `malbank-app-logs-staging-us1` |
| Production | `prod` | `p` | `prod-card-process` | `malbank-app-logs-prod-us1` |

<!-- INSTRUCTION: The "Short Code" column is optional and used when character
     limits are tight. If your project does not need short codes, remove it. -->

### 4.2 Environment Placement Rules

<!-- INSTRUCTION: Document WHERE in the name the environment appears for each
     resource type. Inconsistent placement is a common source of confusion. -->

| Resource Type | Environment Position | Example |
|---------------|---------------------|---------|
| Lambda Functions | Prefix (leading) | `dev-card-process` |
| API Gateway Routes | First path segment | `/prod/card/v1/accounts` |
| DynamoDB Tables | Suffix (trailing) | `malbank-card-customer-dev` |
| S3 Buckets | Before region code | `malbank-app-logs-prod-us1` |
| IAM Roles | Suffix (trailing) | `malbank-card-execution-role-prod` |
| Security Groups | Suffix (trailing) | `malbank-card-alb-sg-dev` |
| CloudWatch Log Groups | Second path level | `/malbank/prod/card/lambda/...` |
| SSM Parameters | Second path level | `/malbank/dev/auth/endpoint/db` |
| Terraform Resources | Prefix (leading) | `dev_card_process` |

---

## 5. Tagging Standards

<!-- INSTRUCTION: Tags complement naming conventions by providing metadata
     that cannot fit in the name itself. Update the required tags list to
     match your organization's tagging policy. -->

### 5.1 Required Tags

Every resource that supports tagging must include all of the following tags:

| Tag Key | Description | Example Value | Required |
|---------|-------------|---------------|----------|
| `Environment` | Deployment environment | `dev`, `test`, `staging`, `prod` | Yes |
| `Service` | Owning service or microservice | `card-service`, `payment-service` | Yes |
| `ManagedBy` | Tool or team managing the resource | `terraform`, `eks-operator`, `manual` | Yes |
| `CostCenter` | Cost allocation center code | `CC-1001`, `CC-2048` | Yes |
| `Project` | Project or initiative identifier | `BE_Infra`, `FIP-1688` | Yes |
| `Owner` | Team or individual responsible | `platform-team`, `arthur.ren` | Yes |
| `CreatedDate` | Date resource was created | `2025-03-10` | Yes |

<!-- INSTRUCTION: CostCenter values must match your organization's finance
     system codes. Coordinate with the finance team before enforcing this tag. -->

### 5.2 Optional Tags

| Tag Key | Description | Example Value |
|---------|-------------|---------------|
| `Backup` | Backup policy identifier | `daily`, `weekly`, `none` |
| `DataClassification` | Sensitivity level of data | `public`, `internal`, `confidential`, `restricted` |
| `ExpiresAfter` | Auto-deletion date for temporary resources | `2025-12-31` |
| `Repository` | Source code repository URL | `github.com/org/BE_Infra` |
| `Version` | Application version deployed | `v2.1.0` |

### 5.3 Tag Enforcement

<!-- INSTRUCTION: Describe how tags are enforced. Common methods: AWS SCPs,
     AWS Config rules, Terraform sentinel policies, CI/CD pipeline checks. -->

Resources missing required tags must:

1. Be rejected by the CI/CD pipeline during `terraform plan`.
2. Trigger an alert to the `#platform-alerts` Slack channel if detected in an existing environment.
3. Be remediated within 5 business days of detection.

---

## 6. Validation

### 6.1 Automated Validation Script

<!-- INSTRUCTION: Adapt this bash script to validate naming conventions in your
     CI/CD pipeline. It checks Terraform plan output for naming violations.
     It can also be run locally during development. -->

```bash
#!/usr/bin/env bash
# naming-validator.sh -- Validate resource naming conventions
# Usage: ./naming-validator.sh <terraform-plan-json>
# Exit:  0 = all names valid, 1 = violations detected

set -euo pipefail
PLAN_FILE="${1:?Usage: $0 <terraform-plan-json>}"
VIOLATIONS=0

# -- Configuration ------------------------------------------------------------
ORG_PREFIX="[ORGANIZATION_PREFIX]"
# INSTRUCTION: Set your organization prefix. Example: "malbank"

LAMBDA_PATTERN="^(dev|test|staging|prod)-[a-z0-9-]+$"
DYNAMODB_PATTERN="^${ORG_PREFIX}-[a-z0-9-]+-(dev|test|staging|prod)$"
S3_PATTERN="^${ORG_PREFIX}-[a-z0-9-]+-(dev|test|staging|prod)-[a-z0-9]+$"

# -- Validation Functions -----------------------------------------------------
validate_names() {
  local rtype="$1" pattern="$2" jq_filter="$3"
  echo "=== Validating ${rtype} ==="
  names=$(jq -r "${jq_filter}" "$PLAN_FILE" 2>/dev/null || true)
  for name in $names; do
    [[ ! "$name" =~ $pattern ]] && { echo "  VIOLATION: '$name'"; ((VIOLATIONS++)); }
  done
}

validate_names "Lambda" "$LAMBDA_PATTERN" \
  '.planned_values.root_module.resources[] | select(.type=="aws_lambda_function") | .values.function_name // empty'
validate_names "DynamoDB" "$DYNAMODB_PATTERN" \
  '.planned_values.root_module.resources[] | select(.type=="aws_dynamodb_table") | .values.name // empty'
validate_names "S3" "$S3_PATTERN" \
  '.planned_values.root_module.resources[] | select(.type=="aws_s3_bucket") | .values.bucket // empty'

# -- Tag Validation -----------------------------------------------------------
# INSTRUCTION: Update REQUIRED_TAGS to match Section 5.1.
echo "=== Validating Required Tags ==="
REQUIRED_TAGS=("Environment" "Service" "ManagedBy" "CostCenter" "Project")
jq -c '.planned_values.root_module.resources[] | select(.values.tags!=null)' "$PLAN_FILE" \
  2>/dev/null | while read -r res; do
  addr=$(echo "$res" | jq -r '.address')
  for tag in "${REQUIRED_TAGS[@]}"; do
    echo "$res" | jq -e ".values.tags.${tag}" >/dev/null 2>&1 \
      || echo "  VIOLATION: ${addr} missing '${tag}'"
  done
done

# -- Summary ------------------------------------------------------------------
[ "$VIOLATIONS" -gt 0 ] && { echo "FAILED: ${VIOLATIONS} violation(s)."; exit 1; } \
  || { echo "PASSED: All names comply."; exit 0; }
```

<!-- INSTRUCTION: Integrate this script into your CI/CD pipeline after
     `terraform plan` but before `terraform apply`. Example:
       - name: Validate naming conventions
         run: ./scripts/naming-validator.sh tfplan.json -->

### 6.2 Naming Checklist

<!-- INSTRUCTION: Use this checklist during code review to manually verify
     naming compliance. Check each box before approving a PR. -->

- [ ] All new resource names follow the patterns defined in Section 3.
- [ ] Environment suffix matches the target deployment environment (Section 4).
- [ ] No resource name exceeds the character limit for its type.
- [ ] All required tags (Section 5.1) are present on taggable resources.
- [ ] Tag values use the correct format (e.g., `dev` not `development`).
- [ ] S3 bucket names contain only lowercase letters, numbers, and hyphens.
- [ ] DynamoDB table names include the organization prefix.
- [ ] IAM role names end with `-role-{env}` and policies end with `-policy-{env}`.
- [ ] Security group names end with `-sg-{env}`.
- [ ] CloudWatch log groups use the hierarchical `/org/service/type/name` pattern.
- [ ] SSM parameter paths use the hierarchical `/org/env/service/category/name` pattern.
- [ ] Terraform resource addresses use underscores and follow `env_service_name` pattern.
- [ ] No abbreviations are used that are not in the approved abbreviation registry.
- [ ] Names do not include environment-identifying terms in unexpected positions.

### 6.3 Common Mistakes

<!-- INSTRUCTION: Add entries based on naming violations that have occurred in
     your project. Update this table during retrospectives. -->

| # | Mistake | Incorrect | Correct | Ref |
|---|---------|-----------|---------|-----|
| 1 | Missing environment in Lambda name | `card-process` | `dev-card-process` | 3.1 |
| 2 | Underscore in S3 bucket name | `malbank_app_logs_dev_us1` | `malbank-app-logs-dev-us1` | 3.5 |
| 3 | Uppercase in DynamoDB table name | `Malbank-Card-Customer-Dev` | `malbank-card-customer-dev` | 3.3 |
| 4 | Wrong environment suffix format | `malbank-card-customer-development` | `malbank-card-customer-dev` | 4.1 |
| 5 | Missing organization prefix | `card-customer-dev` | `malbank-card-customer-dev` | 3.3 |
| 6 | Missing `-sg` suffix on security group | `malbank-card-alb-dev` | `malbank-card-alb-sg-dev` | 3.7 |
| 7 | Using camelCase in resource name | `malbank-cardProcess-dev` | `malbank-card-process-dev` | 2.1 |
| 8 | Missing `-role` suffix on IAM role | `malbank-card-execution-dev` | `malbank-card-execution-role-dev` | 3.6 |
| 9 | Wrong position of environment | `card-dev-process` | `dev-card-process` | 4.2 |
| 10 | Exceeding character limit | `malbank-very-long-service-name-...` | Shorten to fit limit | 2.3 |

---

## 7. Migration Guide

<!-- INSTRUCTION: Use this section when renaming existing resources to comply
     with these naming conventions. Renaming is often a destructive operation,
     so plan carefully. -->

### 7.1 When to Rename

Resources should be renamed when:

1. **Governance Audit**: A compliance review identifies naming violations.
2. **Standardization Sprint**: A dedicated effort to align all resources with this document.
3. **New Resource Introduction**: A resource is being created or migrated and does not yet conform.
4. **Incident Root Cause**: A naming-related confusion contributed to an operational incident.

<!-- INSTRUCTION: Establish a policy on whether renaming is mandatory for
     existing resources or only enforced on new resources. Many teams choose
     "enforce on new, migrate existing opportunistically" to avoid risk. -->

### 7.2 Migration Process

**Step 1: Impact Assessment**

```
[ ] Identify all references to the current resource name:
    - Terraform state
    - Application code / environment variables
    - IAM policies and trust relationships
    - CloudWatch alarms and dashboards
    - Documentation and runbooks
    - CI/CD pipeline configurations
```

<!-- INSTRUCTION: Add project-specific reference sources to the checklist
     above. For example, if your project uses Step Functions that reference
     Lambda ARNs, add that to the list. -->

**Step 2: Create the New Resource**

1. Create the new resource with the correct name alongside the existing one.
2. Validate it is functional and identical in configuration.
3. Run integration tests against the new resource name.

**Step 3: Migrate References**

1. Update all Terraform configurations, application config, and IAM policies.
2. Update monitoring, alerting, and dashboard configurations.
3. Update documentation and runbooks.

**Step 4: Validate**

```
[ ] All integration tests pass with new resource name
[ ] No references to old resource name in application code
[ ] No references to old resource name in Terraform state
[ ] CloudWatch metrics flowing for new resource
[ ] Rollback procedure documented and tested
```

**Step 5: Decommission Old Resource**

1. Verify zero traffic to the old resource (check CloudWatch metrics for 24 hours minimum).
2. Delete the old resource via Terraform (not manually).
3. Announce the rename in the team channel and update the asset inventory.

### 7.3 Backward Compatibility

<!-- INSTRUCTION: Define how the project handles backward compatibility during
     the renaming transition period. Especially important for API routes. -->

| Resource Type | Strategy | Transition Period |
|---------------|----------|-------------------|
| Lambda Functions | Deploy both names; route traffic via alias | [DURATION, e.g., 30 days] |
| API Gateway Routes | Redirect old route to new route | [DURATION, e.g., 90 days] |
| DynamoDB Tables | Duplicate to new table; old read-only | [DURATION, e.g., 60 days] |
| S3 Buckets | Replicate to new bucket; CloudFront redirect | [DURATION, e.g., 30 days] |
| IAM Roles | New role with same policies; update trust | [DURATION, e.g., 14 days] |
| Security Groups | Attach new SG before detaching old | [DURATION, e.g., 7 days] |
| SSM Parameters | New parameter; update consumers; delete old | [DURATION, e.g., 14 days] |

<!-- INSTRUCTION: Replace [DURATION] placeholders with actual transition
     periods approved by your team. Shorter periods reduce operational
     overhead but require faster consumer migration. -->

### 7.4 Rollback Procedure

If a migration causes issues:

1. Revert Terraform configuration to the previous commit and run `terraform apply`.
2. Verify the old resource still exists and is operational.
3. Conduct a blameless post-mortem. Update this guide with lessons learned.

<!-- INSTRUCTION: Add project-specific rollback steps. For example, if your
     project uses blue/green Lambda deployments, document how to switch
     traffic back to the old alias. -->

---

## Footer

| Field | Value |
|-------|-------|
| **Version** | 1.0.0 |
| **Last Updated** | [DATE] |
| **Next Review Date** | [DATE + 90 days] |
| **Document Owner** | [TEAM_OR_INDIVIDUAL] |
| **Approved By** | [APPROVER_NAME] |

<!-- INSTRUCTION: Update footer fields:
     - Version: Semantic versioning. Minor for additions, major for breaking changes.
     - Last Updated: Date of the most recent change.
     - Next Review Date: Schedule periodic reviews (recommended: quarterly).
     - Document Owner: Team or individual responsible for this document.
     - Approved By: Engineering manager or architect who approved these standards.
     Remove this instruction comment after filling in the fields. -->

<!-- ==============================================================================
     END OF NAMING CONVENTION RULES TEMPLATE
     ==============================================================================
     This template is part of the spec-coding-templates collection.
     For questions or contributions, contact the platform engineering team.
     ============================================================================ -->
