# 基线与差距分析 - ALB + Fargate 基础设施（Java 卡服务）

**文档编号**: GAP_ALB_FARGATE_JAVA_CARD_SERVICE

**关联议题**: #1880 - ALB + Fargate 基础设施（Java 容器部署）

**创建日期**: 2025-01-15

**分支**: feature/1880-alb-fargate-java-card-service

**作者**: arthurren

**审阅者**: platform-team-lead, devops-lead

**状态**: 分析阶段

**依赖项**: #1843 (PostgreSQL 集群基础设施 - 已配置), #1876 (VPC 网络基础 - 已配置)

---

## 概述

### 当前基线状态: 🔴 **基础设施亟需现代化**

- **计算**: 15% (EC2 实例手动扩缩容，无容器化)
- **网络**: 10% (Classic Load Balancer，单一公有子网，无 API Gateway)
- **部署**: 5% (手动 SSH 部署，无容器 CI/CD 流水线)
- **可观测性**: 25% (基础 CloudWatch 指标，非结构化日志)
- **安全**: 10% (硬编码凭据，无 TLS 终止，无密钥轮换)
- **基础设施即代码**: 30% (部分 CloudFormation 模板，无 Terraform)

### 核心发现

1. ✅ **已有 VPC 基础**: ap-northeast-1 区域已配置含公有子网的 VPC（议题 #1876），提供了基础网络层
2. ✅ **PostgreSQL 集群**: RDS PostgreSQL 集群已就绪并正常运行（议题 #1843），作为持久化层
3. ✅ **AWS 账户结构**: 已建立多环境账户策略（dev/test/staging/prod）并配置 IAM 基线
4. ❌ **ECS Fargate 计算**: Java 卡服务尚不存在 ECS 集群、任务定义或容器编排
5. ❌ **Application Load Balancer**: 无 ALB 的 HTTPS 终止、基于路径的路由或健康检查配置
6. ❌ **容器注册表**: 无 ECR 仓库用于 Java 容器镜像，无卡服务的 Dockerfile
7. ❌ **API Gateway 集成**: 无 REST API、VPC Link 或请求/响应转换层
8. ❌ **CI/CD 流水线**: 无针对容器化部署的自动化构建、测试和部署流水线
9. ❌ **Terraform 模块**: Java 卡服务基础设施栈无 Terraform 覆盖

### 关键差距: 从手动 EC2 迁移至容器化 Fargate

**需求**（议题 #1880）: 将 Java 卡服务从手动管理的 EC2 实例迁移至 ECS Fargate，配备 ALB、API Gateway 集成和全自动化 CI/CD。

#### 方案 A: ECS EC2 启动类型（已评估但未选用）
- **自管计算**: 在团队直接管理的 EC2 实例上运行 ECS
- **优势**: 对底层主机 OS 有更多控制，支持通过 EBS 持久化存储，按实例费用可预测
- **未选用原因**: 增加了补丁管理、容量规划和实例生命周期管理的运维负担。与平台团队的无服务器优先策略相悖。

#### 方案 B: ECS Fargate + ALB + API Gateway ✅ **已选用**

**无服务器容器编排**:
- **ECS Fargate**: 完全托管的计算，自动扩缩容，无需管理实例
- **Application Load Balancer**: HTTPS 终止、基于路径的路由、健康检查
- **API Gateway**: 面向公网的 REST API，通过 VPC Link 连接私有 ALB
- **ECR**: 私有容器注册表，含镜像扫描

**优势**:
- ✅ 完全消除 EC2 实例管理开销
- ✅ 基于 CPU/内存利用率和请求数自动扩缩容
- ✅ ALB 提供原生健康检查和优雅连接排空
- ✅ API Gateway 增加速率限制、流量控制和请求校验

**✅ 决策已定**: 根据议题 #1880 要求，已选定 ECS Fargate 方案进行实施

---

## 第 0 节: 架构对比

### 0.1 当前架构 (EC2 + Classic Load Balancer)

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

### 0.2 目标架构 (ECS Fargate + ALB + API Gateway)

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

**环境隔离架构**:

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

## 第 1 节: 当前架构基线

### 1.1 计算基础设施

#### ✅ 已有资产

**EC2 实例配置** (`infrastructure/ec2/card-service/`):
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

