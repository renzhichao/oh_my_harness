# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **documentation-only** collection of structured specification templates ("Spec Coding Templates") for infrastructure and platform engineering projects. There is no application code, build system, or test suite. The repository contains Markdown templates, filled examples, and bilingual documentation (English + Chinese).

## Repository Structure

- `templates/` — 8 blank templates forming the Spec Coding pipeline (~7,000 lines total)
- `examples/` — Filled versions of templates 01–04 for a reference project (ALB + Fargate Java service, Issue #1880)
- `cn/` — Full Chinese translation (`cn/README.md`, `cn/templates/`, `cn/examples/`)
- `README.md` — Comprehensive usage guide and methodology reference

## The Four-Document Pipeline

Templates are used sequentially; each builds on the previous:

1. **01_GAP_Analysis_Template** → Baseline current state, identify gaps
2. **02_Requirements_Template** → Convert gaps into measurable requirements (FR-X.X.X IDs)
3. **03_FIP_Template** → Architecture design with Mermaid diagrams, risk assessment
4. **04_Task_List_Template** → Phased task breakdown with dependencies and effort estimates

Four supporting templates apply cross-cutting:
- **05_Naming_Rules** — Resource naming standards (referenced by all templates)
- **06_Failure_Patterns** — Known failure modes (referenced in FIP Section 5)
- **07_Auto_Task_Config** — YAML config driving automated task execution via Sub-Agent architecture (v2)
- **08_Infra_DevOps_Dependency_Rules** — Environment promotion and deployment safety rules

## Template Conventions

All templates share these patterns:

- `[PLACEHOLDER]` markers for project-specific content to replace
- `<!-- INSTRUCTION: ... -->` HTML comments guiding fill-in; removed after completion
- Severity ratings: `🔴 CRITICAL` (blocks delivery), `🟡 HIGH` (required), `🟢 MEDIUM` (deferrable)
- Status markers: `✅` done, `❌` pending, `🔄` in progress, `⏳` blocked, `🚫` cancelled
- Document lifecycle: `Draft → Under Review → Approved → Implementation In Progress → Complete`
- Filled file naming: `{TYPE}_{project_name}.md` (e.g., `GAP_ALB_Fargate_java_service.md`)

## Task Granularity Standards (Template 04)

Task List enforces decomposition thresholds — any task exceeding these must be split into subtasks:
- **LOC**: >500 lines → decompose
- **Effort**: >4 person-hours → decompose
- **Complexity**: HIGH → decompose unless justified

Each task must include: Status, Priority, Estimated Time, Estimated LOC, Complexity, Description, Acceptance Criteria, Related Files, Commit Message, and Dependencies.

## Making Changes

When editing templates or documentation:

- Maintain bilingual parity — changes to English content under `templates/` or `examples/` must be reflected in `cn/` (same directory structure, same filenames). After editing English files, verify `cn/` counterparts exist and update them.
- Keep the severity/status marker systems consistent across all documents
- Preserve the `<!-- INSTRUCTION: -->` comment format in templates
- Use Mermaid diagram syntax for architecture visuals in the FIP template
- Template numbering (01–08) must remain stable; they are cross-referenced throughout
- Template 07 (Auto Task Config) uses a v2 Sub-Agent architecture — tasks run in isolated Sub-Agent contexts with parallel execution support

## Cross-Template References

Templates reference each other and must stay consistent:
- GAP Analysis gaps → Requirements (gap IDs become FR- IDs)
- Requirements → FIP (FR- IDs drive architecture decisions)
- FIP → Task List (implementation plan → phased tasks)
- Task List → Auto Task Config (tasks drive automation YAML)
- Naming Rules (05) applies to ALL resource references across every template
- Failure Patterns (06) is referenced in FIP Section 5 (Risk Assessment)
- Infra Dependencies (08) constrains FIP Section 1 (Architecture Design)

## Scope Boundaries

- **Automation capability**: Template 07 specifies configuration schemas and prompt templates for automated execution — it does NOT provide a runnable execution engine. Teams must supply their own scheduler, Sub-Agent runtime, state storage, and Git integration.
- **Domain focus**: Templates are designed for infrastructure and platform engineering (AWS, Terraform, ALB/Fargate, etc.). They do NOT cover AI/ML-specific concerns such as dataset lineage, model versioning, prompt versioning, eval benchmarks, hallucination guardrails, or model rollback. AI projects may use this framework for the engineering management layer, but will need supplementary templates for model/data concerns.
- **Domain adaptation**: Naming Rules (Template 05) and examples use AWS/IaC patterns. Teams using other cloud providers or non-IaC stacks should treat these as reference patterns and replace domain-specific conventions with their own.

## License

Proprietary — personal non-commercial learning use only. Copyright (c) 2026 Ren Zhichao. No redistribution or commercial use without written permission.
