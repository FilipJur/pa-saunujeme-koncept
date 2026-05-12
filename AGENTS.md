# Saunujeme — CMS Section Development

## Project Context

This project produces **self-contained HTML sections** for a CMS manager to paste directly into a content management system. Each section is a standalone file with no JavaScript dependencies — only HTML classes + vanilla CSS.

The existing `index.html` contains a hero section built with Vite. New sections are decoupled from the build system and live as static HTML files.

## File Structure

```
saunujeme-koncept/
├── public/
│   ├── shared.css              # Shared base styles (dev only)
│   ├── pa-typizovane.html      # Example section
│   └── pa-... .html            # Other sections
├── index.html                  # Existing hero (not relevant for new sections)
└── AGENTS.md                   # This file
```

**New sections go in `public/`.** Files in `public/` are served by Vite during development but are **not processed by the build pipeline**.

## Naming Convention (Non-Negotiable)

**Every CSS class MUST use the `pa-` prefix.** This ensures zero collision with the CMS's existing stylesheets or third-party CSS.

### BEM Pattern

```
.pa-{section-name}__{element}--{modifier}
```

Examples:
- `.pa-features` — section block
- `.pa-features__grid` — element inside the section
- `.pa-features__card--highlighted` — modifier variant
- `.pa-features__title` — element
- `.pa-btn--primary` — reusable component with modifier

### Rules
- **Always prefix with `pa-`** — no exceptions.
- **Never use bare tag selectors** (e.g., `h2 { ... }`) — always class-based.
- **Never use IDs for styling** — IDs are reserved for anchor links only.
- **Scope section-specific styles under the section class** to minimize specificity wars.

## Development Workflow

### 1. Create a New Section

Create a new file in `public/`:

```html
<!-- public/pa-{section-name}.html -->
<!doctype html>
<html lang="cs">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{Section Title} — Dev Preview</title>
    <link rel="stylesheet" href="shared.css" />
    <style>
      /* Section-specific styles */
      .pa-{section-name} {
        /* ... */
      }
    </style>
  </head>
  <body>
    <section class="pa-{section-name}">
      <!-- Content -->
    </section>
  </body>
</html>
```

### 2. Shared CSS (`public/shared.css`)

During development, each section links to `shared.css` for:
- Google Fonts import (Jost)
- CSS custom properties (design tokens)
- Minimal reset (box-sizing, margin/padding on `body` only)
- Base typography rules

**Do not put section-specific styles in `shared.css`.**

### 3. CMS Handoff Process

When a section is complete, the agent must produce a **single self-contained file** for the CMS manager:

1. **Inline the shared styles**: Copy the contents of `public/shared.css` into a `<style>` tag.
2. **Append section-specific styles**: Copy the section's `<style>` block contents after the shared styles.
3. **Remove the `<link>` to `shared.css`** — it is no longer needed.
4. **Output**: One HTML file with one `<style>` tag containing everything.

Example handoff structure:

```html
<!doctype html>
<html lang="cs">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <style>
      /* === SHARED BASE STYLES === */
      @import url("https://fonts.googleapis.com/css2?family=Jost:wght@400;700&display=swap");
      :root { /* tokens */ }
      /* ... */

      /* === SECTION-SPECIFIC STYLES === */
      .pa-section-name { /* ... */ }
    </style>
  </head>
  <body>
    <section class="pa-section-name">
      <!-- Content -->
    </section>
  </body>
</html>
```

## Design Tokens (from `shared.css`)

```css
:root {
  /* Colors */
  --pa-color-dark: #14161a;
  --pa-color-gold: #bf9b70;
  --pa-color-white: #ffffff;
  --pa-color-text: #6b6375;
  --pa-color-text-heading: #08060d;

  /* Typography */
  --pa-font-base: "Jost", system-ui, sans-serif;
  --pa-font-mono: ui-monospace, Consolas, monospace;

  /* Spacing */
  --pa-space-xs: 4px;
  --pa-space-sm: 8px;
  --pa-space-md: 16px;
  --pa-space-lg: 24px;
  --pa-space-xl: 32px;
  --pa-space-2xl: 48px;
  --pa-space-3xl: 64px;

  /* Breakpoints (for reference in media queries) */
  /* Mobile:  max-width: 768px  */
  /* Tablet:  max-width: 1024px */
}
```

## Technical Constraints

1. **No JavaScript** — sections must work without any JS.
2. **No external CSS files in production** — everything in one `<style>` tag.
3. **No build dependencies** — the final HTML must render identically when pasted into any CMS.
4. **Vanilla CSS only** — no Sass, Less, PostCSS nesting (unless you inline the compiled output).
5. **Responsive by default** — use modern responsive patterns (CSS Grid `autofit`/`minmax`, Flexbox `wrap`, `clamp()` for fluid sizing). Mobile breakpoint at `@media (max-width: 768px)`.
6. **Always use responsive grid** — `grid-template-columns: repeat(auto-fit, minmax(min(100%, Npx), 1fr))` is the default layout pattern for card collections. Never use fixed `px` values for `width` or `height` on layout containers or cards.
6. **Font**: Jost (400, 700) loaded via Google Fonts `@import`.
7. **No outer containers** — do not set `max-width`, outer `padding`, or container queries on the root section element. The CMS provides the wrapping container. Sections should fill their parent and only control internal layout.

## Reusable Components

If a pattern repeats across sections (e.g., buttons, cards), define it in `shared.css` with the `pa-` prefix. See `public/shared.css` for the current button component (`.pa-btn`) and its variants.

**Rules:**
- Use design-token custom properties for colors (`var(--pa-color-*)`).
- Round arbitrary Figma values to sensible increments (e.g. `13.72549057006836px` → `14px`, `26.5px` → `27px`).
- Avoid overly precise numbers — readability over 1:1 Figma fidelity.

## Agent Checklist for New Sections

Before declaring a section complete, verify:

- [ ] All CSS classes use the `pa-` prefix.
- [ ] BEM naming is consistent: `.pa-section__element--modifier`.
- [ ] No bare tag selectors or ID selectors for styling.
- [ ] Mobile styles exist under `@media (max-width: 768px)`.
- [ ] Jost font is imported via Google Fonts.
- [ ] CSS custom properties from `shared.css` are used where applicable.
- [ ] The section file is ready for manual verification at `http://localhost:5173/pa-{section-name}.html`.
- [ ] A handoff-ready version (shared + section styles inlined in one `<style>`) can be produced.

**Note:** Do not use Playwright MCP or any other automated verification tool. The user will verify sections manually.
