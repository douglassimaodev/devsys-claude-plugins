---
name: create-module
description: Use when creating a new business module — purpose and phase, workflows per role, ownership rules, domain model, tables, endpoints, screens, tests and docs.
---

# Skill: create-module

The project's module list (if any) is in `project-description.md`.

## Module Creation Steps
1. Define module purpose and which phase it belongs to (see `project-description.md` / `docs/MILESTONES.md`).
2. Define business workflows for each role involved.
3. Define permissions and ownership rules (who can see/change what).
4. Define domain model, including status enums and allowed transitions.
5. Define database tables.
6. Define API endpoints.
7. Define frontend screens per role/area.
8. Define tests.
9. Update docs.

## Structure
Follow the project layout in `project-description.md` and the stack conventions skill (e.g. `dotnet-angular:stack-conventions`) for where each piece goes:
- Domain entities, enums, DTOs and request models
- Application service + validators
- Persistence configuration, migration and repository
- API controllers
- Frontend feature route (lazy-loaded, guarded by role), list/detail pages, create/edit form, API service, models, shared components if needed
- Tests

Do not create CQRS command/query folders unless explicitly requested.

## Documentation
Update `docs/Modules.md`, `docs/API.md`, `docs/Database.md`, `docs/BusinessRules.md`, `docs/MILESTONES.md` (those that exist).

## Output
- Module summary
- Data model
- API model
- UI model (per role)
- Permissions and ownership rules
- Financial impact (if any)
- Implementation plan
- Validation checklist
