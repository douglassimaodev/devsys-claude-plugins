# Design — <Product name>

Status: proposed | approved (ADR-xxxx) · Owner: UI_DESIGNER · Last updated: <date>

## 1. Personality
- Feels like: <3–5 words>
- Must not feel like: <e.g. a generic admin template, a bank, a children's toy>
- Audience and context of use: <who, on what device, in what situation>
- References (and what to take from each): <links / names>

## 2. Palette

| Token | Hex | Role | Contrast check |
|---|---|---|---|
| `color-bg` | | page background | |
| `color-surface` | | cards, panels | |
| `color-text` | | body text | AA on bg/surface |
| `color-primary` | | the one primary action | AA with its text |
| `color-accent` | | the brand's memorable colour, used sparingly | |
| `color-success` / `-warning` / `-danger` / `-info` | | states | |

Dark mode: yes/no — if yes, the same table for dark.

## 3. Typography
- Display: <family> — where and why
- Text: <family> — where and why
- Scale (rem): <e.g. 0.75 / 0.875 / 1 / 1.125 / 1.375 / 1.75 / 2.25 / 3>
- Weights in use: <…> · Line height: <…> · Max line length: <…>
- Loading: self-hosted, `font-display: swap`, subsets for the project's languages

## 4. Shape, depth, spacing
- Radius scale and where each applies: <e.g. 4 inputs / 8 cards / 16 hero media / pill for tags>
- Elevation levels (max 3) and what earns each
- Spacing scale: <4-pt based: 4 8 12 16 24 32 48 64 96>
- Grid: <columns / gutters / max content width> for phone, tablet, desktop

## 5. Motion
- Landing: <the one orchestrated moment>
- Portal: <feedback motion only: expand, confirm, route change>
- Durations/easing tokens · reduced-motion behaviour

## 6. Imagery and icons
- Photography: <subject, light, framing, people or not>
- Illustration: <none / style>
- Icons: <one set, size grid, stroke>

## 7. Voice
- Buttons: <verb + object>
- Empty states: <pattern + example>
- Errors: <pattern + example>
- Dates, money, numbers: <formats per locale>

## 8. Component recipes
For each signature component: purpose, anatomy (ASCII sketch), states, do/don't.
- <e.g. Listing card>
- <e.g. Search bar>
- <e.g. Data table>
- <e.g. KPI tile>
- <e.g. Form field + validation>
- <e.g. Empty state>

## 9. Landing page
Section plan (ASCII wireframe per breakpoint), the hero concept, the primary CTA, proof elements.

## 10. Portal
Starts from the house style in `surfaces/portal.md`. Record only the project's choices: sidebar default mode and colour (light/dark), icon set, KPI tiles for this domain, chart types, badge colours per status, dark-mode palette, and any deviation from the house style (with the reason).

## 11. Decisions that were changed because they were defaults
<what, why>
