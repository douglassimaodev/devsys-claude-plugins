---
name: documentation
description: Use when creating or updating project documentation — business rules, requirements, architecture, ADRs, database, API, UI, milestones and release notes.
---

# Skill: documentation

## Documentation Rules
- Keep docs clear and actionable.
- Document business rules separately from technical implementation.
- Update docs when behavior changes.
- Prefer examples for workflows.
- Keep architecture decisions in ADR files.
- Keep release notes current.
- When a project-level fact changes (domain, rules, phasing, open decisions now decided), update `project-description.md` too.

## Common Documents
- `project-description.md` (project source of truth for the AI workflow)
- `docs/Vision.md`
- `docs/BusinessRules.md`
- `docs/FunctionalRequirements.md`
- `docs/NonFunctionalRequirements.md`
- `docs/Architecture.md`
- `docs/Database.md`
- `docs/API.md`
- `docs/UI.md`
- `docs/Modules.md`
- `docs/MILESTONES.md` (scope, phases and progress)
- `docs/PROGRESS.md` (current state and session log)
- `docs/ReleasePlan.md`
- `docs/ADR/*.md`

Only create the ones the project actually uses; `project-description.md` may list which exist.

## Output
1. Documents updated
2. Why they changed
3. Important decisions captured
4. Follow-up documentation gaps
