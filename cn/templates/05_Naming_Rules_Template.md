# 命名规范规则 - [项目标题]

<!-- ==============================================================================
     指令：SPEC 编码模板 - 命名规则
     ==============================================================================
     本模板定义了基础设施和平台工程项目的命名规范标准。涵盖 AWS 资源、数据库对象、
     Terraform 定义和标签策略。一致的命名可以减少运维混乱、加速故障排查并强化治理。

     如何使用本模板：
     1. 将所有 [PLACEHOLDER] 标记替换为项目特定的内容。
     2. 删除不适用于您项目的资源章节。
     3. 遵循行内注释（<!-- 指令：... -->）中的指导。
     4. 通过 CI/CD 验证脚本强制执行规则（参见第 6 节）。
     5. 引入新资源类型时更新本文档。
     ============================================================================ -->

**文档编号**：NAMING_[PROJECT_NAME]
<!-- 指令：使用简短的大写字母、下划线分隔的标识符。
     示例：NAMING_ALB_Fargate, NAMING_payment_service_v2 -->

**议题**：#[ISSUE_NUMBER] - [议题标题]
<!-- 指令：引用 GitHub/Jira 议题编号及其标题。
     示例：#2132 - Dev PostgreSQL NLB 路由修复 -->

**创建日期**：[DATE]
<!-- 指令：使用本文档首次创建的日期。
     示例：2025-03-10 -->

**分支**：[BRANCH_NAME]
<!-- 指令：实施工作所在的特性或修复分支。
     示例：bugfix/2132-dev-postgresql-nlb-routing-fix -->

**状态**：草稿
<!-- 指令：跟踪文档生命周期：
     - "草稿"（初始编写）
     - "评审中"（团队反馈）
     - "已批准"（准备执行）
     - "执行中"（CI/CD 验证已激活）
     - "已修订"（反馈或新资源后更新） -->

---

## 目录

