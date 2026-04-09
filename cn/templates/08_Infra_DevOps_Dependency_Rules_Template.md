# 基础设施与 DevOps 依赖规则 - [PROJECT_TITLE]

<!-- ==============================================================================
     指令：SPEC 编码模板 - 基础设施与 DevOps 依赖规则
     ==============================================================================
     本模板定义了基础设施和平台工程项目的依赖规则。涵盖环境晋升、组件部署
     顺序、跨服务依赖、部署阻断规则、命名/标签规范、验证检查清单以及常见
     运维模式。

     如何使用本模板：
     1. 将所有 [PLACEHOLDER] 标记替换为项目特定的内容。
     2. 删除或调整不适用于本项目的章节。
     3. 按照内联注释 (<!-- INSTRUCTION: ... -->) 的指引进行操作。
     4. 始终保持严重等级系统的一致性：
        - 🔴 硬阻断 (HARD BLOCKER)  = 问题解决前部署无法继续
        - 🟡 软阻断 (SOFT BLOCKER)  = 在有记录的例外情况下可以继续部署
        - 🟢 建议 (ADVISORY)        = 最佳实践；在下一次迭代中处理
     5. 在适用的地方直接使用 ASCII 图表；仅在拓扑变更需要时重新绘制。
     6. 保持环境表、依赖矩阵和阻断规则表与实际的 Terraform 模块
        和 CI/CD 流水线同步。
     ============================================================================ -->

**文档 ID**: DEPS_[PROJECT_NAME]
<!-- 指令：使用简短的、大写的、下划线分隔的标识符。
     示例：DEPS_ALB_Fargate_java_service, DEPS_2132_postgresql_nlb_routing -->

**Issue**: #[ISSUE_NUMBER] - [Issue 标题]
<!-- 指令：引用 GitHub/Jira 的 issue 编号及其标题。
     示例：#2132 - Dev PostgreSQL NLB 路由修复 -->

**创建日期**: [DATE]
<!-- 指令：使用本文档首次创建的日期。
     示例：2025-03-20 -->

**分支**: [BRANCH_NAME]
<!-- 指令：实施工作所在的特性分支。
     示例：bugfix/2132-dev-postgresql-nlb-routing-fix -->

**状态**: 草稿
<!-- 指令：跟踪文档生命周期：
     - "草稿" (初始编写)
     - "审核中" (同行评审)
     - "已批准" (可实施)
     - "已实施" (规则已在流水线中执行)
     - "已废弃" (被更新版本取代) -->

---

## 目录

