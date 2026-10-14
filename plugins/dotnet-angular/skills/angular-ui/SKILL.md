---
name: angular-ui
description: How to build distinctive UI in this Angular + Syncfusion stack — design tokens in SCSS/CSS variables, restyling Syncfusion so it doesn't look like a stock admin template, the style guide page, the landing page with SSR/prerendering and SEO, the portal shell, images, fonts, motion, and taking Playwright screenshots for the visual review loop. Load with team-workflow:ui-design for any UI work.
---

# Skill: angular-ui

Design decisions come from `docs/DESIGN.md` (see `team-workflow:ui-design`). This skill is how to implement them in Angular with Syncfusion. Check the version notes for the pinned Angular major first.

## 1. Tokens — `src/sass/`

```text
src/sass/
├── _tokens.scss          ← the only place raw values live (from DESIGN.md)
├── _theme.scss           ← emits tokens as CSS custom properties (:root, and dark if used)
├── _syncfusion.scss      ← Syncfusion base theme import + overrides mapped to tokens
├── _typography.scss      ← @font-face (self-hosted), type scale classes
├── _mixins.scss          ← breakpoints, focus ring, visually-hidden
└── styles.scss           ← imports in this order: tokens, theme, typography, syncfusion, globals
```

```scss
// _tokens.scss — names follow DESIGN.md
$color: (
  bg: #…, surface: #…, text: #…, text-muted: #…,
  primary: #…, primary-contrast: #…, accent: #…,
  success: #…, warning: #…, danger: #…, info: #…,
  border: #…
);
$radius: (sm: 4px, md: 8px, lg: 16px, pill: 999px);
$space: (1: 4px, 2: 8px, 3: 12px, 4: 16px, 5: 24px, 6: 32px, 7: 48px, 8: 64px, 9: 96px);
$shadow: (1: …, 2: …, 3: …);
$font: (display: ('…', serif), text: ('…', sans-serif));
$breakpoint: (sm: 600px, md: 960px, lg: 1280px);
```

```scss
// _theme.scss
@use 'sass:map';
@use 'tokens' as t;

:root {
  @each $name, $value in t.$color { --color-#{$name}: #{$value}; }
  @each $name, $value in t.$radius { --radius-#{$name}: #{$value}; }
  @each $name, $value in t.$space  { --space-#{$name}: #{$value}; }
  @each $name, $value in t.$shadow { --shadow-#{$name}: #{$value}; }
  --font-display: #{map.get(t.$font, display)};
  --font-text: #{map.get(t.$font, text)};
}
```

Components use only `var(--…)`. A raw hex, px size, radius or shadow in a component stylesheet is a review finding.

## 2. Make Syncfusion look like *this* product

Syncfusion's stock theme is what makes screens look like a generic CRM. Restyle it once, centrally:

1. Pick the base theme closest to `DESIGN.md` (e.g. Tailwind 3 or Fluent 2 for clean/neutral, Material 3 for soft/rounded) and import only the component styles you use, in `_syncfusion.scss`.
2. Map the theme's variables (the newer themes expose CSS custom properties — confirm the names in the installed version) to your tokens: primary, surface, text, border, radius, font family, focus ring.
3. Override what variables can't reach with **low-specificity** rules in `_syncfusion.scss` only (`.e-grid .e-headercell`, `.e-btn.e-primary`…). Never `::ng-deep` or `!important` scattered through components.
4. Typical overrides that change the feel the most: grid header (weight, case, background), row height and hover, button radius/weight, input height/border/focus ring, chips/badges, dialog radius and shadow, toast style, date picker header.
5. Verify every overridden component on the style guide page in all states.

## 3. Style guide page — `/styleguide`

A route (lazy, disabled in production builds via an environment flag) that renders: colour tokens with contrast ratios, the type scale, spacing/radius/shadow samples, and every component in default/hover/focus/disabled/error/loading states — buttons, inputs, selects, date picker, grid with real-looking data, chips, dialog, toast, empty state, skeletons, KPI tile, cards. It is the first screen the UI designer builds and the reference everyone checks against.

## 4. Landing page

