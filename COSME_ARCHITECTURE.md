# Cosme Theme Architecture

> **Status:** Planning document only. No implementation included.
>
> **Base:** Shopify Skeleton Theme v0.1.0
>
> **Target:** Premium editorial Beauty theme for the Shopify Theme Store

---

## A. Current Skeleton Architecture

Cosme is initialized from the official [Shopify Skeleton Theme](https://github.com/Shopify/skeleton-theme)—a minimal, opinionated starter that demonstrates Shopify's modern theme architecture patterns.

### Architecture summary

| Layer | Current state |
|---|---|
| **Layouts** | `theme.liquid` (main), `password.liquid` (storefront password) |
| **Templates** | 12 JSON templates + 1 Liquid template (`gift_card.liquid`) |
| **Section groups** | `header-group.json`, `footer-group.json` |
| **Sections** | 14 section files (global, page-type mains, demo) |
| **Theme blocks** | 2 blocks: `text.liquid`, `group.liquid` |
| **Snippets** | 3 snippets: `css-variables`, `meta-tags`, `image` |
| **Assets** | `critical.css`, 3 SVG icons, 1 demo SVG |
| **JavaScript** | None |
| **Build tooling** | None (no npm, Vite, Tailwind, etc.) |
| **Linting** | Theme Check (`theme-check:recommended`) |

### Key architectural patterns in Skeleton

1. **JSON templates** — Every page type uses a JSON template referencing one or more sections. Merchants customize page structure via the Theme Editor without code changes.

2. **Section groups** — Header and footer are rendered via `{% sections 'header-group' %}` / `{% sections 'footer-group' %}` in `layout/theme.liquid`, enabling grouped, persistent global sections.

3. **Theme blocks** — Reusable, nestable components in `/blocks` with their own schemas. Sections like `custom-section.liquid` accept `@theme` blocks via `{% content_for 'blocks' %}`.

4. **Co-located CSS** — Component styles live in `{% stylesheet %}` tags within sections, blocks, and snippets. Shopify deduplicates these at runtime.

5. **Critical CSS separation** — Global reset, typography baseline, and the section grid system live in `assets/critical.css`, preloaded on every page.

6. **CSS custom properties from settings** — `snippets/css-variables.liquid` maps global theme settings to `:root` variables consumed across the theme.

7. **Section grid layout** — `.shopify-section` uses CSS Grid with `--content-grid` to support both constrained and full-width child elements (`.full-width` utility).

8. **LiquidDoc** — Snippets and blocks include `{% doc %}` headers documenting parameters and usage.

9. **Translation keys** — User-facing strings use `{{ 'key' | t }}`. Schema strings use `t:` prefix keys in `locales/en.default.schema.json`.

10. **Shopify native integrations** — `shopify-account` web component, `payment_button`, `structured_data`, font picker with preload, `inline_asset_content` for SVG icons.

### Current file inventory

```
.
├── assets/
│   ├── critical.css
│   ├── icon-account.svg
│   ├── icon-cart.svg
│   └── shoppy-x-ray.svg          # demo asset — replace later
├── blocks/
│   ├── group.liquid
│   └── text.liquid
├── config/
│   ├── settings_data.json        # empty current preset
│   └── settings_schema.json      # typography, layout, colors
├── layout/
│   ├── password.liquid
│   └── theme.liquid
├── locales/
│   ├── en.default.json
│   └── en.default.schema.json
├── sections/
│   ├── 404.liquid
│   ├── article.liquid
│   ├── blog.liquid
│   ├── cart.liquid
│   ├── collection.liquid
│   ├── collections.liquid
│   ├── custom-section.liquid
│   ├── footer-group.json
│   ├── footer.liquid
│   ├── header-group.json
│   ├── header.liquid
│   ├── hello-world.liquid        # demo section — replace later
│   ├── page.liquid
│   ├── password.liquid
│   ├── product.liquid
│   ├── search.liquid
│   └── password.liquid
├── snippets/
│   ├── css-variables.liquid
│   ├── image.liquid
│   └── meta-tags.liquid
└── templates/
    ├── 404.json
    ├── article.json
    ├── blog.json
    ├── cart.json
    ├── collection.json
    ├── gift_card.liquid
    ├── index.json
    ├── list-collections.json
    ├── page.json
    ├── password.json
    ├── product.json
    └── search.json
```

### Current global settings (`config/settings_schema.json`)

| Group | Settings |
|---|---|
| Typography | Primary font (`font_picker`) |
| Layout | Page width (90rem / 110rem), page margin (10–100px) |
| Colors | Background, foreground, input corner radius |

### Current gaps (intentional in Skeleton, to be addressed in Cosme)

- No customer account templates (`templates/customers/*`)
- No JavaScript (variant selection, cart drawer, media gallery, etc.)
- No app block support (`@app` not declared in section/block schemas)
- Minimal accessibility (only `role="search"` on search form)
- Minimal responsive behavior (one breakpoint in `hello-world.liquid`)
- No design system beyond basic CSS variables
- Page-type sections are functional stubs, not editorial/commerce-ready
- Hardcoded English in `hello-world.liquid` (violates i18n best practice)
- `hello-world` demo section on homepage
- No cart drawer, quick add, or predictive search
- No product recommendations, complementary products, or Shop Pay installments UI
- Gift card template references `settings.logo` which is not defined in settings schema

---

## B. Proposed Cosme Architecture

Cosme preserves Shopify's native theme architecture entirely. All storefront rendering remains **Liquid + JSON templates + sections/blocks + snippets + CSS + minimal vanilla JS**. No framework migration.

### Design principles

1. **Editorial-first, commerce-capable** — Beauty brands need magazine-style layouts with strong typography, imagery, and whitespace, while product discovery and conversion paths remain first-class.

2. **Composition over monoliths** — Small, focused sections and blocks compose into flexible page layouts. Avoid mega-sections with dozens of unrelated settings.

3. **Design tokens via CSS custom properties** — Global theme settings map to `:root` variables. Section/block settings map to scoped inline CSS variables or modifier classes.

4. **Snippets for logic, blocks for editor** — Reusable rendering logic lives in snippets (not editable by merchants). Merchant-customizable UI lives in blocks and sections.

5. **Progressive enhancement** — Core functionality works without JavaScript. JS enhances variant selection, cart interactions, media galleries, and animations.

6. **Theme Store ready from day one** — Accessibility, performance, app compatibility, and Shopify feature support are architectural requirements, not afterthoughts.

### Architectural layers

```
┌─────────────────────────────────────────────────────────┐
│  Layout (theme.liquid / password.liquid)                │
│  ├── css-variables snippet                              │
│  ├── critical.css                                       │
│  ├── meta-tags snippet                                  │
│  ├── header-group (section group)                       │
│  ├── content_for_layout (JSON template sections)        │
│  └── footer-group (section group)                       │
├─────────────────────────────────────────────────────────┤
│  JSON Templates (index, product, collection, …)         │
│  └── Ordered list of sections + section settings        │
├─────────────────────────────────────────────────────────┤
│  Sections (page modules, global chrome)                 │
│  ├── Theme blocks (@theme)                              │
│  ├── Section blocks (inline, section-specific)          │
│  └── App blocks (@app) where appropriate                │
├─────────────────────────────────────────────────────────┤
│  Theme Blocks (/blocks) — nestable editor components    │
├─────────────────────────────────────────────────────────┤
│  Snippets — shared rendering, no editor exposure        │
├─────────────────────────────────────────────────────────┤
│  Assets — critical.css, icons, optional shared JS         │
└─────────────────────────────────────────────────────────┘
```

---

## C. Directory Structure

Proposed Cosme directory structure. **Do not create empty placeholder files yet** — this is the target layout.

```
.
├── assets/
│   ├── critical.css                    # Global reset, tokens consumption, grid system
│   ├── cosme.js                        # Minimal shared JS entry (deferred, module)
│   └── icons/                          # SVG icon set
│       ├── icon-account.svg
│       ├── icon-cart.svg
│       ├── icon-search.svg
│       ├── icon-menu.svg
│       ├── icon-close.svg
│       └── …
│
├── blocks/
│   ├── _accordion-item.liquid          # Private/nested block (leading underscore convention)
│   ├── accordion.liquid
│   ├── badge.liquid
│   ├── button.liquid
│   ├── heading.liquid
│   ├── icon.liquid
│   ├── image.liquid
│   ├── group.liquid                    # Preserve from Skeleton
│   ├── product-card.liquid
│   ├── rich-text.liquid
│   ├── text.liquid                     # Preserve from Skeleton (extend)
│   └── video.liquid
│
├── config/
│   ├── settings_data.json
│   └── settings_schema.json
│
├── layout/
│   ├── password.liquid
│   └── theme.liquid
│
├── locales/
│   ├── en.default.json
│   ├── en.default.schema.json
│   └── (future locale files added at Theme Store submission)
│
├── sections/
│   ├── global/
│   │   ├── header.liquid
│   │   ├── footer.liquid
│   │   ├── announcement-bar.liquid
│   │   ├── cart-drawer.liquid
│   │   └── predictive-search.liquid
│   ├── homepage/
│   │   ├── hero-editorial.liquid
│   │   ├── featured-collection.liquid
│   │   ├── image-with-text.liquid
│   │   ├── multicolumn.liquid
│   │   ├── testimonials.liquid
│   │   ├── logo-list.liquid
│   │   ├── newsletter.liquid
│   │   └── rich-text.liquid
│   ├── product/
│   │   ├── main-product.liquid
│   │   ├── product-recommendations.liquid
│   │   └── recently-viewed.liquid
│   ├── collection/
│   │   ├── main-collection.liquid
│   │   ├── collection-banner.liquid
│   │   └── featured-collection-grid.liquid
│   ├── cart/
│   │   └── main-cart.liquid
│   ├── search/
│   │   └── main-search.liquid
│   ├── blog/
│   │   ├── main-blog.liquid
│   │   └── main-article.liquid
│   ├── page/
│   │   └── main-page.liquid
│   ├── utility/
│   │   ├── 404.liquid
│   │   ├── password.liquid
│   │   └── custom-section.liquid       # Preserve flexible block container
│   ├── header-group.json
│   └── footer-group.json
│
├── snippets/
│   ├── design-system/
│   │   ├── css-variables.liquid        # Preserve — extend with full token set
│   │   ├── css-utilities.liquid        # Optional utility classes if needed
│   │   └── focus-styles.liquid         # Shared focus-visible patterns
│   ├── components/
│   │   ├── button.liquid
│   │   ├── form-field.liquid
│   │   ├── price.liquid
│   │   ├── product-card.liquid
│   │   ├── product-badge.liquid
│   │   ├── product-media.liquid
│   │   ├── variant-picker.liquid
│   │   ├── pagination.liquid
│   │   ├── breadcrumbs.liquid
│   │   ├── icon.liquid
│   │   └── image.liquid                # Preserve from Skeleton — extend
│   ├── layout/
│   │   ├── skip-link.liquid
│   │   └── section-wrapper.liquid
│   └── meta/
│       ├── meta-tags.liquid            # Preserve from Skeleton
│       └── structured-data.liquid      # Extract product JSON-LD if it grows
│
└── templates/
    ├── 404.json
    ├── article.json
    ├── blog.json
    ├── cart.json
    ├── collection.json
    ├── gift_card.liquid
    ├── index.json
    ├── list-collections.json
    ├── page.json
    ├── page.contact.json               # Alternate page template example
    ├── password.json
    ├── product.json
    ├── search.json
    └── customers/
        ├── account.json
        ├── activate_account.json
        ├── addresses.json
        ├── login.json
        ├── order.json
        ├── register.json
        └── reset_password.json
```

> **Note on section subdirectories:** Shopify requires section `.liquid` files directly in `/sections` (not nested subfolders). The structure above uses logical grouping in documentation; actual files will use naming prefixes (e.g., `main-product.liquid`, `hero-editorial.liquid`) flat in `/sections`.

---

## D. Responsibilities of Each Directory

### `/assets`

- **`critical.css`** — Only CSS required on every page: reset, base typography, layout grid, focus defaults, reduced-motion overrides. Keep under ~14KB gzipped target.
- **`cosme.js`** — Single deferred ES module for shared interactivity (cart drawer, variant sync, disclosure toggles). Section-specific JS stays in `{% javascript %}` tags.
- **`/icons`** — SVG icons using `currentColor` and CSS variable stroke width (matching existing icon pattern).

### `/blocks`

Theme blocks merchants can add, remove, reorder, and nest within supporting sections.

- **Layout blocks** — `group` (row/column wrapper), `accordion`
- **Content blocks** — `heading`, `text`, `rich-text`, `button`
- **Media blocks** — `image`, `video`
- **Commerce blocks** — `product-card`, `badge`
- Blocks include `{% doc %}`, `{% schema %}`, `{% stylesheet %}`, and `{{ block.shopify_attributes }}`
- Use `@theme` nesting in parent blocks; leading `_` prefix for blocks not meant as top-level presets

### `/config`

- **`settings_schema.json`** — Global theme settings exposed in Theme Editor (design tokens, behavior toggles)
- **`settings_data.json`** — Auto-generated preset data; treat as runtime state, not source of truth

### `/layout`

- **`theme.liquid`** — Document shell: CSS variables, critical CSS, meta tags, header/footer groups, skip link, `content_for_layout`
- **`password.liquid`** — Minimal layout for password-protected storefront

### `/locales`

- **`en.default.json`** — All customer-facing translatable strings
- **`en.default.schema.json`** — Theme Editor labels, option names, section names
- Hierarchical keys, max 3 levels, snake_case
- Sentence case for all UI text

### `/sections`

Full-width page modules. Each section owns its markup, scoped CSS, optional JS, and schema.

- **Global sections** — Persistent chrome (header, footer, announcement, cart drawer)
- **Homepage sections** — Editorial/marketing modules with presets for quick assembly
- **Page-type main sections** — One primary section per template type (`main-product`, `main-collection`, etc.) with `disabled_on` for header/footer groups
- **Section groups** — JSON files grouping global sections

### `/snippets`

Reusable Liquid fragments **not** exposed in the Theme Editor.

- **`/design-system`** — Token output, shared CSS patterns
- **`/components`** — Atomic UI rendered via `{% render %}` with typed parameters (LiquidDoc)
- **`/layout`** — Structural helpers (skip link, section wrappers)
- **`/meta`** — SEO and structured data

### `/templates`

JSON files defining which sections appear on each page type and their default order/settings.

- Prefer JSON templates over Liquid templates
- Exception: `gift_card.liquid` (Shopify requirement)
- Alternate templates use Shopify naming (`page.contact.json`, `product.featured.json`)

---

## E. Planned Reusable Components

### Design system primitives (snippets)

| Component | Snippet | Purpose |
|---|---|---|
| Image | `snippets/cosme-image.liquid` | Responsive image with optional link, srcset, aspect ratio, lazy loading |
| Icon | `snippets/cosme-icon.liquid` | Inline SVG by name with accessible title |
| Button | `snippets/cosme-button.liquid` | Primary/secondary/tertiary/link variants |
| Form field | `snippets/cosme-form-field.liquid` | Label, input, error, hint pattern |
| Price | `snippets/cosme-price.liquid` | Regular, sale, compare-at, unit price, from-price |
| Badge | `snippets/cosme-product-badge.liquid` | Sale, sold out, new, custom |
| Product card | `snippets/components/product-card.liquid` | Image, title, price, badges, quick add optional |
| Product media | `snippets/components/product-media.liquid` | Image/video/3D model rendering |
| Variant picker | `snippets/components/variant-picker.liquid` | Swatches, buttons, dropdowns |
| Pagination | `snippets/components/pagination.liquid` | Accessible pagination replacing default filter |
| Breadcrumbs | `snippets/components/breadcrumbs.liquid` | Collection/product navigation |
| Skip link | `snippets/layout/skip-link.liquid` | Keyboard skip to main content |

### Theme blocks (merchant-editable)

| Block | Purpose |
|---|---|
| `group` | Flex row/column container for nesting blocks |
| `heading` | Semantic heading with level + style settings |
| `text` / `rich-text` | Body copy with typography presets |
| `button` | CTA with link + style |
| `image` | Editorial image with caption, aspect ratio |
| `video` | Shopify-hosted or external video |
| `product-card` | Single product display in grids/carousels |
| `badge` | Promotional label overlay |
| `accordion` | FAQ/collapsible content group |

### CSS design tokens (via `css-variables.liquid`)

| Token category | Examples |
|---|---|
| Typography | Font families (heading, body, accent), scale, line heights, letter spacing |
| Colors | Background, foreground, accent, sale, border, overlay, badge variants |
| Spacing | Section spacing scale, grid gap, component padding |
| Containers | Page width, margin, content max-width |
| Buttons | Height, padding, radius, border width |
| Forms | Input height, radius, border, focus ring |
| Images | Aspect ratios, border radius, overlay opacity |
| Icons | Size scale, stroke width |
| Animations | Duration, easing (respecting reduced motion) |
| Breakpoints | Documented constants used in media queries |

---

## F. Planned Global Settings

> Documented for planning only. **Do not implement yet.**

### Theme info
- Theme name: Cosme
- Version, author, documentation URL, support URL

### Typography
- Heading font (`font_picker`)
- Body font (`font_picker`)
- Accent/label font (`font_picker`, optional)
- Base font size scale
- Heading scale multiplier

### Colors
- Background
- Foreground (text)
- Accent (links, CTAs)
- Accent contrast (auto-calculated or manual override)
- Sale / badge colors
- Border color
- Overlay color + opacity
- Input background

### Layout
- Page width (range or select presets)
- Page margin
- Section vertical spacing (compact / default / spacious)
- Grid horizontal gap

### Buttons
- Primary button background / text / radius
- Secondary button style
- Button border width

### Forms
- Input corner radius (preserve existing setting)
- Input border color
- Focus ring color / width

### Product cards
- Image aspect ratio (portrait for beauty editorial default)
- Show vendor / show rating / show secondary image on hover
- Quick add toggle

### Cart
- Cart type: page / drawer
- Show cart note
- Show free shipping bar (threshold setting)

### Social media
- Social account links (used in footer)

### Favicon & logo
- Logo, logo width, favicon

### Animation
- Enable/disable motion globally (pairs with `prefers-reduced-motion`)
- Transition duration preset

### Search
- Enable predictive search
- Search suggestions toggle

---

## G. Planned Section Architecture

### Section categories

#### 1. Global (section groups)

| Section | Blocks | Notes |
|---|---|---|
| `announcement-bar` | Text, link | Dismissible, optional carousel |
| `header` | Logo, menu, search, account, cart | Sticky option, transparent overlay on homepage |
| `footer` | Menu columns, newsletter, social, payment icons | Multi-column block layout |
| `cart-drawer` | — | Rendered globally when cart type = drawer |
| `predictive-search` | — | Modal/overlay search results |

#### 2. Homepage / marketing (presets for Theme Editor)

| Section | Blocks | Notes |
|---|---|---|
| `hero-editorial` | Image, heading, text, button | Full-width editorial hero |
| `featured-collection` | — (uses product cards) | Grid or carousel |
| `image-with-text` | Image, heading, text, button | 50/50 and offset layouts |
| `multicolumn` | Column blocks (icon, heading, text) | Brand values, rituals, ingredients |
| `testimonials` | Quote blocks | Beauty social proof |
| `logo-list` | Logo blocks | Press / brand partners |
| `newsletter` | — | Email capture |
| `rich-text` | Heading, text, button | Editorial copy blocks |
| `custom-section` | `@theme` blocks | Flexible container (preserve Skeleton pattern) |

#### 3. Page-type main sections

| Section | Template | Key features |
|---|---|---|
| `main-product` | `product.json` | Media gallery, variant picker, buy buttons, pickup availability, sticky ATC, tabs/accordion for details |
| `main-collection` | `collection.json` | Filters (Shopify Search & Discovery), sort, grid/list, pagination |
| `main-cart` | `cart.json` | Line items, quantities, notes, checkout |
| `main-search` | `search.json` | Faceted results, product/article/page tabs |
| `main-blog` | `blog.json` | Editorial article grid |
| `main-article` | `article.json` | Hero image, metadata, content, comments |
| `main-page` | `page.json` | Page title + content |
| `404` | `404.json` | Branded not-found |
| `password` | `password.json` | Storefront password |

#### 4. Reusable commerce sections

| Section | Usage |
|---|---|
| `product-recommendations` | Product template, cart |
| `recently-viewed` | Product template (JS + localStorage) |
| `collection-banner` | Collection template header |
| `featured-collection-grid` | Any template |

### Section schema conventions

- `"tag": "section"` with optional `"class": "cosme-section cosme-section--{name}"`
- Use `"disabled_on": { "groups": ["header", "footer"] }` for main content sections
- Declare `"blocks": [{ "type": "@theme" }, { "type": "@app" }]` where app blocks add merchant value
- Provide `"presets"` with beauty-industry-friendly names and categories
- Use `visible_if` for conditional settings (already demonstrated in Skeleton `group` block)
- Single-property settings → CSS variables; multi-property → modifier classes

---

## H. Planned Template Architecture

### Template strategy

All templates remain JSON except `gift_card.liquid`. Each template defines an ordered section list merchants can customize.

| Template | Sections (default order) | Notes |
|---|---|---|
| `index.json` | Hero, featured collection, image-with-text, multicolumn, newsletter | Replace hello-world |
| `product.json` | main-product, product-recommendations | Consider alternate templates later |
| `collection.json` | collection-banner, main-collection | |
| `list-collections.json` | collections grid (rename/refactor) | |
| `cart.json` | main-cart | |
| `search.json` | main-search | |
| `blog.json` | main-blog | |
| `article.json` | main-article | |
| `page.json` | main-page | |
| `404.json` | 404 | |
| `password.json` | password (layout: password) | |
| `customers/*.json` | Customer account sections | Required for Theme Store |

### Alternate templates (future)

- `product.featured.json` — Editorial long-form product layout
- `collection.editorial.json` — Lookbook-style collection
- `page.contact.json` — Contact form section
- `page.ingredients.json` — Beauty-specific content layout

### Template rules

- Never hardcode content in templates; all content flows through section settings/blocks
- Keep `settings_data.json` and template JSON auto-generated comments — avoid manual edits that fight the Theme Editor
- Use section groups in layout, not duplicated in each template

---

## I. CSS Architecture

### Three-tier CSS model (preserve and extend Skeleton pattern)

```
Tier 1: assets/critical.css
        Global reset, document baseline, .shopify-section grid, reduced-motion

Tier 2: snippets/css-variables.liquid
        :root design tokens from theme settings (rendered in <head>)

Tier 3: {% stylesheet %} in sections/blocks/snippets
        Component-scoped styles, deduplicated by Shopify
```

### Naming convention: BEM with `cosme-` namespace

```css
.cosme-product-card { }
.cosme-product-card__media { }
.cosme-product-card__title { }
.cosme-product-card--featured { }
```

Existing Skeleton classes (`.shopify-section`, `.full-width`, `.image`) are preserved as Shopify/platform conventions.

### Settings → CSS mapping rules

| Setting type | Implementation |
|---|---|
| Single CSS property | Inline style CSS variable: `style="--gap: {{ setting }}px"` |
| Multiple properties | Modifier class: `class="cosme-button cosme-button--primary"` |
| Color settings | CSS variable on `:root` via `css-variables.liquid` |
| Conditional visibility | Liquid `{% if %}` — never CSS-only hiding of essential content |

### Responsive CSS

- Mobile-first media queries
- Breakpoint constants (documented, not necessarily as CSS variables):
  - `sm`: 576px
  - `md`: 768px
  - `lg`: 990px
  - `xl`: 1200px
- Section grid system from Skeleton handles page-level horizontal layout
- Component-level responsive behavior in component stylesheets

### What NOT to add

- No Tailwind, Sass, PostCSS build pipeline
- No CSS-in-JS
- No external CSS frameworks
- Avoid `@import` chains in assets

---

## J. JavaScript Architecture

### Current state

Skeleton includes **zero JavaScript**. This is intentional and should remain the starting mindset.

### Proposed minimal JS strategy

| Principle | Detail |
|---|---|
| **Location** | `{% javascript %}` in sections/snippets for component-specific code; one shared `assets/cosme.js` for cross-page utilities |
| **Loading** | `defer` on shared module; no render-blocking scripts |
| **Modules** | ES modules, no bundler required initially |
| **No dependencies** | Vanilla JS only; no jQuery, Alpine, React |
| **Progressive enhancement** | Forms work with full page reload; JS enhances UX |
| **Shopify APIs** | Use Section Rendering API, Cart API (`/cart/add.js`, `/cart/change.js`), Product JSON |

### Planned JS modules (implement only when needed)

| Module | Purpose | Scope |
|---|---|---|
| `cart-drawer.js` | Open/close drawer, AJAX cart updates | Global |
| `variant-picker.js` | Sync variant selection, URL, price, media | Product |
| `product-media.js` | Gallery thumbnails, zoom optional | Product |
| `quantity-input.js` | Increment/decrement with min/max | Cart, product |
| `disclosure.js` | Accordion, mobile menu, filters | Global |
| `predictive-search.js` | Fetch and display search suggestions | Header |
| `recently-viewed.js` | localStorage product tracking | Product |
| `animation.js` | Intersection Observer reveals | Optional, respects reduced motion |

### JS conventions

- Custom elements only when justified (Shopify provides `shopify-account`; follow that pattern)
- Event delegation over per-element listeners where possible
- Publish/subscribe via simple custom events for cross-component communication (e.g., `cart:updated`)
- All interactive elements must be keyboard accessible without JS

---

## K. Responsive Strategy

### Layout approach

1. **Mobile-first CSS** — Base styles target mobile; enhance at breakpoints.
2. **Skeleton grid preservation** — The `.shopify-section` content grid handles horizontal containment; components manage their own internal responsive layout.
3. **Editorial breakpoints** — Beauty themes often use generous desktop layouts that collapse to single-column mobile stacks.

### Component responsive patterns

| Component | Mobile | Tablet | Desktop |
|---|---|---|---|
| Header | Hamburger menu, icon bar | Partial inline nav | Full inline nav, mega menu optional |
| Product grid | 2 columns | 3 columns | 3–4 columns (setting) |
| Editorial hero | Stacked text over image | Side-by-side | Full bleed with overlay text |
| Footer | Accordion columns | 2 columns | 4 columns |
| Product media | Swipe carousel | Thumbnails below | Thumbnails left |

### Touch & pointer

- Minimum 44×44px tap targets on interactive elements
- Hover effects (secondary image, underline) must have non-hover equivalents
- `@media (hover: hover)` for hover-only enhancements

### Testing matrix

- iOS Safari, Android Chrome (mobile)
- Safari, Chrome, Firefox (desktop)
- 320px minimum viewport width
- Theme Editor desktop and mobile preview modes

---

## L. Accessibility Strategy

### Current Skeleton baseline

- Semantic HTML partially used (`header`, `footer`, `dialog` styles in critical CSS)
- `role="search"` on search form
- `lang` attribute on `<html>`
- Product structured data in meta-tags
- No skip link, limited focus styles, no aria labels on icon buttons, no live regions

### Cosme accessibility requirements (Theme Store aligned)

| Area | Implementation |
|---|---|
| **Skip link** | First focusable element in `theme.liquid` via `skip-link` snippet |
| **Focus management** | Visible `:focus-visible` styles in critical.css; trap focus in cart drawer/modals |
| **Icon buttons** | `aria-label` on cart, search, menu, close (translated) |
| **Images** | Required alt text; decorative images use `alt=""` |
| **Forms** | Associated `<label>`, `aria-describedby` for errors, `required` attributes |
| **Variant picker** | Radio group semantics or `aria-pressed` buttons; announce changes via `aria-live` |
| **Cart updates** | `aria-live="polite"` region for cart count changes |
| **Motion** | `@media (prefers-reduced-motion: reduce)` disables animations |
| **Color contrast** | Theme settings must not allow WCAG AA failures for text; document safe ranges |
| **Heading hierarchy** | One `<h1>` per page; sections use configurable heading levels defaulting correctly |
| **Keyboard** | All menus, drawers, accordions, carousels fully keyboard operable |
| **Screen readers** | Use `visually-hidden` utility class for supplementary text |

### Testing

- axe DevTools / Lighthouse accessibility audit on all template types
- Manual keyboard-only navigation test
- VoiceOver (macOS/iOS) and NVDA (Windows) spot checks

---

## M. Performance Strategy

### Current Skeleton strengths (preserve)

- Minimal asset payload (one CSS file, no JS)
- Font preconnect + preload for primary font
- `font-display: swap` on all `@font-face` declarations
- `critical.css` preloaded
- Component CSS deduplication via `{% stylesheet %}`
- Native `image_tag` / `image_url` for CDN-optimized images

### Cosme performance targets

| Metric | Target |
|---|---|
| Lighthouse Performance (mobile) | ≥ 60 with realistic demo content (Theme Store minimum) |
| critical.css | < 14KB gzipped |
| Shared JS | < 30KB gzipped total |
| LCP | Optimize hero images with `fetchpriority="high"`, avoid lazy-loading above-fold |
| CLS | Explicit image width/height/aspect-ratio on all media |
| INP | Minimal main-thread work; defer non-critical JS |

### Implementation rules

- Lazy-load below-fold images (`loading="lazy"`)
- Preload hero/LCP image only
- Limit font variants to weights actually used (Skeleton loads 4 variants — audit for Cosme)
- Use Section Rendering API for partial updates instead of full page reloads
- No external render-blocking resources except Shopify CDN fonts
- Avoid massive DOM from mega-carousels; paginate or limit items
- Use `{%- -%}` whitespace control in hot-path Liquid

---

## N. App Compatibility Strategy

### Current state

No `@app` block support. Sections do not expose app block slots.

### Cosme app compatibility plan

1. **Declare app blocks** in marketing and content sections where merchants commonly install apps:
   ```json
   "blocks": [
     { "type": "@theme" },
     { "type": "@app" }
   ]
   ```

2. **Priority sections for `@app` support:**
   - `main-product` (reviews, upsells, subscriptions)
   - `main-collection` (filters, badges)
   - `main-cart` (upsells, shipping)
   - `footer` (chat, trust badges)
   - Homepage marketing sections

3. **Do not override app blocks** with custom implementations of common app categories (reviews, subscriptions) — provide hooks instead.

4. **Shopify app embed blocks** — Ensure `{{ content_for_header }}` remains untouched; app embeds render automatically.

5. **Section Rendering API** — Product and cart sections must use standard Shopify form and DOM hooks so apps can attach.

6. **Avoid z-index wars** — Document z-index scale; cart drawer/modals reserve high values but leave room for app chat widgets.

---

## O. Shopify Theme Store Compliance Considerations

### Requirements checklist (architectural implications)

| Requirement | Cosme approach |
|---|---|
| **All template types** | Add customer templates; ensure gift card works |
| **Theme Editor support** | All sections have schemas, presets, and meaningful settings |
| **Responsive** | Mobile-first, tested on multiple devices |
| **Accessibility** | WCAG 2.1 AA target (Section L) |
| **Performance** | Section M targets |
| **Browser support** | Last 2 versions of major browsers |
| **Translations** | All strings via locale files; English included |
| **Settings defaults** | Professional beauty-industry demo defaults |
| **Demo store** | Requires curated demo content (separate from architecture) |
| **No external dependencies** | No npm packages in theme |
| **License & branding** | Update `theme_info` from Skeleton to Cosme |
| **Contact forms** | Use Shopify form tag or supported pattern |
| **Checkout** | Cannot customize; ensure cart → checkout path works |
| **Search & Discovery** | Support filtering on collection pages |
| **Customer accounts** | Support new Customer Accounts (`shopify-account` — already in header) |
| **Markets / localization** | Use Shopify localization objects; avoid hardcoded currency |
| **Subscriptions / selling plans** | Product form must support selling plan selectors |
| **Pickup availability** | Display in product section when available |
| **Unit pricing** | Render via `unit_price_with_measurement` where applicable |

### Documentation deliverables (later phases)

- Theme documentation site
- Setting guides for beauty merchants
- FAQ for common customizations

---

## P. Things That Must NOT Be Changed

These are non-negotiable Shopify platform contracts:

1. **Directory roles** — `/sections`, `/snippets`, `/blocks`, `/layout`, `/templates`, `/config`, `/locales`, `/assets` serve their platform-defined purposes.

2. **`{{ content_for_header }}` and `{{ content_for_layout }}`** — Required in layouts; never remove or reorder before header injection.

3. **JSON template format** — Templates must remain valid JSON with `sections` and `order` keys.

4. **Section schema structure** — Valid JSON in `{% schema %}` tags; validate against Shopify JSON schemas.

5. **Theme blocks architecture** — Use `/blocks` with `{% content_for 'blocks' %}` / `{% content_for 'block' %}` patterns.

6. **Section groups** — Header/footer via group JSON files and `{% sections 'group-name' %}`.

7. **Shopify Liquid objects and tags** — Use native `form`, `paginate`, `render`, `section`, routes, cart, product objects.

8. **No build-step requirement** — Theme must work when uploaded directly to Shopify without compilation.

9. **Co-located `{% stylesheet %}` and `{% javascript %}`** — Do not move all CSS/JS to monolithic asset files.

10. **Translation filter usage** — All customer-facing strings via `{{ 'key' | t }}`.

11. **App block/extension compatibility** — Never block or hide `content_for_header` app injections.

12. **`settings_data.json` auto-generation** — Do not treat as hand-edited config.

13. **Gift card template** — Must remain Liquid (`gift_card.liquid`).

14. **Checkout** — No checkout.liquid customization (deprecated/unavailable for Theme Store themes).

---

## Q. Recommended Development Order for the Project

This order respects dependencies identified from the Skeleton starting point:

| Step | Phase | Rationale |
|---|---|---|
| 1 | Architecture foundation | Document conventions, finalize naming, update theme_info |
| 2 | Design system | CSS tokens, typography, colors, base components — everything depends on this |
| 3 | Global theme settings | Expose tokens in Theme Editor via settings_schema |
| 4 | Global layout | Extend theme.liquid, skip link, section grid refinements |
| 5 | Header and navigation | Required before any page preview looks coherent |
| 6 | Core snippets | button, price, image, product-card, icon — shared by all page types |
| 7 | Footer | Complete global chrome |
| 8 | Product | Core commerce page; validates design system + JS patterns |
| 9 | Collection | Grid, filters, pagination — second most important commerce page |
| 10 | Cart | Cart page + cart drawer decision |
| 11 | Search | Predictive search integration with header |
| 12 | Homepage | Editorial sections using established blocks/snippets |
| 13 | Blog and content pages | Article, blog, page templates |
| 14 | Customer pages | Required for Theme Store |
| 15 | Utility pages | 404, password, gift card, list-collections |
| 16 | Animation system | After core UX is stable |
| 17 | Responsive refinement | Cross-template QA pass |
| 18 | Accessibility | Audit and remediate |
| 19 | Performance | Optimize fonts, images, JS payload |
| 20 | App compatibility | Add @app blocks, verify popular apps |
| 21 | Testing | Cross-browser, Theme Editor, translation completeness |
| 22 | Theme Store preparation | Demo store, documentation, submission assets |

---

## Development Conventions

Active rules for Cosme development. All contributors and future phases must follow these conventions.

| Convention | Rule |
|---|---|
| **Theme name** | Cosme |
| **CSS namespace** | `cosme-` prefix on custom classes |
| **CSS naming** | BEM-style (e.g. `.cosme-product-card__title`, `.cosme-button--primary`) |
| **Liquid reusable rendering** | Snippets (`{% render 'snippet' %}`) |
| **Merchant-editable components** | Sections and theme blocks |
| **Global sections** | Section groups (`header-group`, `footer-group`) |
| **Templates** | JSON templates remain JSON; `gift_card.liquid` remains Liquid |
| **JavaScript** | Vanilla JS only — no React, Vue, or jQuery |
| **CSS** | No Tailwind or external CSS framework |
| **Build pipeline** | None unless a later decision explicitly requires one |
| **Section files** | Flat in `/sections` — use descriptive file names (e.g. `main-product.liquid`, `hero-editorial.liquid`), not subdirectories |

Platform contracts from section P (e.g. `content_for_header`, `content_for_layout`, translation filters, `@theme` blocks) are mandatory and not overridden by these conventions.

---

## Design System Foundation (Phase 2.1)

Implemented foundation for all future Cosme components. Visual palette uses neutral defaults; token architecture supports a future premium beauty palette without structural changes.

### Settings → CSS variables flow

```
config/settings_schema.json
        ↓
snippets/css-variables.liquid  ({% style %} :root { … })
        ↓
CSS custom properties consumed by critical.css, sections, blocks, snippets
```

Merchant changes in Theme Editor update settings; `css-variables.liquid` re-renders token values on each request.

### Token categories

| Category | CSS variables | Source |
|---|---|---|
| **Typography** | `--font-body--*`, `--font-heading--*`, `--font-size-base`, `--line-height-*`, `--letter-spacing-*` | Settings (fonts, base size) + static defaults |
| **Colors** | `--color-background`, `--color-foreground`, `--color-muted`, `--color-accent`, `--color-accent-contrast`, `--color-border`, `--color-input-background`, `--color-overlay`, `--overlay-opacity`, `--color-sale`, `--color-success`, `--color-error` | Settings |
| **Spacing** | `--spacing-xs` … `--spacing-2xl`, `--spacing-section` | Static scale + section spacing setting |
| **Layout** | `--page-width`, `--page-margin`, `--content-width`, `--grid-gap` | Settings |
| **Components** | `--radius-button`, `--radius-input`, `--radius-card`, `--border-width`, `--focus-ring-width` | Settings |
| **Motion** | `--transition-duration`, `--transition-easing`, `--animation-duration` | Static defaults |

Skeleton legacy aliases preserved: `--font-primary--family`, `--style-border-radius-inputs`.

### critical.css responsibility

Loaded on every page via `layout/theme.liquid`. Contains only:

- CSS reset and document baseline
- Body and heading typography (consumes tokens)
- Link and form control baseline
- `.cosme-visually-hidden`, `:focus-visible`, `prefers-reduced-motion`
- Shopify `.shopify-section` grid architecture

Component styles remain in `{% stylesheet %}` tags within sections, blocks, and snippets.

### Accessibility foundation

- `.cosme-visually-hidden` — screen-reader-only content
- Global `:focus-visible` ring using `--focus-ring-width` and `--color-accent`
- `:focus:not(:focus-visible)` outline removed to avoid mouse-focus rings
- `prefers-reduced-motion: reduce` disables transitions and animations

### Responsive conventions

Mobile-first. Breakpoints are documented in `critical.css` comments and used directly in component stylesheets:

| Name | Width |
|---|---|
| sm | 576px |
| md | 768px |
| lg | 990px |
| xl | 1200px |

Breakpoints are not exposed as CSS custom properties (invalid in `@media` conditions).

### Component snippets (Phase 2.2.1)

Reusable UI primitives live as flat snippets with the `cosme-` prefix (required for LiquidDoc + Theme Check compatibility):

| Snippet | Render tag |
|---|---|
| `snippets/cosme-icon.liquid` | `{% render 'cosme-icon' %}` |
| `snippets/cosme-button.liquid` | `{% render 'cosme-button' %}` |
| `snippets/cosme-image.liquid` | `{% render 'cosme-image' %}` |
| `snippets/cosme-price.liquid` | `{% render 'cosme-price' %}` |
| `snippets/cosme-product-badge.liquid` | `{% render 'cosme-product-badge' %}` |
| `snippets/cosme-form-field.liquid` | `{% render 'cosme-form-field' %}` |

Skeleton `snippets/image.liquid` remains a compatibility wrapper delegating to `cosme-image`.

### Product card (Phase 2.2.2)

`snippets/cosme-product-card.liquid` — reusable grid card composing `cosme-image`, `cosme-price`, and `cosme-product-badge`.

| Parameter | Default | Purpose |
|---|---|---|
| `product` | required | Product object |
| `aspect_ratio` | `3/4` | Portrait editorial ratio via `cosme-image` |
| `image_width` | `800` | CDN max width |
| `sizes` | grid default | Responsive `sizes` attribute |
| `loading` | `lazy` | Overridden to `eager` when `fetchpriority` is set |
| `show_vendor` | `false` | Renders `product.vendor` (stacked above title in default layout; right-aligned in editorial layout) |
| `show_price` | `true` | Renders price via `cosme-price` |
| `show_badge` | `true` | Renders badges via `cosme-product-badge` (sold out → custom → sale) |
| `show_secondary_image` | `false` | CSS crossfade to second image on hover-capable devices |
| `show_quick_view` | `false` | Icon button with `data-quick-view`, `data-product-handle`, `data-product-url`; opens global `cosme-quick-view` dialog |
| `show_add_to_cart` | `false` | Icon submit for single-variant available products; multi-variant products defer to Quick View |
| `layout` | `default` | `default` (stacked) or `editorial` (title/vendor row, price below) |
| `demo_mode` | `false` | Renders placeholder demo card without a Shopify product |
| `demo_title`, `demo_vendor`, `demo_price`, `demo_placeholder` | — | Demo content when `demo_mode` is true |
| `badge_text` | — | Custom badge when product is available |

Structure: sibling links on media and title (no nested anchors). Overlay icon actions on the image; revealed on hover/focus for hover-capable devices, subtly persistent on touch. No JavaScript.

### Global brand settings (Phase 2.3)

Settings-only foundation for upcoming header/footer work. No layout, snippet, or CSS variable changes in this phase.

| Group | Setting ID | Type | Default | Notes |
|---|---|---|---|---|
| **Brand** | `logo` | `image_picker` | — | Optional; theme works without a logo selected |
| | `logo_width` | `range` (50–300px, step 5) | 120px | For header logo rendering (deferred) |
| | `favicon` | `image_picker` | — | Setting only; favicon `<link>` rendering deferred |
| **Social media** | `social_instagram` | `url` | — | Footer/header social links (deferred) |
| | `social_facebook` | `url` | — | |
| | `social_tiktok` | `url` | — | |
| | `social_youtube` | `url` | — | |
| | `social_pinterest` | `url` | — | |
| **Cart** | `cart_type` | `select` (`page` / `drawer`) | `drawer` | Cart drawer implementation deferred |
| **Search** | `enable_predictive_search` | `checkbox` | `true` | Predictive search UI deferred |
| **Animation** | `enable_animations` | `checkbox` | `true` | Scroll/animation JS deferred; `prefers-reduced-motion` unchanged |

Existing typography, layout, color, and component settings preserved unchanged. Labels in `locales/en.default.schema.json`.

---

## Recommended Development Phases

### Phase 1: Architecture foundation

- Finalize this document and team conventions
- Update `theme_info` from Skeleton → Cosme
- Define naming conventions (BEM prefix, file prefixes, locale key hierarchy)
- Set up Theme Check rules and editor config
- Remove/replace demo content (`hello-world`, `shoppy-x-ray.svg`)
- Fix gift card `settings.logo` dependency when logo setting is added

### Phase 2: Design system

- Expand `css-variables.liquid` with full token set
- Extend `critical.css` with focus styles, visually-hidden utility, typography base
- Build core snippets: `button`, `icon`, `price`, `product-badge`, `form-field`
- Extend `image` snippet with lazy loading, aspect ratio, sizes attribute
- Document token → component mapping

### Phase 3: Global theme settings

- Expand `settings_schema.json` with typography, colors, layout, buttons, forms, cart, animation groups
- Wire all settings to CSS variables
- Add logo and favicon settings
- Populate sensible beauty-industry defaults in preset

### Phase 4: Global layout

- Add skip link to `theme.liquid`
- Add `<main id="main-content">` landmark wrapping `content_for_layout`
- Ensure password layout parity
- Refine section grid spacing tokens

### Phase 5: Header and navigation

- Rebuild `header.liquid` with responsive mobile menu
- Integrate search trigger, cart icon with accessible labels
- Support sticky/transparent header modes
- Update `header-group.json` with announcement bar slot
- Preserve `shopify-account` web component integration

### Phase 6: Homepage

- Remove `hello-world` section
- Build editorial sections: hero, featured-collection, image-with-text, multicolumn, newsletter
- Create homepage preset in `index.json`
- Extend theme blocks as needed (heading, button, video)

### Phase 7: Product

- Replace stub `product.liquid` with `main-product` section
- Product media gallery (CSS-first, JS-enhanced)
- Variant picker snippet + JS
- Buy buttons, dynamic checkout, selling plans
- Pickup availability, unit pricing
- Product recommendations section
- Update `product.json` template

### Phase 8: Collection

- Replace stub with `main-collection` section
- Collection banner section
- Filtering (Shopify Search & Discovery compatible markup)
- Sort, pagination snippet
- Responsive product grid with editorial spacing
- Refactor existing `collections.liquid` grid patterns into shared snippets

### Phase 9: Search

- Rebuild search section with accessible form
- Predictive search overlay
- Result type differentiation (product, article, page)

### Phase 10: Cart

- Rebuild cart page section
- Cart drawer section (if global setting = drawer)
- Cart API integration for quantity updates
- Free shipping bar component

### Phase 11: Blog and content pages

- Editorial blog grid and article layouts
- Rich typography for article content
- Comment form accessibility
- Generic page section with optional blocks

### Phase 12: Customer pages

- Create `templates/customers/*.json` and corresponding sections
- Style login, register, account, orders, addresses
- Ensure Customer Accounts compatibility

### Phase 13: Animation system

- CSS transition tokens
- Intersection Observer scroll reveals (optional per section)
- `@media (prefers-reduced-motion: reduce)` overrides globally
- No essential functionality gated behind animation

### Phase 14: Responsive refinement

- Audit all templates at 320px, 768px, 1200px+
- Fix grid breakpoints, typography scaling, touch targets
- Theme Editor mobile preview QA

### Phase 15: Accessibility

- Full keyboard audit
- Screen reader testing
- Color contrast validation across setting ranges
- ARIA live regions for cart and variant updates
- Lighthouse accessibility score remediation

### Phase 16: Performance

- Font weight audit and reduction
- Image sizing audit (no oversized widths)
- JS payload audit
- Lighthouse performance remediation
- Verify critical CSS size budget

### Phase 17: Testing

- Cross-browser testing matrix
- Theme Editor structural testing (add/remove/reorder sections/blocks)
- Translation completeness audit
- Cart/checkout flow end-to-end
- Customer account flows

### Phase 18: Theme Store preparation

- Demo store content curation (beauty editorial)
- Theme documentation
- Marketing assets (screenshots, feature list)
- Final Theme Check / Lighthouse pass
- App compatibility verification with popular beauty apps
- Submit for Shopify Theme Store review

## Implemented Sections

### Collections Editorial (`sections/main-collections-editorial.liquid`)

Premium editorial collection showcase for homepage and marketing templates. Presents Shopify Collections as large image-led editorial panels—not a conventional collection grid.

| Aspect | Detail |
|---|---|
| **Purpose** | Editorial collection discovery with magazine-style layout |
| **Data model** | Section blocks (`type: collection`) backed by Shopify `collection` objects |
| **Section settings** | Color scheme, optional heading, layout (Editorial / Editorial compact), image aspect ratio (Portrait / Landscape / Square / Auto), image fit (Cover / Contain), show card numbers, show navigation, enable carousel, autoplay, autoplay interval (3–8s), show arrows, show pagination |
| **Block settings** | Collection picker, image override, image alt, image focal position, eyebrow, title override, link label override |
| **Fallback / demo** | Preset includes 4 demo blocks (Outdoors, Indoors, Essentials, New Arrivals) with placeholder SVG images when no collection or image is selected |
| **Data priority** | Image: custom → `collection.featured_image` → placeholder; Title: custom → `collection.title`; Link label: custom → generated "Shop {title}"; URL: `collection.url` |
| **Responsive** | Desktop: 2 cards visible; tablet: proportional spacing; mobile: 1 card, touch swipe |
| **Carousel** | Native JS transform track; no external libraries; advances by viewport count (2 desktop / 1 mobile); vertical editorial nav on desktop |
| **Autoplay** | Off when ≤1 card, carousel disabled, reduced motion, or global animations disabled; pauses on hover/interaction; Theme Editor lifecycle cleanup prevents duplicate timers |
| **Color schemes** | Uses `color-{{ section.settings.color_scheme }}` with existing `--color-*` tokens |
| **Accessibility** | Semantic section/article markup, keyboard arrows, aria-live announcements, no nested links, meaningful alt text |
| **Image rendering** | Delegates to `snippets/cosme-image.liquid` |

### Popular Products (`sections/main-popular-products.liquid`)

Premium editorial product showcase for homepage and marketing templates. Presents products in a minimal grid with large square imagery—not a conventional bordered card grid.

| Aspect | Detail |
|---|---|
| **Purpose** | Editorial product discovery with magazine-style spacing and typography |
| **Data source** | Optional `collection` setting; renders up to `products_to_show` products from the selected collection |
| **Section settings** | Eyebrow, heading, collection, products to show (4–16), desktop columns (3/4/5), mobile columns (1/2), color scheme, show navigation, show vendor, show price, show badge, show secondary image, show quick view, show add to cart |
| **Demo fallback** | When no collection is selected or the collection has no products, renders 8 demo cards with placeholder SVG imagery and fixed demo titles/vendors/prices |
| **Product card** | Delegates to `snippets/cosme-product-card.liquid` with `layout: 'editorial'` and interaction params; demo cards use the same component via `demo_mode` |
| **Navigation** | Previous \| Next buttons in header; desktop/tablet paginate one row at a time (by column count); mobile shows full grid without pagination; hidden when only one page or navigation disabled |
| **Progressive enhancement** | All products render in the DOM as a normal grid; JavaScript adds pagination via `hidden` on non-visible items; without JavaScript all products remain visible and navigation stays hidden |
| **No autoplay** | Manual Previous/Next only; no automatic scrolling |
| **Quick View** | Opens global `snippets/cosme-quick-view.liquid` dialog from card triggers |
| **Color schemes** | Uses `color-{{ section.settings.color_scheme }}` with existing `--color-*` tokens |
| **Responsive** | Desktop: 3–5 columns with row pagination; tablet: up to 3 columns with pagination; mobile: 1–2 column grid, all products visible |
| **Accessibility** | Semantic section/header markup, real navigation buttons with disabled states, `aria-live` page announcements, product links remain in DOM |
| **Theme Editor** | Re-inits on `shopify:section:load`; cleans up on `shopify:section:unload` |
| **Animation** | Respects `settings.enable_animations` and `prefers-reduced-motion`; no slide/autoplay transitions |

### Quick View (`snippets/cosme-quick-view.liquid`)

Global reusable Quick View dialog rendered once in `layout/theme.liquid`. Opens from `[data-quick-view]` triggers prepared on `cosme-product-card`.

| Aspect | Detail |
|---|---|
| **Trigger** | `[data-quick-view]` button with `data-product-handle` and `data-product-url` on product cards |
| **Data loading** | Fetches `/products/{handle}.js` on first open; caches product JSON in memory for subsequent opens |
| **Modal pattern** | Native `<dialog>` with backdrop click, Escape, focus return to trigger, and scroll lock via `showModal()` |
| **Content** | Image, title, vendor, price (with compare-at), badges, description excerpt, variant selectors (multi-variant only), quantity control, Add to Cart, View product link |
| **Variants** | Option `<select>` elements; unavailable combinations disabled; Add to Cart disabled for invalid/unavailable variants |
| **Add to Cart** | POST `/cart/add.js`; updates header cart count and dispatches existing `cart:refresh` event for cart drawer refresh |
| **Feedback** | `aria-live` region for success/error; subtle button success state; duplicate submission prevented while request is in flight |
| **Animation** | Respects `settings.enable_animations` and `prefers-reduced-motion` |
| **Color schemes** | Uses `color-{{ settings.color_scheme }}` with existing `--color-*` tokens |
| **Icons** | Cloned from Liquid `<template>` tags wrapping `cosme-icon` (close, plus, minus, chevron-down) |

---

*Document version: 1.0*
*Created: August 2026*
*Base theme: Shopify Skeleton v0.1.0*
