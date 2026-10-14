---
name: architecture-review
description: Use when reviewing architecture before or after implementation — boundaries, domain model, ownership, state workflows, security, database, API, frontend structure and over-engineering risk.
---

# Skill: architecture-review

## Goal
Ensure the system remains simple, scalable, secure and maintainable.

## Inputs
- `docs/Architecture.md` and `docs/ADR/` (authoritative)
- `project-description.md` (domain, invariants, ownership rules)
- The stack conventions skill (baseline)

## Review Areas
- Layer / module boundaries and dependency direction
- Domain model quality
- Resource ownership (data scoping per audience/role)
- Critical status workflows (e.g. orders, bookings, payments) and their state separation
- Security and authorization
- Database design
- API design
- Frontend structure
- Testability
- Maintainability
- Over-engineering risk

## Rules
- Do not introduce CQRS unless explicitly requested.
- Prefer simple application services/use cases.
- Keep business rules in the domain layer.
- Keep infrastructure details out of domain and application.
- Avoid premature microservices.
- Prefer explicit, readable code.

## Output
1. Current architecture summary
2. Issues found
3. Severity for each issue
4. Recommended fixes
