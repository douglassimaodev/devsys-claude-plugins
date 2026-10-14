# Visual review loop

A UI change is not done until someone has **looked at it**. The stack skill (e.g. `dotnet-angular:angular-ui`) explains how to take screenshots; this file says what to do with them.

## Loop (implementer)
1. Take screenshots of every changed route at **phone (390×844)** and **desktop (1440×900)**, in each relevant state (filled, loading, empty, error; logged-in role if it matters).
2. Open each screenshot (Read the image file) and write a short critique against `docs/DESIGN.md` and the checklist below — concrete ("primary button competes with the filter chips: same weight and colour"), never generic ("looks good").
3. Fix, re-shoot, re-critique. Stop when the checklist passes, or after 3 rounds — then list what's left in the handoff.
4. Put the final screenshot paths in the handoff note.

## Checklist
**Direction**
- [ ] Recognisably follows `DESIGN.md` (palette roles, type, radius by hierarchy, imagery, voice); no colour, size, radius or shadow outside the tokens.
- [ ] Doesn't fall into the default kit (see `SKILL.md`) unless `DESIGN.md` asks for it.

**Hierarchy and layout**
- [ ] Squint test: the most important thing and the one primary action are obvious.
- [ ] Alignment is consistent; spacing follows the scale; related things are closer than unrelated things.
- [ ] Text line length is comfortable; nothing is cramped or floating.

**Content and states**
- [ ] Real or realistic content, no lorem ipsum; copy follows the voice rules.
- [ ] Loading, empty and error states exist and are designed.
- [ ] Long names, big numbers and missing images don't break the layout.

**Responsive and accessible**
- [ ] Phone: no horizontal scroll, touch targets ≥ 44 px, primary action reachable.
- [ ] Contrast AA; focus visible; status never conveyed by colour alone; images have alt text.
- [ ] Reduced motion respected.

## For the reviewer and QA
- **Senior reviewer**: on UI changes, look at the screenshots in the handoff (re-shoot if missing or stale) and treat checklist failures as MUST FIX — BLOCKER for accessibility failures or a broken phone layout.
- **QA**: verify the acceptance criteria visually where they describe UI (states, responsive behaviour), using fresh screenshots as evidence.
