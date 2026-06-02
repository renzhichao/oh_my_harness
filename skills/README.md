# Spec Coding Skills for AI Coding Agents

> 将 Spec Coding 模板转换为可被 Claude Code 等 AI Coding Agent 加载和执行的技能系统。
>
> Convert Spec Coding templates into loadable and executable skills for AI Coding Agents like Claude Code.

**版本 / Version**: 1.0 | **更新日期 / Last Updated**: 2026-06-02

---

## 概述 / Overview

这套技能系统让 AI Coding Agent 能够：
- 系统化地引导用户完成规范开发
- 根据项目上下文选择合适的模板
- 执行跨上下文评审和验证
- 配置和执行自动化任务工作流
- 进行全面的缺陷分析

This skill system enables AI Coding Agents to:
- Systematically guide users through specification development
- Apply appropriate templates based on project context
- Conduct cross-context reviews and validation
- Configure and execute automated task workflows
- Perform comprehensive bug analysis

---

## 技能列表 / Skill List

| 技能 ID | 技能名称 / Name | 用途 / Purpose | 模板 / Template |
|---------|------------------|----------------|-----------------|
| `spec:workflow` | 完整工作流编排 | 执行从初始化到执行的完整 Spec Coding 工作流 | All |
| `spec:review` | 跨上下文评审 | 在技术、安全、运营、业务上下文中验证规范 | N/A |
| `spec:bug` | 缺陷分析报告 | 系统化缺陷调查和修复策略 | Template 09 |

---

## 安装和使用 / Installation & Usage

### 方法 1: Claude Code 内置技能目录

将技能文件复制到 Claude Code 的技能目录：

```bash
# 复制技能文件
cp -r /path/to/oh_my_harness/skills/* ~/.claude/skills/

# 或创建符号链接
ln -s /path/to/oh_my_harness/skills/spec-*.md ~/.claude/skills/
```

### 方法 2: 项目本地技能

在项目根目录创建 `.claude/skills/` 目录：

```bash
mkdir -p /path/to/project/.claude/skills/
cp /path/to/oh_my_harness/skills/spec-*.md /path/to/project/.claude/skills/
```

### 方法 3: 直接调用

在对话中直接引用技能文件路径：

```bash
# 使用完整路径
技能: /path/to/oh_my_harness/skills/spec-workflow.md

# 或使用相对路径（如果技能在项目内）
技能: .claude/skills/spec-workflow.md
```

---

## 使用示例 / Usage Examples

### 示例 1: 启动新项目

```bash
# 用户输入
/spec:workflow project_name="ALB_Fargate_Java_Service" work_type="new_service" issue_number="1880"

# Agent 执行
1. 创建 docs/specs/ 目录
2. 复制模板 01-08
3. 引导填写 GAP Analysis
4. 生成 Requirements
5. 执行评审
6. 创建 FIP
7. 执行评审
8. 生成 Task List
9. 配置 AUTO_TASK_CONFIG
```

### 示例 2: 跨上下文评审

```bash
# 用户输入
/spec:review document_type="REQ" document_path="docs/specs/REQ_1880_alb_fargate.md"

# Agent 执行
1. 读取需求文档
2. 应用技术评审检查清单
3. 应用安全评审检查清单
4. 应用运营评审检查清单
5. 应用业务评审检查清单
6. 生成评审报告
7. 提供批准/修改建议
```

### 示例 3: 缺陷分析

```bash
# 用户输入
/spec:bug issue_number="614" issue_title="Blueprint_Run_Button_Not_Working" error_evidence="ModuleNotFoundError: No module named 'kevin.utils'"

# Agent 执行
1. 创建 BAR 文档
2. 执行根因分析（多层调查）
3. 评估影响
4. 开发多个解决方案（A/B/C）
5. 推荐修复策略
6. 定义测试计划
7. 提供时间估算
```

---

## 技能参数说明 / Skill Parameters

### spec:workflow

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `project_name` | string | ✅ | 项目名称或标识符 |
| `work_type` | string | ❌ | 工作类型：new_service, enhancement, bugfix, migration |
| `issue_number` | string | ❌ | GitHub/Jira issue 编号 |

### spec:review

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `document_type` | string | ✅ | 文档类型：REQ 或 FIP |
| `document_path` | string | ✅ | 规范文档的路径 |

### spec:bug

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `issue_number` | string | ✅ | 缺陷 issue 编号 |
| `issue_title` | string | ✅ | 缺陷标题 |
| `error_evidence` | string | ❌ | 错误日志、堆栈跟踪或证据 |

