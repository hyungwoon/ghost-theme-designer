# Ghost Theme Designer

AI-powered Casper Ghost theme design modification agent. Automates CSS/HBS editing, building, and validation for Ghost theme customization.

## Overview

`ghost-theme-designer` is a specialized Claude Code agent that streamlines the process of modifying the Casper Ghost theme. It handles the entire pipeline:

1. **Edit** - Modify CSS (`assets/css/`) and Handlebars templates (`.hbs` files)
2. **Build** - Compile assets using the Gulp build system
3. **Validate** - Test with gscan theme validator
4. **Package** - Create distribution ZIP file

## Usage

Invoke the agent via Claude Code:

```bash
/ghost-theme-designer
```

Then describe what you want to modify:
- "Change the header background color to navy blue"
- "Make the post cards wider on desktop"
- "Add a custom footer section"
- "Modify the dark mode colors"

The agent will automatically handle CSS compilation, template updates, and validation.

## Getting Started

### Prerequisites

- Node.js (v18+)
- Yarn package manager
- Ghost CMS (v5.9.0+)

### Installation

```bash
# Clone and setup
git clone https://github.com/hyungwoon/ghost-theme-designer.git
cd ghost-theme-designer

# Install dependencies
yarn install

# Start development server
yarn dev
```

## Available Commands

| Command | Purpose |
|---------|---------|
| `yarn install` | Install dependencies |
| `yarn dev` | Start dev server with livereload (Gulp watch) |
| `yarn zip` | Build and package theme to `dist/casper.zip` |
| `yarn test` | Build and validate with gscan |
| `yarn test:ci` | Validate with `--fatal --verbose` flags |

## Project Structure

```
casper/
├── assets/
│   ├── css/                    # Source CSS (edit here)
│   │   ├── screen.css         # Main stylesheet
│   │   ├── global.css         # Reset & typography
│   │   └── features/          # Feature-specific styles
│   ├── built/                 # Compiled CSS/JS (auto-generated)
│   └── js/                    # JavaScript sources
├── partials/                  # Handlebars components
│   ├── post-card.hbs         # Post card layout
│   ├── icons/                # SVG icon partials
│   └── lightbox.hbs          # Image lightbox
├── default.hbs               # Base layout template
├── index.hbs                 # Home page template
├── post.hbs                  # Single post template
├── tag.hbs / author.hbs      # Archive pages
└── package.json              # Build config & custom settings
```

## Theme Architecture

### Base Layout
`default.hbs` is the root template containing the HTML structure. All pages render their content into its `{{{body}}}` block.

### CSS Grid System
Posts use a named CSS Grid with three zones:
- `main` (720px default content width)
- `wide` (full-width images/cards)
- `full` (full-bleed sections)

### Custom Settings
Configured in `package.json` under `config.custom`, accessed in templates via `@custom.*`:

- `navigation_layout` - Logo position (cover/middle/stacked)
- `title_font` - Heading font family
- `body_font` - Body font family
- `color_scheme` - Light/Dark/Auto mode
- `header_style` - Header alignment (center/left/hidden)
- `feed_layout` - Post grid type (classic/grid/list)
- `post_image_style` - Featured image size (wide/full/small/hidden)
- `email_signup_text` - Post footer CTA
- `show_recent_posts_footer` - Related posts toggle

### CSS Variables

Key variables in `:root`:
```css
--color-darkgrey: #15171A;
--color-lightgrey: #f1f1f1;
--ghost-accent-color: /* set by Ghost Admin */
--gh-font-heading: /* user-selected heading font */
--gh-font-body: /* user-selected body font */
```

### Dark Mode
Three color schemes controlled by `@custom.color_scheme`:
- **Light** - Default light theme
- **Dark** - `html.dark-mode` class with dark overrides
- **Auto** - `html.auto-color` class with `@media (prefers-color-scheme: dark)` media queries

## Development Workflow

1. Edit source files in `assets/css/` and template files
2. HBS template changes trigger livereload automatically
3. CSS changes compile via PostCSS pipeline
4. View changes in dev server at `http://localhost:3000`
5. Run `yarn zip` to package for upload to Ghost

## Build Pipeline

**CSS**: `assets/css/screen.css` → PostCSS (import → color-mod → autoprefixer → cssnano) → `assets/built/screen.css`

**JS**: `assets/js/lib/*.js` + `assets/js/*.js` → concat → uglify → `assets/built/casper.js`

**Note**: Always edit source files in `assets/css/` and `assets/js/`, never edit `assets/built/` directly.

## Customization Examples

### Change Header Color
Edit `assets/css/screen.css` Section 3 (Site Header):
```css
.gh-head {
  background: var(--gh-accent-color);
}
```

### Add Custom Font
Update `config.custom.title_font` in `package.json`, then in CSS:
```css
h1, h2, h3 {
  font-family: var(--gh-font-heading);
}
```

### Modify Post Card Layout
Edit `partials/post-card.hbs` and adjust CSS in Section 5 (Post Feed).

## Validation

Before deploying, always validate:
```bash
yarn test        # Development validation
yarn test:ci     # CI validation (strict)
```

This runs gscan to check for Ghost theme compatibility issues.

## License

Based on [Casper](https://github.com/TryGhost/Casper) by Ghost Foundation.
Copyright (c) 2013-2025 Ghost Foundation - [MIT License](LICENSE)
