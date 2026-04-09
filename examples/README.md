# Spec Coding Examples - Filled Template Samples

> Example filled specifications for the "ALB + Fargate Infrastructure for Java Container Deployment" project.

---

## About These Examples

These documents are **filled (completed) versions** of the spec-coding-templates for a real infrastructure project. They demonstrate how blank template placeholders translate into production-ready specifications.

### Project Context

| Field | Value |
|-------|-------|
| **Project** | ALB + Fargate Infrastructure for Java Card Service |
| **Issue** | #1880 |
| **Team** | Platform Engineering |
| **Cloud Provider** | AWS |
| **Region** | ap-southeast-1 |
| **Environments** | dev, test, staging, prod |
| **Key Services** | ECS Fargate, ALB, API Gateway, VPC Link, ECR, CloudWatch, IAM |

### What Was Built

A Java Spring Boot microservice deployed on AWS ECS Fargate behind an Application Load Balancer, with API Gateway private integration via VPC Link for external API exposure.

---

## Example Files

| File | Based On Template | Description |
|------|-------------------|-------------|
| `filled-gap-analysis.md` | 01_GAP_Analysis_Template.md | GAP Analysis for ECS Fargate infrastructure - identifies gaps in container orchestration, deployment automation, and service routing |
| `filled-requirements.md` | 02_Requirements_Template.md | Full requirements for ALB, ECS Fargate, API Gateway, container deployment, and multi-environment infrastructure |
| `filled-fip.md` | 03_FIP_Template.md | FIP with Mermaid architecture diagrams, component design, security design, and phased implementation plan |
| `filled-task-list.md` | 04_Task_List_Template.md | 4-phase task breakdown with dependencies, effort estimates, and acceptance criteria |

---

## How to Use These Examples

### Step 1: Read the Blank Template First
Open the template file (e.g., `templates/01_GAP_Analysis_Template.md`) and read through the structure and `<!-- INSTRUCTION: ... -->` comments.

### Step 2: Compare with the Filled Version
Open the corresponding example file (e.g., `examples/filled-gap-analysis.md`) and compare:
- How `[PLACEHOLDER]` markers were replaced with real content
- How instruction comments guided the content creation
- How generic component names became specific service names

### Step 3: Notice the Pipeline Flow
Observe how documents build on each other:
1. **GAP Analysis** gaps become **Requirements** (e.g., "No container platform" → FR-1.2.1 "ECS Cluster")
2. **Requirements** drive **FIP** architecture decisions (e.g., FR-1.1.1 → ALB design in Section 2.1)
3. **FIP** implementation plan becomes **Task List** phases (e.g., FIP Phase 1 → Task List Phase 1)

---

## Key Differences: Template vs. Filled Example

| Aspect | Blank Template | Filled Example |
|--------|---------------|----------------|
| Placeholders | `[PLACEHOLDER]` markers everywhere | Real project-specific values |
| Instructions | `<!-- INSTRUCTION: ... -->` guidance comments | Comments removed after filling |
| Component Names | Generic (Module A, Component 1) | Specific (ALB, ECS Cluster, VPC Link) |
| Tables | Empty rows with column headers | Populated with real data |
| Diagrams | Generic Mermaid templates | Architecture matching the actual system |
| Risk Assessment | Template risk entries | Real project risks with mitigation plans |
| Acceptance Criteria | Unchecked boxes `[ ]` | Checked boxes `[x]` where applicable |

---

## Recommended Learning Path

1. Start with `filled-gap-analysis.md` - understand the problem statement
2. Read `filled-requirements.md` - see how gaps become measurable requirements
3. Study `filled-fip.md` - learn how requirements drive architecture design
4. Review `filled-task-list.md` - observe how design breaks into actionable tasks

Each document takes 10-15 minutes to read. Allow 1 hour for the complete walkthrough.

---

*These examples are based on actual project documentation. Project-specific details have been preserved to demonstrate realistic spec writing.*
