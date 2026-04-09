# Requirements Specification - ALB + Fargate Infrastructure for Java Card Service

**Document ID**: REQ_ALB_FARGATE_JAVA_CARD_SERVICE

**Issue**: #1880 - ALB + Fargate Infrastructure for Java Container Deployment

**Created**: 2025-01-20

**Branch**: feature/1880-alb-fargate-java-card-service

**Status**: Requirements Approved

**Author**: arthurren

**Dependencies**: #1843 (PostgreSQL Cluster Infrastructure)

---

## Executive Summary

### Problem Statement

**Current State**:
- Java Card Service runs on a manually provisioned EC2 instance with no container orchestration
- Deployment requires SSH access to the instance and manual jar replacement, causing downtime
- No load balancing, auto-scaling, or automated health checking exists
- Environment configuration drift between dev, test, staging, and prod is frequent
- Monitoring is limited to basic CPU/memory metrics on the EC2 instance

**Target State**:
- Containerized Java Card Service deployed on ECS Fargate with full orchestration
- Zero-downtime rolling deployments via ALB target group management
- Auto-scaling based on CPU and memory utilization with per-environment thresholds
- Identical infrastructure provisioned across all environments via Terraform modules
- Comprehensive observability with structured logging, Container Insights, and CloudWatch alarms

### Solution Overview