**EC2 特性**:
- ✅ 两台 t3.large 实例（每台 2 vCPU, 8 GB RAM），运行 Java 11 JAR
- ✅ 通过 CloudWatch 告警操作实现自动恢复
- ✅ 基础实例级监控（CPU、网络、磁盘，5 分钟间隔）

#### ❌ 缺少 ECS Fargate 基础设施

```bash
# ECS Fargate Infrastructure (🔴 CRITICAL GAP)
- ❌ No ECS cluster defined for card-service
- ❌ No Fargate task definition (container spec, CPU/memory limits)
- ❌ No ECS service with desired count and auto-scaling policy
- ❌ No capacity provider strategy for Fargate Spot fallback
- ❌ No ECR repository for container image storage
```

**差距严重程度**: 🔴 **严重** - 整个容器编排层缺失。没有 ECS Fargate，服务无法实现自动扩缩容、不可变部署或基础设施即代码对等。

### 1.2 网络与负载均衡

#### ✅ 已有 VPC 基础设施 (议题 #1876)

**VPC 配置** (来自 `infrastructure/terraform/vpc-foundation/`):
- ✅ VPC `vpc-0abc123def456`，位于 ap-northeast-1 (CIDR: 10.0.0.0/16)
- ✅ ap-northeast-1a 和 ap-northeast-1c 中的两个公有子网
- ✅ 已附加 Internet Gateway 用于出站连接
- ✅ 路由表已配置，公有子网可访问互联网

#### ❌ 缺少 Application Load Balancer 和私有网络

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

**差距严重程度**: 🔴 **严重** - Classic Load Balancer 无法路由到 Fargate 任务。ALB 是 ECS 集成、健康检查和 HTTPS 终止的前提条件。

### 1.3 容器化与部署

#### ✅ 已有部署机制

**当前基于 Shell 的部署** (`scripts/deploy-card-service.sh`):
```bash
#!/bin/bash
# Manual deployment script for card service
ssh ec2-user@10.0.1.10 "cd /opt/card-service && ./stop.sh"
scp ./target/card-service.jar ec2-user@10.0.1.10:/opt/card-service/
ssh ec2-user@10.0.1.10 "cd /opt/card-service && ./start.sh"
```

**部署特性**:
- ✅ 每台 EC2 实例上可正常工作的启停脚本
- ✅ 通过 Maven 构建 JAR 制品 (`mvn clean package`)

#### ❌ 缺少容器和 CI/CD 基础设施

```bash
# Container and CI/CD (🔴 CRITICAL GAP)
- ❌ No Dockerfile for Java 17 container image
- ❌ No ECR repository with lifecycle policies
- ❌ No CodePipeline or GitHub Actions workflow for CI/CD
- ❌ No container image build and push stage
- ❌ No ECS service update deployment stage
- ❌ No blue/green or rolling deployment strategy
```

**差距严重程度**: 🔴 **严重** - 没有容器化和 CI/CD，无法推进向 Fargate 的迁移。这是整个项目的基础性差距。

### 1.4 可观测性

#### ✅ 已有监控

**CloudWatch 配置**:
- ✅ 基础 EC2 CPU 利用率告警（阈值: 80%）
- ✅ CloudWatch Agent 采集 EC2 内存/磁盘指标
- ✅ CloudWatch Logs 日志组 `/ec2/card-service`（非结构化文本）

#### ❌ 缺少容器可观测性

```bash
# Observability (🟡 HIGH GAP)
- ❌ No ECS Container Insights enabled
- ❌ No structured JSON logging from Java application
- ❌ No AWS X-Ray distributed tracing
- ❌ No CloudWatch Logs log group for Fargate task stdout/stderr
- ❌ No CloudWatch dashboard for ECS service metrics
- ❌ No alarm on task restart count or deployment failures
```

**差距严重程度**: 🟡 **高** - 可观测性差距影响运维就绪度，但不会阻塞初始部署。可在第 2 阶段之后逐步解决。

### 1.5 安全

#### ✅ 已有安全模式

**当前安全配置**:
- ✅ EC2 安全组限制入站仅来自 Classic LB
- ✅ RDS 安全组允许入站仅来自 EC2 实例
- ✅ EC2 IAM 实例配置文件具备基础 S3 只读访问

#### ❌ 缺少安全控制

