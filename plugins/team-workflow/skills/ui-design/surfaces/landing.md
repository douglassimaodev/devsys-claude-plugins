# Surface: Landing page

Job: a stranger understands what this is within 5 seconds, trusts it within 30, and takes the primary action.

## Structure (adapt; don't fill every slot)
1. **Hero** — the most characteristic thing in the product's world, in the form that suits it: a real photo, a live search box, a short demo, a bold headline. One sentence that says what it is and for whom, one primary CTA. Avoid the default "big number + small label + gradient" treatment unless it is genuinely the best option.
2. **How it works** — only if there really is a sequence (then numbering is meaningful).
3. **What you get** — benefits in the user's words, shown with real product screenshots or real content, not icon grids of features.
4. **Proof** — real reviews, partner logos, numbers that are true. Never invent testimonials or figures; leave a clearly marked placeholder for the user to fill.
5. **Second audience** (if any, e.g. "for partners") — its own short section with its own CTA.
6. **FAQ** — the real objections (price, safety, cancellation).
7. **Final CTA** and a footer with the legal links the project needs.

## Rules
- One orchestrated motion moment at most (e.g. the hero), respecting reduced motion. No fade-up on every section.
- Real imagery from the product's world; if none exists yet, use clearly marked placeholders with the intended subject and crop described.
- Sections differ in rhythm (full-bleed media, split, narrow text) — not a stack of identical centred blocks.
- Mobile first: the hero CTA is visible without scrolling on a 390×844 screen.

## Technical floor
- **SEO**: server-side rendered or prerendered HTML, unique `<title>` and meta description, Open Graph/Twitter tags, canonical URL, `sitemap.xml`, `robots.txt`, structured data where it fits (Organization, LocalBusiness, Product, FAQPage).
- **Performance**: LCP image preloaded and sized; fonts self-hosted with `font-display: swap`; no client-only rendering of above-the-fold content; aim for Lighthouse ≥ 90 on mobile for performance, accessibility, best practices and SEO.
- **Analytics/consent**: only if the project asks for it, and consent-first where the law requires it.
