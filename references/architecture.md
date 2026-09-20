# Theme architecture reference

Based on Shopify's Online Store 2.0 theme architecture. Read this before deciding where a new piece of theme functionality should live.

## Directory structure

Only these top-level directories are supported (no other subdirectories are recognized):

```
.
├── assets      - images, CSS, JS, and other static files
├── blocks      - reusable theme blocks (.liquid), usable across multiple sections
├── config      - settings_schema.json, settings_data.json
├── layout      - theme.liquid (required — this is the only file Shopify actually requires to upload a theme)
├── locales     - translation files (en.default.json, etc.)
├── sections    - section .liquid files, and section group .json files
├── snippets    - reusable Liquid partials, rendered with {% render %}
└── templates
    ├── customers   - legacy customer account page templates
    └── metaobject   - metaobject templates
```

To see a real, complete example, the [Dawn theme repo](https://github.com/Shopify/dawn) is Shopify's own reference implementation and follows every best practice described here.

## How a page is assembled

`layout` → `template` → `section groups` (in header/footer areas) → `sections` → `blocks` → `snippets`, roughly outside-in:

1. **Layout** (`layout/theme.liquid`) is the outermost wrapper — `<html>`, `<head>`, and anything repeated on every page (header, footer, cart drawer, etc). Templates render *inside* the layout via `{{ content_for_layout }}`.
2. **Template** (`templates/*.json` or `templates/*.liquid`) controls what shows on a given page type (home, product, collection, etc.). **JSON templates are just a wrapper listing sections and their settings/order — they contain no markup themselves.** Liquid templates (older style) contain actual code directly. Default Shopify guidance and Theme Store requirements favor JSON templates because they let merchants add/remove/reorder sections in the editor.
   - No templates are required by default, but you need a matching template for any page type you want to render (e.g. a `product` template to render product pages).
   - You can create multiple templates of the same resource type ("alternate templates") to offer merchants different layouts for different products/collections/pages.
3. **Section groups** (JSON files, typically in `sections/`, e.g. `header-group.json`) are containers that let merchants add/remove/reorder *sections* within a specific area of the layout — most commonly the header and footer.
4. **Sections** (`sections/*.liquid`) are the reusable, customizable content modules — hero banners, image-with-text, featured collection, etc. A section can be:
   - Dynamically included via a JSON template or section group (merchant can add/remove/reorder it), or
   - **Statically** included directly in Liquid (`{% section 'name' %}`) in a layout/template — static sections **cannot** be removed or reordered by the merchant, so only do this for things that must always exist.
   - A JSON template/section group can render up to 25 sections, and each section can have up to 50 blocks.
5. **Blocks** are reusable, reorderable sub-items *within* a section (a slide in a slideshow, an FAQ entry, a testimonial card). Three kinds:
   - **Theme blocks** — their own `.liquid` files in `/blocks`, reusable across multiple sections in the theme. Give sections access to them via `blocks: [{"type": "@theme"}]` (accept all) or by naming specific block types (accept specific) in the section schema.
   - **Section blocks** — defined inline inside a single section's `.liquid` file; only usable in that section.
   - **App blocks** — provided by installed apps; sections that support them let merchants add app content without you writing code.
6. **Snippets** (`snippets/*.liquid`) are small reusable pieces of Liquid with **no direct editor UI** — invisible to merchants in the theme editor. Render with `{% render 'snippet-name' %}`. Good for things like a price display or product card markup you reuse in several sections. You can add [LiquidDoc](https://shopify.dev/docs/storefronts/themes/tools/liquid-doc) comments to snippets for better editor tooling support (e.g. in the Shopify VS Code extension).

## `config/`

- `settings_schema.json` defines the **global theme settings** shown in the theme editor's "Theme settings" panel (fonts, colors, spacing, etc.), accessible in Liquid via the `settings` object.
- `settings_data.json` stores the actual current values a merchant has set — you don't usually hand-edit this.
- Section- and block-level settings are defined per-section/block in their own schema instead (see `sections-and-blocks.md`), not here.

## `locales/`

Translation files for theme editor labels, storefront text, and merchant-customizable strings. Needed if you want the theme to support more than one language, and Shopify checks for a missing default locale file as a lint error.

## `assets/`

Images, CSS, and JS. Always reference these with the `asset_url` Liquid filter rather than a hardcoded path, so they're served correctly from Shopify's CDN, e.g.:

```liquid
<link rel="stylesheet" href="{{ 'theme.css' | asset_url }}">
```

You can give a non-binary asset file limited Liquid access by appending `.liquid` to its extension (`theme.css.liquid`, `cart.js.liquid`) — these get access to the `settings` object and Liquid filters, useful for e.g. injecting theme color settings into a CSS file.

## JSON quirks worth knowing

Most theme JSON files support comments (`/* ... */`) and trailing commas — but **not** these, which strip both silently:
- `templates/*.json`
- `sections/*.json` (section groups)
- `config/settings_data.json`
- `locales/*.json`

`config/settings_schema.json` and inline `{% schema %}` tags in sections/blocks *do* support comments — useful for leaving context for other developers, but don't rely on comments surviving in the files listed above.