```bash
# Security (🟡 HIGH GAP)
- ❌ No AWS Secrets Manager integration for database credentials
- ❌ No TLS 1.3 termination at ALB (current: HTTP only)
- ❌ No IAM task execution role for ECS Fargate
- ❌ No IAM task role for runtime AWS API calls
- ❌ No AWS WAF rules on ALB for common attack protection
- ❌ No container image vulnerability scanning in ECR
```

**差距严重程度**: 🟡 **高** - 安全差距使服务面临凭据泄露和未加密流量的风险。必须在生产流量迁移之前解决。

### 1.6 基础设施即代码

#### ✅ 已有 IaC 覆盖

**CloudFormation 模板** (`infrastructure/cloudformation/`):
```yaml
# Partial CloudFormation for VPC (legacy)
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
```

**IaC 特性**:
- ✅ VPC 和 RDS 的部分 CloudFormation 模板（遗留，未维护）
- ✅ 团队有其他项目的 Terraform 经验

#### ❌ 缺少 Terraform 覆盖

```bash
# Terraform Modules (🟡 HIGH GAP)
- ❌ No Terraform module for ECS cluster
- ❌ No Terraform module for ALB + target group + listeners
- ❌ No Terraform module for API Gateway + VPC Link
- ❌ No Terraform module for ECR repository
- ❌ No environment-specific tfvars files (dev/test/staging/prod)
- ❌ No remote state backend configuration for card-service stack
```

**差距严重程度**: 🟡 **高** - 没有 Terraform 覆盖，基础设施无法可靠地在环境间复制或提升。手动 AWS 控制台操作会导致配置漂移。

---

## 第 2 节: ECS Fargate 方案分析

### 2.1 差距分析汇总矩阵

| 差距编号 | 类别 | 当前状态 | 目标状态 | 影响 |
|--------|----------|---------------|--------------|--------|
| GAP-001 | 计算 | EC2 实例 (t3.large) 手动扩缩容 | ECS Fargate (0.5 vCPU, 1 GB) 自动扩缩容 | 🔴 严重 |
| GAP-002 | 网络 | Classic LB, 仅 HTTP, 单一公有子网 | ALB 含 HTTPS/TLS 1.3, 私有子网, API Gateway | 🔴 严重 |
| GAP-003 | 部署 | 手动 SSH, Shell 脚本, JAR 制品 | ECR + Docker + CodePipeline CI/CD | 🔴 严重 |
| GAP-004 | 可观测性 | 基础 CloudWatch, 非结构化日志 | Container Insights, 结构化 JSON, X-Ray 追踪 | 🟡 高 |
| GAP-005 | 安全 | 硬编码凭据, 无 TLS, 无 IAM 角色 | Secrets Manager, TLS 1.3, IAM 任务角色 | 🟡 高 |
| GAP-006 | 基础设施即代码 | 部分 CloudFormation（未维护） | 完整 Terraform 模块覆盖 | 🟡 高 |

### 2.2 对比分析

| 方面 | ECS EC2 启动类型 | ECS Fargate |
|--------|---------------------|-------------|
| **基础设施复杂度** | 高 - 管理 EC2 集群, AMI 补丁, 容量 | 低 - AWS 管理计算, 专注任务配置 |
| **ALB 集成** | 相同的 ALB 集成, 手动实例注册 | 原生目标组集成, 自动注册 |
| **性能** | 冷启动略低（热实例） | Java 容器冷启动约 30-60 秒 |
| **成本（低流量）** | 固定 EC2 成本（约 $120/月, t3.large） | 按任务计费（约 $15/月, 0.5 vCPU 任务） |
| **成本（高流量）** | 持续高利用率时更优 | 线性扩缩容, 无过度配置浪费 |
| **扩缩容** | 手动或自动扩缩组（分钟级） | 服务自动扩缩容（秒级添加任务） |
| **实施时间** | 较慢 - 需要 EC2 配置, AMI 烘焙 | 较快 - 专注任务定义和网络 |

**建议**: ECS Fargate 是推荐方案。更低的运维开销、更快的实施速度以及与平台团队无服务器优先策略的一致性，超过了对于流量模式可预测的卡服务而言冷启动的权衡。

### 2.3 ECS Fargate 实施差距

#### ECR 仓库与容器镜像
```bash
# Container Registry (🔴 CRITICAL GAP)
- ❌ No ECR repository `card-service` with image scanning
- ❌ No Dockerfile (multi-stage: Maven build + JRE 17 runtime)
- ❌ No lifecycle policy to retain last 10 images
- ❌ No container image for Java 17 card-service application
```

