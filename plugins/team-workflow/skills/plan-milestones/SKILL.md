---
name: plan-milestones
description: Plan the whole project from project-description.md and break it into milestones and junior-sized subtasks (creates docs/MILESTONES.md and docs/PROGRESS.md, or a re-plan proposal). Use when starting a new project, or when asked to plan the project, create milestones, build a roadmap or re-plan.
---

# Skill: plan-milestones

Entry point for project planning. The planning itself is done by the `team-workflow:project-planner` subagent, which always runs on Opus regardless of the session model.

## Steps
1. **Check the input.** `project-description.md` must exist at the repository root. If it doesn't, offer to draft it with the user first (product goal, audiences/roles, phase-1 scope, business model, open decisions, locale, workflow profile) — don't plan without it.
2. **Pick the mode.** No `docs/MILESTONES.md` → *Create*. Existing file → *Re-plan* (the planner writes `docs/MILESTONES.proposed.md` and preserves all progress).
3. **Delegate.** Start the `team-workflow:project-planner` subagent with: the mode, the current phase if the user named one, and any constraints the user gave in the conversation (deadline, team size, "MVP only", modules to skip).
4. **If it returns `MISSING INPUT`,** ask the user those questions, update `project-description.md` with the answers, and delegate again.
5. **Present the result** to the user: the milestone list with goals, the pattern-setting subtasks, open decisions and what they block, the planner's assumptions and questions. Don't paste the whole file — it is in `docs/`.
6. **Iterate.** For changes the user asks for, resume the planner with the requested changes (it edits its own output).
7. **Re-plan only:** when the user approves the proposal, replace `docs/MILESTONES.md` with `docs/MILESTONES.proposed.md`, delete the proposal, and add a session-log line to `docs/PROGRESS.md`.
8. **Hand off.** The next step is `TECH_LEAD` refining the first subtask (or straight to implementation if it's already detailed enough). Do not commit — the user reviews the new docs first.
