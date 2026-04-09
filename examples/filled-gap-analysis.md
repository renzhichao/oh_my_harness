# Baseline & GAP Analysis - ALB + Fargate Infrastructure for Java Card Service

**Document ID**: GAP_ALB_FARGATE_JAVA_CARD_SERVICE

**Issue**: #1880 - ALB + Fargate Infrastructure for Java Container Deployment

**Created**: 2025-01-15

**Branch**: feature/1880-alb-fargate-java-card-service

**Author**: arthurren

**Reviewers**: platform-team-lead, devops-lead

**Status**: Analysis Phase

**Dependencies**: #1843 (PostgreSQL Cluster Infrastructure - provisioned), #1876 (VPC Network Foundation - provisioned)

---

## Executive Summary

### Current Baseline Status: 🔴 **Infrastructure Requires Modernization**

- **Compute**: 15% (EC2 instances with manual scaling, no containerization)
- **Networking**: 10% (Classic Load Balancer, single public subnet, no API Gateway)
- **Deployment**: 5% (Manual SSH-based deployment, no CI/CD pipeline for containers)
- **Observability**: 25% (Basic CloudWatch metrics, unstructured logging)
- **Security**: 10% (Hardcoded credentials, no TLS termination, no secret rotation)
- **Infrastructure as Code**: 30% (Partial CloudFormation templates, no Terraform)

### Key Findings

