# shopify-theme-dev

A Claude Skill that teaches [Claude](https://claude.ai) how to build, edit, and ship Shopify themes end-to-end — from `shopify theme init` through local preview, writing Liquid sections/blocks/snippets with proper schema, to pushing and publishing on a live store.

It's based on Shopify's own Online Store 2.0 documentation, so it follows current best practices (JSON templates, section groups, theme blocks) rather than older, pre-2.0 theme patterns still floating around in a lot of tutorials.

## What this skill does

When active, Claude will:

- Scaffold a new theme with `shopify theme init` and walk through local dev setup.
- Know when something should be a **section**, a **block** (theme/section/app), or a **snippet**, and where each file belongs in the theme directory structure.
- Write correct `{% schema %}` blocks — settings, blocks, presets, `max_blocks`, `enabled_on`/`disabled_on` — and follow conventions that keep the theme editor working properly (e.g. `block.shopify_attributes`, never matching on a block's literal `id`).
- Match an existing theme's conventions when editing a repo you already have, instead of imposing a different style.
- Walk through the push/publish flow and a pre-launch checklist (theme check, performance, editor testing, translations, version control) before anything goes live.

## Structure

```
shopify-theme-dev/
├── SKILL.md                          # entry point: workflow, decision logic, command reference
└── references/
    ├── architecture.md               # directory structure, how layout/template/section/block/snippet fit together
    ├── sections-and-blocks.md        # {% schema %} fields, section/block code examples, editor integration
    └── cli-and-deployment.md         # init/dev/push/publish walkthrough + pre-publish checklist
```

`SKILL.md` stays short and points into `references/` for the deep-dive material, so Claude only loads what a given task actually needs.

## Installing

**Claude.ai / Claude Code / Cowork**
1. Download [`shopify-theme-dev.skill`](./shopify-theme-dev.skill) from this repo (or clone the repo and zip the `shopify-theme-dev/` folder yourself — a `.skill` file is just a zip).
2. In Claude, open the file and click **Save skill** (where your org allows skill installs), or drop it into your skills directory if you're running Claude Code locally.

**Manual / any Claude surface with a skills folder**
Copy the `shopify-theme-dev/` folder into wherever your setup loads skills from (for Claude Code, typically `~/.claude/skills/` or your project's `.claude/skills/`).

## Using it

Once installed, you don't need to invoke it by name — it triggers automatically on Shopify theme work: creating a theme, editing Liquid/JSON files, building sections/blocks/snippets, `settings_schema.json` questions, `shopify theme dev/push/publish`, theme-check errors, performance/Theme Store questions, etc. You can also name it directly, e.g. "using the shopify-theme-dev skill, add a testimonials section with blocks."

## Source material

Built from Shopify's official docs:
- [Create a theme](https://shopify.dev/docs/storefronts/themes/getting-started/create)
- [Theme architecture](https://shopify.dev/docs/storefronts/themes/architecture)
- [Sections](https://shopify.dev/docs/storefronts/themes/architecture/sections) / [Section schema](https://shopify.dev/docs/storefronts/themes/architecture/sections/section-schema) / [Blocks](https://shopify.dev/docs/storefronts/themes/architecture/blocks)
- [Best practices for building Shopify themes](https://shopify.dev/docs/storefronts/themes/best-practices)
- [Theme Check](https://github.com/Shopify/theme-check)

Shopify's docs move — if something here looks stale against current `shopify.dev`, that's worth an issue/PR.

## Contributing

PRs welcome — especially real-world section/block patterns, corrections, or additions covering areas not in here yet (metaobjects, customer account templates, app block edge cases, etc.). Keep `SKILL.md` itself lean; put deep material in `references/`.

## License

MIT — use it, fork it, adapt it for your own theme workflow.
