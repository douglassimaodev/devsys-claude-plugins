# devsys-claude-plugins

Reusable Claude Code setup for any project.

| Plugin | What it provides |
|---|---|
| `team-workflow` | Multi-agent dispatcher (injected into every session by a `SessionStart` hook), the PM / TECH_LEAD / ARCHITECT / FULLSTACK_DEVELOPER / QA / NAVIGATOR roles, the session-resume protocol (`docs/PROGRESS.md`), Git Rules, Quality Gate and generic skills (bug-fix, code-review, architecture-review, security-review, performance-review, documentation, release-review, create-feature, create-module). |
| `dotnet-angular` | .NET (latest LTS) + EF Core + MariaDB + Angular (latest stable major) stack conventions with a version policy and per-version notes (`stack-conventions`), a pitfalls checklist (`stack-pitfalls`) and the create-api, create-entity, create-angular-page, create-unit-tests and database-review skills. |

Nothing here is specific to a project. Everything project-specific lives in **`project-description.md`** at the root of the project's repository, and its rules always take precedence over the plugins.

## Using it in a project

1. In the project, create `.claude/settings.json`:
   ```json
   {
     "extraKnownMarketplaces": {
       "devsys": { "source": { "source": "github", "repo": "YOUR-USER/devsys-claude-plugins" } }
     },
     "enabledPlugins": {
       "team-workflow@devsys": true,
       "dotnet-angular@devsys": true
     }
   }
   ```
   When the project is opened, Claude Code asks you to trust the marketplace and installs the plugins.
2. Create `project-description.md` at the root (use the FDS one as a template).
3. Create a minimal `CLAUDE.md` that imports it:
   ```markdown
   @project-description.md
   ```

Manual alternative: `/plugin marketplace add YOUR-USER/devsys-claude-plugins`, then `/plugin install team-workflow@devsys` (or use a local path such as `C:\dev\devsys-claude-plugins` while developing).

## Starting a new project

1. Write `project-description.md`.
2. Run `/team-workflow:plan-milestones`. The `project-planner` agent (Opus) reads the description and creates `docs/MILESTONES.md` (dependency-ordered milestones, junior-sized subtasks with acceptance criteria, `Mx.QG` / `Mx.P` at the end of each) and `docs/PROGRESS.md`. Run it again later to re-plan: it writes `docs/MILESTONES.proposed.md` and never touches finished work.

## Workflow profiles

Each project chooses a profile in the **Workflow profile** section of its `project-description.md`:

```
profile: junior-senior      # or: standard
reviewer: SENIOR_REVIEWER  # or: TECH_LEAD
```

| Profile | Flow |
|---|---|
| `standard` | FULLSTACK_DEVELOPER implements in the main conversation → QA |
| `junior-senior` | `junior-developer` subagent (Sonnet) implements normal subtasks; `senior-developer` subagent (Opus) implements pattern-setting ones (the first of each kind, which the junior then copies) → `senior-reviewer` subagent (Opus, read-only, fresh context each round) reviews → back to the implementer until `REVIEW APPROVED` → `qa` subagent. A 3rd round of changes or an `ESCALATE` verdict stops and asks you. |

To change which models play junior and senior, edit `model:` in `plugins/team-workflow/agents/*.md` (`sonnet`, `opus`, `haiku` or `inherit`).

## QA and the lessons loop

- QA always runs as the `qa` agent (Sonnet, read-only, fresh context): it checks each acceptance criterion and its negative case with executed evidence and runs the full test suite for regressions. It reports missing tests; it doesn't write them.
- The senior reviewer and QA report repeated mistakes as `LESSONS TO RECORD`; the main conversation appends them to the project's `docs/LESSONS.md`, which the junior reads before every subtask. Lessons about the stack itself are worth moving into `stack-pitfalls` or the version notes here, so every project benefits.

## Stack knowledge (dotnet-angular)

| Skill | Purpose |
|---|---|
| `stack-conventions` | Rules + version policy (.NET LTS only, Angular latest stable major, pinned versions win, no unrequested upgrades) + `versions/<framework>-<major>.md` notes |
| `stack-examples` | Reference code for every core pattern (entity, EF config, repository, service + validator, controller, errors, tests, ApiService, signal page, Vitest) |
| `stack-pitfalls` | The mistakes that actually happen — the reviewer's and QA's checklist |

When you move to a new LTS: add `versions/dotnet-<new>.md` and/or `versions/angular-<new>.md`; nothing else changes.

## UI: landing page + portal

- `ui-designer` agent (Opus): writes `docs/DESIGN.md` (the project's design direction — you approve it before any screen is built), builds the design system (tokens, restyled component library, `/styleguide` page, Playwright screenshot setup), and the pattern-setting screens (landing page, portal shell, first table/form/detail/dashboard). It screenshots its work at phone and desktop sizes and critiques it against the design until the checklist passes.
- `ui-design` skill (team-workflow): principles, the two surfaces, the `DESIGN.md` template, the visual review checklist.
- `angular-ui` skill (dotnet-angular): tokens in SCSS/CSS variables, restyling Syncfusion, SSR/prerender + SEO for the landing page, portal shell, screenshots.
- Junior UI subtasks copy the designer's screens and run the same visual loop; the senior reviewer and QA check the screenshots.
- Brand inputs (logo, colours, fonts, references) go in the *Brand and surfaces* section of `project-description.md`.

## Running a milestone

Say "implement M1" (or "continue"). The main conversation runs every subtask through the loop and **keeps going without waiting**, ticking each one off in `docs/MILESTONES.md`. It stops at the milestone's quality gate (`Mx.QG`), at an open business decision, an escalation, a 3rd review round, QA `BLOCKED`, or an unrelated red build. It never commits: at each stop it lists the changed files and suggests commit messages.

## Permissions

The loop runs builds, tests and git read commands many times. Pre-allow them in the project's `.claude/settings.json` (see the FDS one for a ready-made block) so you aren't asked every time; commits, pushes and database updates stay on "ask".

## Another stack?

Create another plugin under `plugins/` (e.g. `node-react`) with its own `stack-conventions` and creation skills, add it to `.claude-plugin/marketplace.json`, and enable it in the project instead of `dotnet-angular`. `team-workflow` stays the same.

## Updating

Edit here, bump `version` in `plugin.json` and `marketplace.json`, and push. In each project: `/plugin marketplace update devsys`.

## Skill names

Skills are prefixed with the plugin name: `team-workflow:qa`, `dotnet-angular:create-api`, and so on. They can also be run as slash commands, e.g. `/team-workflow:bug-fix`.
