---
name: shopify-theme-dev
description: >-
  Build, edit, and ship Shopify themes end-to-end — from `shopify theme init` through local preview, writing Liquid sections/blocks/snippets with proper schema, and pushing/publishing to a live store. Use this skill for ANY Shopify theme or storefront work, even if the user doesn't say the word "skill" or "theme" — creating a new theme, editing an existing theme's Liquid/JSON files, adding or fixing a section, block, snippet, or template, working with `settings_schema.json` or `config/settings_data.json`, theme performance/accessibility questions, `shopify theme dev/push/publish` commands, theme-check errors, or preparing a theme for the Shopify Theme Store. Trigger proactively whenever Liquid, `.liquid` files, section schema, theme editor, Dawn/Skeleton theme, or a Shopify store URL (`*.myshopify.com`) come up.
---

# Shopify theme development

This skill covers the full lifecycle of a Shopify theme: scaffolding it, building out its Liquid/JSON architecture correctly, previewing it locally, and getting it live on a store safely. It's based on Shopify's own docs (Online Store 2.0 architecture), so follow the patterns here rather than older Shopify tutorials — a lot of theme advice online predates sections-everywhere/JSON templates and will lead you to build things the hard way.

Bengali/Banglish is fine for explanations to the user; keep code, commands, file paths, and JSON/Liquid keys exact as written here — Shopify is picky about directory names and schema field names.

## How to work through a theme request

1. **Figure out where the user is in the lifecycle** — new theme vs. editing an existing one vs. deploying/troubleshooting. Don't re-explain setup to someone who's mid-build.
2. **If scaffolding a new theme or explaining setup/CLI**, read `references/cli-and-deployment.md`.
3. **If writing or editing Liquid files** (sections, blocks, snippets, templates, layout), read `references/architecture.md` first for where a file belongs and how the pieces connect, then `references/sections-and-blocks.md` for schema syntax and section/block conventions before writing code.
4. **If the user shares files or a repo**, inspect the actual directory structure and existing schema patterns in their theme before adding new code — match their conventions (naming, settings, CSS approach) rather than imposing a different style.
5. **Before calling something done**, run through the production checklist in `references/cli-and-deployment.md#before-you-publish` and mention anything the user should still check (theme check, performance, translations).

## Core mental model (read before writing any file)

A theme page is assembled from, top to bottom: **layout → template → section groups/sections → blocks → snippets**. Getting confused about which of these a piece of functionality belongs in is the most common mistake, so before writing code, decide:

- Does this need to render on *every* page (header, footer, `<head>` content)? → `layout/theme.liquid`.
- Is this a whole page's content area, and should merchants be able to add/remove/reorder pieces of it in the theme editor? → a JSON **template** made of **sections**.
- Is this a self-contained, reusable content module (hero banner, image-with-text, testimonials)? → a **section** in `sections/`.
- Does the section need repeatable, reorderable sub-items (slides, FAQ items, testimonial cards)? → **blocks** inside that section's schema, or a reusable **theme block** in `blocks/` if it needs to work across multiple sections.
- Is this just reusable Liquid/HTML you don't want to copy-paste (a price display, a product card), with no direct editor UI of its own? → a **snippet** in `snippets/`, rendered with `{% render %}`.

Required directory structure (subdirectories beyond these aren't supported):

```
.
├── assets
├── blocks
├── config
├── layout       ← only layout/theme.liquid is actually required to upload
├── locales
├── sections
├── snippets
└── templates
    ├── customers
    └── metaobject
```

Two things that trip people up constantly:
- **JSON templates are wrappers, not code.** `templates/index.json` just lists which sections render in what order and with what settings — the actual markup lives in the section's `.liquid` file. If someone asks you to "edit the homepage template," you almost always want the *section* being rendered, not the JSON file.
- **`templates/*.json`, `sections/*.json` (section groups), `config/settings_data.json`, and `locales/*.json` don't preserve comments/trailing commas** even though theme JSON generally supports them (e.g. `config/settings_schema.json` does). Don't add comments to the ones that will silently strip them.

## Writing theme code — conventions to follow

- **Liquid + minimal JS.** Prefer server-rendered Liquid and modern native browser features (native lazy-loading, `<details>`, CSS instead of JS where possible) over heavy JavaScript — this is Shopify's own stated performance principle and matters for Theme Store acceptance (min. Lighthouse performance score of 60).
- **Every section needs a `{% schema %}` tag** at the bottom defining at least `name`; add `settings` for anything a merchant should be able to customize without editing code, and `blocks`/`presets` as needed. See `references/sections-and-blocks.md` for the full field list and examples.
- **Reference assets with the `asset_url` filter**, never a hardcoded path, so Shopify's CDN serves them correctly.
- **Give repeated block markup the `{{ block.shopify_attributes }}` attribute** on its container element — the theme editor needs this to let merchants select/highlight blocks in the visual preview. Skipping it silently breaks editor UX.
- **Don't rely on a block's literal `id`** (e.g. `block.id == 'J6d9jV'`) — it's dynamically generated and can change.
- **Static sections** (ones hardcoded into a layout/template rather than added via a JSON template or section group) can't be removed or reordered by merchants — only use `{% section %}` directly in Liquid for things that must always be present; use JSON templates/section groups for anything merchants should control.
- When in doubt about a specific object, tag, or filter's exact behavior, it's fine to note you're going by the documented pattern rather than guessing — Liquid has a lot of edge cases (whitespace control, object availability per template type) that are easy to get subtly wrong.

## Quick command reference

| Task | Command |
|---|---|
| Scaffold a new theme (clones the Skeleton reference theme) | `shopify theme init` |
| Preview locally with hot reload (Chrome only) | `shopify theme dev --store <store>.myshopify.com` |
| Lint Liquid/JSON for errors and deprecated patterns | `shopify theme check` |
| Upload as a new, unpublished theme | `shopify theme push --unpublished` |
| Push updates to an existing theme | `shopify theme push` |
| Make a theme live | `shopify theme publish` |
| See which store you're connected to | `shopify theme info` |

Full walkthroughs, flags, and the pre-launch checklist are in `references/cli-and-deployment.md`.
