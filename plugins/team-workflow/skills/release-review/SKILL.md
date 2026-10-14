---
name: release-review
description: Use before releasing a milestone or version — completeness, business and financial workflows, security, ownership, migrations, builds, tests, docs and deployment readiness, ending in Approved/Not Approved.
---

# Skill: release-review

## Review Areas
- Product completeness (against the phase in `project-description.md` / `docs/MILESTONES.md`)
- Business rules
- Financial workflows
- Operational workflows
- Security
- Resource ownership
- Payment flow, if any (sandbox, webhooks, refunds)
- Data-protection compliance (privacy policy, consent, data minimisation)
- Authentication
- Authorization
- Database migrations
- API consistency
- Frontend build
- Backend build
- Tests
- Documentation
- Performance
- Accessibility
- Deployment readiness

## Required Checks
1. Backend build
2. Frontend build
3. Unit tests
4. Integration tests if available
5. Main user workflows
6. Security-sensitive workflows
7. Documentation update
8. Release notes

## Output
1. Release summary
2. Completed items
3. Blockers
4. Risks
5. Technical debt
6. Required fixes
7. Release decision: Approved / Not Approved
