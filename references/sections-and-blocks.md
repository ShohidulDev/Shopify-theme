# Sections, blocks, and schema reference

## When to use a section vs. a block

Ask: does this piece of content need to be added/removed/reordered **at the template/section-group level** (i.e. merchants can move it relative to *other whole sections*)? → it's a **section**. Does it need to be added/removed/reordered **within** one section, alongside similar sibling items? → it's a **block**.

A section should generally:
- Provide its own default content so a template still looks reasonable with nothing customized.
- Control settings scoped to its own layout/content (e.g. "show/hide subtitle", "columns on desktop").
- Not assume it will always be paired with specific other sections — merchants can freely rearrange, remove, or duplicate sections.

## Basic section anatomy

```liquid
<div class="hero-banner {{ section.settings.color_scheme }}">
  <h2>{{ section.settings.heading }}</h2>
  {% if section.settings.subheading != blank %}
    <p>{{ section.settings.subheading }}</p>
  {% endif %}

  {% for block in section.blocks %}
    {% case block.type %}
      {% when 'button' %}
        <a href="{{ block.settings.link }}" {{ block.shopify_attributes }}>
          {{ block.settings.label }}
        </a>
    {% endcase %}
  {% endfor %}
</div>

{% schema %}
{
  "name": "Hero banner",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "Welcome"
    },
    {
      "type": "text",
      "id": "subheading",
      "label": "Subheading"
    }
  ],
  "blocks": [
    {
      "type": "button",
      "name": "Button",
      "settings": [
        { "type": "text", "id": "label", "label": "Button label" },
        { "type": "url", "id": "link", "label": "Button link" }
      ]
    }
  ],
  "max_blocks": 4,
  "presets": [
    { "name": "Hero banner" }
  ]
}
{% endschema %}
```

## `{% schema %}` fields

| Field | Purpose |
|---|---|
| `name` | Display name in the theme editor. Required. |
| `tag` | HTML wrapper tag rendered around the section (defaults to `div`). |
| `class` | CSS class added to that wrapper. |
| `limit` | Max number of times this section can be added to one template/group. |
| `settings` | Array of section-level setting objects (see setting types below). |
| `blocks` | Array of block type definitions this section accepts, or `[{"type": "@theme"}]` to accept all theme blocks. |
| `max_blocks` | Cap on total blocks a merchant can add. |
| `presets` | Default configurations merchants pick from when adding this section via "Add section". Needed for the section to show up in that picker. |
| `default` | Default settings/blocks used when a section is added without a chosen preset (mainly relevant for section groups). |
| `locales` | Section-scoped translation strings. |
| `enabled_on` / `disabled_on` | Restrict which template types or section groups can use this section. |

Common setting `type` values you'll use constantly: `text`, `textarea`, `richtext`, `image_picker`, `url`, `select`, `checkbox`, `range`, `color`, `color_scheme`, `product`, `collection`, `blog`, `page`, `link_list`, `font_picker`, `video`, `html`. Each needs a unique `id` (used as the settings key) and a `label` (shown in the editor).

## Blocks in depth

- **Section blocks**: defined inline in the section's own `blocks` schema array (as in the example above) — simplest option, scoped to that one section.
- **Theme blocks**: separate `.liquid` files under `/blocks`, each with their own `{% schema %}`, reusable across many sections. A section opts in via its schema:
  ```json
  { "blocks": [{ "type": "@theme" }] }
  ```
  to accept *all* theme blocks, or by listing specific block type names to restrict the picker to just those.
- **App blocks**: contributed by installed apps; supported automatically by any section using JSON templates/section groups — you don't build these yourself, just don't block them.

**Always add `{{ block.shopify_attributes }}` to a block's outer element.** The theme editor's JavaScript uses this attribute to know which DOM element corresponds to which block, so merchants can click to select/highlight it in the live preview. Forgetting this doesn't cause a visible bug in your own testing but breaks the editing experience.

**Never match on a block's literal `id`** — it's generated dynamically and can change between saves. Iterate with `{% for block in section.blocks %}` and branch on `block.type`, not `block.id`, unless you're intentionally working with a specific instance the merchant picked (e.g. via a setting).

## Theme editor & preview integration

- Sections/blocks should update live in the theme editor without a full page reload when merchants add/remove/reorder/select them — this generally falls out naturally from following the patterns above (schema-driven settings, `shopify_attributes` on blocks), but if you're adding custom JS behavior, make sure it re-initializes correctly when the editor swaps section HTML in via AJAX (listen for `shopify:section:load` / `shopify:block:select` etc. rather than assuming your JS only runs once on page load).
- You can detect editor/preview mode in Liquid via `request.design_mode` if you need to change behavior while being edited (e.g. disabling a slideshow's autoplay while a merchant is actively editing it).

## Sections best practices (summary)

- These conventions assume an Online Store 2.0 theme (JSON templates + section groups) — don't apply them if you're stuck maintaining a pre-2.0 theme.
- Sections are available on every page by default; scope them with `enabled_on`/`disabled_on` if a section only makes sense on certain templates.
- Favor giving merchants real settings over hardcoding content, but don't over-fragment either — group closely related content into one section with blocks rather than many tiny single-purpose sections, unless the pieces genuinely need independent reordering.
