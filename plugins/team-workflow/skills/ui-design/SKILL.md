---
name: ui-design
description: Design principles and process for distinctive, polished UI — a project's design direction (docs/DESIGN.md), the landing page and the admin portal, and the screenshot critique loop. Load before designing or building any user-facing screen, design system, theme or style guide, and when reviewing UI.
---

# Skill: ui-design

Most generated UIs look like a generic CRM because nothing told them what *this* product should feel like and nobody looked at the result. This skill fixes both: a written design direction per project, and a mandatory look-and-critique loop.

## The two surfaces

Projects usually have two surfaces with different jobs. Design them as one brand with two densities — never as two unrelated styles.

| | Landing page | Admin portal |
|---|---|---|
| Job | Make a stranger understand, trust and act (sign up, book, contact) | Let a returning user do their work fast and without mistakes |
| Tone | Expressive: one memorable moment, real imagery, generous space | Premium admin-dashboard house style (see `surfaces/portal.md`): calm, card-based, light and dark |
| Density | Low — one idea per section | Medium to high — tables, filters, forms, but with clear rhythm |
| Details | `surfaces/landing.md` | `surfaces/portal.md` |

Other surfaces (a customer app, a partner panel) sit between the two: portal structure, closer-to-landing warmth.

## Step 1 — Design direction (once per project): `docs/DESIGN.md`

Before any screen is built, the project needs `docs/DESIGN.md`, written from the brand section of `project-description.md` (personality, audience, references, existing logo/colours/fonts). Use the template in `design-template.md`. It must decide:

- **Personality** in 3–5 words, and what the product must *not* feel like.
- **Palette**: 4–6 named colours with hex values and their roles (background, surface, text, primary action, accent, states), all checked for WCAG AA contrast.
- **Typography**: one or two deliberately chosen families (not the framework default), with roles and a type scale.
- **Shape and depth**: radius scale tied to hierarchy (not one radius for everything), elevation levels, borders.
- **Spacing and layout**: spacing scale, grid, max line length (under ~75 characters for body text).
- **Motion**: what moves and why (one orchestrated moment on the landing page; feedback motion in the portal).
- **Imagery and icons**: photo style, illustration (if any), one icon set.
- **Voice**: how buttons, empty states and errors are written.
- **Component recipes** for the product's signature pieces (e.g. listing card, search bar, KPI tile, data table, form, empty state).

Then challenge it: if a choice would be the same for any similar product, it is a default, not a decision — change it and note why in the doc. Record the direction as an ADR when the user approves it.

## Step 2 — Design system in code (pattern-setting)

Turn `DESIGN.md` into tokens and themed components (stack specifics: e.g. `dotnet-angular:angular-ui`), plus a **style guide page** that renders every token and component in every state. All later screens use only the tokens and components — no one-off colours, sizes or shadows.

## Step 3 — Build screens, then look at them

Every UI subtask ends with the **visual review loop** in `visual-review.md`: screenshots at phone and desktop sizes, a written critique against `DESIGN.md` and the checklist, fixes, and new screenshots — until the checklist passes (max 3 rounds, then report what's left).

## Rules that keep it from looking generated

- **Ground it in the subject.** The product's world (its materials, places, vocabulary, real photos) is where distinctive choices come from. Use real or realistic content — never lorem ipsum, never "John Doe / Acme".
- **Spend boldness in one place** per screen; keep the rest quiet. Remove one decoration before calling it done.
- **Hierarchy before decoration.** One primary action per view, clearly the strongest element. Secondary actions look secondary.
- **Avoid the default kit** unless the brief asks for it (the portal's card-based house style in `surfaces/portal.md` is a chosen direction, not this default): identical rounded cards with the same soft shadow everywhere; gradient washes as decoration; all-caps tracked labels above every heading; a single highlighted word in a headline; numbered "01/02/03" markers on content that isn't a sequence; fade-up animations on every section; `→` appended to every link.
- **Structure carries meaning.** Borders, dividers, labels and numbering only where they encode something.
- **States are designed, not afterthoughts.** Every data view has loading (skeletons shaped like the content), empty (explains and invites the next action), error (says what happened and how to fix it) and success states.
- **Copy is design.** Plain words from the user's point of view, sentence case, active verbs. A button says what happens ("Book this venue", not "Submit"), and the confirmation uses the same verb ("Booked").
- **Quality floor, always:** responsive down to 360 px, visible keyboard focus, `prefers-reduced-motion` respected, AA contrast, touch targets ≥ 44 px, images sized and lazy-loaded, no layout shift.

## Files in this skill

| File | Use |
|---|---|
| `design-template.md` | Template for `docs/DESIGN.md` |
| `surfaces/landing.md` | Landing page structure, hero, SEO, conversion |
| `surfaces/portal.md` | Admin portal shell, tables, forms, dashboards |
| `visual-review.md` | Screenshot loop and the visual checklist (also used by the reviewer and QA) |