#### ECS 任务定义
```bash
# Task Definition (🔴 CRITICAL GAP)
- ❌ No task definition with 0.5 vCPU / 1 GB memory
- ❌ No container definition with port mapping (8080)
- ❌ No log configuration for CloudWatch Logs
- ❌ No environment variable injection from SSM/Secrets Manager
- ❌ No health check command in container definition
```

#### VPC 与子网配置
```bash
# Private Networking (🔴 CRITICAL GAP)
- ❌ No private subnets for Fargate task placement (10.0.10.0/24, 10.0.11.0/24)
- ❌ No NAT Gateway for private subnet egress
- ❌ No security group for Fargate tasks (allow 8080 from ALB only)
- ❌ No security group for ALB (allow 443 from VPC Link)
```

#### API Gateway 集成
```bash
# API Gateway (🟡 HIGH GAP)
- ❌ No REST API with `/api/v1/cards` resource
- ❌ No VPC Link to NLB/ALB in private subnets
- ❌ No custom domain with ACM certificate
- ❌ No usage plan or throttling configuration
```

#### CI/CD 流水线
```bash
# CI/CD (🟡 HIGH GAP)
- ❌ No CodePipeline with GitHub source stage
- ❌ No CodeBuild stage for Docker build and ECR push
- ❌ No deploy stage with ECS service update
- ❌ No manual approval gate for production deployment
```

---

## 第 3 节: 基础设施差距分析

### 3.1 网络基础设施

#### ✅ 已有网络设施