- [第 1 节：概述](#第-1-节概述)
- [第 2 节：环境依赖规则](#第-2-节环境依赖规则)
- [第 3 节：基础设施组件依赖](#第-3-节基础设施组件依赖)
- [第 4 节：跨服务依赖规则](#第-4-节跨服务依赖规则)
- [第 5 节：部署阻断规则](#第-5-节部署阻断规则)
- [第 6 节：命名与标签依赖](#第-6-节命名与标签依赖)
- [第 7 节：验证检查清单](#第-7-节验证检查清单)
- [第 8 节：常见模式](#第-8-节常见模式)

---

## 第 1 节：概述

### 目的

本文档定义了 [PROJECT_NAME] 的基础设施配置、服务部署和环境晋升的依赖规则。
依赖规则确保资源按正确顺序创建，服务通过已批准的路径通信，并在前置条件不满足时
阻断部署。

<!-- 指令：根据具体项目范围调整目的声明。
     如果项目跨越多个团队，请注明涉及的团队。
     示例："管理跨后端和平台工程团队的 Java Fargate 平台的基础设施配置。" -->

### 范围

| 方面 | 覆盖范围 |
|------|----------|
| **项目** | [PROJECT_NAME] |
| **团队** | [TEAM_1], [TEAM_2] |
| **环境** | dev, test, staging, prod |
| **IaC 工具** | [Terraform / CloudFormation / Pulumi] |
| **CI/CD 平台** | [GitHub Actions / GitLab CI / Jenkins] |
| **云服务商** | [AWS / Azure / GCP] |

<!-- 指令：填写每一行。根据技术栈添加或删除行。
     如果项目使用多个云服务商，请全部列出并注明主/次关系。 -->

### 相关文档

| 文档 | ID | 关系 |
|------|----|------|
| [GAP 分析] | GAP_[PROJECT_NAME] | 基线和差距识别 |
| [需求规格] | REQ_[PROJECT_NAME] | 功能和非功能需求 |
| [架构设计] | ADR_[PROJECT_NAME] | 架构决策记录 |

<!-- 指令：链接到与本项目相关的其他 spec-coding-template 文档。
     如果没有相关文档，请删除此表。 -->

---

## 第 2 节：环境依赖规则

### 2.1 环境晋升顺序

基础设施变更必须遵循以下晋升路径。除非获得 [ROLE_NAME] 的明确例外批准，
否则禁止跳过环境。

<!-- 指令：调整晋升路径以匹配实际的环境流水线。
     根据需要添加或删除环境。
     部分项目包含 "sandbox" 或 "qa" 环境。 -->

```
                     +-------+
                     |  dev  |   <-- 所有变更从这里开始
                     +---+---+
                         |
                         v
                     +-------+
                     | test  |   <-- 自动化集成测试
                     +---+---+
                         |
                         v
                     +---------+
                     | staging |   <-- 预生产一致性检查
                     +----+----+
                          |
                          v
                     +-------+
                     | prod  |   <-- 生产发布
                     +-------+
```

**晋升规则：**

1. 在 `test` 环境中未通过测试运行的情况下，任何变更不得晋升到 `staging`。
2. 在 `staging` 环境中未成功部署的情况下，任何变更不得晋升到 `prod`。
3. 热修复遵循路径 `dev -> staging -> prod`，但要求在 [SLA_HOURS] 小时内完成 PIR（事后复盘）。
<!-- 指令：调整热修复规则。部分组织允许热修复直接进入 staging，
     其他组织要求完整路径。明确事后复盘的 SLA 时限。 -->

### 2.2 环境隔离规则

<!-- 指令：这些规则定义了各环境之间必须保持的隔离方式。
     添加组织合规要求特有的规则。 -->

1. **网络隔离**：每个环境必须位于自己的 VPC 中。禁止非生产环境与生产环境之间的 VPC 对等连接。
2. **IAM 边界**：IAM 角色必须包含环境范围的边界。为 `dev` 创建的角色不得在 `prod` 中被代入。
3. **状态隔离**：Terraform 状态文件必须存储在特定环境的后端中：`[TF_STATE_BUCKET]/[PROJECT]/[ENVIRONMENT]/terraform.tfstate`。
4. **密钥隔离**：密钥必须存储在特定环境的密钥库或参数存储路径中。禁止跨环境共享密钥。
5. **DNS 隔离**：每个环境使用专有的托管区域或子域名：`[ENV].[DOMAIN]`。
6. **日志隔离**：CloudWatch 日志组和 S3 访问日志必须按环境划分。仅允许通过只读跨账户角色进行集中式日志收集。

### 2.3 各环境配置

| 环境 | VPC CIDR | 子网 | ACM 证书 | 集群名称 | 状态存储桶 |
|------|----------|------|----------|----------|------------|
| dev | [DEV_CIDR] | [DEV_SUBNETS] | [DEV_CERT_ARN] | [DEV_CLUSTER] | [DEV_STATE_BUCKET] |
| test | [TEST_CIDR] | [TEST_SUBNETS] | [TEST_CERT_ARN] | [TEST_CLUSTER] | [TEST_STATE_BUCKET] |
| staging | [STAGING_CIDR] | [STAGING_SUBNETS] | [STAGING_CERT_ARN] | [STAGING_CLUSTER] | [STAGING_STATE_BUCKET] |
| prod | [PROD_CIDR] | [PROD_SUBNETS] | [PROD_CERT_ARN] | [PROD_CLUSTER] | [PROD_STATE_BUCKET] |

<!-- 指令：填写所有值。使用 Terraform 变量中的确切 ARN、CIDR 块
     和存储桶名称。如果子网按层划分（公有/私有/数据库），
     请使用逗号分隔的列表或引用单独的子网表。 -->

---

## 第 3 节：基础设施组件依赖

### 3.1 依赖图

以下图示说明了基础设施组件必须创建或更新的顺序。顶部组件必须在下方组件
可以配置之前就已经存在。

<!-- 指令：重新绘制此图以匹配实际的基础设施拓扑。
     以下示例展示了一个常见的 AWS ECS Fargate 配置。
     根据架构需要调整节点名称和箭头。 -->

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

### 3.2 组件依赖矩阵

| 组件 | 依赖于 | 阻断 | TF 模块路径 |
|------|--------|------|-------------|
| VPC | 无 | 安全组、子网、NAT 网关 | `[TF_MODULE_PATH]/vpc` |
| 安全组 | VPC | ALB、ECS 服务、RDS | `[TF_MODULE_PATH]/security-groups` |
| IAM 角色 | 无 | ECS 任务执行、CI/CD | `[TF_MODULE_PATH]/iam` |
| ECS 集群 | VPC、安全组、IAM 角色 | Fargate 服务 | `[TF_MODULE_PATH]/ecs-cluster` |
| ALB / NLB | VPC、安全组 | 目标组、监听器规则 | `[TF_MODULE_PATH]/alb` |
| 目标组 | ALB / NLB | 监听器规则 | `[TF_MODULE_PATH]/target-groups` |
| Fargate 服务 | ECS 集群、ALB、IAM 角色 | API Gateway VPC Link | `[TF_MODULE_PATH]/fargate` |
| ECR 仓库 | IAM 角色 | Fargate 服务 (镜像拉取) | `[TF_MODULE_PATH]/ecr` |
| API Gateway | VPC Link | 路由、集成 | `[TF_MODULE_PATH]/api-gateway` |
| VPC Link | NLB、VPC | API Gateway 私有集成 | `[TF_MODULE_PATH]/vpc-link` |
| ACM 证书 | Route53 (DNS 验证) | ALB HTTPS 监听器 | `[TF_MODULE_PATH]/acm` |
| Route53 记录 | ALB / NLB / API Gateway | DNS 解析 | `[TF_MODULE_PATH]/route53` |

<!-- 指令：此表必须与 Terraform 模块结构保持同步。
     根据实际组件添加或删除行。"阻断" 列列出了在该组件
     就绪之前无法创建的下游组件。 -->

### 3.3 Terraform 模块依赖

<!-- 指令：将下面的树形结构替换为项目 Terraform 代码库中的
     实际模块结构。以目录布局作为事实来源。 -->

```
[PROJECT_ROOT]/
+-- environments/
|   +-- dev/
|   |   +-- main.tf          <-- 根模块 (dev)
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

### 3.4 资源创建顺序

<!-- 指令：以下阶段必须与第 3.1 节的依赖图一致。
     每个阶段列出在该阶段内可以并行创建的资源。 -->

**Phase 1 -- 基础层（无依赖）**

1. VPC、子网、路由表、互联网网关、NAT 网关
2. 安全组（ALB、ECS、数据库）
3. IAM 角色和策略（任务执行、任务角色、CI/CD）
4. ACM 证书（DNS 验证）
5. ECR 仓库

**Phase 2 -- 计算层（依赖于 Phase 1）**

6. ECS 集群
7. CloudWatch 日志组
8. ALB / NLB 及目标组
9. Fargate 服务定义和任务定义
10. ALB 监听器规则和健康检查

**Phase 3 -- 路由和 DNS（依赖于 Phase 2）**

11. VPC Link（API Gateway 私有集成）
12. API Gateway REST API、资源、方法
13. API Gateway VPC Link 集成
14. Route53 DNS 记录（别名指向 ALB / API Gateway）

**Phase 4 -- 验证（依赖于 Phase 3）**

15. 端到端健康检查验证
16. DNS 解析验证
17. TLS 证书验证
18. 冒烟测试套件执行

---

## 第 4 节：跨服务依赖规则

### 4.1 共享基础设施规则

<!-- 指令：共享基础设施是指多个服务所依赖的资源，
     如 VPC、日志存储桶或 KMS 密钥。定义所有权和变更审批规则。 -->

| 共享资源 | 所有者 | 变更审批 | 影响范围 |
|----------|--------|----------|----------|
| VPC | [TEAM_OWNER] | [APPROVAL_PROCESS] | 环境中的所有服务 |
| KMS 密钥 | [TEAM_OWNER] | [APPROVAL_PROCESS] | 所有服务的静态加密 |
| CloudWatch 日志组 | [TEAM_OWNER] | [APPROVAL_PROCESS] | 所有服务的日志 |
| ECR 仓库 | [TEAM_OWNER] | [APPROVAL_PROCESS] | 所有容器服务的镜像存储 |
| Route53 托管区域 | [TEAM_OWNER] | [APPROVAL_PROCESS] | 所有服务的 DNS |

**规则：**

1. 共享基础设施的变更需要至少 [APPROVER_COUNT] 个审批。
2. 共享资源变更必须在指定的维护窗口内部署：[MAINTENANCE_WINDOW]。
3. 在应用任何共享资源变更之前，必须记录回滚计划。
<!-- 指令：明确所需的审批人数和维护窗口。如果组织没有固定的
     维护窗口，请注明"仅变更冻结期"或"任何时间（需审批）。" -->

### 4.2 服务间通信规则

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

<!-- 指令：重新绘制以匹配实际的服务通信拓扑。
     如果服务通过 EventBridge、SQS 或 SNS 通信，
     请在图中包含这些组件。 -->

| 规则 ID | 描述 | 执行方式 |
|---------|------|----------|
| COM-001 | 服务必须通过 API Gateway 或内部 ALB 通信。禁止直接的任务到任务通信。 | 安全组规则 |
| COM-002 | 服务网格通信必须使用 mTLS。 | [SERVICE_MESH_CONFIG] |
| COM-003 | 任何服务不得直接连接到另一服务的数据库。 | IAM 策略边界 |
| COM-004 | 跨账户通信需要 VPC 对等连接或 PrivateLink，并需明确批准。 | 网络 ACL |

### 4.3 数据库访问规则

| 服务 | 数据库 | 访问级别 | 连接方式 | TF 模块 |
|------|--------|----------|----------|---------|
| [SERVICE_A] | [DB_A_NAME] | 读写 | ECS 任务 IAM 认证 | `[TF_MODULE_PATH]/db-a` |
| [SERVICE_B] | [DB_B_NAME] | 只读 | ECS 任务 IAM 认证 | `[TF_MODULE_PATH]/db-b` |
| [SERVICE_C] | [DB_A_NAME] | 只读 | 基于密钥的认证 | `[TF_MODULE_PATH]/db-a-ro` |

<!-- 指令：每个访问数据库的服务都必须列出。如果服务不访问任何数据库，
     则从本表中省略。明确使用的认证方式（IAM 认证、基于密钥的认证等）。 -->

**规则：**

1. 每个服务必须使用自己的 IAM 角色或密钥进行数据库访问。禁止共享凭据。
2. 只读访问应为默认设置。写访问需要在 Terraform 模块中提供明确的理由。
3. 数据库凭据必须每 [ROTATION_DAYS] 天通过 Secrets Manager 轮换。
4. 任何服务不得在 `staging` 或 `prod` 环境中拥有任何数据库的超级用户或管理员权限。

---

## 第 5 节：部署阻断规则

### 5.1 硬阻断

<!-- 指令：硬阻断会完全阻止部署。它们代表继续操作会导致数据丢失、
     安全暴露或服务中断的情况。 -->

| 阻断 ID | 条件 | 检测方式 | 解决方案 |
|---------|------|----------|----------|
| HB-001 | Terraform plan 显示将销毁生产数据存储资源 | `terraform plan` 输出 | 手动审查 plan；添加 `lifecycle prevent_destroy` |
| HB-002 | 安全组允许 0.0.0.0/0 在非 HTTP 端口上的入站流量 | `tfsec` / `checkov` 扫描 | 将 CIDR 限制为已知范围 |
| HB-003 | IAM 策略授予 `*:*` 对 `*` 资源的权限 | `tfsec` / IAM Access Analyzer | 将策略范围限定为最小权限 |
| HB-004 | ACM 证书已过期或处于待验证状态 | `aws acm describe-certificate` | 重新签发或重新验证证书 |
| HB-005 | ECS 任务定义在 prod 中使用特权容器 | `checkov` 扫描 | 移除 `privileged = true` |
| HB-006 | 数据库迁移包含不可逆的数据丢失操作 (DROP TABLE) | 迁移审查脚本 | 添加可逆迁移；同行评审 |
| HB-007 | [CUSTOM_BLOCKER] | [DETECTION_METHOD] | [RESOLUTION_STEPS] |

<!-- 指令：添加项目特定的硬阻断。保持检测方式具体化
     （工具名称、命令或检查项）。解决方案必须是可操作的步骤，
     而不是模糊的指示。 -->

### 5.2 软阻断

<!-- 指令：软阻断会生成警告但不会停止部署。
     必须确认并跟踪以便后续修复。 -->

| 阻断 ID | 条件 | 检测方式 | 解决方案 | 默认操作 |
|---------|------|----------|----------|----------|
| SB-001 | Terraform plan 显示超过 [THRESHOLD] 个资源变更 | `terraform plan` 摘要 | 审查并确认 | 警告 + 需确认 |
| SB-002 | 容器镜像标签为 `latest` 而非 SHA 或版本号 | CI 流水线检查 | 将镜像固定到摘要或版本标签 | 警告 |
| SB-003 | 新资源未定义冒烟测试 | 测试框架 | 在流水线中添加冒烟测试 | 警告 |
| SB-004 | 资源标签不完整（缺少必需标签） | `tfsec` / 自定义脚本 | 添加缺失的标签 | 警告 + 创建工单 |
| SB-005 | 日志保留期未设置或超过 365 天 | `checkov` 扫描 | 设置适当的保留期 | 警告 |
| SB-006 | [CUSTOM_SOFT_BLOCKER] | [DETECTION_METHOD] | [RESOLUTION_STEPS] | [DEFAULT_ACTION] |

### 5.3 环境特定阻断

| 环境 | 附加阻断 | 理由 |
|------|----------|------|
| dev | 无 | 开发环境允许更快的迭代 |
| test | 所有硬阻断 + SB-001、SB-002 | 基线质量门禁 |
| staging | 所有硬阻断 + 所有软阻断 | 必须反映生产就绪状态 |
| prod | 所有硬阻断 + 所有软阻断 + [ROLE_NAME] 的手动审批 | 生产环境的最高安全级别 |

<!-- 指令：调整各环境的阻断升级策略。部分组织要求变更咨询委员会 (CAB)
     审批生产部署。如适用，在此记录该要求。 -->

---

## 第 6 节：命名与标签依赖

### 6.1 资源命名依赖

<!-- 指令：资源名称通常编码了环境、服务和区域信息。
     如果下游资源的名称派生自上游资源，
     请在此记录该依赖关系。 -->

| 资源类型 | 命名模式 | 依赖于 | 示例 |
|----------|----------|--------|------|
| VPC | `[project]-[env]-vpc` | 项目名称、环境 | `myapp-dev-vpc` |
| 子网 | `[project]-[env]-[tier]-subnet-[az]` | VPC 名称 | `myapp-dev-private-subnet-a` |
| 安全组 | `[project]-[env]-[service]-sg` | VPC、服务名称 | `myapp-dev-api-sg` |
| IAM 角色 | `[project]-[env]-[service]-[purpose]-role` | 服务名称 | `myapp-dev-api-exec-role` |
| ECS 集群 | `[project]-[env]-cluster` | 项目名称、环境 | `myapp-dev-cluster` |
| ALB | `[project]-[env]-[service]-alb` | VPC、安全组 | `myapp-dev-api-alb` |
| 目标组 | `[project]-[env]-[service]-tg` | ALB、VPC | `myapp-dev-api-tg` |
| ECR 仓库 | `[project]/[service]` | 服务名称 | `myapp/api-service` |
| S3 存储桶 | `[project]-[env]-[purpose]-[account-id]` | 账户 ID | `myapp-dev-logs-123456789` |
| CloudWatch 日志组 | `/aws/ecs/[project]-[env]-[service]` | ECS 集群 | `/aws/ecs/myapp-dev-api` |

**规则：**

1. 资源名称不得超过 [AWS_NAMING_LIMIT] 个字符。
2. 名称只能使用小写字母、数字和连字符。IAM 角色名称中允许使用下划线。
3. 禁止硬编码资源名称。所有名称必须从 Terraform 变量派生。

### 6.2 各资源类型的必需标签

| 标签键 | 适用资源 | 格式 | 示例 | 执行工具 |
|--------|----------|------|------|----------|
| `Environment` | 所有资源 | `dev`、`test`、`staging`、`prod` | `prod` | `tfsec` |
| `Project` | 所有资源 | 项目简称 | `myapp` | `tfsec` |
| `Service` | 计算、数据库、ALB | 服务简称 | `api-service` | `tfsec` |
| `Owner` | 所有资源 | 团队或个人邮箱 | `platform@example.com` | `tfsec` |
| `CostCenter` | 所有资源 | 成本中心代码 | `CC-1234` | `tfsec` |
| `ManagedBy` | 所有资源 | `terraform` | `terraform` | `tfsec` |
| `Repository` | 所有资源 | 仓库 URL | `github.com/org/repo` | `tfsec` |
| `DataClassification` | 存储、数据库 | `public`、`internal`、`confidential` | `confidential` | `checkov` |

<!-- 指令：使用组织特定的标签扩展此表。如果使用 AWS 标签策略
     或服务控制策略进行强制执行，请在 "执行工具" 列中注明。 -->

### 6.3 基于标签的访问控制规则

| 规则 | 标签条件 | 效果 | 范围 |
|------|----------|------|------|
| TBA-001 | `Environment = prod` 且调用者不在 [PROD_ACCESS_ROLE] 中 | 拒绝修改操作 | 所有生产资源 |
| TBA-002 | `DataClassification = confidential` 且操作为 `s3:GetObject` | 仅允许经批准的角色访问 | S3 存储桶 |
| TBA-003 | `ManagedBy != terraform` | 拒绝所有操作 (手动变更) | 所有资源 |
| TBA-004 | 缺少 `Environment` 标签 | 拒绝资源创建 | 所有资源 |

<!-- 指令：将这些规则映射到实际的 IAM 策略或服务控制策略。
     如果组织使用 ABAC (基于属性的访问控制)，
     请在此记录标签架构和策略绑定。 -->

---

## 第 7 节：验证检查清单

### 7.1 部署前检查清单

<!-- 指令：在启动部署之前必须验证每个复选框。
     添加项目特定的检查项。这些检查应尽可能在 CI/CD 中自动化。 -->

- [ ] Terraform `plan` 无错误且已由 [MIN_REVIEWERS] 名团队成员审查
- [ ] 所有硬阻断（第 5.1 节）已解决或已记录例外情况
- [ ] 所有软阻断（第 5.2 节）已确认
- [ ] 容器镜像已构建、扫描并推送到 ECR，使用版本化标签（非 `latest`）
- [ ] 数据库迁移脚本已在低等级环境中测试
- [ ] 安全组规则已审查；无过度宽松的入站规则
- [ ] IAM 策略遵循最小权限原则；无 `*:*` 对 `*` 资源的权限
- [ ] ACM 证书已签发并针对目标域名完成验证
- [ ] Route53 DNS 记录已准备（创建或更新）
- [ ] 回滚计划已记录并测试
- [ ] 部署安排在已批准的维护窗口内
- [ ] 利益相关者已收到部署窗口通知

### 7.2 部署后验证

- [ ] Terraform `apply` 无错误完成
- [ ] ECS 任务正在运行且已通过健康检查
- [ ] ALB 目标组显示健康目标
- [ ] API Gateway 在 `/health` 端点返回预期响应
- [ ] DNS 解析返回正确的 IP 地址或别名
- [ ] TLS 证书有效且服务于正确的域名
- [ ] CloudWatch 日志正在写入正确的日志组
- [ ] 告警和警报已配置且功能正常
- [ ] 冒烟测试套件通过，成功率不低于 [MIN_PASS_RATE]%
- [ ] 后续的 `terraform plan` 中无意外资源变更

### 7.3 环境就绪检查清单

<!-- 指令：仅在所有检查通过后将环境标记为就绪。
     此检查清单应在初始设置期间每个环境完成一次，
     并在任何共享基础设施变更后重新验证。 -->

| 检查项 | dev | test | staging | prod |
|--------|-----|------|---------|------|
| VPC 和子网已配置 | [ ] | [ ] | [ ] | [ ] |
| 安全组已创建 | [ ] | [ ] | [ ] | [ ] |
| IAM 角色和策略已创建 | [ ] | [ ] | [ ] | [ ] |
| ECS 集群已激活 | [ ] | [ ] | [ ] | [ ] |
| ECR 仓库已存在 | [ ] | [ ] | [ ] | [ ] |
| ALB / NLB 已配置 | [ ] | [ ] | [ ] | [ ] |
| ACM 证书有效 | [ ] | [ ] | [ ] | [ ] |
| Route53 托管区域已配置 | [ ] | [ ] | [ ] | [ ] |
| CloudWatch 日志组已创建 | [ ] | [ ] | [ ] | [ ] |
| 密钥已存储在 Secrets Manager | [ ] | [ ] | [ ] | [ ] |
| Terraform 状态后端已配置 | [ ] | [ ] | [ ] | [ ] |

### 7.4 回滚验证

<!-- 指令：这些检查验证回滚是否已成功完成，
     且失败的部署没有残留状态。 -->

- [ ] 先前的 Terraform 状态已从备份恢复
- [ ] ECS 任务正在运行先前的容器镜像版本
- [ ] ALB 目标组指向先前的任务集
- [ ] DNS 记录反映部署前状态（如已更改）
- [ ] 数据库迁移已回滚（如适用）
- [ ] CloudWatch 警报已恢复正常状态
- [ ] 已创建事件工单并附有根因摘要
- [ ] 事后复盘会议已在 [POSTMORTEM_SLA] 小时内安排

---

## 第 8 节：常见模式

### 8.1 新服务接入模式

<!-- 指令：此模式描述将新服务添加到基础设施的逐步流程。
     每个步骤必须在下一步开始前完成。 -->

**前置条件**：服务已定义 Dockerfile 和 CI 流水线。

**步骤：**

1. **创建 Terraform 模块**：`[TF_MODULE_PATH]/[NEW_SERVICE]/`
   - 添加 `main.tf`、`variables.tf`、`outputs.tf`
   - 定义 ECS 任务定义、服务和目标组
   - 引用共享的 VPC、安全组和 IAM 角色

2. **在环境模块中注册服务**：更新 `environments/[ENV]/main.tf`
   - 为 `[NEW_SERVICE]` 添加模块块
   - 传入环境特定的变量

3. **创建 ECR 仓库**：添加到共享 ECR 模块或创建独立模块。
   - 设置生命周期策略以保留最近 [IMAGE_RETENTION_COUNT] 个镜像

4. **配置 ALB 监听器规则**：添加路由规则以将流量转发到新目标组。
   - 路径模式：`[SERVICE_PATH]`

5. **创建 API Gateway 集成**（如适用）：添加 VPC Link 集成以进行私有 API 暴露。

6. **添加监控**：为 CPU、内存和任务计数创建 CloudWatch 告警。

7. **更新文档**：在依赖矩阵（第 3.2 节）和数据库访问表（第 4.3 节）中添加服务条目。

8. **在各环境中部署**：遵循第 2.1 节的晋升顺序。

### 8.2 数据库迁移模式

**前置条件**：迁移脚本已审查并在本地测试。

**步骤：**

1. **创建可逆迁移**：每个 `UP` 迁移必须有对应的 `DOWN` 迁移。
2. **在 dev 中测试**：对 dev 数据库运行迁移。验证模式变更和数据完整性。
3. **在 test 中运行**：作为 CI 流水线的一部分执行。验证应用兼容性。
4. **在 staging 中预演**：对生产数据的副本运行。验证性能影响。
5. **应用到 prod**：在维护窗口内执行，并激活回滚计划。
6. **验证**：运行数据完整性检查。监控应用错误率 [MONITORING_WINDOW] 分钟。
7. **完成**：移除回滚标志。如果权限有变更，更新数据库访问表（第 4.3 节）。

<!-- 指令：明确使用的迁移工具（Flyway、Liquibase、Alembic 等）
     以及项目特定的约定，如迁移文件的命名模式。 -->

### 8.3 多区域模式

<!-- 指令：此模式仅适用于需要多区域部署的项目。
     如果项目是单区域的，请删除本节。 -->

**前置条件**：主区域已完全部署。DR 区域已确定。

**步骤：**

1. **复制基础基础设施**：使用相同的 Terraform 模块，配合区域特定的变量，在 DR 区域部署 VPC、子网、安全组和 IAM 角色。
2. **配置跨区域资源**：根据需要设置 S3 跨区域复制、DynamoDB 全局表或 RDS 只读副本。
3. **部署计算层**：在 DR 区域部署 ECS 集群和 Fargate 服务。
4. **配置 DNS 故障转移**：创建 Route53 健康检查和故障转移路由策略。
   - 主区域：`[REGION_PRIMARY]`
   - 次区域：`[REGION_DR]`
5. **测试故障转移**：模拟主区域故障并验证自动 DNS 故障转移。
6. **记录 RTO/RPO**：记录实际达到的恢复时间目标和恢复点目标。
   - RTO 目标：[RTO_TARGET]
   - RPO 目标：[RPO_TARGET]
7. **安排定期 DR 测试**：每 [DR_DRILL_FREQUENCY] 个月进行故障转移演练。

---

## 页脚

| 字段 | 值 |
|------|----|
| **版本** | 1.0 |
| **最后更新** | [DATE] |
| **作者** | [AUTHOR_NAME] |
| **审批人** | [APPROVER_NAME] |
| **下次评审日期** | [REVIEW_DATE] |

<!-- 指令：每次重大变更时更新版本号。使用语义化版本号 (MAJOR.MINOR)。
     对于持续维护的项目，下次评审日期距离上次更新不得超过 6 个月。 -->
