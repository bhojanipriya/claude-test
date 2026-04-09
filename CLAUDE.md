# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Skeleton** is a minimal, foundational Shopify theme by Shopify, designed as a starting point — not a fully-featured theme. It must stay minimalist and avoid legacy or non-recommended Shopify features.

- Online Store 2.0: JSON templates throughout, sections everywhere, app block support
- No external JS libraries — sections use inline `{% javascript %}` tags (auto-deduped by Shopify)
- CSS custom property defaults live in `assets/variables.css`; at runtime `snippets/css-variables.liquid` overrides them with live theme editor values

---

## CLI Commands

```bash
shopify theme dev --store=your-store.myshopify.com      # Dev server with hot reload
shopify theme push                                       # Upload theme to store
shopify theme pull                                       # Download from store
shopify theme push --only sections/header.liquid        # Push a single file
shopify theme check                                     # Lint (Theme Check, config in .theme-check.yml)
shopify theme share                                     # Generate shareable preview URL
```

---

## Tech Stack

- **Templates**: JSON templates only (`templates/*.json`) — except `gift_card.liquid`
- **CSS**: `assets/critical.css` loaded on every page (preloaded); per-section styles via inline `{% stylesheet %}` tags
- **JS**: Inline `{% javascript %}` tags within sections/blocks — no bundler, no external libraries
- **CSS variables**: Defined in `snippets/css-variables.liquid` using a `{% style %}` tag (not `{% stylesheet %}`— intentional, to avoid deduplication), sourced from `config/settings_schema.json` values

---

## Project Structure

```
/
├── assets/
│   ├── critical.css           # Only global stylesheet — loaded (preloaded) on every page
│   ├── variables.css          # Static CSS custom property defaults (overridden at runtime by css-variables.liquid)
│   └── *.svg                  # Icon assets (account, cart, etc.)
├── blocks/
│   ├── group.liquid           # Container/grouping block
│   └── text.liquid            # Text block with style variants (title/subtitle/normal)
├── config/
│   ├── settings_schema.json   # Theme Editor settings (fonts, colors, spacing)
│   └── settings_data.json     # Saved setting values — do not commit live store data
├── layout/
│   ├── theme.liquid           # Global shell: css-variables → critical.css → header-group → content → footer-group
│   └── password.liquid        # Coming-soon/password page layout
├── locales/
│   ├── en.default.json        # Storefront strings
│   └── en.default.schema.json # Theme Editor label strings
├── sections/
│   ├── header-group.json      # Section group config (JSON, not Liquid) — defines which sections appear in header
│   ├── footer-group.json      # Section group config (JSON, not Liquid) — defines which sections appear in footer
│   ├── header.liquid          # Site header
│   ├── footer.liquid          # Site footer
│   ├── product.liquid         # Product page section
│   ├── collection.liquid      # Collection page section
│   └── *.liquid               # One section per page type + custom-section/hello-world examples
├── snippets/
│   ├── css-variables.liquid   # Injects theme settings as CSS custom properties
│   ├── image.liquid           # Reusable image rendering helper
│   └── meta-tags.liquid       # SEO/social meta tags
└── templates/
    ├── gift_card.liquid        # Only non-JSON template
    └── *.json                  # JSON template per page type — Shopify admin may overwrite these
```

---

## Architecture

### CSS Variable Pattern
Theme settings (fonts, colors, spacing) are exposed as CSS custom properties via `snippets/css-variables.liquid`, rendered in `layout/theme.liquid` before `critical.css`. Component styles reference these variables inline.

Two schema patterns for section styling:
- **CSS variable** — for single properties (e.g., `--text-align: {{ section.settings.text_alignment }}`)
- **CSS class modifier** — for multiple related properties (e.g., `.text--title`, `.text--subtitle`)

### Section/Block Inline Styles
Each section and block defines its own styles and JS inline:
```liquid
{% stylesheet %}
  .my-section { color: var(--color-foreground); }
{% endstylesheet %}

{% javascript %}
  // section-scoped JS here
{% endjavascript %}
```
Shopify automatically deduplicates these across multiple section instances.

### Layout Structure
`layout/theme.liquid` loads CSS variables → `critical.css` → section groups (`header-group`, `footer-group`) → page content. There is no separate JS bundle loaded in the layout.

### Blocks
`blocks/` contains reusable theme blocks (e.g., `group.liquid`, `text.liquid`) that sections reference. Blocks follow the same inline stylesheet/javascript pattern. Key conventions:
- Use `{% doc %}` tags inside block files for schema documentation
- Always include `{{ block.shopify_attributes }}` on the root element (enables merchant editing tools)
- Use `{ "type": "@theme" }` in a section's `blocks` schema to allow any theme block type
- Define `presets` in the block schema to provide default configurations

### Font Loading
`layout/theme.liquid` preconnects to `https://fonts.shopifycdn.com` and preloads only the base font variant. Additional weights/styles load on-demand via `@font-face` with `font-display: swap`. System fonts skip the preload entirely (`settings.type_primary_font.system?` check).

---

## CI / Linting

GitHub Actions runs `shopify/theme-check-action@v2` on every push. Run the same check locally before pushing:

```bash
shopify theme check   # uses .theme-check.yml (extends theme-check:recommended)
```

---

## Contributing Constraints

Per `CONTRIBUTING.md`, this theme must remain a **minimal foundational starting point**:
- No fully-featured implementations
- No legacy or non-recommended Shopify patterns
- Open an issue before proposing new features
- Requires signing the Shopify CLA for contributions

## Resources

- Liquid reference:    https://shopify.dev/docs/api/liquid
- Section schema docs: https://shopify.dev/docs/themes/architecture/sections/section-schema
- Block schema docs:   https://shopify.dev/docs/themes/architecture/blocks
- Shopify CLI:         https://shopify.dev/docs/themes/tools/cli
- Theme Check:         https://shopify.dev/docs/themes/tools/theme-check