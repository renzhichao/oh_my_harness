# Spec Coding 示例 — 已填写模板样本

> "ALB + Fargate 基础设施 for Java 容器部署" 项目的已填写规范示例。

---

## 关于这些示例

这些文档是 spec-coding-templates 的**已填写（完成）版本**，基于一个真实的基础设施项目。它们展示了如何将空白模板中的占位符转化为生产就绪的规范文档。

### 项目背景

| 字段 | 值 |
|------|------|
| **项目** | ALB + Fargate 基础设施 for Java Card Service |
| **Issue** | #1880 |
| **团队** | 平台工程团队 |
| **云服务商** | AWS |
| **区域** | ap-southeast-1 |
| **环境** | dev, test, staging, prod |
| **核心服务** | ECS Fargate, ALB, API Gateway, VPC Link, ECR, CloudWatch, IAM |

### 构建内容

一个 Java Spring Boot 微服务，部署在 AWS ECS Fargate 上，前置 Application Load Balancer，通过 VPC Link 与 API Gateway 私有集成实现外部 API 暴露。

---

## 示例文件

| 文件 | 基于模板 | 描述 |
|------|----------|------|
| `filled-gap-analysis.md` | 01_GAP_Analysis_Template.md | ECS Fargate 基础设施的 GAP 差距分析 — 识别容器编排、部署自动化和服务路由方面的差距 |
| `filled-requirements.md` | 02_Requirements_Template.md | ALB、ECS Fargate、API Gateway、容器部署和多环境基础设施的完整需求 |
| `filled-fip.md` | 03_FIP_Template.md | 含 Mermaid 架构图、组件设计、安全设计和分阶段实现计划的 FIP |
| `filled-task-list.md` | 04_Task_List_Template.md | 4 阶段任务分解，含依赖关系、工作量估算和验收标准 |

---

## 如何使用这些示例

### 第一步：先阅读空白模板
打开模板文件（例如 `templates/01_GAP_Analysis_Template.md`），阅读其结构和 `<!-- INSTRUCTION: ... -->` 指令注释。

### 第二步：与已填写版本对比
打开对应的示例文件（例如 `examples/filled-gap-analysis.md`），对比以下内容：
- `[PLACEHOLDER]` 占位符如何被替换为实际内容
- 指令注释如何引导内容创建
- 通用组件名称如何变为具体服务名称

### 第三步：观察流水线流转
观察文档之间的构建关系：
1. **GAP 差距分析**的差距转化为**需求规格说明**（例如："没有容器平台" → FR-1.2.1 "ECS 集群"）
2. **需求规格说明**驱动**FIP** 的架构决策（例如：FR-1.1.1 → 第 2.1 节的 ALB 设计）
3. **FIP** 的实现方案转化为**任务清单**的阶段（例如：FIP 阶段 1 → 任务清单阶段 1）

---

## 关键差异：空白模板 vs 已填写示例

| 方面 | 空白模板 | 已填写示例 |
|------|----------|------------|
| 占位符 | `[PLACEHOLDER]` 标记随处可见 | 真实的项目特定值 |
| 指令 | `<!-- INSTRUCTION: ... -->` 指导注释 | 填写完成后删除注释 |
| 组件名称 | 通用的（模块 A、组件 1） | 具体的（ALB、ECS Cluster、VPC Link） |
| 表格 | 只有列标题的空行 | 填充了真实数据 |
| 图表 | 通用的 Mermaid 模板 | 匹配实际系统的架构图 |
| 风险评估 | 模板风险条目 | 真实的项目风险与缓解方案 |
| 验收标准 | 未勾选的复选框 `[ ]` | 适用处已勾选的复选框 `[x]` |

---

## 推荐学习路径

1. 从 `filled-gap-analysis.md` 开始 — 理解问题描述
2. 阅读 `filled-requirements.md` — 看差距如何转化为可度量的需求
3. 研究 `filled-fip.md` — 学习需求如何驱动架构设计
4. 审阅 `filled-task-list.md` — 观察设计如何分解为可执行的任务

每份文档阅读时间约 10-15 分钟。完整走读建议预留 1 小时。

---

*这些示例基于真实的项目文档。项目特定的细节已保留，以展示真实的规范编写过程。*