- [1. 概述](#1-概述)
- [2. 通用原则](#2-通用原则)
- [3. 资源命名规则](#3-资源命名规则)
- [4. 环境后缀规则](#4-环境后缀规则)
- [5. 标签标准](#5-标签标准)
- [6. 验证](#6-验证)
- [7. 迁移指南](#7-迁移指南)

---

## 1. 概述

### 1.1 目的

<!-- 指令：说明命名规范对本项目为何重要。
     参考不一致命名曾导致的运维痛点，或需要标准化的合规要求。 -->

本文档为 **[PROJECT_TITLE]** 的所有基础设施资源、代码制品和配置对象建立强制性的命名规范。目标如下：

- **可发现性** -- 工程师无需打开控制台即可按名称定位任何资源。
- **可追溯性** -- 每个资源名称可关联到所属服务、环境和成本中心。
- **一致性** -- 自动化流水线可以程序化地解析和验证资源名称。
- **合规性** -- 名称满足 [POLICY_REFERENCE] 中定义的组织治理和审计要求。

<!-- 指令：将 [POLICY_REFERENCE] 替换为要求命名标准的内部策略文档
     或合规框架。如果不存在，请删除合规性要点或替换为"内部最佳实践"。 -->

### 1.2 范围

<!-- 指令：列出本文档管理的资源类别。
     根据项目的技术栈增减类别。 -->

这些规则适用于所有环境中的以下资源类型：

| 类别 | 资源 |
|------|------|
| 计算 | Lambda 函数、ECS 服务、EC2 实例 |
| 网络 | API Gateway、VPC、子网、安全组、NLB/ALB |
| 存储 | S3 存储桶、DynamoDB 表、EFS、EBS 卷 |
| 数据库 | RDS 实例、Aurora 集群、表列、索引 |
| IAM | 角色、策略、实例配置文件 |
| 可观测性 | CloudWatch 日志组、告警、仪表盘 |
| 配置 | SSM 参数、Secrets Manager 密钥 |
| 基础设施即代码 | Terraform 资源、模块、状态文件 |
| CI/CD | CodePipeline 流水线、CodeBuild 项目、Jenkins 任务 |

<!-- 指令：调整上表以匹配项目实际的资源清单。
     删除未使用类别的行，添加缺失类别的行（例如 Kafka 主题、Redis 集群、WAF 规则）。 -->

---

## 2. 通用原则

<!-- 指令：以下原则具有普适性，请勿删除。
     您可以添加项目特定的原则，但此处列出的四项必须保留作为基础。 -->

### 2.1 一致性原则

同一类型的所有资源在每个环境中必须遵循相同的命名模式。`dev` 环境中的 Lambda 函数必须使用与 `prod` 中对应资源相同的结构模式。

**规则**：如果两名工程师独立命名同一概念资源，结果名称必须完全一致。

### 2.2 描述性原则

资源名称必须能够传达其**用途**、**所属服务**和**环境**，无需读者查阅额外文档。

**规则**：新团队成员必须能仅从名称判断资源的用途。

### 2.3 简洁性原则

名称应在保持描述性的同时尽可能简短。避免冗余的前缀或后缀，这些信息已通过资源类型或标签传达。

**规则**：名称最大长度不得超过目标资源中最严格的平台限制（例如 Lambda 64 字符，S3 63 字符）。

<!-- 指令：如果项目目标平台有更严格的限制，请更新字符限制。
     DynamoDB 表允许 255 字符，安全组允许 255 字符，IAM 角色名允许 64 字符，
     S3 存储桶名允许 63 字符。 -->

### 2.4 环境感知原则

每个可部署资源必须在名称中编码其目标环境。这可以防止意外的跨环境操作，并使资源归属关系一目了然。

**规则**：没有环境标识的资源名称无效。

---

## 3. 资源命名规则

<!-- 指令：每个子章节定义特定资源类型的命名模式。
     删除您不使用的资源的章节。按照相同格式（模式、理由、示例表）
     添加新章节。 -->

### 3.1 Lambda 函数

**模式**：
```
{stage}-{abbreviated-service}-{function-name}
```

<!-- 指令：为您的项目定义 "abbreviated-service" 的含义。
     它应该是 2-5 个字符的短代码。例如："card" 代表 card-service，
     "auth" 代表 authentication-service，"pmt" 代表 payment-service。
     在团队维基中维护服务缩写注册表。 -->

**理由**：前导的 stage 段在排序视图和 CloudWatch 日志路径中按环境分组函数。

**分隔符**：连字符（`-`）| **字符限制**：64 字符

| 资源 | 模式 | 示例 |
|------|------|------|
| 卡片处理 | `{stage}-card-process` | `dev-card-process` |
| 认证校验 | `{stage}-auth-validate-token` | `staging-auth-validate-token` |
| 通知发送 | `{stage}-notif-send-email` | `prod-notif-send-email` |
| 报表生成 | `{stage}-rpt-generate-monthly` | `test-rpt-generate-monthly` |
| 数据同步处理器 | `{stage}-sync-[ENTITY]` | `dev-sync-customer` |

### 3.2 API Gateway 路由

**模式**：
```
/{stage}/{service}/{version}/{resource}[/{resource-id}[/{sub-resource}]]
```

<!-- 指令：调整路由模式以匹配您的 API 设计标准。
     一些团队偏好 /api/{service}/{version}/{resource} 而不在路径中
     包含 stage（改用 stage 变量）。选择一种约定并在此文档中记录。 -->

**理由**：基于 URL 的版本控制允许同时部署多个 API 版本，并为消费者提供清晰的路由。

| 资源 | 模式 | 示例 |
|------|------|------|
| 列出账户 | `GET /{stage}/card/v1/accounts` | `GET /prod/card/v1/accounts` |
| 按 ID 获取账户 | `GET /{stage}/card/v1/accounts/{id}` | `GET /prod/card/v1/accounts/acc-12345` |
| 创建交易 | `POST /{stage}/pmt/v1/transactions` | `POST /dev/pmt/v1/transactions` |
| 健康检查 | `GET /{stage}/{service}/v1/health` | `GET /staging/auth/v1/health` |

### 3.3 DynamoDB 表

**模式**：
```
malbank-{service}-{entity}-{environment}
```

<!-- 指令：将 "malbank" 替换为您组织的前缀。
     前缀确保全局唯一性并在 AWS 控制台中按组显示表。
     一些团队使用公司名称或项目代码。 -->

**理由**：DynamoDB 表名在 AWS 账户内是全局的。组织前缀防止与其他项目的表发生冲突。

**分隔符**：连字符（`-`）| **字符限制**：255 字符

| 资源 | 模式 | 示例 |
|------|------|------|
| 客户档案 | `malbank-{service}-customer-{env}` | `malbank-card-customer-dev` |
| 交易记录 | `malbank-pmt-transaction-{env}` | `malbank-pmt-transaction-prod` |
| 会话存储 | `malbank-auth-session-{env}` | `malbank-auth-session-staging` |
| 审计日志 | `malbank-{service}-audit-{env}` | `malbank-pmt-audit-test` |
| 配置数据 | `malbank-{service}-config-{env}` | `malbank-core-config-dev` |

### 3.4 数据库列和属性

<!-- 指令：这些规则适用于关系型数据库（PostgreSQL、MySQL）
     和 NoSQL 属性名（DynamoDB、MongoDB）。如果您的 ORM 或框架
     强制使用不同的约定，请调整大小写规则。 -->

**列/属性大小写**：表列和 DynamoDB 属性使用 `camelCase`。
**索引大小写**：二级索引和约束名使用 `PascalCase`。

**理由**：`camelCase` 与 JavaScript/TypeScript 生态系统和 DynamoDB 单表设计保持一致。索引使用 `PascalCase` 在视觉上区分访问模式与数据属性。

| 资源 | 约定 | 示例 |
|------|------|------|
| 表列 | `camelCase` | `customerId`, `createdAt`, `accountBalance` |
| 主键属性 | `camelCase` | `pk`, `sk`, `userId` |
| GSI 名称 | `PascalCase` | `GSI_ByCustomer`, `GSI_ByStatus` |
| LSI 名称 | `PascalCase` | `LSI_CreatedAt` |
| 外键列 | `camelCase` 加 `Id` 后缀 | `accountId`, `transactionId` |
| 布尔列 | `camelCase` 加 `is/has` 前缀 | `isActive`, `hasVerified` |
| 时间戳列 | `camelCase` 加 `At` 后缀 | `createdAt`, `updatedAt`, `deletedAt` |

<!-- 指令：如果您的项目在 PostgreSQL 中使用 snake_case 作为列名，
     请替换上述规则并相应更新示例表。
     记录任何偏离 camelCase 的理由。 -->

### 3.5 S3 存储桶

**模式**：
```
{organization}-{purpose}-{environment}-{region-code}
```

<!-- 指令：S3 存储桶名在所有 AWS 账户中必须全局唯一。
     使用简短的区域代码："us1" 代表 us-east-1，"ap1" 代表 ap-southeast-1。 -->

**理由**：全局唯一性要求结合环境和区域标识，用于多区域部署。

**分隔符**：连字符（`-`）| **字符限制**：63 字符
**限制**：仅允许小写字母、数字和连字符。不允许下划线或大写字母。

| 资源 | 模式 | 示例 |
|------|------|------|
| 应用日志 | `malbank-app-logs-{env}-{region}` | `malbank-app-logs-prod-us1` |
| 文档上传 | `malbank-docs-upload-{env}-{region}` | `malbank-docs-upload-dev-us1` |
| Terraform 状态 | `malbank-tfstate-{env}-{region}` | `malbank-tfstate-prod-us1` |
| 数据湖原始层 | `malbank-datalake-raw-{env}-{region}` | `malbank-datalake-raw-staging-us1` |
| 制品存储 | `malbank-artifacts-{env}-{region}` | `malbank-artifacts-test-us1` |

### 3.6 IAM 角色和策略

**角色模式**：`{organization}-{service}-{purpose}-role-{environment}`
**策略模式**：`{organization}-{service}-{purpose}-policy-{environment}`

<!-- 指令：末尾的 "-role" / "-policy" 后缀用于在日志和信任关系文档中
     区分角色和策略。 -->

**分隔符**：连字符（`-`）| **字符限制**：64 字符

| 资源 | 模式 | 示例 |
|------|------|------|
| Lambda 执行角色 | `malbank-{service}-execution-role-{env}` | `malbank-card-execution-role-dev` |
| ECS 任务角色 | `malbank-{service}-task-role-{env}` | `malbank-pmt-task-role-prod` |
| S3 读取策略 | `malbank-{service}-s3-read-policy-{env}` | `malbank-rpt-s3-read-policy-staging` |
| 跨账户角色 | `malbank-{service}-xaccount-role-{env}` | `malbank-core-xaccount-role-prod` |
| CI/CD 部署角色 | `malbank-deploy-pipeline-role-{env}` | `malbank-deploy-pipeline-role-dev` |

### 3.7 安全组

**模式**：`{organization}-{service}-{purpose}-sg-{environment}`

<!-- 指令：包含环境后缀可保持与其他命名模式的一致性，
     并有助于日志分析，即使安全组的作用域是 VPC，
     在技术上已经按环境隔离。 -->

**分隔符**：连字符（`-`）| **字符限制**：255 字符

| 资源 | 模式 | 示例 |
|------|------|------|
| ALB 安全组 | `malbank-{service}-alb-sg-{env}` | `malbank-card-alb-sg-prod` |
| 数据库安全组 | `malbank-{service}-db-sg-{env}` | `malbank-auth-db-sg-dev` |
| Lambda 安全组 | `malbank-{service}-lambda-sg-{env}` | `malbank-pmt-lambda-sg-staging` |
| 出站安全组 | `malbank-{service}-egress-sg-{env}` | `malbank-core-egress-sg-prod` |
| 内部服务安全组 | `malbank-{service}-internal-sg-{env}` | `malbank-rpt-internal-sg-test` |

### 3.8 CloudWatch 日志组

**模式**：`/{organization}/{service}/{resource-type}/{resource-name}`

<!-- 指令：使用正斜杠创建分层日志结构。
     这样可以在 /{organization}/{service} 级别设置保留策略，
     并级联到所有子日志组。 -->

**分隔符**：正斜杠（`/`）用于层级，连字符（`-`）用于段内

| 资源 | 模式 | 示例 |
|------|------|------|
| Lambda 日志 | `/malbank/{service}/lambda/{function-name}` | `/malbank/card/lambda/dev-card-process` |
| ECS 任务日志 | `/malbank/{service}/ecs/{task-family}` | `/malbank/pmt/ecs/payment-processor` |
| API Gateway 日志 | `/malbank/{service}/apigw/{api-name}` | `/malbank/auth/apigw/auth-api-prod` |
| 应用日志 | `/malbank/{service}/app/{component}` | `/malbank/core/app/user-service` |

### 3.9 SSM 参数

**模式**：`/{organization}/{environment}/{service}/{category}/{parameter-name}`

<!-- 指令：常见类别：config、secret、feature-flag、endpoint、arn。
     对包含凭证或敏感配置值的参数使用 SecureString 类型。 -->

**分隔符**：正斜杠（`/`）用于层级，连字符（`-`）用于段内

| 资源 | 模式 | 示例 |
|------|------|------|
| 数据库端点 | `/malbank/{env}/{service}/endpoint/db` | `/malbank/dev/auth/endpoint/db` |
| API 密钥（密钥） | `/malbank/{env}/{service}/secret/api-key` | `/malbank/prod/pmt/secret/api-key` |
| 功能开关 | `/malbank/{env}/{service}/feature-flag/{flag}` | `/malbank/staging/card/feature-flag/new-ui` |
| Lambda ARN | `/malbank/{env}/{service}/arn/{function}` | `/malbank/dev/pmt/arn/process-payment` |
| 配置值 | `/malbank/{env}/{service}/config/{key}` | `/malbank/prod/core/config/max-retries` |

### 3.10 Terraform 资源

**资源块模式**：`resource "aws_{type}" "{environment}_{service}_{name}" { ... }`
**模块模式**：`module "{environment}_{service}_{purpose}" { ... }`

<!-- 指令：Terraform 资源地址是状态内部的，不影响实际的 AWS 资源名称
     （通过 "name" 参数设置）。一致的内部命名使 Terraform 代码可导航。 -->

**分隔符**：下划线（`_`）遵循 Terraform 惯例

| 资源 | 模式 | 示例 |
|------|------|------|
| Lambda 函数 | `resource "aws_lambda_function" "dev_card_process"` | -- |
| S3 存储桶 | `resource "aws_s3_bucket" "prod_docs_upload"` | -- |
| IAM 角色 | `resource "aws_iam_role" "staging_auth_execution"` | -- |
| 安全组 | `resource "aws_security_group" "dev_card_alb"` | -- |
| 模块调用 | `module "prod_card_alb"` | -- |

<!-- 指令：每个资源块内的 "name" 或 "name_prefix" 参数
     必须遵循第 3.1-3.9 节中的 AWS 资源模式。
     Terraform 内部名称独立于 AWS 资源名称。 -->

---

## 4. 环境后缀规则

<!-- 指令：每个可部署资源必须编码其目标环境。
     根据项目的部署流水线增减环境。 -->

### 4.1 环境映射

| 环境 | 后缀 | 短代码 | 示例（Lambda） | 示例（S3 存储桶） |
|------|------|--------|----------------|-------------------|
| 开发 | `dev` | `d` | `dev-card-process` | `malbank-app-logs-dev-us1` |
| 测试 | `test` | `t` | `test-card-process` | `malbank-app-logs-test-us1` |
| 预发布 | `staging` | `s` | `staging-card-process` | `malbank-app-logs-staging-us1` |
| 生产 | `prod` | `p` | `prod-card-process` | `malbank-app-logs-prod-us1` |

<!-- 指令："短代码" 列是可选的，在字符限制紧张时使用。
     如果您的项目不需要短代码，请删除该列。 -->

### 4.2 环境位置规则

<!-- 指令：记录每种资源类型中环境标识出现在名称的哪个位置。
     位置不一致是造成混乱的常见原因。 -->

| 资源类型 | 环境位置 | 示例 |
|----------|----------|------|
| Lambda 函数 | 前缀（前导） | `dev-card-process` |
| API Gateway 路由 | 首个路径段 | `/prod/card/v1/accounts` |
| DynamoDB 表 | 后缀（尾随） | `malbank-card-customer-dev` |
| S3 存储桶 | 区域代码之前 | `malbank-app-logs-prod-us1` |
| IAM 角色 | 后缀（尾随） | `malbank-card-execution-role-prod` |
| 安全组 | 后缀（尾随） | `malbank-card-alb-sg-dev` |
| CloudWatch 日志组 | 第二路径层级 | `/malbank/prod/card/lambda/...` |
| SSM 参数 | 第二路径层级 | `/malbank/dev/auth/endpoint/db` |
| Terraform 资源 | 前缀（前导） | `dev_card_process` |

---

## 5. 标签标准

<!-- 指令：标签通过提供名称中无法容纳的元数据来补充命名规范。
     更新必需标签列表以匹配您组织的标签策略。 -->

### 5.1 必需标签

每个支持标签的资源必须包含以下所有标签：

| 标签键 | 描述 | 示例值 | 必需 |
|--------|------|--------|------|
| `Environment` | 部署环境 | `dev`, `test`, `staging`, `prod` | 是 |
| `Service` | 所属服务或微服务 | `card-service`, `payment-service` | 是 |
| `ManagedBy` | 管理资源的工具或团队 | `terraform`, `eks-operator`, `manual` | 是 |
| `CostCenter` | 成本分配中心代码 | `CC-1001`, `CC-2048` | 是 |
| `Project` | 项目或计划标识符 | `BE_Infra`, `FIP-1688` | 是 |
| `Owner` | 负责的团队或个人 | `platform-team`, `arthur.ren` | 是 |
| `CreatedDate` | 资源创建日期 | `2025-03-10` | 是 |

<!-- 指令：CostCenter 值必须与您组织财务系统的代码匹配。
     在强制执行此标签之前，请与财务团队协调。 -->

### 5.2 可选标签

| 标签键 | 描述 | 示例值 |
|--------|------|--------|
| `Backup` | 备份策略标识 | `daily`, `weekly`, `none` |
| `DataClassification` | 数据敏感级别 | `public`, `internal`, `confidential`, `restricted` |
| `ExpiresAfter` | 临时资源的自动删除日期 | `2025-12-31` |
| `Repository` | 源代码仓库 URL | `github.com/org/BE_Infra` |
| `Version` | 部署的应用版本 | `v2.1.0` |

### 5.3 标签执行

<!-- 指令：描述如何强制执行标签。常见方法：AWS SCP、
     AWS Config 规则、Terraform Sentinel 策略、CI/CD 流水线检查。 -->

缺少必需标签的资源必须：

1. 在 `terraform plan` 阶段被 CI/CD 流水线拒绝。
2. 如果在现有环境中被发现，向 `#platform-alerts` Slack 频道发送告警。
3. 在发现后 5 个工作日内完成整改。

---

## 6. 验证

### 6.1 自动化验证脚本

<!-- 指令：调整此 bash 脚本以在您的 CI/CD 流水线中验证命名规范。
     它检查 Terraform 计划输出中的命名违规。
     也可以在开发期间本地运行。 -->

```bash
#!/usr/bin/env bash
# naming-validator.sh -- 验证资源命名规范
# 用法：./naming-validator.sh <terraform-plan-json>
# 退出码：0 = 所有名称合规, 1 = 检测到违规

set -euo pipefail
PLAN_FILE="${1:?用法: $0 <terraform-plan-json>}"
VIOLATIONS=0

# -- 配置 ------------------------------------------------------------
ORG_PREFIX="[ORGANIZATION_PREFIX]"
# 指令：设置您的组织前缀。示例："malbank"

LAMBDA_PATTERN="^(dev|test|staging|prod)-[a-z0-9-]+$"
DYNAMODB_PATTERN="^${ORG_PREFIX}-[a-z0-9-]+-(dev|test|staging|prod)$"
S3_PATTERN="^${ORG_PREFIX}-[a-z0-9-]+-(dev|test|staging|prod)-[a-z0-9]+$"

# -- 验证函数 -----------------------------------------------------
validate_names() {
  local rtype="$1" pattern="$2" jq_filter="$3"
  echo "=== 验证 ${rtype} ==="
  names=$(jq -r "${jq_filter}" "$PLAN_FILE" 2>/dev/null || true)
  for name in $names; do
    [[ ! "$name" =~ $pattern ]] && { echo "  违规: '$name'"; ((VIOLATIONS++)); }
  done
}

validate_names "Lambda" "$LAMBDA_PATTERN" \
  '.planned_values.root_module.resources[] | select(.type=="aws_lambda_function") | .values.function_name // empty'
validate_names "DynamoDB" "$DYNAMODB_PATTERN" \
  '.planned_values.root_module.resources[] | select(.type=="aws_dynamodb_table") | .values.name // empty'
validate_names "S3" "$S3_PATTERN" \
  '.planned_values.root_module.resources[] | select(.type=="aws_s3_bucket") | .values.bucket // empty'

# -- 标签验证 -----------------------------------------------------------
# 指令：更新 REQUIRED_TAGS 以匹配第 5.1 节。
echo "=== 验证必需标签 ==="
REQUIRED_TAGS=("Environment" "Service" "ManagedBy" "CostCenter" "Project")
jq -c '.planned_values.root_module.resources[] | select(.values.tags!=null)' "$PLAN_FILE" \
  2>/dev/null | while read -r res; do
  addr=$(echo "$res" | jq -r '.address')
  for tag in "${REQUIRED_TAGS[@]}"; do
    echo "$res" | jq -e ".values.tags.${tag}" >/dev/null 2>&1 \
      || echo "  违规: ${addr} 缺少 '${tag}'"
  done
done

# -- 汇总 ------------------------------------------------------------------
[ "$VIOLATIONS" -gt 0 ] && { echo "失败: ${VIOLATIONS} 个违规。"; exit 1; } \
  || { echo "通过: 所有名称合规。"; exit 0; }
```

<!-- 指令：将此脚本集成到您的 CI/CD 流水线中，在
     `terraform plan` 之后、`terraform apply` 之前执行。示例：
       - name: 验证命名规范
         run: ./scripts/naming-validator.sh tfplan.json -->

### 6.2 命名检查清单

<!-- 指令：在代码评审期间使用此检查清单手动验证命名合规性。
     在批准 PR 前勾选每一项。 -->

- [ ] 所有新资源名称遵循第 3 节定义的模式。
- [ ] 环境后缀与目标部署环境匹配（第 4 节）。
- [ ] 没有资源名称超过其类型的字符限制。
- [ ] 所有可标记资源上存在必需标签（第 5.1 节）。
- [ ] 标签值使用正确格式（例如 `dev` 而不是 `development`）。
- [ ] S3 存储桶名仅包含小写字母、数字和连字符。
- [ ] DynamoDB 表名包含组织前缀。
- [ ] IAM 角色名以 `-role-{env}` 结尾，策略以 `-policy-{env}` 结尾。
- [ ] 安全组名称以 `-sg-{env}` 结尾。
- [ ] CloudWatch 日志组使用分层 `/org/service/type/name` 模式。
- [ ] SSM 参数路径使用分层 `/org/env/service/category/name` 模式。
- [ ] Terraform 资源地址使用下划线并遵循 `env_service_name` 模式。
- [ ] 不使用未经批准缩写注册表中的缩写。
- [ ] 名称不在意外位置包含环境标识术语。

### 6.3 常见错误

<!-- 指令：根据项目中发生过的命名违规添加条目。
     在回顾会议期间更新此表。 -->

| # | 错误 | 错误示例 | 正确示例 | 参考 |
|---|------|----------|----------|------|
| 1 | Lambda 名称缺少环境标识 | `card-process` | `dev-card-process` | 3.1 |
| 2 | S3 存储桶名中使用下划线 | `malbank_app_logs_dev_us1` | `malbank-app-logs-dev-us1` | 3.5 |
| 3 | DynamoDB 表名中使用大写字母 | `Malbank-Card-Customer-Dev` | `malbank-card-customer-dev` | 3.3 |
| 4 | 环境后缀格式错误 | `malbank-card-customer-development` | `malbank-card-customer-dev` | 4.1 |
| 5 | 缺少组织前缀 | `card-customer-dev` | `malbank-card-customer-dev` | 3.3 |
| 6 | 安全组缺少 `-sg` 后缀 | `malbank-card-alb-dev` | `malbank-card-alb-sg-dev` | 3.7 |
| 7 | 资源名中使用 camelCase | `malbank-cardProcess-dev` | `malbank-card-process-dev` | 2.1 |
| 8 | IAM 角色缺少 `-role` 后缀 | `malbank-card-execution-dev` | `malbank-card-execution-role-dev` | 3.6 |
| 9 | 环境标识位置错误 | `card-dev-process` | `dev-card-process` | 4.2 |
| 10 | 超出字符限制 | `malbank-very-long-service-name-...` | 缩短以适应限制 | 2.3 |

---

## 7. 迁移指南

<!-- 指令：在重命名现有资源以符合这些命名规范时使用本章节。
     重命名通常是破坏性操作，请谨慎规划。 -->

### 7.1 何时重命名

以下情况应重命名资源：

1. **治理审计**：合规审查发现命名违规。
2. **标准化冲刺**：专门将所有资源与本文件对齐的努力。
3. **新资源引入**：正在创建或迁移的资源尚未合规。
4. **事件根因**：与命名相关的混乱导致了运维事件。

<!-- 指令：制定关于重命名是否对现有资源强制执行还是仅对新资源
     强制执行的政策。许多团队选择"新资源强制执行，现有资源
     适时迁移"以避免风险。 -->

### 7.2 迁移流程

**步骤 1：影响评估**

```
[ ] 识别对当前资源名称的所有引用：
    - Terraform 状态
    - 应用代码 / 环境变量
    - IAM 策略和信任关系
    - CloudWatch 告警和仪表盘
    - 文档和运维手册
    - CI/CD 流水线配置
```

<!-- 指令：在上方的检查清单中添加项目特定的引用来源。
     例如，如果您的项目使用 Step Functions 引用 Lambda ARN，
     请将其添加到列表中。 -->

**步骤 2：创建新资源**

1. 使用正确名称在现有资源旁边创建新资源。
2. 验证其功能正常且配置相同。
3. 对新资源名称运行集成测试。

**步骤 3：迁移引用**

1. 更新所有 Terraform 配置、应用配置和 IAM 策略。
2. 更新监控、告警和仪表盘配置。
3. 更新文档和运维手册。

**步骤 4：验证**

```
[ ] 所有集成测试使用新资源名称通过
[ ] 应用代码中没有对旧资源名称的引用
[ ] Terraform 状态中没有对旧资源名称的引用
[ ] CloudWatch 指标正在为新资源流动
[ ] 回滚程序已记录并测试
```

**步骤 5：下线旧资源**

1. 验证旧资源流量为零（检查 CloudWatch 指标至少 24 小时）。
2. 通过 Terraform（而非手动）删除旧资源。
3. 在团队频道中宣布重命名并更新资产清单。

### 7.3 向后兼容性

<!-- 指令：定义项目在重命名过渡期间如何处理向后兼容性。
     对于 API 路由尤其重要。 -->

| 资源类型 | 策略 | 过渡期 |
|----------|------|--------|
| Lambda 函数 | 同时部署两个名称；通过别名路由流量 | [时长，例如 30 天] |
| API Gateway 路由 | 将旧路由重定向到新路由 | [时长，例如 90 天] |
| DynamoDB 表 | 复制到新表；旧表设为只读 | [时长，例如 60 天] |
| S3 存储桶 | 复制到新存储桶；CloudFront 重定向 | [时长，例如 30 天] |
| IAM 角色 | 创建具有相同策略的新角色；更新信任关系 | [时长，例如 14 天] |
| 安全组 | 先附加新安全组再分离旧安全组 | [时长，例如 7 天] |
| SSM 参数 | 创建新参数；更新消费端；删除旧参数 | [时长，例如 14 天] |

<!-- 指令：将 [时长] 占位符替换为团队批准的实际过渡期。
     较短的过渡期减少运维开销，但需要更快的消费端迁移。 -->

### 7.4 回滚程序

如果迁移导致问题：

1. 将 Terraform 配置恢复到前一个提交并运行 `terraform apply`。
2. 验证旧资源仍然存在且可运行。
3. 进行无责备的事后复盘。用经验教训更新本指南。

<!-- 指令：添加项目特定的回滚步骤。例如，如果您的项目使用
     蓝/绿 Lambda 部署，记录如何将流量切回旧别名。 -->

---

## 页脚

| 字段 | 值 |
|------|-----|
| **版本** | 1.0.0 |
| **最后更新** | [DATE] |
| **下次评审日期** | [DATE + 90 天] |
| **文档负责人** | [TEAM_OR_INDIVIDUAL] |
| **批准人** | [APPROVER_NAME] |

<!-- 指令：更新页脚字段：
     - 版本：语义化版本。新增为小版本，破坏性变更为大版本。
     - 最后更新：最近更改的日期。
     - 下次评审日期：安排定期评审（建议：每季度）。
     - 文档负责人：负责本文档的团队或个人。
     - 批准人：批准这些标准的工程经理或架构师。
     填写完字段后删除此指令注释。 -->

<!-- ==============================================================================
     命名规范规则模板结束
     ==============================================================================
     本模板是 spec-coding-templates 集合的一部分。
     如有问题或贡献，请联系平台工程团队。
     ============================================================================ -->
