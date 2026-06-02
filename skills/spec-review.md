---
name: spec:review
description: Conduct cross-context review of specifications
parameters:
  document_type:
    type: string
    required: true
    description: Type of document to review (REQ or FIP)
  document_path:
    type: string
    required: true
    description: Path to the specification document
---

# Cross-Context Specification Review

You are conducting a **cross-context review** for the **{{document_type}}** document at `{{document_path}}`.

## Review Framework

### Context 1: Technical Review
Validate:
- Architecture is sound and feasible
- Technology choices are justified
- Implementation approach is viable
- Performance requirements are realistic

### Context 2: Security Review
Validate:
- Security is designed in, not bolted on
- Vulnerabilities are addressed
- Encryption and auth specified
- Compliance requirements met

### Context 3: Operational Review
Validate:
- Deployability is considered
- Monitoring and observability defined
- Maintenance procedures specified
- Rollback capabilities exist

### Context 4: Business Review
Validate:
- Aligns with business objectives
- User needs are reflected
- Cost and timeline are acceptable
- Risk is manageable

## Review Process

1. **Read the specification document**
2. **Apply review checklist for each context**
3. **Document findings** with severity (🔴🟡🟢)
4. **Identify action items** with owners
5. **Make recommendation**: APPROVED | APPROVED_WITH_CHANGES | NEEDS_REVISION | REJECTED

## Output Format

```markdown
# Review Report: {{document_type}} for {{document_path}}

## Review Summary
- Overall Status: [APPROVED | APPROVED_WITH_CHANGES | NEEDS_REVISION | REJECTED]
- Review Date: [date]
- Reviewers: [list]

## Findings by Context

### Technical Review
- [ ] Finding 1 — [severity] — [action required]

### Security Review
- [ ] Finding 1 — [severity] — [action required]

### Operational Review
- [ ] Finding 1 — [severity] — [action required]

### Business Review
- [ ] Finding 1 — [severity] — [action required]

## Action Items
- [ ] Item 1 — [owner] — [due date]

## Recommendation
[Brief justification for recommendation]
```

Begin by reading the document and conducting the review.