1. ✅ **Existing VPC Foundation**: VPC with public subnets already provisioned in ap-northeast-1 (Issue #1876), provides base networking layer
2. ✅ **PostgreSQL Cluster**: RDS PostgreSQL cluster available and operational (Issue #1843), serves as persistence layer
3. ✅ **AWS Account Structure**: Multi-environment account strategy (dev/test/staging/prod) established with IAM baselines
4. ❌ **ECS Fargate Compute**: No ECS cluster, task definitions, or container orchestration exists for the Java card service
5. ❌ **Application Load Balancer**: No ALB with HTTPS termination, path-based routing, or health check configuration
6. ❌ **Container Registry**: No ECR repository for Java container images, no Dockerfile for the card service
7. ❌ **API Gateway Integration**: No REST API, VPC Link, or request/response transformation layer
8. ❌ **CI/CD Pipeline**: No automated build, test, and deploy pipeline for container-based deployments
9. ❌ **Terraform Modules**: No Terraform coverage for the Java card service infrastructure stack

### Critical Gap: Manual EC2 to Containerized Fargate Migration

**Requirement** (Issue #1880): Migrate the Java card service from manually-managed EC2 instances to ECS Fargate with ALB, API Gateway integration, and fully automated CI/CD.

#### Option A: ECS EC2 Launch Type (Considered but Not Selected)
- **Self-managed compute**: Run ECS on EC2 instances the team manages directly
- **Benefits**: More control over underlying host OS, support for persistent storage via EBS, predictable per-instance cost
- **Not Selected**: Adds operational burden for patch management, capacity planning, and instance lifecycle management. Contradicts the platform team's serverless-first directive.

#### Option B: ECS Fargate + ALB + API Gateway ✅ **SELECTED**

**Serverless container orchestration**:
- **ECS Fargate**: Fully managed compute with auto-scaling, no instance management
- **Application Load Balancer**: HTTPS termination, path-based routing, health checks
- **API Gateway**: Public-facing REST API with VPC Link to private ALB
- **ECR**: Private container registry with image scanning

**Benefits**:
- ✅ Eliminates EC2 instance management overhead entirely
- ✅ Auto-scaling based on CPU/memory utilization and request count
- ✅ ALB provides native health checks and graceful drain connections
- ✅ API Gateway adds rate limiting, throttling, and request validation

**✅ Decision Made**: ECS Fargate approach selected for implementation per Issue #1880 requirements

---

## Section 0: Architecture Comparison

### 0.1 Current Architecture (EC2 + Classic Load Balancer)

```
┌─────────────────────────────────────────────────────────────┐
│                    Internet / Client                          │
│              Public HTTPS requests to card API               │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │   Classic Load Balancer       │
        │   - HTTP only (port 80)       │
        │   - No path-based routing     │
        │   - No health checks          │
        └───────────────┬───────────────┘
                        │
        ┌───────────────┴───────────────┐
        │                               │
        ▼                               ▼
┌───────────────┐              ┌───────────────┐
│  EC2 (t3.large)│              │  EC2 (t3.large)│
│  10.0.1.10    │              │  10.0.1.11    │
│  Java JAR     │              │  Java JAR     │
│  Manual deploy│              │  Manual deploy│
└───────┬───────┘              └───────┬───────┘
        │                               │
        └───────────────┬───────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │  RDS PostgreSQL  │
              │  card-db.cluster │
              └──────────────────┘
```

### 0.2 Target Architecture (ECS Fargate + ALB + API Gateway)

```
┌─────────────────────────────────────────────────────────────┐
│                    Internet / Client                          │
│              HTTPS requests to api.example.com                │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │   API Gateway (REST API)      │
        │   - Rate limiting (1000/min)  │
        │   - Request validation        │
        │   - Custom domain + ACM cert  │
        └───────────────┬───────────────┘
                        │ VPC Link
                        ▼
        ┌───────────────────────────────┐
        │   ALB (Internal)              │
        │   - HTTPS (TLS 1.3)           │
        │   - Path: /api/v1/cards/*     │
        │   - Health: /health           │
        │   - Draining: 300s            │
        └───────────────┬───────────────┘
                        │
        ┌───────────────┴───────────────┐
        │                               │
        ▼                               ▼
┌───────────────┐              ┌───────────────┐
│  Fargate Task │              │  Fargate Task │
│  0.5 vCPU     │              │  0.5 vCPU     │
│  1.0 GB RAM   │              │  1.0 GB RAM   │
│  Java 17 JRE  │              │  Java 17 JRE  │
│  Docker       │              │  Docker       │
└───────┬───────┘              └───────┬───────┘
        │                               │
        └───────────────┬───────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │  RDS PostgreSQL  │
              │  card-db.cluster │
              └──────────────────┘
```

**Environment Isolation Architecture**:

```
┌───────────────────────────────────────────────────────┐
│         ECS Cluster: card-service-cluster              │
│                                                        │
│  Environment Isolation via namespace:                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │   dev   │  │  test   │  │ staging │  │   prod  │  │
│  │ 1 task  │  │ 1 task  │  │ 2 tasks │  │ 3 tasks │  │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │
└───────────────────────────────────────────────────────┘
```

---

## Section 1: Current Architecture Baseline

### 1.1 Compute Infrastructure

#### ✅ EXISTING ASSETS

**EC2 Instance Configuration** (`infrastructure/ec2/card-service/`):
```hcl
# Current EC2 provisioning for Java card service
resource "aws_instance" "card_service" {
  count         = 2
  ami           = "ami-0a1b2c3d4e5f6g7h8"
  instance_type = "t3.large"
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "card-service-${count.index}"
    Env  = "production"
  }
}
```

**EC2 Features**:
- ✅ Two t3.large instances (2 vCPU, 8 GB RAM each) running Java 11 JAR
- ✅ Auto-recovery via CloudWatch alarm action recovery
- ✅ Basic instance-level monitoring (CPU, network, disk at 5-minute intervals)

#### ❌ MISSING ECS FARGATE INFRASTRUCTURE

```bash
# ECS Fargate Infrastructure (🔴 CRITICAL GAP)
- ❌ No ECS cluster defined for card-service
- ❌ No Fargate task definition (container spec, CPU/memory limits)
- ❌ No ECS service with desired count and auto-scaling policy
- ❌ No capacity provider strategy for Fargate Spot fallback
- ❌ No ECR repository for container image storage
```

**GAP SEVERITY**: 🔴 **CRITICAL** - The entire container orchestration layer is absent. Without ECS Fargate, the service cannot achieve auto-scaling, immutable deployments, or infrastructure-as-code parity.

### 1.2 Networking and Load Balancing

#### ✅ EXISTING VPC INFRASTRUCTURE (Issue #1876)

**VPC Configuration** (from `infrastructure/terraform/vpc-foundation/`):
- ✅ VPC `vpc-0abc123def456` in ap-northeast-1 (CIDR: 10.0.0.0/16)
- ✅ Two public subnets in ap-northeast-1a and ap-northeast-1c
- ✅ Internet Gateway attached for outbound connectivity
- ✅ Route tables configured for public subnet internet access

#### ❌ MISSING APPLICATION LOAD BALANCER AND PRIVATE NETWORKING

```bash
# ALB and Private Networking (🔴 CRITICAL GAP)
- ❌ No Application Load Balancer (internal or internet-facing)
- ❌ No ALB target group for card-service Fargate tasks
- ❌ No ALB listener rules (HTTPS on 443, redirect HTTP to HTTPS)
- ❌ No private subnets for Fargate task placement
- ❌ No NAT Gateway for private subnet outbound connectivity
- ❌ No VPC Link for API Gateway to ALB integration
- ❌ No ACM certificate for TLS termination
```

**GAP SEVERITY**: 🔴 **CRITICAL** - The Classic Load Balancer cannot route to Fargate tasks. ALB is a prerequisite for ECS integration, health checks, and HTTPS termination.

### 1.3 Containerization and Deployment

#### ✅ EXISTING DEPLOYMENT MECHANISM

**Current Shell-Based Deployment** (`scripts/deploy-card-service.sh`):
```bash
#!/bin/bash
# Manual deployment script for card service
ssh ec2-user@10.0.1.10 "cd /opt/card-service && ./stop.sh"
scp ./target/card-service.jar ec2-user@10.0.1.10:/opt/card-service/
ssh ec2-user@10.0.1.10 "cd /opt/card-service && ./start.sh"
```

**Deployment Features**:
- ✅ Working stop/start shell scripts on each EC2 instance
- ✅ JAR artifact built via Maven (`mvn clean package`)

#### ❌ MISSING CONTAINER AND CI/CD INFRASTRUCTURE

```bash
# Container and CI/CD (🔴 CRITICAL GAP)
- ❌ No Dockerfile for Java 17 container image
- ❌ No ECR repository with lifecycle policies
- ❌ No CodePipeline or GitHub Actions workflow for CI/CD
- ❌ No container image build and push stage
- ❌ No ECS service update deployment stage
- ❌ No blue/green or rolling deployment strategy
```

**GAP SEVERITY**: 🔴 **CRITICAL** - Without containerization and CI/CD, the migration to Fargate cannot proceed. This is the foundational gap for the entire project.

### 1.4 Observability

#### ✅ EXISTING MONITORING

**CloudWatch Configuration**:
- ✅ Basic EC2 CPU utilization alarms (threshold: 80%)
- ✅ CloudWatch Agent for memory/disk metrics on EC2
- ✅ CloudWatch Logs log group `/ec2/card-service` (unstructured text)

#### ❌ MISSING CONTAINER OBSERVABILITY

```bash
# Observability (🟡 HIGH GAP)
- ❌ No ECS Container Insights enabled
- ❌ No structured JSON logging from Java application
- ❌ No AWS X-Ray distributed tracing
- ❌ No CloudWatch Logs log group for Fargate task stdout/stderr
- ❌ No CloudWatch dashboard for ECS service metrics
- ❌ No alarm on task restart count or deployment failures
```

**GAP SEVERITY**: 🟡 **HIGH** - Observability gaps impact operational readiness but do not block initial deployment. Can be addressed incrementally after Phase 2.

### 1.5 Security

#### ✅ EXISTING SECURITY PATTERNS

**Current Security Configuration**:
- ✅ EC2 security groups restrict inbound to Classic LB only
- ✅ RDS security group allows inbound from EC2 instances only
- ✅ IAM instance profile for EC2 with basic S3 read access

#### ❌ MISSING SECURITY CONTROLS

```bash
# Security (🟡 HIGH GAP)
- ❌ No AWS Secrets Manager integration for database credentials
- ❌ No TLS 1.3 termination at ALB (current: HTTP only)
- ❌ No IAM task execution role for ECS Fargate
- ❌ No IAM task role for runtime AWS API calls
- ❌ No AWS WAF rules on ALB for common attack protection
- ❌ No container image vulnerability scanning in ECR
```

**GAP SEVERITY**: 🟡 **HIGH** - Security gaps expose the service to credential leakage and unencrypted traffic. Must be addressed before production traffic migration.

### 1.6 Infrastructure as Code

#### ✅ EXISTING IaC COVERAGE

**CloudFormation Templates** (`infrastructure/cloudformation/`):
```yaml
# Partial CloudFormation for VPC (legacy)
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
```

**IaC Features**:
- ✅ Partial CloudFormation templates for VPC and RDS (legacy, not maintained)
- ✅ Team experience with Terraform from other projects

#### ❌ MISSING TERRAFORM COVERAGE

```bash
# Terraform Modules (🟡 HIGH GAP)
- ❌ No Terraform module for ECS cluster
- ❌ No Terraform module for ALB + target group + listeners
- ❌ No Terraform module for API Gateway + VPC Link
- ❌ No Terraform module for ECR repository
- ❌ No environment-specific tfvars files (dev/test/staging/prod)
- ❌ No remote state backend configuration for card-service stack
```

**GAP SEVERITY**: 🟡 **HIGH** - Infrastructure cannot be reliably reproduced or promoted across environments without Terraform coverage. Manual AWS Console changes create configuration drift.

---

## Section 2: ECS Fargate Approach Analysis

### 2.1 Gap Analysis Summary Matrix

| Gap ID | Category | Current State | Target State | Impact |
|--------|----------|---------------|--------------|--------|
| GAP-001 | Compute | EC2 instances (t3.large) with manual scaling | ECS Fargate (0.5 vCPU, 1 GB) with auto-scaling | 🔴 CRITICAL |
| GAP-002 | Networking | Classic LB, HTTP only, single public subnet | ALB with HTTPS/TLS 1.3, private subnets, API Gateway | 🔴 CRITICAL |
| GAP-003 | Deployment | Manual SSH, shell scripts, JAR artifacts | ECR + Docker + CodePipeline CI/CD | 🔴 CRITICAL |
| GAP-004 | Observability | Basic CloudWatch, unstructured logs | Container Insights, structured JSON, X-Ray tracing | 🟡 HIGH |
| GAP-005 | Security | Hardcoded credentials, no TLS, no IAM roles | Secrets Manager, TLS 1.3, IAM task roles | 🟡 HIGH |
| GAP-006 | Infrastructure as Code | Partial CloudFormation (unmaintained) | Full Terraform module coverage | 🟡 HIGH |

### 2.2 Comparative Analysis

| Aspect | ECS EC2 Launch Type | ECS Fargate |
|--------|---------------------|-------------|
| **Infrastructure Complexity** | High - manage EC2 fleet, AMI patching, capacity | Low - AWS manages compute, focus on task config |
| **ALB Integration** | Same ALB integration, manual instance registration | Native target group integration, automatic registration |
| **Performance** | Slightly lower cold start (warm instances) | Cold start ~30-60s for Java containers |
| **Cost (Low Traffic)** | Fixed EC2 cost (~$120/mo per t3.large) | Pay-per-task (~$15/mo per 0.5 vCPU task) |
| **Cost (High Traffic)** | Better at sustained high utilization | Scales linearly, no over-provisioning waste |
| **Scaling** | Manual or autoscaling group (minutes) | Service auto-scaling (seconds to add tasks) |
| **Implementation Time** | Slower - requires EC2 provisioning, AMI baking | Faster - focus on task definition and networking |

**Recommendation**: ECS Fargate is the preferred approach. Lower operational overhead, faster implementation, and alignment with the platform team serverless-first strategy outweigh the cold start trade-off for a card service with predictable traffic patterns.

### 2.3 ECS Fargate Implementation Gaps

#### ECR Repository and Container Image
```bash
# Container Registry (🔴 CRITICAL GAP)
- ❌ No ECR repository `card-service` with image scanning
- ❌ No Dockerfile (multi-stage: Maven build + JRE 17 runtime)
- ❌ No lifecycle policy to retain last 10 images
- ❌ No container image for Java 17 card-service application
```

#### ECS Task Definition
```bash
# Task Definition (🔴 CRITICAL GAP)
- ❌ No task definition with 0.5 vCPU / 1 GB memory
- ❌ No container definition with port mapping (8080)
- ❌ No log configuration for CloudWatch Logs
- ❌ No environment variable injection from SSM/Secrets Manager
- ❌ No health check command in container definition
```

#### VPC and Subnet Configuration
```bash
# Private Networking (🔴 CRITICAL GAP)
- ❌ No private subnets for Fargate task placement (10.0.10.0/24, 10.0.11.0/24)
- ❌ No NAT Gateway for private subnet egress
- ❌ No security group for Fargate tasks (allow 8080 from ALB only)
- ❌ No security group for ALB (allow 443 from VPC Link)
```

#### API Gateway Integration
```bash
# API Gateway (🟡 HIGH GAP)
- ❌ No REST API with `/api/v1/cards` resource
- ❌ No VPC Link to NLB/ALB in private subnets
- ❌ No custom domain with ACM certificate
- ❌ No usage plan or throttling configuration
```

#### CI/CD Pipeline
```bash
# CI/CD (🟡 HIGH GAP)
- ❌ No CodePipeline with GitHub source stage
- ❌ No CodeBuild stage for Docker build and ECR push
- ❌ No deploy stage with ECS service update
- ❌ No manual approval gate for production deployment
```

---

## Section 3: Infrastructure GAP Analysis

### 3.1 Networking Infrastructure

#### ✅ EXISTING NETWORKING

**VPC Configuration** (from Issue #1876):
- ✅ VPC `vpc-0abc123def456` in ap-northeast-1 (10.0.0.0/16)
- ✅ Public subnets in ap-northeast-1a (10.0.0.0/24) and ap-northeast-1c (10.0.1.0/24)
- ✅ Internet Gateway and public route tables

#### ❌ MISSING PRIVATE NETWORKING FOR FARGATE

```bash
# Private Subnets and NAT (🟡 HIGH GAP)
- ❌ No private subnets in ap-northeast-1a (10.0.10.0/24) and ap-northeast-1c (10.0.11.0/24)
- ❌ No NAT Gateway in each AZ for private subnet egress
- ❌ No private route tables with NAT Gateway default route
- ❌ No VPC Link integration subnet group
```

**GAP SEVERITY**: 🟡 **HIGH** - Fargate tasks should run in private subnets for security. NAT Gateway enables outbound connectivity without exposing tasks to the internet.

### 3.2 Security Configuration

#### ✅ EXISTING SECURITY PATTERNS

**EC2 Security Groups** (reference):
- ✅ Security group allows inbound TCP/8080 from Classic LB security group
- ✅ RDS security group allows inbound TCP/5432 from EC2 security group

#### ❌ MISSING FARGATE SECURITY CONTROLS

```bash
# Security (🟡 HIGH GAP)
- ❌ No Secrets Manager secret for RDS credentials with auto-rotation
- ❌ No ACM certificate in ap-northeast-1 for `*.api.example.com`
- ❌ No IAM task execution role with ECR pull + CloudWatch Logs + Secrets Manager access
- ❌ No IAM task role for runtime AWS SDK calls (S3, SQS)
- ❌ No WAF WebACL on ALB for SQL injection and XSS protection
```

**GAP SEVERITY**: 🟡 **HIGH** - Security controls must be in place before production traffic. Development can proceed with self-signed certificates and basic IAM roles.

### 3.3 IAM Roles and Permissions

#### ✅ EXISTING IAM PATTERNS

**EC2 IAM** (current):
- ✅ Instance profile with `AmazonS3ReadOnlyAccess` for artifact downloads
- ✅ Instance profile with `CloudWatchAgentServerPolicy` for metrics

**Other ECS Services** (reference):
- ✅ `infrastructure/terraform/java-fargate-alb/` has IAM patterns for ECS task roles
- ✅ Task execution role template with ECR and CloudWatch Logs permissions

#### ❌ MISSING CARD-SERVICE IAM ROLES

```bash
# IAM Roles (🟡 HIGH GAP)
- ❌ No task execution role `card-service-task-execution-role`
- ❌ No task role `card-service-task-role` with least-privilege policies
- ❌ No IAM policy for Secrets Manager read access (RDS credentials)
- ❌ No IAM policy for KMS decrypt (if encryption at rest required)
```

**GAP SEVERITY**: 🟡 **HIGH** - IAM roles are required for Fargate task launch. Reference implementation exists and can be adapted.

### 3.4 Monitoring and Logging

#### ✅ EXISTING MONITORING

**CloudWatch Setup**:
- ✅ CloudWatch Agent on EC2 for memory and disk metrics
- ✅ Log group `/ec2/card-service` with 30-day retention
- ✅ SNS topic `ops-alerts` for alarm notifications

#### ❌ MISSING CONTAINER MONITORING

```bash
# Container Monitoring (🟢 MEDIUM GAP)
- ❌ No CloudWatch Container Insights for ECS cluster
- ❌ No structured JSON log format in Java application
- ❌ No CloudWatch dashboard `card-service-ops`
- ❌ No alarms on CPU utilization > 70%, memory > 80%
- ❌ No X-Ray tracing for request correlation
```

**GAP SEVERITY**: 🟢 **MEDIUM** - Monitoring can be enhanced incrementally post-launch. Basic CloudWatch Logs will be available from Fargate task stdout by default.

### 3.5 Auto-Scaling Configuration

#### ✅ EXISTING AUTO-SCALING PATTERNS

**Reference Auto-Scaling** (from `infrastructure/terraform/java-fargate-alb/`):
- ✅ ECS service auto-scaling based on CPU utilization target tracking
- ✅ Scalable target with min/max task count configuration
- ✅ Step scaling policy for rapid scale-out on high request count

#### ❌ MISSING CARD-SERVICE AUTO-SCALING

```bash
# Auto-Scaling (🟢 MEDIUM GAP)
- ❌ No scalable target for card-service ECS service
- ❌ No target tracking policy (CPU target: 65%, memory target: 75%)
- ❌ No ALB request count per target scaling metric
- ❌ No scheduled scaling for business hours (min 3) vs off-hours (min 1)
```

**GAP SEVERITY**: 🟢 **MEDIUM** - Auto-scaling is not required for initial deployment. A fixed task count (2) is sufficient for launch, with scaling added based on observed traffic patterns.

---

## Section 4: Implementation GAP Analysis

### 4.1 Terraform Infrastructure Code

#### ✅ EXISTING TERRAFORM MODULES

**Reference Modules**:
- ✅ `infrastructure/terraform/java-fargate-alb/` - Full ECS Fargate + ALB pattern for Java services
- ✅ `infrastructure/terraform/vpc-foundation/` - VPC module that can be extended with private subnets

#### ❌ MISSING CARD-SERVICE TERRAFORM

```bash
# Terraform Modules (🔴 CRITICAL GAP)
- ❌ No `infrastructure/terraform/card-service/` module directory
- ❌ No `main.tf` with provider and backend configuration
- ❌ No `ecs.tf` with cluster, task definition, and service resources
- ❌ No `alb.tf` with ALB, target group, listener, and rules
- ❌ No `apigateway.tf` with REST API and VPC Link
- ❌ No `ecr.tf` with repository and lifecycle policy
- ❌ No `iam.tf` with task execution and task roles
- ❌ No environment tfvars files for dev/test/staging/prod
```

**GAP SEVERITY**: 🔴 **CRITICAL** - All infrastructure must be defined in Terraform. The reference module provides a working pattern to adapt.

### 4.2 CI/CD Pipeline

#### ✅ EXISTING CI/CD PATTERNS

**Current Deployment**:
- ✅ Maven build produces JAR artifact (`mvn clean package`)
- ✅ GitHub repository with branch protection rules
- ✅ Existing CodePipeline for other Java services (reference)

#### ❌ MISSING CARD-SERVICE CI/CD

```bash
# CI/CD Pipeline (🟡 HIGH GAP)
- ❌ No CodePipeline `card-service-pipeline`
- ❌ No CodeBuild project for Docker build (Dockerfile + ECR push)
- ❌ No source stage with GitHub webhook trigger
- ❌ No deploy stage with ECS service update action
- ❌ No manual approval stage before production deploy
- ❌ No buildspec.yml for container image build
```

**GAP SEVERITY**: 🟡 **HIGH** - CI/CD enables repeatable deployments. The reference pipeline from the Java Fargate ALB project can be adapted.

### 4.3 Configuration Management

#### ✅ EXISTING CONFIGURATION

**Current Configuration**:
- ✅ `application.yml` with Spring Boot externalized configuration
- ✅ Environment variable overrides for database URL and credentials
- ✅ SSM Parameter Store used for non-secret configuration values

#### ❌ MISSING CONTAINER CONFIGURATION

```bash
# Container Configuration (🟢 MEDIUM GAP)
- ❌ No environment variable injection via ECS task definition
- ❌ No Secrets Manager secret references in task definition
- ❌ No SSM Parameter Store references for feature flags
- ❌ No log driver configuration for Fluentd or FireLens
```

**GAP SEVERITY**: 🟢 **MEDIUM** - Configuration management can be incrementally improved. Initial deployment can use environment variables with Secrets Manager references.

---

## Section 5: Migration Path Analysis

### 5.1 Current State to Target State

**Current Architecture**:
```
Internet
    |
    v
Classic Load Balancer (HTTP/80)
    |
    v
EC2 t3.large x2 (Java 11 JAR, manual deploy)
    |
    v
RDS PostgreSQL
```

**Target Architecture**:
```
Internet
    |
    v
API Gateway (HTTPS/443, rate limiting)
    | (VPC Link)
    v
ALB Internal (HTTPS/443, TLS 1.3)
    |
    v
ECS Fargate Tasks (Java 17 Docker, auto-scaling)
    |
    v
RDS PostgreSQL (unchanged)
```

### 5.2 Migration Steps (Detailed)

#### Phase 1: Foundation (Week 1)
1. ✅ Create private subnets and NAT Gateway in Terraform (`vpc.tf`)
2. ✅ Provision ECR repository `card-service` with lifecycle policy
3. ✅ Create IAM task execution role and task role with least-privilege policies
4. ✅ Request ACM certificate for `api.example.com` in ap-northeast-1
5. ✅ Create Secrets Manager secret for RDS credentials with rotation enabled

#### Phase 2: Compute Layer (Week 2)
1. ✅ Write Dockerfile (multi-stage: Maven build + Eclipse Temurin JRE 17)
2. ✅ Define ECS task definition with 0.5 vCPU, 1 GB memory, port 8080
3. ✅ Create ECS cluster `card-service-cluster`
4. ✅ Create ALB (internal) with target group, HTTPS listener, health check path `/health`
5. ✅ Create ECS service with desired count 2, attach to ALB target group

#### Phase 3: Routing and Integration (Week 3)
1. ✅ Create API Gateway REST API with `/api/v1/cards` resource
2. ✅ Configure VPC Link to ALB in private subnets
3. ✅ Set up custom domain with ACM certificate
4. ✅ Configure Route53 DNS record for `api.example.com`

#### Phase 4: CI/CD and Validation (Week 4)
1. ✅ Create CodePipeline with GitHub source, CodeBuild, and ECS deploy stages
2. ✅ Build and push initial container image to ECR
3. ✅ Deploy to dev environment and validate health checks
4. ✅ Promote to staging, run integration test suite
5. ✅ Production cutover: switch DNS from Classic LB to API Gateway

#### Phase 5: Observability and Optimization (Week 5)
1. ✅ Enable CloudWatch Container Insights on ECS cluster
2. ✅ Create CloudWatch dashboard with service metrics
3. ✅ Configure auto-scaling policies (CPU target: 65%, min 2, max 10)
4. ✅ Decommission EC2 instances and Classic Load Balancer

---

## Section 6: Risk Assessment

### 6.1 High Risk Areas

1. **RISK-001: ECS Fargate Cold Start Latency**
   - **Risk**: Java container cold starts may take 30-60 seconds, causing timeout errors during scale-up events or task replacement
   - **Mitigation**: Set minimum healthy count to 2 tasks, configure ALB health check interval to 15 seconds with 3 healthy threshold, pre-warm tasks before traffic spikes using scheduled scaling

2. **RISK-002: Classic LB to ALB Migration Traffic Disruption**
   - **Risk**: DNS cutover from Classic LB to API Gateway may cause dropped connections during propagation (TTL: 300 seconds)
   - **Mitigation**: Use weighted Route53 routing (90/10 split), monitor error rates on new endpoint for 30 minutes, maintain Classic LB for rollback during cutover window

3. **RISK-003: Container Image Vulnerability Exposure**
   - **Risk**: Base image or dependency vulnerabilities in the Docker container may go undetected without automated scanning
   - **Mitigation**: Enable ECR image scanning on push, integrate Trivy scan in CodeBuild pipeline, block deployment on CRITICAL/HIGH severity findings

4. **RISK-004: Secrets Manager Rotation Downtime**
   - **Risk**: Database credential rotation may cause brief connection failures if the Java application does not handle credential refresh
   - **Mitigation**: Implement connection pool refresh logic in Spring Boot, use rotation with `test` mode before `apply`, schedule rotation during low-traffic window (02:00 JST)

### 6.2 Medium Risk Areas

1. **RISK-005: API Gateway Throttling Misconfiguration**
   - **Risk**: Default rate limits may be too aggressive (1000 req/min) or too permissive for production traffic patterns
   - **Mitigation**: Analyze current traffic volume from Classic LB access logs, set initial burst limit at 200% of peak observed traffic, create CloudWatch alarm on 429 responses

2. **RISK-006: Cross-AZ Fargate Task Distribution**
   - **Risk**: Fargate tasks may not distribute evenly across ap-northeast-1a and ap-northeast-1c, reducing availability during AZ failure
   - **Mitigation**: Configure ECS service with `azAware` capacity provider strategy, ensure private subnets exist in both AZs with equal capacity

3. **RISK-007: Terraform State Drift During Migration**
   - **Risk**: Manual AWS Console changes during migration may drift from Terraform state, causing plan/apply conflicts
   - **Mitigation**: Lock down IAM write permissions to console users, enforce all changes via Terraform PR, run `terraform plan` in CI before apply

---

## Section 7: Recommendations

### 7.1 Selected Approach: ECS Fargate + ALB + API Gateway ✅ **DECISION MADE**

**Decision**: ECS Fargate with Application Load Balancer and API Gateway has been selected for the Java card service migration per Issue #1880 requirements.

**Rationale**:
1. ✅ **Operational Efficiency**: Eliminates EC2 instance management (patching, capacity planning, SSH access), reducing operational toil by an estimated 8 hours per month
2. ✅ **Auto-Scaling**: Fargate service auto-scaling responds to traffic changes in seconds versus minutes for EC2 autoscaling groups, improving cost efficiency during low-traffic periods
3. ✅ **Security Posture**: ALB provides TLS 1.3 termination, API Gateway adds rate limiting and WAF integration, eliminating hardcoded credentials via Secrets Manager
4. ✅ **Deployment Velocity**: CI/CD pipeline with container images enables daily deployments versus the current monthly manual release cycle
5. ✅ **Platform Consistency**: Aligns with existing Java Fargate ALB pattern already in production for other services, enabling knowledge reuse and shared modules

**Implementation Priority**:
1. **High**: Terraform modules for ECS cluster, ALB, and IAM roles (blocks all downstream work)
2. **High**: Dockerfile and ECR repository (enables container builds)
3. **High**: Private subnets and NAT Gateway (required for Fargate task placement)
4. **Medium**: API Gateway and VPC Link (can be deferred, direct ALB access as interim)
5. **Low**: X-Ray tracing and advanced observability (post-launch enhancement)

### 7.2 Alternative Approach: ECS EC2 Launch Type - Not Selected

**Rationale**:
1. ✅ **Lower Cold Start**: Warm EC2 instances eliminate Java cold start latency
2. ✅ **Persistent Storage**: EBS volumes available for local caching or state
3. ✅ **Cost at Scale**: Cheaper at sustained high utilization (>70% average CPU)

**ECS EC2 Launch Type was considered but not selected**:
- Requires EC2 instance lifecycle management (patching, scaling, replacement)
- Adds 2-3 weeks to implementation timeline for AMI pipeline and capacity planning
- Contradicts platform team serverless-first strategy established in Q4 2024

---

## Section 8: Success Criteria

### 8.1 ECS Fargate Approach Success Criteria

**Infrastructure**:
- [ ] ECS cluster `card-service-cluster` running in ap-northeast-1 with Fargate capacity provider
- [ ] ALB target group health checks passing on `/health` endpoint for all registered tasks
- [ ] ECR repository `card-service` with image scanning enabled and lifecycle policy retaining last 10 images
- [ ] Private subnets in both ap-northeast-1a and ap-northeast-1c with NAT Gateway for egress
- [ ] All infrastructure defined in Terraform with environment-specific tfvars for dev/test/staging/prod

**Integration**:
- [ ] API Gateway REST API serving requests at `https://api.example.com/api/v1/cards/*`
- [ ] VPC Link connecting API Gateway to internal ALB in private subnets
- [ ] Route53 DNS record with ACM certificate for HTTPS termination
- [ ] Secrets Manager secret for RDS credentials with rotation enabled and tested
- [ ] IAM task roles with least-privilege policies (no `AdministratorAccess` or `*:*`)

**Performance**:
- [ ] p99 API response latency under 500ms (measured at API Gateway)
- [ ] ECS Fargate task startup time under 90 seconds (including Java application bootstrap)
- [ ] Auto-scaling responds to 2x traffic spike within 3 minutes
- [ ] Zero-downtime deployment during ECS service update (rolling deployment)

---

## Section 9: Dependencies

### 9.1 External Dependencies

- **AWS ECS Fargate**: Status: existing - Service limit increase request submitted (current: 2 tasks, target: 10 tasks per service)
- **AWS ALB**: Status: existing - Available in ap-northeast-1, no limit increase needed
- **AWS API Gateway**: Status: existing - REST API available, VPC Link supported in ap-northeast-1
- **ACM Certificate**: Status: new - Certificate request for `*.api.example.com` submitted, DNS validation pending

### 9.2 Internal Dependencies

- ✅ **VPC Foundation** (Issue #1876): Provisioned - VPC, public subnets, IGW available
- ✅ **RDS PostgreSQL** (Issue #1843): Operational - Card database cluster running in ap-northeast-1
- ❌ **Private Subnets**: Status: new - Must be added to existing VPC before Fargate task deployment
- ❌ **Container Image**: Status: new - Dockerfile and initial build required

### 9.3 Reference Implementations

- **Java Fargate ALB Pattern**: `infrastructure/terraform/java-fargate-alb/`
- **VPC Foundation Module**: `infrastructure/terraform/vpc-foundation/`
- **ECS CI/CD Pipeline**: `infrastructure/codepipeline/java-service-pipeline/`

---

## Section 10: Next Steps

### 10.1 Immediate Actions

1. **Create Terraform Module Structure**
   - Initialize `infrastructure/terraform/card-service/` with `main.tf`, `variables.tf`, `outputs.tf`
   - Expected outcome: Empty module ready for resource definitions

2. **Write Dockerfile for Java Card Service**
   - Create multi-stage Dockerfile based on Eclipse Temurin JRE 17
   - Expected outcome: Container image builds locally and passes health check

3. **Provision Private Subnets and NAT Gateway**
   - Extend VPC Terraform with private subnets in both AZs
   - Expected outcome: Private network path available for Fargate task placement

### 10.2 Short-term Actions

1. **Build ECS Cluster and ALB in Dev**
   - Deploy ECS cluster, task definition, ALB, and ECS service to dev environment
   - Expected outcome: Running Fargate tasks accessible via internal ALB

2. **Configure CI/CD Pipeline**
   - Create CodePipeline with GitHub source, CodeBuild, and ECS deploy stages
   - Expected outcome: Automated build and deploy on push to `feature/1880-*` branch

3. **Set Up API Gateway with VPC Link**
   - Create REST API, VPC Link, and integration with internal ALB
   - Expected outcome: Public API endpoint proxying requests to Fargate tasks

### 10.3 Long-term Actions

1. **Production Migration and DNS Cutover**
   - Deploy to production, configure weighted Route53 routing, monitor for 48 hours
   - Expected outcome: All card service traffic served via ECS Fargate

2. **Decommission Legacy EC2 Infrastructure**
   - Remove EC2 instances, Classic Load Balancer, and associated security groups
   - Expected outcome: Clean infrastructure state with no unused resources

3. **Enhance Observability Stack**
   - Enable X-Ray tracing, structured logging, and CloudWatch Container Insights
   - Expected outcome: Full request tracing and service health visibility

---

## Related Documentation

- **Java Fargate ALB Terraform Module**: `infrastructure/terraform/java-fargate-alb/README.md`
- **VPC Foundation Design**: `devops/docs/design/VPC_Foundation_Design.md`
- **ECS Fargate Best Practices**: `devops/docs/runbooks/ECS_Fargate_Operations.md`
- **GitHub Issue**: #1880 - https://github.com/org/repo/issues/1880
- **FIP Template**: `devops/docs/requirements/FIP_1880_ALB_Fargate_Java_Card_Service.md`

---

**Document Version**: 1.0
**Last Updated**: 2025-01-15
**Changes**: Initial GAP analysis for ALB + Fargate Java card service migration
**Next Review**: 2025-01-22 (after Phase 1 completion)
