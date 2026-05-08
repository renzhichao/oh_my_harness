# 自动任务配置 (AUTO_TASK_CONFIG) - [项目标题]

<!-- ==============================================================================
     指令: SPEC 编码模板 - 自动任务配置 (v2)
     ==============================================================================
     本模板定义了基础设施和平台工程项目的自动化任务执行配置，
     采用 Sub-Agent 架构。每个任务在独立的 Sub-Agent 上下文中执行，
     支持并行执行、上下文隔离和可恢复的工作流。

     如何使用本模板:
     1. 将所有 [占位符] 标记替换为项目特定内容。
     2. 将 YAML 配置块复制到 AUTO_TASK_CONFIG.yaml 文件中。
     3. 按照内联注释（指令标记）的指引进行操作。
     4. 使用字段参考 (第 5 节) 验证您的配置。
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

- [第 1 节: 设计理念](#第-1-节-设计理念)
- [第 2 节: 架构概览](#第-2-节-架构概览)
- [第 3 节: 任务执行生命周期](#第-3-节-任务执行生命周期)
- [第 4 节: 配置模板](#第-4-节-配置模板)
- [第 5 节: 配置字段参考](#第-5-节-配置字段参考)
- [第 6 节: 任务执行规则](#第-6-节-任务执行规则)
- [第 7 节: 与任务列表集成](#第-7-节-与任务列表集成)
- [第 8 节: Orchestrator 执行模板](#第-8-节-orchestrator-执行模板)
- [第 9 节: 并行执行策略](#第-9-节-并行执行策略)
- [第 10 节: 异常处理](#第-10-节-异常处理)
- [第 11 节: 自定义指南](#第-11-节-自定义指南)
- [第 12 节: 故障排除](#第-12-节-故障排除)

---

## 第 1 节: 设计理念

### 1.1 核心原则

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Sub-Agent 任务执行核心原则                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🔒 原生上下文隔离 (Native Context Isolation)                       │
│     ├── 每个任务由独立 Sub-Agent 执行                               │
│     ├── Sub-Agent 拥有独立上下文窗口，天然隔离                      │
│     ├── Orchestrator 主上下文保持干净，不被任务污染                  │
│     └── 无需手动 /clear，隔离由架构保证而非纪律                     │
│                                                                     │
│  🎯 单一职责 (Single Responsibility)                                │
│     ├── 每个 Sub-Agent 只完成一个明确的目标                         │
│     ├── 任务粒度可控，便于追踪和回滚                                │
│     └── 任务完成标准明确（Acceptance Criteria）                     │
│                                                                     │
│  📝 状态持久化 (State Persistence)                                  │
│     ├── 任务状态记录在任务列表文档中                                │
│     ├── 每次状态变更立即持久化                                      │
│     └── 支持任务中断后恢复                                         │
│                                                                     │
│  ⚡ 并行执行 (Parallel Execution)                                   │
│     ├── 无依赖的任务可并行派发至多个 Sub-Agent                      │
│     ├── Orchestrator 负责依赖排序和并发控制                         │
│     └── 通过任务列表和文件系统协调并行结果                          │
│                                                                     │
│  🔄 可重复执行 (Idempotent Execution)                               │
│     ├── 同一任务可安全重复执行                                      │
│     ├── 前置检查确保执行条件满足                                    │
│     └── 失败任务可从断点恢复                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 v1 → v2 核心变化

| 维度 | v1（手动隔离） | v2（Sub-Agent 隔离） |
|------|---------------|---------------------|
| 上下文隔离方式 | 手动 `/clear`、新建会话 | Sub-Agent 天然隔离 |
| 隔离可靠性 | 依赖人为纪律 | 架构级保证 |
| 并行能力 | 不支持（单会话串行） | 原生支持（多 Agent 并行） |
| Orchestrator 上下文 | 被任务细节污染 | 保持干净，只做调度 |
| 任务间通信 | 通过文件/任务列表 | 通过文件/任务列表（不变） |

### 1.3 适用场景

- 具有顺序或并行阶段的多步骤基础设施部署
- 环境提升工作流 (dev、test、staging、prod)
- 需要任务执行一致性和可审计性的项目
- 多模块并行开发以缩短总耗时
- 批量测试编写（可并行、互不依赖）

---

## 第 2 节: 架构概览

### 2.1 两层架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                         两层执行架构                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    Orchestrator（主 Agent）                  │   │
│   │                                                             │   │
│   │  职责：                                                     │   │
│   │  ├── 读取任务列表，解析任务依赖                             │   │
│   │  ├── 选择下一个可执行任务                                   │   │
│   │  ├── 构造 Sub-Agent Prompt                                 │   │
│   │  ├── 派发 Sub-Agent 执行任务                               │   │
│   │  ├── 收集 Sub-Agent 执行结果                               │   │
│   │  ├── 更新任务列表状态                                      │   │
│   │  └── 处理异常和重试                                        │   │
│   │                                                             │   │
│   │  上下文：干净，仅保留调度状态                               │   │
│   └──────────┬──────────────┬──────────────┬────────────────────┘   │
│              │              │              │                         │
│              ▼              ▼              ▼                         │
│   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│   │  Sub-Agent 1 │ │  Sub-Agent 2 │ │  Sub-Agent 3 │              │
│   │  (任务 A)    │ │  (任务 B)    │ │  (任务 C)    │              │
│   │              │ │              │ │              │              │
│   │ 独立上下文   │ │ 独立上下文   │ │ 独立上下文   │              │
│   │ 执行完毕销毁 │ │ 执行完毕销毁 │ │ 执行完毕销毁 │              │
│   └──────────────┘ └──────────────┘ └──────────────┘              │
│                                                                     │
│   通信方式：任务列表文档 + 文件系统 + Git 仓库                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Orchestrator 与 Sub-Agent 职责划分

| 职责 | Orchestrator | Sub-Agent |
|------|-------------|-----------|
| 读取任务列表 | ✅ | ❌（由 Prompt 传入任务详情） |
| 依赖检查 | ✅ | ❌（Orchestrator 保证依赖已满足） |
| 状态管理 | ✅ | ❌（通过返回结果告知） |
| 代码编写 | ❌ | ✅ |
| 测试运行 | ❌ | ✅ |
| Git 提交 | ❌（或代行提交） | ✅ |
| AC 验证 | 接收结果 | ✅（执行验证并报告） |
| 异常重试 | ✅ | ❌（报告失败，Orchestrator 决定） |

---

## 第 3 节: 任务执行生命周期

### 3.1 生命周期图

```
┌──────────┐
│  PENDING │ ◄──────────────────────────────────────────┐
└────┬─────┘                                            │
     │ [Orchestrator 选择任务]                           │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│DEPENDENCY│────►│ 1. 检查前置任务 COMPLETED     │      │
│  CHECK   │     │ 2. 检查前置文件/配置          │      │
└────┬─────┘     └──────────────────────────────┘       │
     │ [依赖满足]                                       │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│ PREPARE  │────►│ 1. 更新任务列表为 RUNNING     │      │
│   AGENT  │     │ 2. 构造 Sub-Agent Prompt      │       │
└────┬─────┘     │ 3. 注入任务详情 + 约束        │       │
     │           └──────────────────────────────┘       │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│  SPAWN   │────►│ 1. 调用 Agent Tool            │      │
│SUB-AGENT │     │ 2. Sub-Agent 在独立上下文执行  │  [失败]
└────┬─────┘     │ 3. 等待执行结果               │      │
     │           └──────────────────────────────┘       │
     │ [返回结果]                                        │
     ▼                                                  │
┌──────────┐     ┌──────────────────────────────┐       │
│  RESULT  │────►│ 1. 解析 Sub-Agent 执行结果    ├──────┘
│  PROCESS │     │ 2. 验证 AC 完成情况           │
└────┬─────┘     │ 3. 检查输出物                 │
     │ [通过]    └──────────────────────────────┘
     ▼
┌──────────┐     ┌──────────────────────────────┐
│ FINALIZE │────►│ 1. Git add & commit          │
│          │     │ 2. 更新任务列表状态           │
└────┬─────┘     │ 3. 记录执行摘要              │
     │           └──────────────────────────────┘
     ▼
┌──────────┐     ┌──────────────────────────────┐
│  NEXT    │────►│ 1. 选择下一个可执行任务      │
│  TASK    │     │ 2. 如有并行任务可同时派发     │
└────┬─────┘     │ 3. 无更多任务则结束           │
     │           └──────────────────────────────┘
     ▼
┌──────────┐
│COMPLETED │
└──────────┘
```

### 3.2 阶段说明

| 阶段 | Orchestrator 动作 | Sub-Agent 动作 |
|------|-------------------|----------------|
| 待执行 (PENDING) | 等待选择 | - |
| 依赖检查 | 验证前置依赖 | - |
| 准备 Agent | 构造 Prompt | - |
| 派发执行 | 调用 Agent Tool | 在独立上下文中执行任务 |
| 结果处理 | 解析结果、验证 AC | - |
| 收尾 | Git commit、更新文档 | - |
| 下一任务 | 选择并派发 | - |
| 已完成 | - | - |

---

## 第 4 节: 配置模板

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
  # 指令: 最大并发 Sub-Agent 数。使用 1 表示顺序执行。范围: 1-5。
  stop_on_failure: true | false
  auto_commit: true | false
  commit_message_prefix: "[prefix]"

sub_agent:
  # 指令: Sub-Agent 派发配置。
  default_mode: foreground | background | worktree
  # 指令: foreground=等待结果, background=派发后继续, worktree=文件隔离。
  subagent_type: general-purpose | [自定义类型]
  # 指令: 可选。匹配工具中可用的 Agent 类型。
  max_retries: 2
  # 指令: 每个失败任务的最大重试次数。范围: 0-3。
  retry_backoff: [30, 60]
  # 指令: 重试间隔秒数。长度必须匹配 max_retries。

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
  - condition: "工作目录有未提交的更改"
    action: "stop"
  - condition: "目标环境不可达"
    action: "stop"
  - condition: "验证门禁失败"
    action: "ask"
  - condition: "分支与基础分支发生偏离"
    action: "ask"

environments:
  order: ["dev", "test", "staging", "prod"]
  deploy_sequence: true | false
  # 指令: 设为 true 时，dev 必须成功才能进入 test，以此类推。
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

## 第 5 节: 配置字段参考

<!-- 指令: 使用此表验证您的 YAML 配置。检查"必填"列以识别必需字段。 -->

### 5.1 执行字段

| 字段 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `execution.mode` | string | 是 | `auto` | 自动化级别: `auto`, `semi-auto`, `manual` |
| `execution.max_parallel_tasks` | integer | 是 | `3` | 最大并发 Sub-Agent 数。使用 `1` 表示顺序执行 |
| `execution.stop_on_failure` | boolean | 是 | `true` | 当任何任务失败时中止所有执行 |
| `execution.auto_commit` | boolean | 否 | `true` | 每个任务后自动提交 |
| `execution.commit_message_prefix` | string | 否 | `"[automate]"` | 自动生成提交消息的前缀 |

### 5.2 Sub-Agent 字段

| 字段 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `sub_agent.default_mode` | string | 否 | `foreground` | 默认派发模式: `foreground`, `background`, `worktree` |
| `sub_agent.subagent_type` | string | 否 | `general-purpose` | 用于专业任务的 Agent 类型 |
| `sub_agent.max_retries` | integer | 否 | `2` | 每个失败任务的最大重试次数 |
| `sub_agent.retry_backoff` | integer[] | 否 | `[30, 60]` | 重试间隔秒数 |

### 5.3 分支与提交字段

| 字段 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `branch.name` | string | 是 | - | 自动化工作使用的功能分支名称 |
| `branch.create_if_missing` | boolean | 否 | `true` | 如果分支不存在则创建 |
| `branch.base_branch` | string | 是 | `"main"` | 用于创建分支和合并的基础分支 |
| `commit.auto_stage` | boolean | 否 | `true` | 自动暂存所有已更改的文件 |
| `commit.conventional_commits` | boolean | 否 | `true` | 强制使用 Conventional Commits 格式 |
| `commit.message_template` | string | 否 | `"[type]([scope]): [description]"` | 包含令牌的模板: `[type]`, `[scope]`, `[description]`, `[task_id]` |

### 5.4 跟踪与验证字段

| 字段 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
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

## 第 6 节: 任务执行规则

### 6.1 执行顺序策略

```yaml
execution_order:
  strategy: "dependency_first"
  # 指令: 可用策略:
  #   dependency_first: 优先执行所有前置依赖任务
  #   phase_sequential: 同阶段内可并行，跨阶段必须顺序
  #   priority_based: 高优先级任务优先派发

  parallel:
    enabled: true
    # 指令: 启用并行派发至多个 Sub-Agent。
    max_concurrent: [number]
    # 指令: 最大并发 Sub-Agent 数。应与 execution.max_parallel_tasks 一致。
    strategy: "dependency_level"
    # 指令: dependency_level = 同依赖层级的任务可并行执行。
```

### 6.2 任务选择算法

```python
def get_next_tasks(task_list: List[Task], max_concurrent: int = 3) -> List[Task]:
    """
    获取下一批可执行的任务（支持并行派发）

    选择规则：
    1. 状态为 PENDING
    2. 所有前置依赖状态为 COMPLETED
    3. 按任务 ID 顺序选择（稳定排序）
    4. 不超过 max_concurrent 个并发任务
    """
    ready_tasks = []

    for task in sorted(task_list, key=lambda t: t.id):
        if task.status != TaskStatus.PENDING:
            continue

        dependencies_met = all(
            dep.status == TaskStatus.COMPLETED
            for dep in task.dependencies
        )

        if not dependencies_met:
            continue

        ready_tasks.append(task)

        if len(ready_tasks) >= max_concurrent:
            break

    return ready_tasks
```

### 6.3 Sub-Agent 模式选择矩阵

| 场景 | 推荐模式 | 理由 |
|------|---------|------|
| 顺序任务（A→B→C） | foreground | 后续任务依赖前序结果 |
| 独立任务（A、B、C 无关） | background | 并行执行节省时间 |
| 可能修改相同文件 | worktree | 文件系统级隔离防冲突 |
| 单一简单任务 | foreground | 无需复杂隔离 |
| 大批量测试编写 | background | 可并行，互不依赖 |

---

## 第 7 节: 与任务列表集成

<!-- 指令: AUTO_TASK_CONFIG 通过 status_tracking.file 与任务列表 (模板 04) 集成。 -->

### 7.1 状态更新触发器

```yaml
task_list_update:
  update_triggers:
    - on_task_start       # Sub-Agent 派发前
    - on_task_complete    # Sub-Agent 成功返回后
    - on_task_failure     # Sub-Agent 失败后

  fields_to_update:
    on_task_start:
      - status: "RUNNING"
      - start_time: "当前时间"
      - agent_mode: "foreground | background | worktree"

    on_task_complete:
      - status: "COMPLETED"
      - end_time: "当前时间"
      - commit_id: "Git Commit SHA"
      - commit_summary: "改动摘要"
      - acceptance_criteria: "逐项标记完成"

    on_task_failure:
      - status: "FAILED"
      - failure_reason: "失败原因"
```

### 7.2 任务状态机

```
                         ┌───────────┐
              ┌─────────►│ CANCELLED │
              │          └───────────┘
              │ [用户取消]
              │
         ┌────┴────┐    [依赖未满足]    ┌─────────┐
         │ PENDING │◄──────────────────│ BLOCKED │
         └────┬────┘                   └────▲────┘
              │                              │
              │ [派发 Sub-Agent]             │ [依赖未完成]
              │                              │
              ▼                              │
         ┌─────────┐                         │
         │ RUNNING │─────────────────────────┘
         └────┬────┘
              │ [Sub-Agent 返回结果]
        ┌─────┴──────┐
        │            │
        ▼            ▼
   ┌──────────┐  ┌────────┐
   │COMPLETED │  │ FAILED │
   └──────────┘  └────┬───┘
                       │ [Orchestrator 重试]
                       ▼
                  ┌─────────┐
                  │ PENDING │
                  └─────────┘
```

### 7.3 Sub-Agent 结果回传协议

<!-- 指令: Sub-Agent 执行完毕后必须按此固定格式返回结果。 -->

```markdown
## Sub-Agent 执行报告

### 执行结果
- **状态**: SUCCESS / FAILED
- **失败原因**: （仅 FAILED 时填写）

### 变更文件
- <文件1> — <变更说明>
- <文件2> — <变更说明>

### Acceptance Criteria 验证
- [x] AC1: <描述>
- [x] AC2: <描述>
- [ ] AC3: <描述> — <未完成原因>

### Git 提交
- Commit ID: <SHA>（如 Sub-Agent 已提交）
- 或: 未提交，需 Orchestrator 代行

### 备注
- <任何需要注意的问题>
```

---

## 第 8 节: Orchestrator 执行模板

### 8.1 Orchestrator 主 Prompt

<!-- 指令: 使用此 Prompt 模板指示 Orchestrator Agent。
     将 [占位符] 替换为项目特定的路径和设置。 -->

```markdown
# 任务编排指令（Orchestrator）

## 角色
你是任务编排器（Orchestrator），负责按任务列表顺序派发 Sub-Agent 执行任务。
你不直接编写代码，而是通过 Agent Tool 委托 Sub-Agent 执行。

## 执行上下文
- **任务列表路径**: [path/to/task-list.md]
- **起始任务 ID**: [TASK-001]
- **最大并行数**: [3]

## 执行步骤

### Step 1: 读取任务列表
读取任务列表文档，构建任务依赖图。

### Step 2: 循环派发
重复以下步骤直到所有任务完成：

1. **选择任务**: 调用 get_next_tasks() 获取可执行任务
2. **构造 Prompt**: 为每个任务生成 Sub-Agent Prompt（见 8.2）
3. **派发执行**: 调用 Agent Tool
   - 有后续依赖 → foreground（等待完成）
   - 无后续依赖 → background（并行执行）
4. **处理结果**: 解析 Sub-Agent 返回报告
5. **更新状态**: 更新任务列表
6. **Git 提交**: 如 Sub-Agent 未提交，代行提交

### Step 3: 输出总结报告

## 重要约束
1. 不要直接编写代码或修改文件（任务列表更新除外）
2. 所有开发工作由 Sub-Agent 完成
3. 每次 Sub-Agent 返回后立即更新任务列表
4. 遇到 FAILED 任务时，分析原因并决定是否重试
5. 并行任务不要超过最大并行数
```

### 8.2 Sub-Agent 任务 Prompt 模板

<!-- 指令: 使用此模板为每个 Sub-Agent 生成 Prompt。
     从任务列表中提取方括号内的内容填充。 -->

```markdown
# 任务执行指令（Sub-Agent）

## 任务信息
- **任务 ID**: [TASK-XXX]
- **任务描述**: [从任务列表提取的完整描述]
- **所属 Phase**: [Phase N]

## 前置任务摘要
<!-- 指令: 如果本任务有已完成的前序任务，包含其摘要。无前序任务则省略此节。 -->
- [TASK-YYY]: [完成摘要，如 "已创建项目目录结构，Commit: abc1234"]

## Acceptance Criteria
<!-- 指令: 从任务列表中复制本任务的 AC 列表。 -->
- [ ] [AC1 描述]
- [ ] [AC2 描述]

## 参考文档
<!-- 指令: 从任务列表中复制参考文档路径。 -->
- [path/to/reference-doc]

## 执行要求

### 代码变更
1. 按任务描述完成开发工作
2. 确保所有 Acceptance Criteria 满足
3. 代码风格与项目现有风格保持一致

### 验证
1. 逐项检查 Acceptance Criteria
2. 运行测试用例（如有）
3. 验证输出物存在且正确

### Git 提交
1. 使用 `git add` 添加变更文件
2. 使用以下格式提交：
   ```
   [type]([task-id]): [简短描述]

   - [改动点1]
   - [改动点2]

   Task: [TASK-XXX]
   ```
3. 记录 Commit ID

## 输出要求
完成后，输出固定格式的执行报告：

```
## Sub-Agent 执行报告

### 执行结果
- **状态**: SUCCESS / FAILED
- **失败原因**: （仅 FAILED 时填写）

### 变更文件
- <文件> — <变更说明>

### Acceptance Criteria 验证
- [x] AC1: <描述>
- [ ] AC2: <描述> — <未完成原因>

### Git 提交
- Commit ID: <SHA>

### 备注
- <任何需要注意的问题>
```

## 重要约束
1. 只完成指定任务，不要执行其他任务
2. 不要修改任务范围外的文件
3. 如遇问题无法继续，报告 FAILED 并说明原因
```

### 8.3 并行派发模板

<!-- 指令: 当同一依赖层级的多个任务可以并行执行时使用。 -->

```markdown
# 并行任务派发指令

## 可并行任务
以下任务无依赖关系，可同时派发：

### Sub-Agent 1: [TASK-XXX]
- **任务描述**: [描述]
- **Acceptance Criteria**: [AC 列表]
- **执行模式**: background

### Sub-Agent 2: [TASK-YYY]
- **任务描述**: [描述]
- **Acceptance Criteria**: [AC 列表]
- **执行模式**: background

## 派发方式
在单条消息中发出多个 Agent tool calls（并行调用）：

Agent Call 1:
- description: "[TASK-XXX]: [简短描述]"
- prompt: <TASK-XXX 完整 Prompt>
- run_in_background: true

Agent Call 2:
- description: "[TASK-YYY]: [简短描述]"
- prompt: <TASK-YYY 完整 Prompt>
- run_in_background: true

## 收集结果
等待所有 background Sub-Agent 完成，逐个处理结果。
```

### 8.4 任务恢复 Prompt

<!-- 指令: 在恢复先前失败或中断的任务时使用。 -->

```markdown
# 任务恢复指令

## 执行上下文
- **任务列表路径**: [path/to/task-list.md]
- **恢复任务 ID**: [TASK-XXX]

## 恢复步骤

### Step 1: 评估当前状态
1. 读取任务列表获取 [TASK-XXX] 当前状态
2. 检查 Git 工作目录状态
3. 分析已完成的工作和待完成的工作

### Step 2: 构造恢复 Prompt
为 Sub-Agent 生成恢复执行 Prompt，包含：
1. 原始任务描述和 AC
2. 已完成的工作（从 Git diff 分析）
3. 待完成的工作
4. 加上"从断点继续"的指示

### Step 3: 派发恢复 Sub-Agent
调用 Agent Tool 执行恢复任务。
```

---

## 第 9 节: 并行执行策略

### 9.1 依赖图分析

```
<!-- 指令: 替换为您的实际任务依赖图。
     识别哪些层级可以并行化。 -->

示例依赖图：

    TASK-001 (PENDING)
        │
        ├── TASK-002 (depends on 001)
        │       │
        │       └── TASK-004 (depends on 002, 003)
        │
        └── TASK-003 (depends on 001)

执行层次分析：
  Level 0: [TASK-001]              → 串行执行
  Level 1: [TASK-002, TASK-003]    → 可并行执行
  Level 2: [TASK-004]              → 串行执行
```

### 9.2 并行调度规则

| 规则 | 描述 |
|------|------|
| 同层级无依赖 | 同一依赖层级中无相互依赖的任务可并行派发 |
| 跨层级 | 不同层级的任务必须串行执行 |
| 最大并发 | 并行数不超过 max_concurrent |
| 失败隔离 | 一个任务失败时，暂停同层级后续派发 |

### 9.3 并行安全约束

| 约束 | 说明 | 检查方式 |
|------|------|----------|
| 文件冲突 | 并行任务不应修改相同文件 | Prompt 中声明变更范围 |
| 依赖完整 | 前置任务必须 COMPLETED | 任务列表状态检查 |
| 资源限制 | 并发不超过 max_concurrent | Orchestrator 计数 |
| 失败隔离 | 一个失败不影响其他 | 各 Sub-Agent 独立 |

---

## 第 10 节: 异常处理

### 10.1 异常类型和处理策略

| 异常类型 | 描述 | Orchestrator 处理 |
|----------|------|-------------------|
| 前置依赖未满足 | 依赖任务未完成 | 标记 BLOCKED，稍后重试 |
| Sub-Agent 执行失败 | Sub-Agent 返回 FAILED | 分析原因，决定重试/跳过/中止 |
| Sub-Agent 超时 | Sub-Agent 长时间未返回 | 等待超时后标记 FAILED |
| AC 验证失败 | 输出不满足要求 | 重新派发（附加修复指示） |
| Git 提交失败 | 提交冲突或权限问题 | 暂停，等待人工解决 |
| 任务列表更新失败 | 文件锁定或格式错误 | 重试或人工更新 |
| 并行任务冲突 | 并行任务修改了相同文件 | 串行化重试或使用 worktree |

### 10.2 失败处理流程

```
Sub-Agent FAILED
       │
       ▼
Orchestrator 分析
Sub-Agent 报告
       │
  ┌────┴────────┬──────────────┬──────────────┐
  │             │              │              │
  ▼             ▼              ▼              ▼
可重试        需修复         需人工介入      应取消
  │           Prompt           │              │
  ▼             ▼              ▼              ▼
重置为       修复后         暂停并         标记为
PENDING      重新派发       通知用户       CANCELLED
重新派发     Sub-Agent

重试限制：每个任务最多重试 max_retries 次（默认: 2）
超限后标记为 FAILED 并暂停后续依赖任务，请求人工介入
```

---

## 第 11 节: 自定义指南

<!-- 指令: 使用本节扩展默认模板之外的配置。 -->

### 11.1 Sub-Agent 类型配置

```yaml
# 指令: 为专业任务配置 Sub-Agent 类型。
sub_agent_types:
  foreground:
    use_when: "任务有后续依赖，需顺序执行"
    tool_params:
      description: "任务简短描述"
      prompt: "完整任务 Prompt"

  background:
    use_when: "任务无后续依赖，可并行执行"
    tool_params:
      description: "任务简短描述"
      prompt: "完整任务 Prompt"
      run_in_background: true

  worktree:
    use_when: "任务可能产生文件冲突，需完全隔离"
    tool_params:
      description: "任务简短描述"
      prompt: "完整任务 Prompt"
      isolation: "worktree"
```

### 11.2 自定义阻塞条件

```yaml
blocking_conditions:
  # 指令: 每个条件需要 "condition" (字符串) 和 "action" (stop|skip|ask)。
  # 按顺序评估；首次匹配生效。

  - condition: "数据库迁移未完成"
    action: "ask"
  - condition: "SSL 证书将在 7 天内过期"
    action: "stop"
  - condition: "ECS 服务期望实例数不匹配"
    action: "ask"
```

### 11.3 环境特定覆盖

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
          - "curl -sf http://dev-internal.[DOMAIN]/health"
    staging:
      execution:
        mode: semi-auto
      validation:
        post_task:
          - "curl -sf https://staging.[DOMAIN]/health"
    prod:
      execution:
        mode: semi-auto
        max_parallel_tasks: 1
        # 指令: Prod 环境严格顺序执行。
      validation:
        post_task:
          - "curl -sf https://www.[DOMAIN]/health"
```

---

## 第 12 节: 故障排除

<!-- 指令: 每个条目包括症状、根本原因和解决方案。 -->

### 12.1 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 任务未执行 | `execution.mode` 设置为 `manual` | 更改为 `auto` 或 `semi-auto` |
| 状态未更新 | `status_tracking.file` 路径不正确 | 验证路径相对于项目根目录且文件存在 |
| 提交未创建 | `auto_commit` 或 `auto_stage` 为 `false` | 将两者都设置为 `true` 以实现完全自动提交 |
| 找不到分支 | `branch.name` 与现有分支不匹配 | 将 `create_if_missing` 设置为 `true` 或验证名称 |
| Sub-Agent 超时 | 任务过于复杂或环境问题 | 拆分为更小的任务；检查网络连通性 |
| 并行任务冲突 | 共享文件修改 | 使用 `worktree` 模式或减少 `max_parallel_tasks` |
| AC 验证失败 | 实现不完整 | 根据失败报告附加修复指示后重新派发 |
| 重试次数超限 | 持续性失败 | 人工调查根因后重新派发 |
| 上下文污染 | Sub-Agent 泄漏到 Orchestrator | 验证使用正确的 Sub-Agent 派发而非内联执行 |

### 12.2 调试模式

```yaml
# 指令: 合并到 main 分支前禁用调试模式。
debug:
  enabled: true
  log_file: "debug/auto_task_debug.log"
  verbosity: "detailed"
  # 指令: 级别: "minimal" (仅错误), "standard" (+警告), "detailed" (完整跟踪)。
  include_environment_variables: false
  # 指令: 警告: 设为 true 会在日志中暴露密钥。仅在本地调试时使用。
  dry_run: false
  # 指令: 设为 true 时，仅报告操作而不执行。用于配置验证。
```

---

## 附录

### A. 快速参考卡片

```
┌─────────────────────────────────────────────────────────────────────┐
│              Sub-Agent 任务自动执行快速参考                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Orchestrator 启动                                                  │
│     1. 读取任务列表                                                 │
│     2. 构建依赖图                                                   │
│     3. 选择可执行任务                                               │
│     4. 为每个任务构造 Sub-Agent Prompt                              │
│     5. 通过 Agent Tool 派发                                        │
│                                                                     │
│  并行执行                                                           │
│     1. 同层级无依赖任务 → background 模式                          │
│     2. 在单条消息中发出多个 Agent tool calls                        │
│     3. 等待所有 background Sub-Agent 完成                          │
│     4. 逐个处理结果                                                 │
│                                                                     │
│  任务完成                                                           │
│     1. 解析 Sub-Agent 执行报告                                     │
│     2. 验证 Acceptance Criteria                                    │
│     3. Git commit（如 Sub-Agent 未提交）                           │
│     4. 更新任务列表状态                                            │
│     5. 选择下一批任务                                               │
│                                                                     │
│  任务失败                                                           │
│     1. 分析 Sub-Agent 报告的失败原因                               │
│     2. 决定：重试 / 修复后重试 / 人工介入                          │
│     3. 重试次数 ≤ max_retries（默认: 2）                           │
│     4. 超限后标记 FAILED 并暂停                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### B. Commit Message 规范

```
<type>(<task-id>): <简短描述>

<详细说明（可选）>

Task: <TASK-ID>
```

**Type 类型**: `feat` (新功能), `fix` (Bug 修复), `docs` (文档), `test` (测试), `refactor` (重构), `chore` (构建/工具)

### C. Agent Tool 参数速查

```yaml
# Agent Tool 派发模式
foreground:
  description: "任务简短描述"
  prompt: "<完整 Sub-Agent 任务 Prompt>"

background:
  description: "任务简短描述"
  prompt: "<完整 Sub-Agent 任务 Prompt>"
  run_in_background: true

worktree:
  description: "任务简短描述"
  prompt: "<完整 Sub-Agent 任务 Prompt>"
  isolation: "worktree"

# 可选 subagent_type 值
available_types:
  general-purpose: "通用任务（默认）"
  python-expert: "Python 开发任务"
  frontend-architect: "前端开发任务"
  quality-engineer: "测试和质量保障任务"
  security-engineer: "安全相关任务"
  refactoring-expert: "代码重构任务"
```

---

**版本**: 2.0
<!-- 指令: 在重大更改时递增版本号。语义化版本控制:
     主版本 (3.0): 破坏性字段变更
     次版本 (2.1): 新增可选字段
     修订版本 (2.0.1): 文档更正 -->

**最后更新**: [DATE]
<!-- 指令: 每次修改配置时更新此日期。 -->
