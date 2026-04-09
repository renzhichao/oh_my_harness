# 自动任务配置 (AUTO_TASK_CONFIG) - [项目标题]

<!-- ==============================================================================
     指令: SPEC 编码模板 - 自动任务配置
     ==============================================================================
     本模板定义了基础设施和平台工程项目的自动化任务执行配置。
     它提供了一个结构化的 YAML 配置，用于驱动任务自动化、分支策略、
     提交约定、状态跟踪和验证门禁。

     如何使用本模板:
     1. 将所有 [占位符] 标记替换为项目特定内容。
     2. 将 YAML 配置块复制到项目根目录或 .spec/ 目录下
        名为 AUTO_TASK_CONFIG.yaml 的文件中。
     3. 按照内联注释 (<!-- 指令: ... -->) 的指引进行操作。
     4. 使用字段参考 (第 3 节) 验证您的配置。
     5. 将此配置与任务列表文档 (模板 04) 配对使用。
     ============================================================================ -->

**文档 ID**: AUTOCONFIG_[PROJECT_NAME]
<!-- 指令: 简短、大写、下划线分隔。示例: AUTOCONFIG_ALB_FARGATE_JAVA_SERVICE -->

**Issue**: #[ISSUE_NUMBER] - [Issue 标题]
<!-- 指令: 引用 GitHub/Jira issue。示例: #1880 - ALB + Fargate 基础设施 -->

**创建日期**: [DATE]
<!-- 指令: 首次创建日期。示例: 2025-01-15 -->

**分支**: [BRANCH_NAME]
<!-- 指令: 必须与 YAML 配置中的 branch.name 匹配。示例: feature/1880-alb-fargate-java-service -->

**状态**: 草稿
<!-- 指令: 生命周期: 草稿 -> 评审 -> 已批准 -> 活跃 -> 已完成 -->

---

## 目录

