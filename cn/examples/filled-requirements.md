# 需求规格说明 - Java Card Service 的 ALB + Fargate 基础设施

**文档 ID**: REQ_ALB_FARGATE_JAVA_CARD_SERVICE

**Issue**: #1880 - ALB + Fargate Infrastructure for Java Container Deployment

**创建日期**: 2025-01-20

**分支**: feature/1880-alb-fargate-java-card-service

**状态**: 需求已批准

**作者**: arthurren

**依赖**: #1843 (PostgreSQL Cluster Infrastructure)

---

## 概要

### 问题陈述

**当前状态**:
- Java Card Service 运行在手动配置的 EC2 实例上，没有容器编排
- 部署需要 SSH 访问实例并手动替换 jar 包，导致停机
- 没有负载均衡、自动扩缩容或自动化健康检查
- dev、test、staging 和 prod 环境之间配置漂移频繁
- 监控仅限于 EC2 实例上的基本 CPU/内存指标

**目标状态**:
- 容器化的 Java Card Service 部署在 ECS Fargate 上，具有完整的编排能力
- 通过 ALB 目标组管理实现零停机滚动部署
- 基于 CPU 和内存利用率的自动扩缩容，具有按环境配置的阈值
- 通过 Terraform 模块在所有环境中配置相同的基础设施
- 全面的可观测性，包括结构化日志、Container Insights 和 CloudWatch 警报

### 解决方案概述