---

## 扩展开发 / Extension Development

### 创建新技能

1. 在 `skills/` 目录创建新文件：`spec-{name}.md`
2. 添加 frontmatter 元数据
3. 定义执行步骤
4. 指定输出格式

示例：

```markdown
---
name: spec:custom
description: Custom skill description
parameters:
  param1:
    type: string
    required: true
    description: Parameter description
---

# Skill Title

You are performing [skill purpose].

## Steps
1. Step 1
2. Step 2

## Output
Generate [output format]

Begin by [starting instruction].
```

---

## 技能架构 / Skill Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Spec Coding Skills System                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐     ┌─────────────────┐               │
│  │ Core Workflow   │────→│ Context Skills  │               │
│  │    Skills       │     │                 │               │
│  └─────────────────┘     └─────────────────┘               │
│           │                       │                          │
│           ▼                       ▼                          │
│  ┌─────────────────┐     ┌─────────────────┐               │
│  │ Review &        │────→│ Specialized     │               │
│  │ Validation      │     │ Skills          │               │
│  └─────────────────┘     └─────────────────┘               │
│           │                                                │
│           ▼                                                │
│  ┌─────────────────┐                                       │
│  │ Template        │                                       │
│  │ Integration     │───────────────────────┐               │
│  └─────────────────┘                       │               │
│                                             ▼               │
│                              ┌─────────────────────┐        │
│                              │ Spec Coding          │        │
│                              │ Templates (01-09)   │        │
│                              └─────────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 质量保证 / Quality Assurance

### 技能验证清单

每个技能应满足：

- [ ] **清晰的触发条件**: 何时调用此技能
- [ ] **明确的输入参数**: 需要什么信息
- [ ] **结构化执行步骤**: 如何执行
- [ ] **可验证的输出**: 产生什么结果
- [ ] **质量检查点**: 如何验证成功

### 模板集成验证

- [ ] 技能能正确加载模板
- [ ] 能解析 INSTRUCTION 注释
- [ ] 能指导用户填写模板
- [ ] 能验证模板完成度
- [ ] 能应用粒度阈值（任务清单）
- [ ] 能检查 Mermaid 语法
- [ ] 能验证 YAML 配置

---

## 限制和注意事项 / Limitations & Considerations

### 执行环境

- 技能在 Claude Code 环境中执行
- 需要 Agent Tool 支持用于自动化执行
- 某些技能需要外部运行时支持

### 状态管理

- 技能执行状态需要持久化
- 文档生命周期需要跟踪
- 评审发现需要记录

### 依赖关系

- 技能之间有依赖关系（如评审需要规范完成）
- 模板之间有交叉引用
- 需要按正确顺序执行

---

## 故障排除 / Troubleshooting

### 技能加载失败

```bash
# 检查技能文件是否存在
ls ~/.claude/skills/spec-*.md

# 检查 frontmatter 格式
head -10 skills/spec-workflow.md

# 验证 YAML 语法
python -c "import yaml; yaml.safe_load(open('skills/spec-workflow.md'))"
```

### 执行问题

1. **技能不响应**: 确保参数完整
2. **模板找不到**: 检查模板路径
3. **验证失败**: 检查文档格式

---

## 路线图 / Roadmap

### v1.1 (计划中)

- [ ] 添加 `spec:init` 技能（独立初始化）
- [ ] 添加 `spec:gap` 技能（GAP 分析）
- [ ] 添加 `spec:req` 技能（需求规格）
- [ ] 添加 `spec:fip` 技能（架构设计）
- [ ] 添加 `spec:task` 技能（任务分解）
- [ ] 添加 `spec:auto` 技能（自动化配置）
- [ ] 添加模板验证检查点
- [ ] 添加进度跟踪机制

### v2.0 (未来)

- [ ] 支持自定义工作流
- [ ] 集成 Git 操作
- [ ] 自动化提交消息生成
- [ ] 集成测试执行
- [ ] 支持多语言模板

---

## 贡献 / Contributing

欢迎改进和扩展技能系统。

贡献指南：
1. Fork 项目
2. 创建技能分支
3. 添加/改进技能
4. 更新文档
5. 提交 Pull Request

---

## 许可证 / License

Proprietary — 个人非商业学习使用
版权所有 © 2026 Ren Zhichao

未经书面许可，不得重新分发或商业使用。

---

**维护者 / Maintainer**: Spec Coding Templates Team
**联系方式 / Contact**: 通过 GitHub Issues