- [第 1 节: 概述](#第-1-节-概述)
- [第 2 节: 配置模板](#第-2-节-配置模板)
- [第 3 节: 配置字段参考](#第-3-节-配置字段参考)
- [第 4 节: 使用示例](#第-4-节-使用示例)
- [第 5 节: 与任务列表集成](#第-5-节-与任务列表集成)
- [第 6 节: 自定义指南](#第-6-节-自定义指南)
- [第 7 节: 故障排除](#第-7-节-故障排除)

---

## 第 1 节: 概述

### 1.1 什么是 AUTO_TASK_CONFIG?

AUTO_TASK_CONFIG 是一个基于 YAML 的配置文件，用于驱动基础设施和平台工程工作流的自动化任务执行。它定义了任务如何执行、分支和提交如何管理、状态如何跟踪，以及在继续之前必须通过哪些验证门禁。它与任务列表文档 (模板 04) 配对使用，以实现完整的自动化覆盖。

### 1.2 何时使用本模板

- 具有顺序或并行阶段的多步骤基础设施部署
- 环境提升工作流 (dev、test、staging、prod)
- 需要验证门禁的批量配置任务
- 需要任务执行一致性和可审计性的项目

### 1.3 与任务列表文档的集成

<!-- 指令: 确保 status_tracking.file 指向正确的任务列表文档。 -->

```
[AUTO_TASK_CONFIG.yaml] ----引用----> [task-list.md]
        |                                    |
        | 驱动执行                            | 定义任务
        v                                    v
   [自动化引擎] ----更新------> [task-list.md 状态]
```

---

## 第 2 节: 配置模板

<!-- 指令: 将下方的 YAML 块复制到 AUTO_TASK_CONFIG.yaml 中。
     将所有 [占位符] 值替换为项目特定配置。
     配置确定后删除内联 # 注释。 -->

```yaml
# ==============================================================================
# AUTO_TASK_CONFIG - [项目名称]
# 指令: 将 [项目名称] 替换为您的项目标识符。
# ==============================================================================

execution:
  mode: auto | semi-auto | manual
  # 指令: auto=完全自动化, semi-auto=在门禁处暂停, manual=仅跟踪
  max_parallel_tasks: [number]
  # 指令: 最大并发任务数。使用 1 表示顺序执行。范围: 1-5。
  stop_on_failure: true | false
  auto_commit: true | false
  commit_message_prefix: "[prefix]"

branch:
  name: "[branch-name]"
  create_if_missing: true | false
  base_branch: "main"

commit:
  auto_stage: true | false
  # 指令: 设为 true 时，自动暂存所有已更改的文件。
  conventional_commits: true | false
  message_template: "[type]([scope]): [description]"
  # 指令: 可用令牌: [type], [scope], [description], [task_id]

status_tracking:
  file: "[path/to/task-list.md]"
  # 指令: 任务列表 Markdown 文件的路径 (模板 04)。相对于项目根目录。
  format: "emoji" | "text"
  # 指令: emoji=[ ]/[x] 复选框; text=TODO/DONE/BLOCKED/SKIPPED。
  update_frequency: "after_each_task" | "after_each_phase" | "manual"

progress_documentation:
  enabled: true | false
  file: "[path/to/progress.md]"
  include_timestamps: true | false

blocking_conditions:
  # 指令: 中止或改变执行的条件。操作: stop, skip, ask。
  - condition: "Uncommitted changes in working directory"
    action: "stop"
  - condition: "Target environment unreachable"
    action: "stop"
  - condition: "Validation gate failure"
    action: "ask"
  - condition: "Branch divergence from base"
    action: "ask"

environments:
  order: ["dev", "test", "staging", "prod"]
  deploy_sequence: true | false
  # 指令: 设为 true 时，dev 必须成功才能进入 test，test 必须成功才能进入 staging，以此类推。
  require_approval: ["staging", "prod"]
  # 指令: 自动化在这些环境中暂停以等待人工确认。

validation:
  pre_task:
    - "[validation command]"
    # 指令: 示例: "terraform validate", "aws sts get-caller-identity"
  post_task:
    - "[validation command]"
    # 指令: 示例: "terraform plan -detailed-exitcode"
  pre_phase:
    - "[validation command]"
    # 指令: 示例: "terraform init -backend-config=env.hcl"
  post_phase:
    - "[validation command]"
    # 指令: 示例: "pytest tests/integration/"
```

---

## 第 3 节: 配置字段参考

<!-- 指令: 使用此表验证您的 YAML 配置。
     检查"必填"列以识别必需字段。 -->

| 字段 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `execution.mode` | string | 是 | `auto` | 自动化级别: `auto`, `semi-auto`, `manual` |
| `execution.max_parallel_tasks` | integer | 是 | `3` | 最大并发任务数。使用 `1` 表示顺序执行 |
| `execution.stop_on_failure` | boolean | 是 | `true` | 当任何任务失败时中止所有执行 |
| `execution.auto_commit` | boolean | 否 | `true` | 每个任务后自动提交 |
| `execution.commit_message_prefix` | string | 否 | `"[automate]"` | 自动生成提交消息的前缀 |
| `branch.name` | string | 是 | - | 自动化工作使用的功能分支名称 |
| `branch.create_if_missing` | boolean | 否 | `true` | 如果分支不存在则创建 |
| `branch.base_branch` | string | 是 | `"main"` | 用于创建分支和合并的基础分支 |
| `commit.auto_stage` | boolean | 否 | `true` | 自动暂存所有已更改的文件 |
| `commit.conventional_commits` | boolean | 否 | `true` | 强制使用 Conventional Commits 格式 |
| `commit.message_template` | string | 否 | `"[type]([scope]): [description]"` | 包含令牌的模板: `[type]`, `[scope]`, `[description]`, `[task_id]` |
| `status_tracking.file` | string | 是 | - | 任务列表 Markdown 文件的路径 |
| `status_tracking.format` | string | 否 | `"emoji"` | 状态显示: `emoji` 或 `text` |
| `status_tracking.update_frequency` | string | 否 | `"after_each_task"` | 更新频率: `after_each_task`, `after_each_phase`, `manual` |
| `progress_documentation.enabled` | boolean | 否 | `true` | 启用自动进度日志记录 |
| `progress_documentation.file` | string | 否 | `"progress.md"` | 进度日志文件的路径 |
| `progress_documentation.include_timestamps` | boolean | 否 | `true` | 在条目中包含 ISO 8601 时间戳 |
| `blocking_conditions[].condition` | string | 是 | - | 阻塞条件的描述 |
| `blocking_conditions[].action` | string | 是 | `"stop"` | 操作: `stop`, `skip`, 或 `ask` |
| `environments.order` | string[] | 否 | `["dev","test","staging","prod"]` | 有序的环境名称 |
| `environments.deploy_sequence` | boolean | 否 | `true` | 强制顺序提升 |
| `environments.require_approval` | string[] | 否 | `["staging","prod"]` | 需要人工审批的环境 |
| `validation.pre_task` | string[] | 否 | `[]` | 每个任务执行前运行的命令 |
| `validation.post_task` | string[] | 否 | `[]` | 每个任务执行后运行的命令 |
| `validation.pre_phase` | string[] | 否 | `[]` | 每个阶段开始前运行的命令 |
| `validation.post_phase` | string[] | 否 | `[]` | 每个阶段完成后运行的命令 |

---

## 第 4 节: 使用示例

<!-- 指令: 根据您的项目调整这些示例。每个都是可以复制和修改的
     完整配置。 -->

### 4.1 基础示例 - 基础设施部署

```yaml
# AUTO_TASK_CONFIG - Terraform VPC 部署
# 指令: 单环境 Terraform 部署，带有最小化验证。

execution:
  mode: auto
  max_parallel_tasks: 1
  stop_on_failure: true
  auto_commit: true
  commit_message_prefix: "[tf-deploy]"

branch:
  name: "feature/1920-vpc-network-setup"
  create_if_missing: true
  base_branch: "main"

commit:
  auto_stage: true
  conventional_commits: true
  message_template: "feat(network): [description] - refs #[task_id]"

status_tracking:
  file: "devops/docs/tasks/TASK_LIST_1920.md"
  format: "emoji"
  update_frequency: "after_each_task"

progress_documentation:
  enabled: true
  file: "devops/docs/progress/PROGRESS_1920.md"
  include_timestamps: true

blocking_conditions:
  - condition: "Uncommitted changes in working directory"
    action: "stop"
  - condition: "Terraform state lock detected"
    action: "stop"

environments:
  order: ["dev"]
  deploy_sequence: false
  require_approval: []

validation:
  pre_task:
    - "terraform fmt -check"
    - "terraform validate"
  post_task:
    - "terraform plan -detailed-exitcode"
  pre_phase:
    - "terraform init -backend-config=dev.hcl"
  post_phase:
    - "terraform output -json > devops/outputs/dev-vpc.json"
```

### 4.2 高级示例 - 多环境带门禁

```yaml
# AUTO_TASK_CONFIG - Java Fargate 服务多环境部署
# 指令: 完整的环境提升，带有审批门禁和验证。

execution:
  mode: semi-auto
  max_parallel_tasks: 2
  stop_on_failure: true
  auto_commit: true
  commit_message_prefix: "[fargate-deploy]"

branch:
  name: "feature/1880-alb-fargate-java-service"
  create_if_missing: false
  base_branch: "main"

commit:
  auto_stage: true
  conventional_commits: true
  message_template: "[type]([scope]): [description]"

status_tracking:
  file: "devops/docs/tasks/TASK_LIST_1880.md"
  format: "emoji"
  update_frequency: "after_each_phase"

progress_documentation:
  enabled: true
  file: "devops/docs/progress/PROGRESS_1880.md"
  include_timestamps: true

blocking_conditions:
  - condition: "Uncommitted changes in working directory"
    action: "stop"
  - condition: "Target environment unreachable"
    action: "stop"
  - condition: "Validation gate failure"
    action: "ask"
  - condition: "Branch divergence from base"
    action: "ask"
  - condition: "ALB target group health check failing"
    action: "stop"

environments:
  order: ["dev", "test", "staging", "prod"]
  deploy_sequence: true
  require_approval: ["staging", "prod"]

validation:
  pre_task:
    - "terraform fmt -check"
    - "terraform validate"
    - "aws sts get-caller-identity"
  post_task:
    - "terraform plan -detailed-exitcode"
    - "aws elbv2 describe-target-health --target-group-arn $TG_ARN"
  pre_phase:
    - "terraform init -backend-config=env/$ENV.hcl"
  post_phase:
    - "pytest tests/integration/test_$ENV.py"
    - "terraform output -json > devops/outputs/$ENV-outputs.json"
```

### 4.3 最小示例 - 单任务自动化

```yaml
# AUTO_TASK_CONFIG - 快速 Lambda 部署
# 指令: 小型任务的最小配置。仅填写必填字段。

execution:
  mode: auto
  max_parallel_tasks: 1
  stop_on_failure: true

branch:
  name: "bugfix/2105-lambda-timeout-fix"
  create_if_missing: true
  base_branch: "main"

status_tracking:
  file: "devops/docs/tasks/TASK_LIST_2105.md"
  format: "text"
  update_frequency: "after_each_task"
```

---

## 第 5 节: 与任务列表集成

<!-- 指令: AUTO_TASK_CONFIG 通过 status_tracking.file 与任务列表 (模板 04) 集成。 -->

### 5.1 引用机制

`status_tracking.file` 字段指向一个任务列表 Markdown 文件。自动化引擎读取任务 ID/描述，按阶段顺序执行任务，并根据 `update_frequency` 更新状态字段。

### 5.2 状态更新流程

```
                     AUTO_TASK_CONFIG.yaml
                            |
                 [读取配置设置]
                            |
                            v
   +--------------------------------------------------+
   |              自动化引擎                             |
   |  1. 读取任务列表文件                                 |
   |     v                                              |
   |  2. 解析任务和阶段                                   |
   |     v                                              |
   |  3. 对于每个任务:                                    |
   |     +--> 运行 pre_task 验证                          |
   |     +--> 执行任务                                    |
   |     +--> 运行 post_task 验证                         |
   |     +--> 更新任务列表中的状态                          |
   |     +--> 写入进度日志条目                             |
   |     +--> 提交更改 (如果 auto_commit=true)             |
   |     v                                              |
   |  4. 对于每个阶段:                                    |
   |     +--> 运行 pre_phase 验证                         |
   |     +--> 执行阶段中的所有任务                          |
   |     +--> 运行 post_phase 验证                        |
   +--------------------------------------------------+
                   |                  |
                   v                  v
        [task-list.md]      [progress.md 已更新]
```

### 5.3 进度跟踪示例

执行前的任务列表:

```markdown
### 阶段 1: 网络基础设施
- [ ] T1.1 创建 VPC 和子网
- [ ] T1.2 配置路由表
```

T1.1 完成后的任务列表:

```markdown
### 阶段 1: 网络基础设施
- [x] T1.1 创建 VPC 和子网
- [ ] T1.2 配置路由表
```

进度日志条目:

```
## 2025-01-15T09:32:14Z - T1.1 创建 VPC 和子网
- 状态: 已完成
- 耗时: 47s
- 验证: pre_task 通过, post_task 通过
- 提交: feat(network): create VPC and subnets - refs T1.1
```

---

## 第 6 节: 自定义指南

<!-- 指令: 使用本节扩展默认模板之外的配置。 -->

### 6.1 添加自定义阻塞条件

```yaml
blocking_conditions:
  # 指令: 每个条件需要 "condition" (字符串) 和 "action" (stop|skip|ask)。
  # 按顺序评估；首次匹配生效。

  - condition: "Database migration pending"
    action: "ask"
    # 指令: "ask" 让操作者决定是否继续。

  - condition: "SSL certificate expiring within 7 days"
    action: "stop"
    # 指令: "stop" 对于安全相关条件最安全。

  - condition: "ECS service desired count mismatch"
    action: "ask"
```

### 6.2 自定义验证命令

验证命令是必须返回退出代码 0 的 shell 命令。自动化引擎提供以下环境变量: `$ENV` (当前环境), `$PHASE` (当前阶段), `$TASK_ID` (当前任务标识符)。

```yaml
validation:
  pre_task:
    # 指令: 任务执行前的基础设施验证。
    - "terraform fmt -check -recursive"
    - "tflint --init --config=.tflint.hcl"
  post_task:
    # 指令: 任务完成后的部署验证。
    - "curl -sf $HEALTH_CHECK_URL/health || exit 1"
  pre_phase:
    # 指令: 阶段级环境准备。
    - "aws ssm get-parameter --name /config/$ENV/api-endpoint"
  post_phase:
    # 指令: 阶段级集成验证。
    - "pytest tests/smoke/ -m $ENV --tb=short -q"
```

### 6.3 环境特定覆盖

```yaml
# 指令: 基础配置适用于所有环境。
execution:
  mode: semi-auto
  stop_on_failure: true

environments:
  order: ["dev", "staging", "prod"]
  deploy_sequence: true
  require_approval: ["staging", "prod"]

  # 指令: 每个环境的覆盖配置与基础配置浅层合并。
  overrides:
    dev:
      execution:
        mode: auto
        # 指令: Dev 环境完全自动化运行。
      validation:
        post_task:
          - "curl -sf http://dev-internal.example.com/health"
    staging:
      execution:
        mode: semi-auto
      validation:
        post_task:
          - "curl -sf https://staging.example.com/health"
    prod:
      execution:
        mode: semi-auto
        max_parallel_tasks: 1
        # 指令: Prod 环境严格顺序执行。
      validation:
        post_task:
          - "curl -sf https://www.example.com/health"
          - "python3 scripts/verify_prod_readiness.py"
```

---

## 第 7 节: 故障排除

<!-- 指令: 每个条目包括症状、根本原因和解决方案。 -->

### 7.1 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 任务未执行 | `execution.mode` 设置为 `manual` | 更改为 `auto` 或 `semi-auto` |
| 状态未更新 | `status_tracking.file` 路径不正确 | 验证路径相对于项目根目录且文件存在 |
| 提交未创建 | `auto_commit` 或 `auto_stage` 为 `false` | 将两者都设置为 `true` 以实现完全自动提交 |
| 找不到分支 | `branch.name` 与现有分支不匹配 | 将 `create_if_missing` 设置为 `true` 或验证名称 |
| 验证总是失败 | 命令返回非零退出代码 | 在目标环境中手动测试验证命令 |
| 执行意外停止 | 触发了阻塞条件 | 检查 blocking_conditions 并查看进度日志 |
| 并行任务互相干扰 | 共享状态或文件冲突 | 将 `max_parallel_tasks` 减少到 `1` |
| 环境提升被阻止 | 前一环境验证失败 | 检查前一环境的 post_task 验证输出 |
| 提交消息被拒绝 | 模板不符合 `conventional_commits` | 确保模板生成有效的 conventional 格式 |
| 进度日志未创建 | `progress_documentation.enabled` 为 `false` | 设置为 `true` 并验证文件路径 |

### 7.2 调试模式

```yaml
# 指令: 合并到 main 分支前禁用调试模式。
debug:
  enabled: true
  log_file: "debug/auto_task_debug.log"
  # 指令: 相对于项目根目录的路径。目录必须存在。
  verbosity: "detailed"
  # 指令: 级别: "minimal" (仅错误), "standard" (+警告), "detailed" (完整跟踪)。
  include_environment_variables: false
  # 指令: 警告: 设为 true 会在日志中暴露密钥。仅在本地调试时使用。
  dry_run: false
  # 指令: 设为 true 时，仅报告操作而不执行。用于配置验证。
```

---

**版本**: 1.0
<!-- 指令: 在重大更改时递增版本号。语义化版本控制:
     主版本 (2.0): 破坏性字段变更
     次版本 (1.1): 新增可选字段
     修订版本 (1.0.1): 文档更正 -->

**最后更新**: [DATE]
<!-- 指令: 每次修改配置时更新此日期。 -->