**VPC 配置** (来自议题 #1876):
- ✅ VPC `vpc-0abc123def456`，位于 ap-northeast-1 (10.0.0.0/16)
- ✅ ap-northeast-1a (10.0.0.0/24) 和 ap-northeast-1c (10.0.1.0/24) 中的公有子网
- ✅ Internet Gateway 和公有路由表

#### ❌ 缺少 Fargate 所需的私有网络

```bash
# Private Subnets and NAT (🟡 HIGH GAP)
- ❌ No private subnets in ap-northeast-1a (10.0.10.0/24) and ap-northeast-1c (10.0.11.0/24)
- ❌ No NAT Gateway in each AZ for private subnet egress
- ❌ No private route tables with NAT Gateway default route
- ❌ No VPC Link integration subnet group
```

**差距严重程度**: 🟡 **高** - Fargate 任务应运行在私有子网中以确保安全。NAT Gateway 提供出站连接，无需将任务暴露于互联网。

### 3.2 安全配置

#### ✅ 已有安全模式

**EC2 安全组** (参考):
- ✅ 安全组允许来自 Classic LB 安全组的入站 TCP/8080
- ✅ RDS 安全组允许来自 EC2 安全组的入站 TCP/5432

#### ❌ 缺少 Fargate 安全控制

```bash
# Security (🟡 HIGH GAP)
- ❌ No Secrets Manager secret for RDS credentials with auto-rotation
- ❌ No ACM certificate in ap-northeast-1 for `*.api.example.com`
- ❌ No IAM task execution role with ECR pull + CloudWatch Logs + Secrets Manager access
- ❌ No IAM task role for runtime AWS SDK calls (S3, SQS)
- ❌ No WAF WebACL on ALB for SQL injection and XSS protection
```

**差距严重程度**: 🟡 **高** - 安全控制必须在生产流量上线前到位。开发环境可使用自签名证书和基础 IAM 角色推进。

### 3.3 IAM 角色与权限

#### ✅ 已有 IAM 模式

**EC2 IAM** (当前):
- ✅ 实例配置文件具有 `AmazonS3ReadOnlyAccess`，用于制品下载
- ✅ 实例配置文件具有 `CloudWatchAgentServerPolicy`，用于指标采集

**其他 ECS 服务** (参考):
- ✅ `infrastructure/terraform/java-fargate-alb/` 包含 ECS 任务角色的 IAM 模式
- ✅ 任务执行角色模板，含 ECR 和 CloudWatch Logs 权限

#### ❌ 缺少卡服务 IAM 角色

```bash
# IAM Roles (🟡 HIGH GAP)
- ❌ No task execution role `card-service-task-execution-role`
- ❌ No task role `card-service-task-role` with least-privilege policies
- ❌ No IAM policy for Secrets Manager read access (RDS credentials)
- ❌ No IAM policy for KMS decrypt (if encryption at rest required)
```

**差距严重程度**: 🟡 **高** - IAM 角色是 Fargate 任务启动的必要条件。已有参考实现可直接适配。

### 3.4 监控与日志

#### ✅ 已有监控

**CloudWatch 配置**:
- ✅ EC2 上的 CloudWatch Agent 采集内存和磁盘指标
- ✅ 日志组 `/ec2/card-service`，保留 30 天
- ✅ SNS 主题 `ops-alerts` 用于告警通知

#### ❌ 缺少容器监控

```bash
# Container Monitoring (🟢 MEDIUM GAP)
- ❌ No CloudWatch Container Insights for ECS cluster
- ❌ No structured JSON log format in Java application
- ❌ No CloudWatch dashboard `card-service-ops`
- ❌ No alarms on CPU utilization > 70%, memory > 80%
- ❌ No X-Ray tracing for request correlation
```

**差距严重程度**: 🟢 **中等** - 监控可在上线后逐步增强。Fargate 任务的标准输出默认即可输出到 CloudWatch Logs。

### 3.5 自动扩缩容配置

#### ✅ 已有自动扩缩容模式

**参考自动扩缩容** (来自 `infrastructure/terraform/java-fargate-alb/`):
- ✅ 基于 CPU 利用率目标跟踪的 ECS 服务自动扩缩容
- ✅ 含最小/最大任务数配置的可扩展目标
- ✅ 高请求数时快速扩容的步进缩放策略

#### ❌ 缺少卡服务自动扩缩容

```bash
# Auto-Scaling (🟢 MEDIUM GAP)
- ❌ No scalable target for card-service ECS service
- ❌ No target tracking policy (CPU target: 65%, memory target: 75%)
- ❌ No ALB request count per target scaling metric
- ❌ No scheduled scaling for business hours (min 3) vs off-hours (min 1)
```

**差距严重程度**: 🟢 **中等** - 自动扩缩容在初始部署中并非必需。固定任务数（2）足以支撑上线，后续根据观测到的流量模式添加扩缩容。

---

## 第 4 节: 实施差距分析

### 4.1 Terraform 基础设施代码

#### ✅ 已有 Terraform 模块

**参考模块**:
- ✅ `infrastructure/terraform/java-fargate-alb/` - 完整的 ECS Fargate + ALB 模式（Java 服务）
- ✅ `infrastructure/terraform/vpc-foundation/` - 可扩展私有子网的 VPC 模块

#### ❌ 缺少卡服务 Terraform

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

**差距严重程度**: 🔴 **严重** - 所有基础设施必须以 Terraform 定义。参考模块提供了可适配的工作模式。

### 4.2 CI/CD 流水线

#### ✅ 已有 CI/CD 模式

**当前部署**:
- ✅ Maven 构建产出 JAR 制品 (`mvn clean package`)
- ✅ GitHub 仓库配置了分支保护规则
- ✅ 其他 Java 服务的 CodePipeline（参考）

#### ❌ 缺少卡服务 CI/CD

```bash
# CI/CD Pipeline (🟡 HIGH GAP)
- ❌ No CodePipeline `card-service-pipeline`
- ❌ No CodeBuild project for Docker build (Dockerfile + ECR push)
- ❌ No source stage with GitHub webhook trigger
- ❌ No deploy stage with ECS service update action
- ❌ No manual approval stage before production deploy
- ❌ No buildspec.yml for container image build
```

**差距严重程度**: 🟡 **高** - CI/CD 支持可重复部署。Java Fargate ALB 项目的参考流水线可直接适配。

### 4.3 配置管理

#### ✅ 已有配置

**当前配置**:
- ✅ `application.yml`，Spring Boot 外部化配置
- ✅ 通过环境变量覆盖数据库 URL 和凭据
- ✅ SSM Parameter Store 用于非敏感配置值

#### ❌ 缺少容器配置

```bash
# Container Configuration (🟢 MEDIUM GAP)
- ❌ No environment variable injection via ECS task definition
- ❌ No Secrets Manager secret references in task definition
- ❌ No SSM Parameter Store references for feature flags
- ❌ No log driver configuration for Fluentd or FireLens
```

**差距严重程度**: 🟢 **中等** - 配置管理可逐步改进。初始部署可使用环境变量配合 Secrets Manager 引用。

---

## 第 5 节: 迁移路径分析

### 5.1 从当前状态到目标状态

**当前架构**:
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

**目标架构**:
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

### 5.2 迁移步骤（详细）

#### 第 1 阶段: 基础设施（第 1 周）
1. ✅ 在 Terraform 中创建私有子网和 NAT Gateway (`vpc.tf`)
2. ✅ 配置 ECR 仓库 `card-service` 并设置生命周期策略
3. ✅ 创建 IAM 任务执行角色和任务角色，采用最小权限策略
4. ✅ 在 ap-northeast-1 申请 `api.example.com` 的 ACM 证书
5. ✅ 创建 Secrets Manager 密钥用于 RDS 凭据，启用轮换

#### 第 2 阶段: 计算层（第 2 周）
1. ✅ 编写 Dockerfile（多阶段: Maven 构建 + Eclipse Temurin JRE 17）
2. ✅ 定义 ECS 任务定义（0.5 vCPU, 1 GB 内存, 端口 8080）
3. ✅ 创建 ECS 集群 `card-service-cluster`
4. ✅ 创建 ALB（内部），配置目标组、HTTPS 监听器、健康检查路径 `/health`
5. ✅ 创建 ECS 服务，期望任务数 2，挂载到 ALB 目标组

#### 第 3 阶段: 路由与集成（第 3 周）
1. ✅ 创建 API Gateway REST API，配置 `/api/v1/cards` 资源
2. ✅ 配置 VPC Link 连接私有子网中的 ALB
3. ✅ 设置自定义域名并绑定 ACM 证书
4. ✅ 配置 Route53 DNS 记录指向 `api.example.com`

#### 第 4 阶段: CI/CD 与验证（第 4 周）
1. ✅ 创建 CodePipeline，包含 GitHub 源、CodeBuild 和 ECS 部署阶段
2. ✅ 构建并推送初始容器镜像至 ECR
3. ✅ 部署至开发环境并验证健康检查
4. ✅ 提升至预发布环境，运行集成测试套件
5. ✅ 生产切换: 将 DNS 从 Classic LB 切换至 API Gateway

#### 第 5 阶段: 可观测性与优化（第 5 周）
1. ✅ 在 ECS 集群上启用 CloudWatch Container Insights
2. ✅ 创建 CloudWatch 仪表板，展示服务指标
3. ✅ 配置自动扩缩容策略（CPU 目标: 65%, 最小 2, 最大 10）
4. ✅ 下线 EC2 实例和 Classic Load Balancer

---

## 第 6 节: 风险评估

### 6.1 高风险领域

1. **RISK-001: ECS Fargate 冷启动延迟**
   - **风险**: Java 容器冷启动可能需要 30-60 秒，在扩容事件或任务替换期间导致超时错误
   - **缓解措施**: 设置最小健康任务数为 2，配置 ALB 健康检查间隔为 15 秒、健康阈值为 3 次，通过计划扩缩容在流量高峰前预热任务

2. **RISK-002: 从 Classic LB 迁移至 ALB 的流量中断**
   - **风险**: 从 Classic LB 到 API Gateway 的 DNS 切换可能在传播期间导致连接中断（TTL: 300 秒）
   - **缓解措施**: 使用 Route53 加权路由（90/10 分配），在新端点监控错误率 30 分钟，在切换窗口期间保留 Classic LB 以便回滚

3. **RISK-003: 容器镜像漏洞暴露**
   - **风险**: Docker 容器中的基础镜像或依赖漏洞可能在无自动扫描的情况下未被检测到
   - **缓解措施**: 启用 ECR 推送时镜像扫描，在 CodeBuild 流水线中集成 Trivy 扫描，对 CRITICAL/HIGH 严重级别发现阻止部署

4. **RISK-004: Secrets Manager 轮换导致的停机**
   - **风险**: 数据库凭据轮换可能在 Java 应用未正确处理凭据刷新时导致短暂连接失败
   - **缓解措施**: 在 Spring Boot 中实现连接池刷新逻辑，先使用 `test` 模式轮换再 `apply`，在低流量时段（02:00 JST）安排轮换

### 6.2 中等风险领域

1. **RISK-005: API Gateway 流量控制配置不当**
   - **风险**: 默认速率限制可能对生产流量模式过于激进（1000 请求/分钟）或过于宽松
   - **缓解措施**: 分析 Classic LB 访问日志中的当前流量，将初始突发限制设为观测峰值流量的 200%，创建 429 响应的 CloudWatch 告警

2. **RISK-006: 跨可用区 Fargate 任务分布**
   - **风险**: Fargate 任务可能未在 ap-northeast-1a 和 ap-northeast-1c 之间均匀分布，降低可用区故障期间的可用性
   - **缓解措施**: 为 ECS 服务配置 `azAware` 容量提供者策略，确保两个可用区中均有私有子网且容量相等

3. **RISK-007: 迁移期间的 Terraform 状态漂移**
   - **风险**: 迁移期间通过 AWS 控制台手动操作可能导致与 Terraform 状态漂移，引发 plan/apply 冲突
   - **缓解措施**: 锁定控制台用户的 IAM 写入权限，强制所有变更通过 Terraform PR 执行，在 CI 中 apply 前运行 `terraform plan`

---

## 第 7 节: 建议

### 7.1 选定方案: ECS Fargate + ALB + API Gateway ✅ **决策已定**

**决策**: 根据议题 #1880 要求，已选定 ECS Fargate 配合 Application Load Balancer 和 API Gateway 进行 Java 卡服务迁移。

**理由**:
1. ✅ **运维效率**: 消除 EC2 实例管理（补丁、容量规划、SSH 访问），预计每月减少约 8 小时运维工作
2. ✅ **自动扩缩容**: Fargate 服务自动扩缩容可在数秒内响应流量变化，而 EC2 自动扩缩组需要分钟级，在低流量期间提高成本效率
3. ✅ **安全态势**: ALB 提供 TLS 1.3 终止，API Gateway 增加速率限制和 WAF 集成，通过 Secrets Manager 消除硬编码凭据
4. ✅ **部署速度**: 基于容器镜像的 CI/CD 流水线支持每日部署，而当前为每月手动发布周期
5. ✅ **平台一致性**: 与已投产的其他服务的 Java Fargate ALB 模式对齐，支持知识复用和模块共享

**实施优先级**:
1. **高**: ECS 集群、ALB 和 IAM 角色的 Terraform 模块（阻塞所有下游工作）
2. **高**: Dockerfile 和 ECR 仓库（启用容器构建）
3. **高**: 私有子网和 NAT Gateway（Fargate 任务放置所需）
4. **中**: API Gateway 和 VPC Link（可延后，以直接 ALB 访问作为过渡）
5. **低**: X-Ray 追踪和高级可观测性（上线后增强）

### 7.2 备选方案: ECS EC2 启动类型 - 未选用

**理由**:
1. ✅ **更低冷启动**: 热的 EC2 实例消除 Java 冷启动延迟
2. ✅ **持久存储**: EBS 卷可用于本地缓存或状态
3. ✅ **规模化成本**: 持续高利用率（平均 CPU >70%）时更便宜

**ECS EC2 启动类型已评估但未选用**:
- 需要 EC2 实例生命周期管理（补丁、扩缩容、替换）
- AMI 流水线和容量规划增加 2-3 周实施时间
- 与 2024 年 Q4 确立的平台团队无服务器优先策略相悖

---

## 第 8 节: 成功标准

### 8.1 ECS Fargate 方案成功标准

**基础设施**:
- [ ] ECS 集群 `card-service-cluster` 在 ap-northeast-1 运行，使用 Fargate 容量提供者
- [ ] ALB 目标组健康检查在 `/health` 端点对所有已注册任务通过
- [ ] ECR 仓库 `card-service` 已启用镜像扫描，生命周期策略保留最近 10 个镜像
- [ ] ap-northeast-1a 和 ap-northeast-1c 均有私有子网，配置 NAT Gateway 用于出站
- [ ] 所有基础设施以 Terraform 定义，配有 dev/test/staging/prod 环境专用 tfvars

**集成**:
- [ ] API Gateway REST API 在 `https://api.example.com/api/v1/cards/*` 提供服务
- [ ] VPC Link 连接 API Gateway 至私有子网中的内部 ALB
- [ ] Route53 DNS 记录配合 ACM 证书实现 HTTPS 终止
- [ ] Secrets Manager 密钥用于 RDS 凭据，轮换已启用并测试通过
- [ ] IAM 任务角色采用最小权限策略（无 `AdministratorAccess` 或 `*:*`）

**性能**:
- [ ] p99 API 响应延迟低于 500ms（在 API Gateway 测量）
- [ ] ECS Fargate 任务启动时间低于 90 秒（含 Java 应用引导）
- [ ] 自动扩缩容在 2 倍流量突增下 3 分钟内响应
- [ ] ECS 服务更新期间零停机部署（滚动部署）

---

## 第 9 节: 依赖项

### 9.1 外部依赖

- **AWS ECS Fargate**: 状态: 已有 - 已提交服务配额提升请求（当前: 每服务 2 个任务, 目标: 10 个任务）
- **AWS ALB**: 状态: 已有 - ap-northeast-1 可用，无需提升配额
- **AWS API Gateway**: 状态: 已有 - REST API 可用，ap-northeast-1 支持 VPC Link
- **ACM 证书**: 状态: 新增 - `*.api.example.com` 证书请求已提交，DNS 验证待完成

### 9.2 内部依赖

- ✅ **VPC 基础** (议题 #1876): 已配置 - VPC, 公有子网, IGW 可用
- ✅ **RDS PostgreSQL** (议题 #1843): 运行中 - 卡数据库集群在 ap-northeast-1 运行
- ❌ **私有子网**: 状态: 新增 - 必须在 Fargate 任务部署前添加到现有 VPC
- ❌ **容器镜像**: 状态: 新增 - 需编写 Dockerfile 并完成初始构建

### 9.3 参考实现

- **Java Fargate ALB 模式**: `infrastructure/terraform/java-fargate-alb/`
- **VPC 基础模块**: `infrastructure/terraform/vpc-foundation/`
- **ECS CI/CD 流水线**: `infrastructure/codepipeline/java-service-pipeline/`

---

## 第 10 节: 后续步骤

### 10.1 立即行动

1. **创建 Terraform 模块结构**
   - 初始化 `infrastructure/terraform/card-service/`，包含 `main.tf`, `variables.tf`, `outputs.tf`
   - 预期成果: 空模块就绪，可开始定义资源

2. **编写 Java 卡服务 Dockerfile**
   - 基于 Eclipse Temurin JRE 17 创建多阶段 Dockerfile
   - 预期成果: 容器镜像可本地构建并通过健康检查

3. **配置私有子网和 NAT Gateway**
   - 在 VPC Terraform 中扩展两个可用区的私有子网
   - 预期成果: 私有网络路径可用于 Fargate 任务放置

### 10.2 短期行动

1. **在开发环境构建 ECS 集群和 ALB**
   - 将 ECS 集群、任务定义、ALB 和 ECS 服务部署至开发环境
   - 预期成果: 运行中的 Fargate 任务可通过内部 ALB 访问

2. **配置 CI/CD 流水线**
   - 创建 CodePipeline，包含 GitHub 源、CodeBuild 和 ECS 部署阶段
   - 预期成果: 推送至 `feature/1880-*` 分支时自动构建和部署

3. **设置 API Gateway 和 VPC Link**
   - 创建 REST API、VPC Link，集成内部 ALB
   - 预期成果: 公网 API 端点代理请求至 Fargate 任务

### 10.3 长期行动

1. **生产迁移与 DNS 切换**
   - 部署至生产环境，配置 Route53 加权路由，监控 48 小时
   - 预期成果: 所有卡服务流量由 ECS Fargate 承载

2. **下线遗留 EC2 基础设施**
   - 移除 EC2 实例、Classic Load Balancer 及关联安全组
   - 预期成果: 干净的基础设施状态，无闲置资源

3. **增强可观测性栈**
   - 启用 X-Ray 追踪、结构化日志和 CloudWatch Container Insights
   - 预期成果: 完整的请求追踪和服务健康可见性

---

## 相关文档

- **Java Fargate ALB Terraform 模块**: `infrastructure/terraform/java-fargate-alb/README.md`
- **VPC 基础设计**: `devops/docs/design/VPC_Foundation_Design.md`
- **ECS Fargate 最佳实践**: `devops/docs/runbooks/ECS_Fargate_Operations.md`
- **GitHub 议题**: #1880 - https://github.com/org/repo/issues/1880
- **FIP 模板**: `devops/docs/requirements/FIP_1880_ALB_Fargate_Java_Card_Service.md`

---

**文档版本**: 1.0
**最后更新**: 2025-01-15
**变更说明**: ALB + Fargate Java 卡服务迁移的初始差距分析
**下次审阅**: 2025-01-22（第 1 阶段完成后）