1. Application Load Balancer - Internal ALB with HTTPS termination and health check routing
2. ECS Fargate Cluster - Per-environment container orchestration with auto-scaling
3. API Gateway Integration - VPC Link routing for /card/v1/* paths through internal NLB
4. Container Deployment Pipeline - ECR repository with multi-stage Docker builds and image scanning
5. Observability Stack - CloudWatch Logs, Container Insights, and metric alarms with SNS notifications

### Architecture Decision

#### Option A: EC2 + Docker Compose (Considered but Not Selected)
- **Approach**: Run Docker containers on dedicated EC2 instances managed via Docker Compose
- **Benefits**: Lower cost for steady-state workloads, full control over the host OS, simpler networking
- **Not Selected Because**: Manual scaling overhead, no built-in service discovery, patching burden on the host OS, no native integration with ALB target groups for rolling deploys

#### Option B: ALB + ECS Fargate ✅ **SELECTED**

**Approach**:
- **Application Load Balancer**: Internal ALB with HTTPS termination, routing traffic to ECS tasks
- **ECS Fargate**: Serverless container compute with awsvpc network mode, per-environment clusters
- **API Gateway**: REST API with VPC Link for private integration with the internal NLB

**Rationale**:
- Serverless compute eliminates host OS management and patching responsibility
- Native ALB integration provides rolling deployment with configurable minimum healthy percent
- Auto-scaling built into ECS with target tracking policies for CPU and memory
- Per-second billing aligns cost with actual usage during low-traffic periods

### Architecture Flow

```
+---------------------------------------------------------------+
|                      API Gateway (Regional)                    |
|                  GET/POST /card/v1/*                           |
+-------------------------------+-------------------------------+
                                |
                                v
                +-------------------------------+
                |        VPC Link               |
                |   (Private NLB Target)        |
                +---------------+---------------+
                                |
                                v
                +-------------------------------+
                |   Internal ALB (HTTPS:443)     |
                |   HTTP:80 -> HTTPS redirect    |
                +---------------+---------------+
                                |
                                v
                +-------------------------------+
                |   ECS Fargate Service          |
                |   Task: 1 vCPU, 2 GB RAM       |
                +---------------+---------------+
                                |
                +---------------+---------------+
                |                               |
                v                               v
+-------------------------------+  +---------------------------+
|   Amazon RDS PostgreSQL       |  |   External Services       |
|   (IAM Auth, Secrets Manager) |  |   (via HTTPS outbound)    |
+-------------------------------+  +---------------------------+
```

### Multi-Environment Architecture

```
+-------------------------------------------------------------------+
|            Java Card Service - ECS Fargate Infrastructure          |
|                                                                    |
|  +-------------+  +-------------+  +-------------+  +-----------+ |
|  |    dev      |  |    test     |  |   staging   |  |   prod    | |
|  | 1 task      |  | 1 task      |  | 2 tasks     |  | 3 tasks   | |
|  | 0.5 vCPU    |  | 1 vCPU      |  | 1 vCPU      |  | 1 vCPU    | |
|  | 1 GB RAM    |  | 2 GB RAM    |  | 2 GB RAM    |  | 2 GB RAM  | |
|  +-------------+  +-------------+  +-------------+  +-----------+ |
|                                                                    |
|  Shared: ECR Repository, IAM Policies, KMS Key, SSM Parameters    |
+-------------------------------------------------------------------+
```

### Goals

| Goal ID | Description | Priority |
|---------|-------------|----------|
| GOAL-001 | Deploy internal ALB with HTTPS termination in each environment | CRITICAL |
| GOAL-002 | Provision ECS Fargate cluster with per-environment isolation | CRITICAL |
| GOAL-003 | Configure auto-scaling with CPU and memory target tracking | CRITICAL |
| GOAL-004 | Establish VPC Link for API Gateway private integration | HIGH |
| GOAL-005 | Implement rolling deployment with minimum 50% healthy tasks | HIGH |
| GOAL-006 | Set up CloudWatch Logs with JSON structured format | HIGH |
| GOAL-007 | Configure multi-environment Terraform with environment tfvars | HIGH |
| GOAL-008 | Create ECR repository with image scanning and lifecycle policy | HIGH |
| GOAL-009 | Implement health check endpoints for ALB target group | MEDIUM |
| GOAL-010 | Document operational runbook for deployment and rollback | MEDIUM |
| GOAL-011 | Set up CloudWatch alarms for CPU, memory, and 5xx error rate | MEDIUM |
| GOAL-012 | Validate infrastructure with post-deploy integration tests | MEDIUM |

### Success Criteria

| Metric | Current State | Target State | Measurement |
|--------|--------------|--------------|-------------|
| Deployment Frequency | Manual, bi-weekly | Automated, on-demand | CI/CD pipeline deployment count |
| Service Availability | ~99.0% (single instance) | 99.9% (multi-AZ Fargate) | CloudWatch uptime check |
| Response Time P99 | Unmeasured | < 500 ms | CloudWatch Latency metric |
| Infrastructure as Code Coverage | 0% (manual EC2) | 100% (Terraform) | Terraform plan drift detection |
| Environment Parity | Low (manual config) | Full (shared modules + env tfvars) | Terraform plan comparison |
| Automated Test Coverage | None | Post-deploy smoke tests per environment | Test suite pass rate |

> **Important Note**: This project depends on the PostgreSQL cluster provisioned in issue #1843. The database endpoint, security group, and Secrets Manager credentials must be available before ECS task deployment. Database migration is out of scope and tracked separately.

---

## Section 1: Functional Requirements

### 1.1 Application Load Balancer Infrastructure

#### FR-1.1.1: ALB with HTTPS Termination

The internal Application Load Balancer serves as the entry point for all traffic to the Java Card Service. It terminates TLS connections on port 443 using certificates from AWS Certificate Manager, enforces HTTPS by redirecting all HTTP traffic on port 80, and forwards requests to the ECS Fargate target group. The ALB must be deployed in private subnets across two Availability Zones to satisfy high availability requirements.

**Acceptance Criteria**:

- [ ] Internal ALB is provisioned in each target environment (dev, test, staging, prod)
- [ ] HTTPS listener is configured on port 443 with TLS 1.3 security policy
- [ ] HTTP listener on port 80 redirects all requests to HTTPS with HTTP 301
- [ ] ACM certificate is attached and auto-renewal is confirmed
- [ ] Target group health checks return HTTP 200 for all registered targets
- [ ] ALB access logs are enabled and written to a dedicated S3 bucket

**Infrastructure Requirements**:

```yaml
# ALB Configuration
component: aws_lb
name: java-card-service-alb
scheme: internal
ip_address_type: ipv4
load_balancer_type: application

listeners:
  - port: 443
    protocol: HTTPS
    ssl_policy: ELBSecurityPolicy-TLS13-1-2-2021-06
    certificate_arn: "arn:aws:acm:ap-northeast-2:ACCOUNT_ID:certificate/CERT_ID"
    default_action:
      type: forward
      target_group_arn: "${aws_lb_target_group.java_card.arn}"
  - port: 80
    protocol: HTTP
    default_action:
      type: redirect
      redirect_config:
        protocol: HTTPS
        port: "443"
        status_code: HTTP_301

target_groups:
  - name: java-card-service-tg
    port: 8080
    protocol: HTTP
    target_type: ip
    health_check:
      enabled: true
      path: "/actuator/health"
      interval_seconds: 30
      healthy_threshold: 3
      unhealthy_threshold: 3
      timeout_seconds: 5
      matcher: "200"
```

**Multi-Environment Configuration**:

```yaml
# Per-environment ALB configuration
environments:
  dev:
    domain: java-card-service.dev.internal.example.com
    idle_timeout: 60
    deletion_protection: false
  test:
    domain: java-card-service.test.internal.example.com
    idle_timeout: 60
    deletion_protection: false
  staging:
    domain: java-card-service.staging.internal.example.com
    idle_timeout: 60
    deletion_protection: true
  prod:
    domain: java-card-service.prod.internal.example.com
    idle_timeout: 30
    deletion_protection: true
```

> **Critical Requirement**: The ALB MUST be internal-facing (scheme: internal) in all environments. No internet-facing ALB is permitted per the organization security policy for internal microservices.

#### FR-1.1.2: Internal ALB Configuration

The ALB is deployed into private application subnets across two Availability Zones (ap-northeast-2a and ap-northeast-2c). Security groups restrict inbound traffic to the VPC CIDR range on port 443 and allow all outbound traffic. The ALB communicates with ECS tasks via the target group on port 8080.

**Acceptance Criteria**:

- [ ] ALB subnets are private application subnets in two AZs
- [ ] Security group inbound allows TCP/443 from VPC CIDR only
- [ ] Security group outbound allows TCP/8080 to ECS task security group
- [ ] ALB is registered as a target of the internal NLB for API Gateway integration

### 1.2 ECS Fargate Infrastructure

#### FR-1.2.1: ECS Cluster (Multi-Environment)

Each environment receives a dedicated ECS cluster with the Fargate capacity provider. Clusters follow the naming convention `java-card-service-{env}` and include standard tags for environment, cost-center, and managed-by. The Fargate capacity provider strategy uses the default weight of 1 for on-demand capacity.

**Acceptance Criteria**:

- [ ] ECS cluster is created in each target environment with Fargate capacity provider
- [ ] Cluster naming follows convention: java-card-service-{environment}
- [ ] Cluster tags include Environment, CostCenter, and ManagedBy
- [ ] Fargate capacity provider is associated with the cluster

#### FR-1.2.2: Task Definition Template

The task definition defines the container runtime specification for the Java Card Service. It uses the awsvpc network mode required by Fargate, allocates 1 vCPU (1024 CPU units) and 2 GB memory, and references the ECR repository for the container image. Environment variables are injected from SSM Parameter Store, and logs are forwarded to CloudWatch Logs in a structured JSON format.

**Acceptance Criteria**:

- [ ] Task definition uses awsvpc network mode with Fargate compatibility
- [ ] CPU is set to 1024 units (1 vCPU) and memory to 2048 MB (2 GB)
- [ ] Container image references the ECR repository URI with configurable tag
- [ ] Environment variables are injected from SSM Parameter Store
- [ ] Log group is configured for CloudWatch Logs with JSON prefix
- [ ] Task role grants least-privilege permissions for SSM, Secrets Manager, and KMS

**Infrastructure Requirements**:

```yaml
# Task Definition Template
task_definition:
  family: "java-card-service-task"
  network_mode: awsvpc
  requires_compatibilities:
    - FARGATE
  cpu: "1024"
  memory: "2048"
  execution_role_arn: "${aws_iam_role.ecs_execution.arn}"
  task_role_arn: "${aws_iam_role.ecs_task.arn}"
  container_definitions:
    - name: "java-card-service"
      image: "${aws_ecr_repository.java_card.repository_url}:${var.image_tag}"
      essential: true
      port_mappings:
        - container_port: 8080
          protocol: tcp
      log_configuration:
        log_driver: awslogs
        options:
          awslogs-group: "/ecs/java-card-service"
          awslogs-region: "ap-northeast-2"
          awslogs-stream-prefix: "ecs"
      environment_files:
        - type: ssm
          value: "${aws_ssm_parameter.service_config.arn}"
      secrets:
        - name: DB_PASSWORD
          value_from: "arn:aws:secretsmanager:ap-northeast-2:ACCOUNT_ID:secret:java-card-db"
```

#### FR-1.2.3: Service Deployment Capability

The ECS service manages the desired task count and integrates with the ALB target group for traffic routing. Deployments use the rolling update strategy (ECS default) with a minimum healthy percent of 50%, ensuring at least half of the tasks remain healthy during deployments. The service is configured to automatically replace failed tasks.

**Acceptance Criteria**:

- [ ] ECS service is configured with desired task count per environment
- [ ] Service integrates with ALB target group via load balancer configuration block
- [ ] Deployment uses rolling update with minimum_healthy_percent = 50
- [ ] Service auto-recovery is enabled for stopped or failed tasks
- [ ] Deployment circuit breaker is enabled with rollback on failure

#### FR-1.2.4: Auto-Scaling Configuration

Auto-scaling is configured using target tracking policies for CPU and memory utilization. Scale-out and scale-in cooldown periods prevent thrashing. Capacity bounds are environment-specific: dev (1-2 tasks), test (1-3 tasks), staging (2-6 tasks), and prod (3-10 tasks).

**Acceptance Criteria**:

- [ ] Auto-scaling policy targets CPU utilization at the per-environment threshold
- [ ] Auto-scaling policy targets memory utilization at the per-environment threshold
- [ ] Scale-out cooldown period is set to 120 seconds
- [ ] Scale-in cooldown period is set to 300 seconds
- [ ] Minimum and maximum task counts are configurable per environment

**Multi-Environment Configuration**:

```yaml
# Per-environment auto-scaling configuration
environments:
  dev:
    desired_count: 1
    min_capacity: 1
    max_capacity: 2
    cpu_target: 70
    memory_target: 80
  test:
    desired_count: 1
    min_capacity: 1
    max_capacity: 3
    cpu_target: 70
    memory_target: 80
  staging:
    desired_count: 2
    min_capacity: 2
    max_capacity: 6
    cpu_target: 65
    memory_target: 75
  prod:
    desired_count: 3
    min_capacity: 3
    max_capacity: 10
    cpu_target: 60
    memory_target: 70
```

### 1.3 API Gateway Integration

#### FR-1.3.1: VPC Link Configuration

The API Gateway uses a VPC Link to route requests from the public or private API Gateway endpoint to the internal NLB, which forwards traffic to the ALB. The VPC Link is created with private subnets and the VPC Link security group allows inbound TCP traffic from the API Gateway service on the NLB listener port.

**Acceptance Criteria**:

- [ ] VPC Link is created targeting the internal NLB in each environment
- [ ] VPC Link uses private application subnets for network connectivity
- [ ] VPC Link health checks pass consistently
- [ ] API Gateway integration references the VPC Link ID

#### FR-1.3.2: Route Configuration

API Gateway routes are configured for the /card/v1/* path pattern. Supported HTTP methods are GET and POST only. Throttling limits are applied per route with a rate of 100 requests per second and a burst of 200. No authentication is applied at the API Gateway level; authentication is handled at the application layer.

**Acceptance Criteria**:

- [ ] Routes are configured for /card/v1/* path pattern
- [ ] HTTP methods are restricted to GET and POST only
- [ ] Throttling limits are set: rate 100 rps, burst 200
- [ ] Integration connects to VPC Link with proxy integration type

### 1.4 Container Deployment Capability

#### FR-1.4.1: Container Image Build and Push

The Java Card Service uses a multi-stage Docker build. The first stage compiles the Java application using Gradle with Eclipse Temurin JDK 21. The second stage produces a minimal runtime image using Eclipse Temurin JRE 21 Alpine. Images are tagged with the git commit SHA and the semantic version from the build. ECR image scanning is enabled on every push, and a lifecycle policy retains the last 30 images.

**Acceptance Criteria**:

- [ ] ECR repository is created with name java-card-service
- [ ] Container image is built using multi-stage Dockerfile from the project root
- [ ] Image is tagged with git commit SHA and semantic version
- [ ] ECR image scanning is enabled on push
- [ ] Lifecycle policy retains last 30 images and expires untagged images after 7 days

**Infrastructure Requirements**:

```yaml
# ECR Configuration
ecr_repository:
  name: "java-card-service"
  image_tag_mutability: "IMMUTABLE"
  scan_on_push: true
  lifecycle_policy:
    rules:
      - rulePriority: 1
        description: "Keep last 30 tagged images"
        selection:
          tagStatus: tagged
          countType: imageCountMoreThan
          countNumber: 30
        action:
          type: expire
      - rulePriority: 2
        description: "Remove untagged images older than 7 days"
        selection:
          tagStatus: untagged
          countType: sinceImagePushed
          countUnit: days
          countNumber: 7
        action:
          type: expire
```

### 1.5 Database Access

#### FR-1.5.1: Database Connectivity

The Java Card Service connects to the PostgreSQL cluster provisioned in issue #1843. Database credentials are stored in AWS Secrets Manager with automatic rotation enabled (30-day schedule). The ECS task role is granted IAM permissions to read the secret and to connect to the database via IAM authentication. The security group allows ingress on TCP/5432 from the ECS task security group only.

**Acceptance Criteria**:

- [ ] Database connection string is stored in SSM Parameter Store as SecureString
- [ ] Database password is stored in Secrets Manager with 30-day rotation
- [ ] Security group allows ingress from ECS task security group on TCP/5432 only
- [ ] ECS task role has Secrets Manager read access for the database credential secret

### 1.6 Multi-Environment Infrastructure

#### FR-1.6.1: Environment Isolation 🔴 **CRITICAL**

Each environment (dev, test, staging, prod) operates in full isolation. ECS clusters, ALBs, security groups, CloudWatch log groups, and IAM roles are provisioned per environment with no shared runtime resources. ECR repositories and KMS keys are shared across environments to reduce overhead. Resource names include the environment identifier as a suffix (e.g., java-card-service-alb-dev). All resources carry mandatory tags: Environment, CostCenter, ManagedBy, and Project.

**Acceptance Criteria**:

- [ ] Each environment has dedicated ECS cluster, ALB, and security groups
- [ ] IAM roles are scoped per environment with no cross-environment access
- [ ] Resource names include environment identifier suffix
- [ ] All resources carry mandatory tags (Environment, CostCenter, ManagedBy, Project)

**Per-Environment Breakdown**:

| Resource | dev | test | staging | prod |
|----------|-----|------|---------|------|
| ECS Cluster | Yes | Yes | Yes | Yes |
| Internal ALB | Yes | Yes | Yes | Yes |
| ECS Service | Yes | Yes | Yes | Yes |
| Auto-Scaling | No | No | Yes | Yes |
| API Gateway Route | No | Yes | Yes | Yes |
| CloudWatch Alarms | No | No | Yes | Yes |
| Deletion Protection | No | No | Yes | Yes |

---

## Section 2: Non-Functional Requirements

### 2.1 Performance

#### Response Time

- /card/v1/* endpoints P99 latency MUST be under 500 ms
- /card/v1/* endpoints P95 latency MUST be under 200 ms
- /actuator/health endpoint MUST respond within 100 ms

#### Throughput

- System MUST handle 1000 concurrent requests per second in production
- System MUST handle 200 concurrent requests per second in staging

### 2.2 Availability

#### Service Availability

- Target availability: 99.9% (measured monthly)
- Maximum acceptable downtime: 43 minutes per month
- Recovery Time Objective (RTO): 15 minutes
- Recovery Point Objective (RPO): 5 minutes

#### High Availability

- [ ] Application is deployed across 2 Availability Zones (ap-northeast-2a, ap-northeast-2c)
- [ ] ALB is configured with multi-AZ target group registrations
- [ ] ECS service minimum healthy percent ensures at least 1 task during deployment
- [ ] Database has a standby replica in a different AZ (provided by #1843)

### 2.3 Security

#### Network Security

- All inter-service communication MUST use TLS 1.2 or higher
- ECS tasks MUST run in private subnets with no direct internet access
- Security groups MUST follow least-privilege ingress/egress rules
- ALB MUST be internal-facing (scheme: internal) in all environments

#### IAM Security

- ECS task roles MUST follow least-privilege principle
- No IAM roles with AdministratorAccess or wildcard (*:*) permissions
- All IAM policies MUST be reviewed and approved before deployment
- ECS execution role MUST be scoped to ECR pull, CloudWatch Logs, and SSM only

#### Data Security

- Data at rest MUST be encrypted using AWS KMS customer-managed keys
- Data in transit MUST be encrypted using TLS 1.2 or higher
- Database credentials MUST be stored in Secrets Manager with automatic rotation
- SSM parameters containing sensitive values MUST use SecureString type with KMS encryption

### 2.4 Scalability

#### Auto-Scaling

- System MUST auto-scale based on CPU and memory utilization
- Scale-out MUST complete within 120 seconds of threshold breach
- Maximum capacity MUST support 3x baseline load in production (3 to 10 tasks)

#### Horizontal Scaling

- [ ] System supports adding Fargate tasks without downtime
- [ ] Stateless application design enables horizontal scaling (no local session state)
- [ ] Database connection pooling supports concurrent task connections

### 2.5 Observability

#### Logging

- Application logs MUST be sent to CloudWatch Logs log group /ecs/java-card-service
- Log format MUST be structured JSON with fields: timestamp, level, message, trace_id, request_id
- Log retention MUST be 30 days for dev/test, 90 days for staging/prod

#### Metrics

- CloudWatch Container Insights MUST be enabled for ECS cluster
- Custom metrics MUST be published for request_count, error_count, and latency_percentiles
- Dashboard MUST be created for key metrics: CPU, memory, task count, ALB latency, 5xx rate

#### Alarms

- Alarm for CPU utilization exceeding 80% for 5 minutes (staging, prod)
- Alarm for memory utilization exceeding 85% for 5 minutes (staging, prod)
- Alarm for ALB 5xx error rate exceeding 5% over 2 minutes (staging, prod)
- Alarm for target group healthy host count dropping below minimum (staging, prod)
- Alarm notifications MUST be sent to SNS topic subscribed by the on-call Slack channel

### 2.6 Deployment

#### CI/CD Pipeline

- Pipeline MUST trigger on push to feature/1880-* and main branches
- Pipeline stages MUST include: build, test, docker-build, terraform-plan, terraform-apply
- Terraform plan MUST be reviewed before apply in staging and prod environments

#### Deployment Strategy

- Dev environment MUST auto-deploy on merge to feature/1880-* branch
- Test environment MUST auto-deploy on merge to main branch
- Staging environment MUST require terraform plan review before apply
- Prod environment MUST require manual approval gate before apply

---

## Section 3: Infrastructure Requirements

### 3.1 Required AWS Services

| Service | Usage | Region | Environment |
|---------|-------|--------|-------------|
| ECS Fargate | Container orchestration for Java Card Service | ap-northeast-2 | all |
| Application Load Balancer | Internal traffic distribution and TLS termination | ap-northeast-2 | all |
| API Gateway (REST) | VPC Link integration for /card/v1/* routes | ap-northeast-2 | test, staging, prod |
| Network Load Balancer | Internal NLB as VPC Link target | ap-northeast-2 | test, staging, prod |
| EC2 Container Registry | Container image storage and scanning | ap-northeast-2 | all (shared) |
| CloudWatch Logs | Structured application log aggregation | ap-northeast-2 | all |
| CloudWatch Container Insights | ECS cluster performance metrics | ap-northeast-2 | all |
| IAM | Task roles, execution roles, and policies | ap-northeast-2 | all |
| VPC | Private subnets, security groups, route tables | ap-northeast-2 | all |
| KMS | Encryption key management for secrets and logs | ap-northeast-2 | all (shared) |
| SSM Parameter Store | Environment variable and config injection | ap-northeast-2 | all |
| Secrets Manager | Database credential storage and rotation | ap-northeast-2 | all |
| ACM | TLS certificate management for ALB HTTPS | ap-northeast-2 | all |
| S3 | ALB access logs storage | ap-northeast-2 | all |
| SNS | Alarm notification delivery | ap-northeast-2 | staging, prod |

### 3.2 Terraform Infrastructure

```
infrastructure/terraform/java-fargate-alb/
+-- environments/
|   +-- dev/
|   |   +-- main.tf
|   |   +-- variables.tf
|   |   +-- outputs.tf
|   |   +-- providers.tf
|   |   +-- terraform.tfvars
|   |   +-- backend.tf
|   +-- test/          # same structure as dev
|   +-- staging/       # same structure as dev
|   +-- prod/          # same structure as dev
+-- modules/
|   +-- alb/
|   |   +-- main.tf
|   |   +-- variables.tf
|   |   +-- outputs.tf
|   +-- ecs/
|   |   +-- main.tf
|   |   +-- variables.tf
|   |   +-- outputs.tf
|   +-- api_gateway/
|   |   +-- main.tf
|   |   +-- variables.tf
|   |   +-- outputs.tf
|   +-- ecr/
|   |   +-- main.tf
|   |   +-- variables.tf
|   |   +-- outputs.tf
|   +-- monitoring/
|       +-- main.tf
|       +-- variables.tf
|       +-- outputs.tf
+-- shared/
    +-- iam/
    |   +-- roles.tf
    |   +-- policies.tf
    +-- networking/
        +-- security_groups.tf
```

### 3.3 Container Image Requirements

#### ECR Repository

- Repository name MUST follow convention: java-card-service
- Image tagging MUST include: git commit SHA (7-char short), semantic version (e.g., 1.2.3)
- Image scanning MUST be enabled on push
- Lifecycle policy MUST retain last 30 tagged images and remove untagged images after 7 days

#### Image Build

- Base image: eclipse-temurin:21-jre-alpine (runtime stage)
- Build strategy: multi-stage (build stage with eclipse-temurin:21-jdk, runtime stage with JRE Alpine)
- Dockerfile location: Dockerfile (project root)

---

## Section 4: Integration Requirements

### 4.1 External Service Access

| External Service | Protocol | Auth Method | Data Format | Direction |
|-----------------|----------|-------------|-------------|-----------|
| PostgreSQL Cluster (#1843) | TCP (5432) | IAM Auth + Secrets Manager | SQL | Outbound |
| API Gateway (via VPC Link) | HTTPS | None (app-level auth) | JSON | Inbound |

### 4.2 API Gateway Integration

- **Gateway Type**: REST API
- **Endpoint Type**: Regional
- **Authentication**: None at gateway level (handled by application)
- **Throttling**: Rate 100 rps, Burst 200
- **VPC Link**: Targeting internal NLB on port 80

---

## Section 5: Deployment Requirements

### 5.1 Phased Infrastructure Deployment

#### Phase 1: Development Environment
- **Target**: dev
- **Entry Criteria**: VPC and subnet infrastructure available, PostgreSQL dev instance accessible
- **Actions**:
  1. Deploy ECR repository and IAM roles via Terraform
  2. Deploy ALB, ECS cluster, and ECS service via Terraform
  3. Build and push initial container image to ECR
  4. Verify health checks pass and logs flow to CloudWatch
- **Exit Criteria**: Java Card Service accessible via internal ALB DNS, health check returns HTTP 200

#### Phase 2: Test Environment
- **Target**: test
- **Entry Criteria**: Dev phase complete and validated, PostgreSQL test instance accessible
- **Actions**:
  1. Replicate dev Terraform configuration with test-specific tfvars
  2. Deploy API Gateway route with VPC Link integration
  3. Run integration test suite against test environment
- **Exit Criteria**: All integration tests pass, API Gateway route returns successful responses

#### Phase 3: Staging Environment
- **Target**: staging
- **Entry Criteria**: Test phase complete and validated
- **Actions**:
  1. Deploy staging infrastructure with production-like configuration (2 tasks, auto-scaling)
  2. Enable CloudWatch alarms and SNS notifications
  3. Run performance and load tests against staging
  4. Test deployment rollback procedure
- **Exit Criteria**: Performance tests meet P99 < 500 ms target, rollback procedure verified

#### Phase 4: Production Environment
- **Target**: prod
- **Entry Criteria**: Staging phase complete, all stakeholders approve production deployment
- **Actions**:
  1. Deploy production infrastructure with manual approval gate
  2. Execute production smoke tests
  3. Enable all CloudWatch alarms and verify SNS delivery
  4. Conduct operational runbook review with on-call team
- **Exit Criteria**: Production smoke tests pass, alarms active, on-call team confirms runbook review

### 5.2 Infrastructure Validation

- [ ] All Terraform resources created successfully (terraform plan shows no changes)
- [ ] ALB health checks return HTTP 200 for all targets
- [ ] ECS tasks reach RUNNING status within 120 seconds of deployment
- [ ] CloudWatch logs are flowing from application containers
- [ ] Security group rules allow only required traffic
- [ ] Auto-scaling policies trigger correctly under simulated load

---

## Section 6: Testing Requirements

### 6.1 Infrastructure Validation Tests

| Test ID | Test Description | Expected Result | Environment |
|---------|-----------------|-----------------|-------------|
| IVT-001 | ALB responds on HTTPS port 443 | HTTP 200 response | all |
| IVT-002 | HTTP port 80 redirects to HTTPS | HTTP 301 redirect to port 443 | all |
| IVT-003 | ECS cluster has active services | Service ACTIVE, desired_count matches | all |
| IVT-004 | Target group healthy host count > 0 | healthy_count >= min_capacity | all |
| IVT-005 | CloudWatch log group receives entries | Log stream contains JSON entries | all |
| IVT-006 | API Gateway route returns 200 | /card/v1/health returns HTTP 200 | test, staging, prod |

### 6.2 Multi-Environment Testing

- [ ] Dev infrastructure matches test infrastructure structure (same module references)
- [ ] Staging configuration matches prod configuration (same instance sizes and AZ count)
- [ ] Environment-specific values are correctly applied from tfvars files
- [ ] Cross-environment access is blocked as designed (no dev-to-prod connectivity)

### 6.3 Performance Testing

| Test Scenario | Target Metric | Threshold | Duration |
|---------------|--------------|-----------|----------|
| Baseline load (100 rps) | P99 < 200 ms | 200 ms | 5 min |
| Peak load (1000 rps) | P99 < 500 ms | 500 ms | 10 min |
| Stress test (1500 rps) | No 5xx errors | 0 errors | 15 min |
| Auto-scale trigger | Scale-out within 120 s | 120 s | 10 min |

---

## Section 7: Documentation Requirements

### 7.1 Technical Documentation

- [ ] Architecture decision record (ADR) for ALB + Fargate approach vs EC2 + Docker
- [ ] Terraform module README with input/output variable documentation
- [ ] API specification for /card/v1/* endpoints
- [ ] Integration guide for consuming the Java Card Service from other microservices

### 7.2 Operational Documentation

- [ ] Runbook for deployment procedures (initial deploy and update)
- [ ] Incident response guide for ALB, ECS, and database connectivity failures
- [ ] Scaling operations guide for manual scale-up and scale-down
- [ ] Deployment rollback procedure with step-by-step instructions

---

## Section 8: Dependencies

### 8.1 External Dependencies

| Dependency | Owner | Impact if Delayed | Mitigation |
|------------|-------|-------------------|------------|
| AWS ECS Fargate service quota | AWS Support | Blocks prod deployment if quota insufficient | Request quota increase before Phase 3 |
| ACM certificate issuance | Security Team | Blocks HTTPS setup on ALB | Request certificate in Phase 1 |
| Internal DNS zone delegation | Network Team | Blocks private DNS resolution for ALB | Submit request in Phase 1 |

### 8.2 Internal Dependencies

| Dependency | Issue / PR | Status | Required By |
|------------|-----------|--------|-------------|
| PostgreSQL cluster infrastructure | #1843 | Complete | Phase 1 (dev deployment) |
| VPC and subnet provisioning | Existing | Available | Phase 1 (networking) |
| IAM baseline policies | Existing | Available | Phase 1 (roles) |

### 8.3 Reference Implementations

- **Existing ECS Fargate Module**: `infrastructure/terraform/java-fargate-alb/modules/ecs/main.tf`
- **Environment tfvars Example**: `infrastructure/terraform/java-fargate-alb/environments/dev/terraform.tfvars`
- **ALB Module Reference**: `infrastructure/terraform/java-fargate-alb/modules/alb/main.tf`

---

## Section 9: Acceptance Criteria Summary

### Per-Environment Acceptance

#### Development Environment
- [ ] Infrastructure deployed and validated in dev via terraform apply
- [ ] Java Card Service accessible via internal ALB DNS in dev
- [ ] Health checks passing consistently in dev (HTTP 200 on /actuator/health)
- [ ] Logs flowing to CloudWatch log group /ecs/java-card-service in dev
- [ ] Container image builds and pushes to ECR successfully

#### Test Environment
- [ ] Infrastructure deployed and validated in test via terraform apply
- [ ] Health checks passing consistently in test
- [ ] API Gateway route returns successful response in test
- [ ] Integration tests pass against test environment

#### Staging Environment
- [ ] Infrastructure deployed with production-like configuration in staging (2 tasks, auto-scaling enabled)
- [ ] Performance tests pass with P99 < 500 ms at 1000 rps in staging
- [ ] CloudWatch alarms fire correctly under simulated failure in staging
- [ ] Deployment rollback procedure tested and verified in staging
- [ ] SNS alarm notifications delivered to on-call Slack channel

#### Production Environment
- [ ] Infrastructure deployed and validated in prod with manual approval
- [ ] Health checks passing consistently in prod
- [ ] All CloudWatch alarms active and verified in prod
- [ ] Operational runbooks reviewed and approved by on-call team
- [ ] ALB access logs confirmed writing to S3

### Infrastructure Capability Checklist

- [ ] ALB with HTTPS termination is operational in all environments
- [ ] ECS Fargate cluster is running in each environment
- [ ] Container images are building and pushing to ECR with scanning enabled
- [ ] Auto-scaling policies are configured and tested in staging and prod
- [ ] Environment isolation is verified (no cross-environment IAM or network access)
- [ ] Security groups follow least-privilege rules

### Operations Checklist

- [ ] CloudWatch monitoring dashboards are created for staging and prod
- [ ] Alerting thresholds are configured for CPU, memory, and 5xx error rate
- [ ] On-call team has reviewed operational runbooks
- [ ] Deployment rollback procedure is tested in staging

> **Important Note**: All acceptance criteria MUST be verified in the target environment before marking as complete. Dev environment validation alone is not sufficient for staging or prod sign-off.

---

## Related Documentation

- **GAP Analysis**: `devops/docs/requirements/GAP_1880_alb_fargate_java_card_service.md`
- **Task List**: `devops/docs/requirements/TASKS_1880_alb_fargate_java_card_service.md`
- **GitHub Issue**: #1880 - ALB + Fargate Infrastructure for Java Container Deployment

---

**Document Version**: 2.0
**Last Updated**: 2025-01-20
**Changes**: Version 2.0 - Stakeholder review complete, requirements approved for implementation
**Next Review**: Upon completion of Phase 2 (test environment validation)
