# Ghost Theme Designer Agent — Design Document

## Overview

A specialized Claude Code agent (`ghost-theme-designer`) for modifying the Casper Ghost theme. Focused on CSS/design changes with automated build and ZIP packaging.

## Requirements (from interview)

| Item | Decision |
|------|----------|
| Primary purpose | CSS/design modification |
| Input method | Text description + screenshots/images (mixed) |
| Validation | Auto build → gscan → ZIP |
| Deployment | Ghost(Pro) — manual ZIP upload |
| Explanation style | Concise change summary only |
| Constraints | Preserve Casper base structure |
| Coverage | All theme areas |
| Workflow | Simple → immediate; complex → analyze first |
| Scope | Design only (no code review / security scan) |
| Language | Korean responses; docs in English + Korean |
| Format | Agent + Skill both |
| Ghost knowledge | Casper theme specific |
| Version control | ZIP only (user uploads manually) |
| Name | ghost-theme-designer |

## Architecture

### Approach: Single Integrated Agent (Method A)

One agent handles the full pipeline:

```
Request → Complexity check → [simple] → Edit files
                           → [complex] → Analyze → Report → Edit files
                                                            ↓
                                          gulp build → gscan → yarn zip
```

### Files Created

```
~/.claude/agents/ghost-theme-designer.md    # Claude Code subagent
~/.claude/skills/ghost-theme-designer/SKILL.md  # /ghost-theme slash command
```

### Complexity Threshold

- **Simple** (immediate edit): single color, font size, spacing, single CSS property
- **Complex** (analyze first): layout change, multi-file modification, new component, responsive change

### Build Pipeline

After every modification:

```bash
yarn build   # gulp css + js
yarn test    # gscan validation
yarn zip     # dist/casper.zip
```

## Agent Knowledge Base

### Casper File Map

| File | Purpose |
|------|---------|
| `assets/css/screen.css` | All theme styles (source, edit this) |
| `assets/css/global.css` | CSS reset + base typography |
| `assets/built/screen.css` | Compiled output (never edit directly) |
| `assets/js/*.js` | Theme JS (dropdown, lightbox, infinite-scroll) |
| `assets/built/casper.js` | Compiled JS (never edit directly) |
| `default.hbs` | Base layout (html/head/body) |
| `index.hbs` | Home page post list |
| `post.hbs` | Single post template |
| `page.hbs` | Static page template |
| `tag.hbs` | Tag archive |
| `author.hbs` | Author archive |
| `partials/post-card.hbs` | Post card component |
| `partials/icons/*.hbs` | Inline SVG icons |
| `package.json` | Custom settings + image sizes config |

### CSS Section Map

| Section | Classes | Purpose |
|---------|---------|---------|
| 1. Global | `:root` | CSS variables |
| 2. Layout | `.viewport`, `.outer`, `.inner` | Page structure |
| 3. Site Header | `.site-header`, `.site-header-content` | Cover area |
| 4. Navigation | `.gh-head`, `.gh-head-*` | Top nav bar |
| 5. Post Feed | `.post-feed`, `.post-card`, `.post-card-*` | Article grid |
| 6. Single Post | `.article`, `.gh-canvas`, `.gh-content` | Post content |
| 7. Author | `.author-profile-*` | Author page |
| 8. Tag | `.tag-template` | Tag archive |
| 9. Error | `.error-*` | Error pages |
| 10. Footer | `.site-footer` | Footer |
| 11. Dark Mode | `html.dark-mode`, `html.auto-color` | Color scheme |
| 12. Lightbox | `.pswp` | PhotoSwipe |

### CSS Variables Reference

```css
/* Colors */
--color-darkgrey: #15171A;
--color-midgrey: #738a94;
--color-lightgrey: #f1f1f1;
--color-secondary-text: #979797;
--color-border: #e1e1e1;
--color-wash: #e5eff5;
--color-darkmode: #151719;
--ghost-accent-color: /* set by Ghost Admin */;

/* Fonts */
--font-sans: -apple-system, BlinkMacSystemFont, ...;
--font-serif: Georgia, Times, serif;
--font-mono: Menlo, Courier, monospace;
--gh-font-heading: /* user-selected heading font */;
--gh-font-body: /* user-selected body font */;
```

### Custom Settings (`@custom.*`)

| Setting | Template Access |
|---------|----------------|
| navigation_layout | `@custom.navigation_layout` |
| title_font | `@custom.title_font` |
| body_font | `@custom.body_font` |
| show_publication_cover | `@custom.show_publication_cover` |
| header_style | `@custom.header_style` |
| feed_layout | `@custom.feed_layout` |
| color_scheme | `@custom.color_scheme` |
| post_image_style | `@custom.post_image_style` |
| email_signup_text | `@custom.email_signup_text` |
| show_recent_posts_footer | `@custom.show_recent_posts_footer` |

### Responsive Breakpoints

| Breakpoint | Layout Change |
|-----------|--------------|
| 992px | Desktop nav → compact |
| 991px | Post feed → 2 columns |
| 767px | Mobile: single column, hamburger |
| 650px | Content font size reduction |
| 600px | Heading font size reduction |

### Dark Mode Rule

When modifying any color/background: always update **both** `html.dark-mode` **and** `html.auto-color` selectors.

---

## Korean Translation / 한국어 번역

### 개요

Casper Ghost 테마 수정 전용 Claude Code 에이전트. CSS/디자인 변경에 특화되어 있으며 자동 빌드 및 ZIP 패키징을 지원합니다.

### 아키텍처: 단일 통합 에이전트 (방식 A)

하나의 에이전트가 전체 파이프라인을 처리합니다.

### 복잡도 기준

- **간단** (즉시 수정): 색상, 폰트 크기, 간격, 단일 CSS 속성
- **복잡** (분석 후 수정): 레이아웃 변경, 다중 파일 수정, 새 컴포넌트, 반응형 수정

### 빌드 파이프라인

모든 수정 후 자동 실행:
1. `yarn build` — CSS/JS 컴파일
2. `yarn test` — gscan 테마 검증
3. `yarn zip` — dist/casper.zip 생성
