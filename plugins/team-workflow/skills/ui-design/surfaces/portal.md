# Surface: Admin portal

Job: a returning user finds what they need, acts on it and is sure it worked — quickly, without errors. Polished means *precise and calm*, not decorated.

## House style (default for every portal)

Unless `project-description.md` says otherwise, portals follow the user's preferred **premium admin-dashboard style** — the qualities of templates like UBold (Coderthemes), designed in our own way with our own components. Take the *qualities*, never copy a commercial template's code, assets or exact visuals.

- **Layout**: fixed left sidebar + slim top bar + light-grey page background with white content cards; fluid width by default.
- **Sidebar**: logo at top; menu grouped under small section titles; collapsible groups with icons; active item clearly highlighted. Size modes: *default* (icon + label), *compact* (icon + small label), *condensed* (icons only, labels on hover), and *off-canvas* on phone. Light or dark sidebar is a theme setting.
- **Top bar**: menu toggle, page search, (language switcher if multilingual), light/dark toggle, notifications with a count badge, user menu with name and role.
- **Page header**: breadcrumb + page title on the left, the page's primary action on the right.
- **Dashboard**: a row of 4 KPI cards (icon in a soft-tinted square, value, label, trend vs previous period) → one or two chart panels (smooth area/line with soft fill, donut for shares) → "recent" tables side by side on desktop, stacked on phone.
- **Cards**: white surface, 1 px subtle border *or* very soft shadow (pick one), modest radius, a header row with title and a quiet action link, consistent inner padding.
- **Tables**: comfortable rows, thumbnail/avatar + primary text with muted secondary text beneath, right-aligned amounts with tabular figures, **soft-tinted status badges** (tinted background + strong text of the same hue), row hover, actions at row end.
- **Colour use**: neutral greys for structure; the brand primary for actions and active states; a small set of tinted state colours (success, warning, danger, info) used consistently across badges, KPI icons and charts.
- **Light and dark mode** from day one (follow the system by default, user can switch).
- **Density**: medium-high — 14 px base text in the portal, tighter vertical rhythm than the landing page.

This style *is* card-based; that's the chosen direction for portals, not the generic default the SKILL warns about. What makes each project's portal its own: the brand palette and typography, the icon set, the KPI and chart choices that fit the domain, and well-designed empty and error states.

## Shell
- **Navigation**: left sidebar on desktop (collapsible to icons), bottom bar or drawer on phone. Group items by the user's tasks, not by database tables. Highlight the current section clearly.
- **Top bar**: global search (if useful), context switcher (organisation/branch), notifications, user menu.
- **Page header**: title in the user's words, one primary action on the right, secondary actions in a menu, breadcrumbs only when hierarchy is deep.
- Brand appears through the type, the primary/accent colours, the logo and small details (focus ring, selection colour, illustrations in empty states) — not through gradients or big imagery.

## Data views
- **Tables**: comfortable default density with an option for compact; sticky header; right-aligned numbers with tabular figures; status as small coloured badges with text (never colour alone); row actions on hover *and* reachable by keyboard; bulk actions appear only when rows are selected.
- **Filters**: the 2–4 most used visible, the rest in a "More filters" panel; active filters shown as removable chips; results count visible.
- **Empty states**: say why it's empty and offer the next action ("No bookings yet — share your listing to get the first one").
- **Loading**: skeleton rows shaped like the data, not a centred spinner.

## Forms
- One column for most forms; group related fields with short headings.
- Labels above fields, helper text below, validation on blur and on submit, error text says how to fix it.
- Primary button names the result ("Save listing"); destructive actions are separated and confirmed with the object's name.
- Long forms: sections with a sticky save bar showing unsaved changes.

## Dashboards
- Start from the decisions the user makes on this page; show only the numbers that drive them.
- KPI tiles: the number, what it measures, and the comparison that gives it meaning (vs last period, vs target).
- Charts use the palette's data colours, labelled directly where possible, with readable axes; one chart type per question.

## Feedback
- Every action gets visible feedback in place (button busy state, row updated) and a toast using the same verb ("Listing published").
- Optimistic updates only where reversal is safe.
