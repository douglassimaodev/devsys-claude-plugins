---
name: create-feature
description: Use to implement a complete feature from requirement to code — business, financial and operational analysis, backend and frontend design, tests, docs and validation.
---

# Skill: create-feature

## Workflow
1. Read `project-description.md` and `CLAUDE.md`.
2. Read relevant docs (`docs/Architecture.md`, ADRs, `docs/MILESTONES.md`).
3. Identify the module and business goal.
4. Define acceptance criteria.
5. Check financial and operational impact.
6. Design the backend following the stack conventions (application services, not CQRS by default).
7. Design the frontend following the stack conventions.
8. Implement incrementally, checkpointing `docs/PROGRESS.md`.
9. Add tests.
10. Update docs.
11. Run builds and tests.

## Backend Expectations
- Domain entity/value object when business logic exists.
- Application service/use case.
- Request validator.
- API endpoint/controller.
- Persistence mapping/migration when needed.
- Unit tests (see `create-unit-tests`).

## Frontend Expectations
- Component following the stack's component and state patterns.
- Typed API service.
- Loading, error and empty states.
- Responsive layout.
- Accessibility labels.

## Output
1. Product Analysis
2. Financial Analysis
3. Operations Analysis
4. Solution Design
5. Backend Changes
6. Frontend Changes
7. Tests Added
8. Docs Updated
9. How to Validate
