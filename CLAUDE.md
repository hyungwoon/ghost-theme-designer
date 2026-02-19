# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Casper is the default theme for the Ghost publishing platform (v5.9.0). It uses Handlebars templating, PostCSS for styles, and Gulp as the build system.

## Commands

```bash
yarn install          # Install dependencies
yarn dev              # Start dev server with livereload (gulp watch)
yarn zip              # Build and package theme to dist/casper.zip
yarn test             # Build then validate with gscan
yarn test:ci          # Validate with --fatal --verbose flags
```

## Build Pipeline

CSS: `assets/css/screen.css` → PostCSS (easy-import → color-mod → autoprefixer → cssnano) → `assets/built/screen.css`

JS: `assets/js/lib/*.js` + `assets/js/*.js` → concat → uglify → `assets/built/casper.js`

HBS changes trigger livereload automatically. Edit source files in `assets/css/` and `assets/js/`, never edit `assets/built/` directly.

## Architecture

### Template Hierarchy

`default.hbs` is the base layout (html/head/body wrapper). All other templates insert into its `{{{body}}}` tag:

- `index.hbs` — Home page post list (required). Uses `@custom.header_style`, `@custom.feed_layout`, `@custom.show_publication_cover`
- `post.hbs` — Single post. Uses `@custom.post_image_style`, `@custom.email_signup_text`, `@custom.show_recent_posts_footer`
- `page.hbs` — Static pages (inherits from post.hbs pattern)
- `tag.hbs` / `author.hbs` — Archive pages with site-header-content + post-feed
- `error.hbs` / `error-404.hbs` — Error pages

### CSS Structure

`screen.css` imports `global.css` (reset + base typography) and contains all theme styles in numbered sections:

1. Global (CSS variables in `:root`)
2. Layout (`.viewport`, `.outer`, `.inner`)
3. Site Header
4. Site Navigation (`.gh-head` with 3 layout modes: left-logo, middle-logo, stacked)
5. Post Feed (6-column grid with responsive breakpoints)
6. Single Post (`.gh-canvas` named grid: full/wide/main columns)
7. Author Template
8. Tag Template
9. Error Template
10. Site Footer
11. Dark Mode (`html.dark-mode` and `html.auto-color` overrides)
12. Lightbox (PhotoSwipe)

### Key CSS Variables

```css
--color-darkgrey: #15171A;     --color-midgrey: #738a94;
--color-lightgrey: #f1f1f1;    --color-secondary-text: #979797;
--color-border: #e1e1e1;       --color-darkmode: #151719;
--font-sans: -apple-system, BlinkMacSystemFont, ...;
--font-serif: Georgia, Times, serif;
--font-mono: Menlo, Courier, monospace;
--ghost-accent-color: /* set by Ghost Admin */;
--gh-font-heading: /* user-selected heading font */;
--gh-font-body: /* user-selected body font */;
```

### Content Grid System (`gh-canvas`)

Posts use a named CSS Grid with three width zones:
- `main-start / main-end` — Default content width (720px)
- `wide-start / wide-end` — Wide images/cards
- `full-start / full-end` — Full-bleed content

### Custom Settings

Defined in `package.json` under `config.custom`, accessed via `@custom.*` in templates:

| Setting | Type | Options |
|---------|------|---------|
| `navigation_layout` | select | Logo on cover, Logo in the middle, Stacked |
| `title_font` | select | Modern sans-serif, Elegant serif |
| `body_font` | select | Modern sans-serif, Elegant serif |
| `show_publication_cover` | boolean | Homepage cover image toggle |
| `header_style` | select | Center aligned, Left aligned, Hidden |
| `feed_layout` | select | Classic, Grid, List |
| `color_scheme` | select | Light, Dark, Auto |
| `post_image_style` | select | Wide, Full, Small, Hidden |
| `email_signup_text` | text | Post footer CTA text |
| `show_recent_posts_footer` | boolean | Related posts toggle |

### Partials

- `partials/post-card.hbs` — Post card component used in feeds. Adapts layout based on `@custom.feed_layout` (Classic/Grid/List)
- `partials/lightbox.hbs` — PhotoSwipe image lightbox
- `partials/icons/*.hbs` — Inline SVG icons (search, rss, social, avatar, lock, fire, loader)

### Dark Mode

Three modes controlled by `@custom.color_scheme`:
- **Light**: Default, no special class
- **Dark**: `html.dark-mode` class, all overrides use `html.dark-mode` prefix
- **Auto**: `html.auto-color` class, uses `@media (prefers-color-scheme: dark)` with duplicate rules

When modifying colors, update both `html.dark-mode` and `html.auto-color` selectors.

### PostCSS Features

- `color-mod()` function for color manipulation (e.g., `color-mod(var(--color-darkgrey) l(-5%))`)
- `postcss-easy-import` for `@import` resolution
- Autoprefixer for vendor prefixes

### Responsive Breakpoints

- `992px` — Desktop/tablet navigation switch
- `991px` — Post feed grid collapses to 2 columns
- `767px` — Mobile: single column, hamburger menu, full-screen mobile nav
- `650px` — Content font size reduction
- `600px` — Heading font size reduction
- `500px` — Error page adjustments

### Image Sizes

Configured in `package.json` under `config.image_sizes`: xxs(30w), xs(100w), s(300w), m(600w), l(1000w), xl(2000w). Used with `{{img_url}}` helper's `size` parameter.