1. 应用负载均衡器 - 内部 ALB，具有 HTTPS 终止和健康检查路由
2. ECS Fargate 集群 - 按环境的容器编排，具有自动扩缩容能力
3. API Gateway 集成 - VPC Link 路由，通过内部 NLB 处理 /card/v1/* 路径
4. 容器部署管道 - ECR 仓库，具有多阶段 Docker 构建和镜像扫描
5. 可观测性栈 - CloudWatch Logs、Container Insights 和指标警报，带有 SNS 通知

### 架构决策

#### 方案 A: EC2 + Docker Compose（已考虑但未选择）
- **方法**: 在通过 Docker Compose 管理的专用 EC2 实例上运行 Docker 容器
- **优点**: 稳态工作负载成本更低，对主机操作系统完全控制，网络更简单
- **未选择原因**: 手动扩缩容开销，无内置服务发现，主机操作系统补丁负担，无 ALB 目标组滚动部署的原生集成

#### 方案 B: ALB + ECS Fargate ✅ **已选择**

**方法**:
- **Application Load Balancer**: 内部 ALB，具有 HTTPS 终止，将流量路由到 ECS 任务
- **ECS Fargate**: 无服务器容器计算，使用 awsvpc 网络模式，按环境部署集群
- **API Gateway**: REST API，使用 VPC Link 与内部 NLB 进行私有集成

**理由**:
- 无服务器计算消除了主机操作系统管理和补丁责任
- 原生 ALB 集成提供滚动部署，具有可配置的最低健康百分比
- ECS 内置自动扩缩容，支持 CPU 和内存的目标跟踪策略
- 按秒计费使成本与低流量时段的实际使用量一致

### 架构流程

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

### 多环境架构

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
|  共享: ECR Repository, IAM Policies, KMS Key, SSM Parameters    |
+-------------------------------------------------------------------+
```

### 目标

| 目标 ID | 描述 | 优先级 |
|---------|------|--------|
| GOAL-001 | 在每个环境中部署具有 HTTPS 终止的内部 ALB | CRITICAL |
| GOAL-002 | 配置具有按环境隔离的 ECS Fargate 集群 | CRITICAL |
| GOAL-003 | 配置基于 CPU 和内存目标跟踪的自动扩缩容 | CRITICAL |
| GOAL-004 | 建立 VPC Link 用于 API Gateway 私有集成 | HIGH |
| GOAL-005 | 实现最低 50% 健康任务的滚动部署 | HIGH |
| GOAL-006 | 设置 JSON 结构化格式的 CloudWatch Logs | HIGH |
| GOAL-007 | 配置多环境 Terraform，使用环境 tfvars | HIGH |
| GOAL-008 | 创建具有镜像扫描和生命周期策略的 ECR 仓库 | HIGH |
| GOAL-009 | 实现 ALB 目标组的健康检查端点 | MEDIUM |
| GOAL-010 | 编写部署和回滚的运维操作手册 | MEDIUM |
| GOAL-011 | 设置 CPU、内存和 5xx 错误率的 CloudWatch 警报 | MEDIUM |
| GOAL-012 | 通过部署后集成测试验证基础设施 | MEDIUM |

### 成功标准

| 指标 | 当前状态 | 目标状态 | 度量方式 |
|------|---------|---------|---------|
| 部署频率 | 手动，每两周 | 自动化，按需 | CI/CD 管道部署次数 |
| 服务可用性 | ~99.0%（单实例） | 99.9%（多可用区 Fargate） | CloudWatch 正常运行时间检查 |
| 响应时间 P99 | 未度量 | < 500 ms | CloudWatch 延迟指标 |
| 基础设施即代码覆盖率 | 0%（手动 EC2） | 100%（Terraform） | Terraform plan 漂移检测 |
| 环境一致性 | 低（手动配置） | 完全（共享模块 + 环境 tfvars） | Terraform plan 比较 |
| 自动化测试覆盖率 | 无 | 每个环境的部署后冒烟测试 | 测试套件通过率 |

> **重要提示**: 本项目依赖于 Issue #1843 中配置的 PostgreSQL 集群。数据库端点、安全组和 Secrets Manager 凭据必须在 ECS 任务部署之前可用。数据库迁移不在本范围内，单独跟踪。

---

## 第 1 节: 功能需求

### 1.1 应用负载均衡器基础设施

#### FR-1.1.1: 具有 HTTPS 终止的 ALB

内部 Application Load Balancer 作为所有到 Java Card Service 流量的入口点。它使用 AWS Certificate Manager 的证书在端口 443 上终止 TLS 连接，通过将端口 80 上的所有 HTTP 流量重定向来强制使用 HTTPS，并将请求转发到 ECS Fargate 目标组。ALB 必须部署在跨两个可用区的私有子网中，以满足高可用性要求。

**验收标准**:

- [ ] 在每个目标环境（dev、test、staging、prod）中配置内部 ALB
- [ ] HTTPS 监听器配置在端口 443 上，使用 TLS 1.3 安全策略
- [ ] 端口 80 上的 HTTP 监听器将所有请求重定向到 HTTPS，返回 HTTP 301
- [ ] ACM 证书已附加并确认自动续期
- [ ] 目标组健康检查对所有已注册目标返回 HTTP 200
- [ ] ALB 访问日志已启用并写入专用 S3 存储桶

**基础设施要求**:

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

**多环境配置**:

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

> **关键要求**: ALB 在所有环境中必须是内部面向的（scheme: internal）。根据组织安全策略，内部微服务不允许使用面向互联网的 ALB。

#### FR-1.1.2: 内部 ALB 配置

ALB 部署在跨两个可用区（ap-northeast-2a 和 ap-northeast-2c）的私有应用子网中。安全组限制端口 443 上的入站流量仅来自 VPC CIDR 范围，并允许所有出站流量。ALB 通过端口 8080 上的目标组与 ECS 任务通信。

**验收标准**:

- [ ] ALB 子网是两个可用区中的私有应用子网
- [ ] 安全组入站规则仅允许来自 VPC CIDR 的 TCP/443
- [ ] 安全组出站规则允许到 ECS 任务安全组的 TCP/8080
- [ ] ALB 注册为内部 NLB 的目标，用于 API Gateway 集成

### 1.2 ECS Fargate 基础设施

#### FR-1.2.1: ECS 集群（多环境）

每个环境都有一个专用的 ECS 集群，配备 Fargate 容量提供者。集群遵循命名约定 `java-card-service-{env}`，并包含环境、成本中心和托管方的标准标签。Fargate 容量提供者策略对按需容量使用默认权重 1。

**验收标准**:

- [ ] 在每个目标环境中创建具有 Fargate 容量提供者的 ECS 集群
- [ ] 集群命名遵循约定: java-card-service-{environment}
- [ ] 集群标签包含 Environment、CostCenter 和 ManagedBy
- [ ] Fargate 容量提供者已关联到集群

#### FR-1.2.2: 任务定义模板

任务定义了 Java Card Service 的容器运行时规范。它使用 Fargate 所需的 awsvpc 网络模式，分配 1 vCPU（1024 CPU 单元）和 2 GB 内存，并引用 ECR 仓库作为容器镜像。环境变量从 SSM Parameter Store 注入，日志以结构化 JSON 格式转发到 CloudWatch Logs。

**验收标准**:

- [ ] 任务定义使用 awsvpc 网络模式，兼容 Fargate
- [ ] CPU 设置为 1024 单元（1 vCPU），内存设置为 2048 MB（2 GB）
- [ ] 容器镜像引用 ECR 仓库 URI，标签可配置
- [ ] 环境变量从 SSM Parameter Store 注入
- [ ] 日志组配置为 CloudWatch Logs，使用 JSON 前缀
- [ ] 任务角色授予 SSM、Secrets Manager 和 KMS 的最小权限

**基础设施要求**:

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

#### FR-1.2.3: 服务部署能力

ECS 服务管理期望的任务数量，并与 ALB 目标组集成以进行流量路由。部署使用滚动更新策略（ECS 默认），最低健康百分比为 50%，确保部署期间至少一半任务保持健康。服务配置为自动替换失败的任务。

**验收标准**:

- [ ] ECS 服务按环境配置期望的任务数量
- [ ] 服务通过负载均衡器配置块与 ALB 目标组集成
- [ ] 部署使用滚动更新，minimum_healthy_percent = 50
- [ ] 已启用对已停止或失败任务的自动恢复
- [ ] 已启用部署断路器，失败时自动回滚

#### FR-1.2.4: 自动扩缩容配置

自动扩缩容使用 CPU 和内存利用率的目标跟踪策略进行配置。扩出和缩入冷却期防止抖动。容量边界按环境特定: dev（1-2 个任务）、test（1-3 个任务）、staging（2-6 个任务）和 prod（3-10 个任务）。

**验收标准**:

- [ ] 自动扩缩容策略以按环境阈值为目标跟踪 CPU 利用率
- [ ] 自动扩缩容策略以按环境阈值为目标跟踪内存利用率
- [ ] 扩出冷却期设置为 120 秒
- [ ] 缩入冷却期设置为 300 秒
- [ ] 最小和最大任务数量可按环境配置

**多环境配置**:

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

### 1.3 API Gateway 集成

#### FR-1.3.1: VPC Link 配置

API Gateway 使用 VPC Link 将请求从公共或私有 API Gateway 端点路由到内部 NLB，NLB 再将流量转发到 ALB。VPC Link 使用私有子网创建，VPC Link 安全组允许来自 API Gateway 服务的 NLB 监听器端口上的入站 TCP 流量。

**验收标准**:

- [ ] VPC Link 在每个环境中以内部 NLB 为目标创建
- [ ] VPC Link 使用私有应用子网进行网络连接
- [ ] VPC Link 健康检查持续通过
- [ ] API Gateway 集成引用 VPC Link ID

#### FR-1.3.2: 路由配置

API Gateway 路由配置为 /card/v1/* 路径模式。支持的 HTTP 方法仅限 GET 和 POST。每个路由应用限流限制，速率为每秒 100 个请求，突发为 200。API Gateway 层面不进行身份验证；身份验证由应用层处理。

**验收标准**:

- [ ] 路由配置为 /card/v1/* 路径模式
- [ ] HTTP 方法仅限于 GET 和 POST
- [ ] 限流限制设置: 速率 100 rps，突发 200
- [ ] 集成使用代理集成类型连接到 VPC Link

### 1.4 容器部署能力

#### FR-1.4.1: 容器镜像构建与推送

Java Card Service 使用多阶段 Docker 构建。第一阶段使用 Gradle 和 Eclipse Temurin JDK 21 编译 Java 应用。第二阶段使用 Eclipse Temurin JRE 21 Alpine 生成最小运行时镜像。镜像使用 git commit SHA 和构建的语义版本进行标记。每次推送都启用 ECR 镜像扫描，生命周期策略保留最近 30 个镜像。

**验收标准**:

- [ ] 创建名为 java-card-service 的 ECR 仓库
- [ ] 容器镜像使用项目根目录中的多阶段 Dockerfile 构建
- [ ] 镜像使用 git commit SHA 和语义版本标记
- [ ] 推送时启用 ECR 镜像扫描
- [ ] 生命周期策略保留最近 30 个标记镜像，并在 7 天后过期未标记镜像

**基础设施要求**:

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

### 1.5 数据库访问

#### FR-1.5.1: 数据库连接

Java Card Service 连接到 Issue #1843 中配置的 PostgreSQL 集群。数据库凭据存储在 AWS Secrets Manager 中，启用了自动轮换（30 天计划）。ECS 任务角色被授予读取密钥的 IAM 权限以及通过 IAM 身份验证连接数据库的权限。安全组仅允许来自 ECS 任务安全组的 TCP/5432 入站流量。

**验收标准**:

- [ ] 数据库连接字符串作为 SecureString 存储在 SSM Parameter Store 中
- [ ] 数据库密码存储在 Secrets Manager 中，具有 30 天轮换周期
- [ ] 安全组仅允许来自 ECS 任务安全组的 TCP/5432 入站流量
- [ ] ECS 任务角色具有数据库凭据密钥的 Secrets Manager 读取权限

### 1.6 多环境基础设施

#### FR-1.6.1: 环境隔离 🔴 **CRITICAL**

每个环境（dev、test、staging、prod）在完全隔离的状态下运行。ECS 集群、ALB、安全组、CloudWatch 日志组和 IAM 角色按环境配置，没有共享的运行时资源。ECR 仓库和 KMS 密钥跨环境共享以减少开销。资源名称包含环境标识符作为后缀（例如 java-card-service-alb-dev）。所有资源携带强制性标签: Environment、CostCenter、ManagedBy 和 Project。

**验收标准**:

- [ ] 每个环境具有专用的 ECS 集群、ALB 和安全组
- [ ] IAM 角色按环境划分范围，无跨环境访问
- [ ] 资源名称包含环境标识符后缀
- [ ] 所有资源携带强制性标签（Environment、CostCenter、ManagedBy、Project）

**按环境分解**:

| 资源 | dev | test | staging | prod |
|------|-----|------|---------|------|
| ECS 集群 | 是 | 是 | 是 | 是 |
| 内部 ALB | 是 | 是 | 是 | 是 |
| ECS 服务 | 是 | 是 | 是 | 是 |
| 自动扩缩容 | 否 | 否 | 是 | 是 |
| API Gateway 路由 | 否 | 是 | 是 | 是 |
| CloudWatch 警报 | 否 | 否 | 是 | 是 |
| 删除保护 | 否 | 否 | 是 | 是 |

---

## 第 2 节: 非功能需求

### 2.1 性能

#### 响应时间

- /card/v1/* 端点 P99 延迟必须低于 500 ms
- /card/v1/* 端点 P95 延迟必须低于 200 ms
- /actuator/health 端点必须在 100 ms 内响应

#### 吞吐量

- 生产环境必须处理每秒 1000 个并发请求
- 预发布环境必须处理每秒 200 个并发请求

### 2.2 可用性

#### 服务可用性

- 目标可用性: 99.9%（按月度量）
- 最大可接受停机时间: 每月 43 分钟
- 恢复时间目标（RTO）: 15 分钟
- 恢复点目标（RPO）: 5 分钟

#### 高可用性

- [ ] 应用部署在 2 个可用区（ap-northeast-2a、ap-northeast-2c）
- [ ] ALB 配置了多可用区目标组注册
- [ ] ECS 服务最低健康百分比确保部署期间至少有 1 个任务
- [ ] 数据库在不同可用区有备用副本（由 #1843 提供）

### 2.3 安全

#### 网络安全

- 所有服务间通信必须使用 TLS 1.2 或更高版本
- ECS 任务必须在私有子网中运行，无直接互联网访问
- 安全组必须遵循最小权限入站/出站规则
- ALB 在所有环境中必须是内部面向的（scheme: internal）

#### IAM 安全

- ECS 任务角色必须遵循最小权限原则
- 没有 AdministratorAccess 或通配符（*:*）权限的 IAM 角色
- 所有 IAM 策略必须在部署前审查和批准
- ECS 执行角色必须仅限于 ECR 拉取、CloudWatch Logs 和 SSM

#### 数据安全

- 静态数据必须使用 AWS KMS 客户管理的密钥加密
- 传输中数据必须使用 TLS 1.2 或更高版本加密
- 数据库凭据必须存储在 Secrets Manager 中，启用自动轮换
- 包含敏感值的 SSM 参数必须使用 KMS 加密的 SecureString 类型

### 2.4 可扩展性

#### 自动扩缩容

- 系统必须基于 CPU 和内存利用率自动扩缩容
- 扩出必须在阈值突破后 120 秒内完成
- 最大容量必须支持生产环境中 3 倍基线负载（3 到 10 个任务）

#### 水平扩展

- [ ] 系统支持无停机添加 Fargate 任务
- [ ] 无状态应用设计支持水平扩展（无本地会话状态）
- [ ] 数据库连接池支持并发任务连接

### 2.5 可观测性

#### 日志

- 应用日志必须发送到 CloudWatch Logs 日志组 /ecs/java-card-service
- 日志格式必须是结构化 JSON，包含字段: timestamp、level、message、trace_id、request_id
- 日志保留期 dev/test 为 30 天，staging/prod 为 90 天

#### 指标

- 必须为 ECS 集群启用 CloudWatch Container Insights
- 必须发布 request_count、error_count 和 latency_percentiles 自定义指标
- 必须创建关键指标的仪表板: CPU、内存、任务数量、ALB 延迟、5xx 比率

#### 警报

- CPU 利用率超过 80% 持续 5 分钟时触发警报（staging、prod）
- 内存利用率超过 85% 持续 5 分钟时触发警报（staging、prod）
- ALB 5xx 错误率在 2 分钟内超过 5% 时触发警报（staging、prod）
- 目标组健康主机数降至最低值以下时触发警报（staging、prod）
- 警报通知必须发送到由值班 Slack 频道订阅的 SNS 主题

### 2.6 部署

#### CI/CD 管道

- 管道必须在推送到 feature/1880-* 和 main 分支时触发
- 管道阶段必须包括: build、test、docker-build、terraform-plan、terraform-apply
- 在 staging 和 prod 环境中，Terraform plan 必须在 apply 之前审查

#### 部署策略

- dev 环境在合并到 feature/1880-* 分支时必须自动部署
- test 环境在合并到 main 分支时必须自动部署
- staging 环境在 apply 之前必须要求 terraform plan 审查
- prod 环境在 apply 之前必须要求手动审批关卡

---

## 第 3 节: 基础设施要求

### 3.1 所需 AWS 服务

| 服务 | 用途 | 区域 | 环境 |
|------|------|------|------|
| ECS Fargate | Java Card Service 的容器编排 | ap-northeast-2 | 全部 |
| Application Load Balancer | 内部流量分发和 TLS 终止 | ap-northeast-2 | 全部 |
| API Gateway (REST) | /card/v1/* 路由的 VPC Link 集成 | ap-northeast-2 | test, staging, prod |
| Network Load Balancer | 作为 VPC Link 目标的内部 NLB | ap-northeast-2 | test, staging, prod |
| EC2 Container Registry | 容器镜像存储和扫描 | ap-northeast-2 | 全部（共享） |
| CloudWatch Logs | 结构化应用日志聚合 | ap-northeast-2 | 全部 |
| CloudWatch Container Insights | ECS 集群性能指标 | ap-northeast-2 | 全部 |
| IAM | 任务角色、执行角色和策略 | ap-northeast-2 | 全部 |
| VPC | 私有子网、安全组、路由表 | ap-northeast-2 | 全部 |
| KMS | 密钥和日志的加密密钥管理 | ap-northeast-2 | 全部（共享） |
| SSM Parameter Store | 环境变量和配置注入 | ap-northeast-2 | 全部 |
| Secrets Manager | 数据库凭据存储和轮换 | ap-northeast-2 | 全部 |
| ACM | ALB HTTPS 的 TLS 证书管理 | ap-northeast-2 | 全部 |
| S3 | ALB 访问日志存储 | ap-northeast-2 | 全部 |
| SNS | 警报通知交付 | ap-northeast-2 | staging, prod |

### 3.2 Terraform 基础设施

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

### 3.3 容器镜像要求

#### ECR 仓库

- 仓库名称必须遵循约定: java-card-service
- 镜像标记必须包括: git commit SHA（7 字符短格式）、语义版本（例如 1.2.3）
- 推送时必须启用镜像扫描
- 生命周期策略必须保留最近 30 个标记镜像，并在 7 天后移除未标记镜像

#### 镜像构建

- 基础镜像: eclipse-temurin:21-jre-alpine（运行时阶段）
- 构建策略: 多阶段（构建阶段使用 eclipse-temurin:21-jdk，运行时阶段使用 JRE Alpine）
- Dockerfile 位置: Dockerfile（项目根目录）

---

## 第 4 节: 集成需求

### 4.1 外部服务访问

| 外部服务 | 协议 | 认证方式 | 数据格式 | 方向 |
|---------|------|---------|---------|------|
| PostgreSQL Cluster (#1843) | TCP (5432) | IAM Auth + Secrets Manager | SQL | 出站 |
| API Gateway（通过 VPC Link） | HTTPS | 无（应用层认证） | JSON | 入站 |

### 4.2 API Gateway 集成

- **网关类型**: REST API
- **端点类型**: Regional
- **认证**: 网关层面无认证（由应用处理）
- **限流**: 速率 100 rps，突发 200
- **VPC Link**: 目标为端口 80 上的内部 NLB

---

## 第 5 节: 部署需求

### 5.1 分阶段基础设施部署

#### 第 1 阶段: 开发环境
- **目标**: dev
- **进入条件**: VPC 和子网基础设施可用，PostgreSQL dev 实例可访问
- **操作**:
  1. 通过 Terraform 部署 ECR 仓库和 IAM 角色
  2. 通过 Terraform 部署 ALB、ECS 集群和 ECS 服务
  3. 构建并推送初始容器镜像到 ECR
  4. 验证健康检查通过，日志流向 CloudWatch
- **退出条件**: Java Card Service 可通过内部 ALB DNS 访问，健康检查返回 HTTP 200

#### 第 2 阶段: 测试环境
- **目标**: test
- **进入条件**: dev 阶段完成并验证，PostgreSQL test 实例可访问
- **操作**:
  1. 使用 test 特定的 tfvars 复制 dev Terraform 配置
  2. 部署 API Gateway 路由与 VPC Link 集成
  3. 对 test 环境运行集成测试套件
- **退出条件**: 所有集成测试通过，API Gateway 路由返回成功响应

#### 第 3 阶段: 预发布环境
- **目标**: staging
- **进入条件**: test 阶段完成并验证
- **操作**:
  1. 使用类生产配置部署 staging 基础设施（2 个任务，自动扩缩容）
  2. 启用 CloudWatch 警报和 SNS 通知
  3. 对 staging 运行性能和负载测试
  4. 测试部署回滚程序
- **退出条件**: 性能测试满足 P99 < 500 ms 目标，回滚程序已验证

#### 第 4 阶段: 生产环境
- **目标**: prod
- **进入条件**: staging 阶段完成，所有利益相关者批准生产部署
- **操作**:
  1. 通过手动审批关卡部署生产基础设施
  2. 执行生产冒烟测试
  3. 启用所有 CloudWatch 警报并验证 SNS 交付
  4. 与值班团队进行运维操作手册审查
- **退出条件**: 生产冒烟测试通过，警报激活，值班团队确认手册审查完成

### 5.2 基础设施验证

- [ ] 所有 Terraform 资源成功创建（terraform plan 显示无变更）
- [ ] ALB 健康检查对所有目标返回 HTTP 200
- [ ] ECS 任务在部署后 120 秒内达到 RUNNING 状态
- [ ] CloudWatch 日志从应用容器正常流出
- [ ] 安全组规则仅允许所需流量
- [ ] 自动扩缩容策略在模拟负载下正确触发

---

## 第 6 节: 测试需求

### 6.1 基础设施验证测试

| 测试 ID | 测试描述 | 预期结果 | 环境 |
|---------|---------|---------|------|
| IVT-001 | ALB 在 HTTPS 端口 443 上响应 | HTTP 200 响应 | 全部 |
| IVT-002 | HTTP 端口 80 重定向到 HTTPS | HTTP 301 重定向到端口 443 | 全部 |
| IVT-003 | ECS 集群具有活跃服务 | 服务 ACTIVE，desired_count 匹配 | 全部 |
| IVT-004 | 目标组健康主机数 > 0 | healthy_count >= min_capacity | 全部 |
| IVT-005 | CloudWatch 日志组接收日志条目 | 日志流包含 JSON 条目 | 全部 |
| IVT-006 | API Gateway 路由返回 200 | /card/v1/health 返回 HTTP 200 | test, staging, prod |

### 6.2 多环境测试

- [ ] dev 基础设施与 test 基础设施结构匹配（相同模块引用）
- [ ] staging 配置与 prod 配置匹配（相同实例大小和可用区数量）
- [ ] 环境特定值从 tfvars 文件正确应用
- [ ] 跨环境访问按设计被阻止（无 dev 到 prod 的连接）

### 6.3 性能测试

| 测试场景 | 目标指标 | 阈值 | 持续时间 |
|---------|---------|------|---------|
| 基线负载（100 rps） | P99 < 200 ms | 200 ms | 5 分钟 |
| 峰值负载（1000 rps） | P99 < 500 ms | 500 ms | 10 分钟 |
| 压力测试（1500 rps） | 无 5xx 错误 | 0 错误 | 15 分钟 |
| 自动扩缩容触发 | 120 秒内扩出 | 120 秒 | 10 分钟 |

---

## 第 7 节: 文档需求

### 7.1 技术文档

- [ ] ALB + Fargate 方案与 EC2 + Docker 方案的架构决策记录（ADR）
- [ ] Terraform 模块 README，包含输入/输出变量文档
- [ ] /card/v1/* 端点的 API 规范
- [ ] 其他微服务消费 Java Card Service 的集成指南

### 7.2 运维文档

- [ ] 部署程序操作手册（初始部署和更新）
- [ ] ALB、ECS 和数据库连接故障的 incident response 指南
- [ ] 手动扩容和缩容的扩展操作指南
- [ ] 带有分步说明的部署回滚程序

---

## 第 8 节: 依赖

### 8.1 外部依赖

| 依赖 | 负责方 | 延迟影响 | 缓解措施 |
|------|--------|---------|---------|
| AWS ECS Fargate 服务配额 | AWS Support | 如果配额不足将阻止生产部署 | 在第 3 阶段之前请求配额增加 |
| ACM 证书签发 | 安全团队 | 阻止 ALB 的 HTTPS 设置 | 在第 1 阶段请求证书 |
| 内部 DNS 区域委派 | 网络团队 | 阻止 ALB 的私有 DNS 解析 | 在第 1 阶段提交请求 |

### 8.2 内部依赖

| 依赖 | Issue / PR | 状态 | 需求时间 |
|------|-----------|------|---------|
| PostgreSQL 集群基础设施 | #1843 | 已完成 | 第 1 阶段（dev 部署） |
| VPC 和子网配置 | 现有 | 可用 | 第 1 阶段（网络） |
| IAM 基线策略 | 现有 | 可用 | 第 1 阶段（角色） |

### 8.3 参考实现

- **现有 ECS Fargate 模块**: `infrastructure/terraform/java-fargate-alb/modules/ecs/main.tf`
- **环境 tfvars 示例**: `infrastructure/terraform/java-fargate-alb/environments/dev/terraform.tfvars`
- **ALB 模块参考**: `infrastructure/terraform/java-fargate-alb/modules/alb/main.tf`

---

## 第 9 节: 验收标准总结

### 按环境验收

#### 开发环境
- [ ] 通过 terraform apply 在 dev 中部署并验证基础设施
- [ ] Java Card Service 在 dev 中可通过内部 ALB DNS 访问
- [ ] dev 中健康检查持续通过（/actuator/health 返回 HTTP 200）
- [ ] 日志在 dev 中流向 CloudWatch 日志组 /ecs/java-card-service
- [ ] 容器镜像成功构建并推送到 ECR

#### 测试环境
- [ ] 通过 terraform apply 在 test 中部署并验证基础设施
- [ ] test 中健康检查持续通过
- [ ] test 中 API Gateway 路由返回成功响应
- [ ] 对 test 环境的集成测试通过

#### 预发布环境
- [ ] 在 staging 中使用类生产配置部署基础设施（2 个任务，自动扩缩容已启用）
- [ ] 在 staging 中性能测试通过，P99 < 500 ms（1000 rps）
- [ ] CloudWatch 警报在 staging 中模拟故障时正确触发
- [ ] 在 staging 中测试并验证部署回滚程序
- [ ] SNS 警报通知已交付到值班 Slack 频道

#### 生产环境
- [ ] 通过手动审批在 prod 中部署并验证基础设施
- [ ] prod 中健康检查持续通过
- [ ] 所有 CloudWatch 警报在 prod 中激活并验证
- [ ] 运维操作手册已由值班团队审查和批准
- [ ] ALB 访问日志确认写入 S3

### 基础设施能力清单

- [ ] 具有 HTTPS 终止的 ALB 在所有环境中运行
- [ ] ECS Fargate 集群在每个环境中运行
- [ ] 容器镜像正在构建并推送到 ECR，扫描已启用
- [ ] 自动扩缩容策略在 staging 和 prod 中已配置并测试
- [ ] 环境隔离已验证（无跨环境 IAM 或网络访问）
- [ ] 安全组遵循最小权限规则

### 运维检查清单

- [ ] 已为 staging 和 prod 创建 CloudWatch 监控仪表板
- [ ] 已配置 CPU、内存和 5xx 错误率的警报阈值
- [ ] 值班团队已审查运维操作手册
- [ ] 部署回滚程序已在 staging 中测试

> **重要提示**: 所有验收标准必须在目标环境中验证后才能标记为完成。仅凭 dev 环境验证不足以作为 staging 或 prod 的签署依据。

---

## 相关文档

- **差距分析**: `devops/docs/requirements/GAP_1880_alb_fargate_java_card_service.md`
- **任务列表**: `devops/docs/requirements/TASKS_1880_alb_fargate_java_card_service.md`
- **GitHub Issue**: #1880 - ALB + Fargate Infrastructure for Java Container Deployment

---

**文档版本**: 2.0
**最后更新**: 2025-01-20
**变更说明**: 版本 2.0 - 利益相关者审查完成，需求已批准实施
**下次审查**: 第 2 阶段完成时（test 环境验证）