- **Rendering**: use Angular SSR (`@angular/ssr`) and prerender the landing routes (server route config with prerender render mode), or keep the landing page as its own small Angular project if the portal must stay a pure SPA. Above-the-fold content must be in the HTML.
- **SEO**: per-route `Title` and `Meta` services (title, description, Open Graph, canonical), `sitemap.xml` and `robots.txt` in `public/`, JSON-LD in the page where it fits.
- **Images**: `NgOptimizedImage` (`ngSrc`, explicit width/height, `priority` on the LCP image, `sizes` for responsive), modern formats.
- **Fonts**: self-hosted `woff2` in `public/fonts`, `font-display: swap`, preload the display font used in the hero, subset for the project's languages.
- **Motion**: CSS transitions/animations; one orchestrated moment; wrap in `@media (prefers-reduced-motion: no-preference)`. Don't add the legacy `@angular/animations` package for new work — check the version notes for the current animation API.

## 5. Portal

Implements the portal house style from `team-workflow:ui-design` → `surfaces/portal.md`.

- **Layout state**: a `LayoutService` (`providedIn: 'root'`) with signals for `sidebarMode` (`default | compact | condensed | offcanvas`), `sidebarTheme` (`light | dark`) and `colorScheme` (`light | dark | system`), persisted to `localStorage` (wrapped in try/catch) and applied as attributes on `<html>` (`data-theme`, `data-sidebar`, `data-sidebar-theme`). SCSS switches token values by those attributes — components never check the theme themselves.
- **Dark mode**: a second token map emitted under `[data-theme='dark']`; `system` follows `prefers-color-scheme`. The Syncfusion overrides read the same CSS variables, so grids, inputs and dialogs switch too.
- **Sidebar**: Syncfusion Sidebar (or a plain component) + a menu built from a typed route/menu config (sections → items → children, icon, required role), filtered by the user's role.
- **KPI card, status badge, card header, page header, empty state**: small shared components in `shared/ui/` used everywhere — the badge maps each status to a token pair (tinted background, strong text).
- **Icons**: one icon set chosen in `DESIGN.md` (e.g. Lucide or Tabler); wrap it in one `<app-icon>` component so it can be swapped.
- **Charts**: Syncfusion Charts — spline area with low-opacity fill, light gridlines, no 3D, palette from tokens, tooltips formatted with the project's locale.


- **Shell component** (`core/layout/`): sidebar (modes above, grouped by task), top bar, page header slot with breadcrumb + one primary action, content area with max width for forms and full width for tables. Phone: off-canvas sidebar.
- **Grids**: Syncfusion Grid restyled via `_syncfusion.scss`; server-side paging/sorting/filtering against the API's `PagedResult`; status as badge component (text + colour); row actions keyboard-reachable; skeleton rows while loading; designed empty state.
- **Forms**: Reactive Forms + Syncfusion inputs; one shared `form-field` wrapper component for label, hint and error text so every form looks the same.
- **Charts**: Syncfusion Charts with the token palette; direct labels; no 3D, no gratuitous gradients.
- **Feedback**: `ToastService` with the same verb as the action; button busy state while saving.

## 6. Screenshots for the visual review loop

Playwright is already in the stack. Create once (pattern-setting), then extend the route list per subtask:

```ts
// e2e/ui-shots.spec.ts
import { test, type Page } from '@playwright/test';

type Shot = { name: string; path: string; setup?: (page: Page) => Promise<unknown> };

const shots: Shot[] = [
  { name: 'landing', path: '/' },
  { name: 'styleguide', path: '/styleguide' },
  // Empty state example: mock the API response for this screenshot only
  {
    name: 'portal-products-empty',
    path: '/admin/products',
    setup: (page) =>
      page.route('**/api/v1/Products/GetMyProducts*', (route) =>
        route.fulfill({ json: { items: [], page: 1, pageSize: 20, totalCount: 0 } })),
  },
];

const viewports = {
  phone: { width: 390, height: 844 },
  desktop: { width: 1440, height: 900 },
};

for (const shot of shots) {
  for (const [viewport, size] of Object.entries(viewports)) {
    test(`${shot.name} @ ${viewport}`, async ({ page }) => {
      await page.setViewportSize(size);
      await shot.setup?.(page);
      await page.goto(shot.path);
      await page.waitForLoadState('networkidle');
      await page.screenshot({ path: `.ui-shots/${shot.name}-${viewport}.png`, fullPage: true });
    });
  }
}
```

- `playwright.config.ts`: `webServer` starts the dev server (`npm start`) and `use.baseURL` points at it; logged-in portal shots use a saved `storageState` for a seeded test user.
- Run: `npx playwright test e2e/ui-shots.spec.ts` → images in `.ui-shots/` (add it to `.gitignore`). Browsers must be installed once with `npx playwright install chromium`.
- Then open the images (Read the PNG files) and follow `team-workflow:ui-design` → `visual-review.md`.
