# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **documentation-only** collection of structured specification templates ("Spec Coding Templates") for infrastructure and platform engineering projects. There is no application code, build system, or test suite. The repository contains Markdown templates, filled examples, and bilingual documentation (English + Chinese).

## Repository Structure

- `templates/` — 8 blank templates forming the Spec Coding pipeline
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
- **07_Auto_Task_Config** — YAML config driving automated task execution
- **08_Infra_DevOps_Dependency_Rules** — Environment promotion and deployment safety rules

## Template Conventions

All templates share these patterns:

- `[PLACEHOLDER]` markers for project-specific content to replace
- `<!-- INSTRUCTION: ... -->` HTML comments guiding fill-in; removed after completion
- Severity ratings: `🔴 CRITICAL` (blocks delivery), `🟡 HIGH` (required), `🟢 MEDIUM` (deferrable)
- Status markers: `✅` done, `❌` pending, `🔄` in progress, `⏳` blocked, `🚫` cancelled
- Filled file naming: `{TYPE}_{project_name}.md` (e.g., `GAP_ALB_Fargate_java_service.md`)

## Making Changes

When editing templates or documentation:

- Maintain bilingual parity — changes to English content under `templates/` or `examples/` should be reflected in `cn/`
- Keep the severity/status marker systems consistent across all documents
- Preserve the `<!-- INSTRUCTION: -->` comment format in templates
- Use Mermaid diagram syntax for architecture visuals in the FIP template
- Template numbering (01–08) must remain stable; they are cross-referenced throughout

## License

Proprietary — personal non-commercial learning use only. No redistribution or commercial use without written permission.
