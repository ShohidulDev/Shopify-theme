# CLI workflow: init → dev → push → publish

Based on Shopify's official "Create a theme" tutorial. This is the standard path from an empty folder to a live theme.

## Requirements before starting

- [Shopify CLI](https://shopify.dev/docs/api/shopify-cli) installed.
- A store to work against — a free [development store](https://shopify.dev/docs/storefronts/themes/tools/development-stores) is recommended for building/testing rather than a live store.
- The store's URL (e.g. `example.myshopify.com`).
- A collaborator account, staff account with **Manage themes**/**Themes** permission, or store-owner access on that store. (If they created the dev store, they're already the owner; anyone else needs to be added as staff.)

## Step 1 (optional): Shopify AI Toolkit

Connects an AI coding assistant to Shopify's docs/APIs/CLI context. Only relevant if the user is setting up their own editor tooling — for Claude Code specifically:
```
claude plugin install shopify-ai-toolkit@claude-plugins-official
```
Not required for anything else in this workflow — skip unless asked about it.

## Step 2: Initialize a new theme

```
shopify theme init
```
Prompts for a theme name and clones the [Skeleton reference theme](https://github.com/shopify/skeleton-theme) — a minimal, best-practices-structured starting point — into a folder of that name. Then:
```
cd "<theme-name>"
```
To start from a different theme instead of Skeleton (e.g. Dawn, or a client's existing theme repo), `theme init` also accepts a Git repo URL/path.

## Step 3: Local development server

```
shopify theme dev --store <store>.myshopify.com
```
- `--store` is only required the first time (or to switch stores); after that it remembers the connection. Check the current connection with `shopify theme info`.
- First run prompts a login.
- Uploads the theme as a **development theme** on the store and returns a URL that hot-reloads CSS/section changes in real time against live store data.
- Open the preview at `http://127.0.0.1:9292` — **Google Chrome only** for the hot-reload preview.
- Also generates a shareable preview link and a link straight into the theme editor for that development theme, useful for showing work-in-progress to a client without publishing anything.

## Step 4: Push to the store

Once ready to persist changes beyond the local dev session (share a permanent link, update an existing uploaded theme, or prepare to publish):

First push, as a new unpublished theme:
```
shopify theme push --unpublished
```
Prompts for a name for the theme as it'll appear in the theme library. Subsequent pushes to that same theme:
```
shopify theme push
```

## Step 5: Publish

Only when the theme should go live on the storefront:
```
shopify theme publish
```
Prompts to pick which theme (by name) to publish and confirm. Push all local changes with `theme push` first — publish just switches which already-uploaded theme is active, it doesn't upload anything itself.

## Before you publish (production checklist)

Run through this before making a theme live, especially for client/production work:

1. **Lint with theme check**: `shopify theme check` (bundled with Shopify CLI). Catches Liquid/JSON syntax errors, deprecated tags/filters, undefined objects, missing default locale, unmatched translation keys, missing `content_for_*` in `theme.liquid`, and some performance smells (parser-blocking JS, missing `width`/`height` on `img`, excessive JS/CSS). Fix everything it flags before shipping — these are the same checks Shopify runs for Theme Store submissions.
2. **Performance**: Shopify's Theme Store minimum bar is an average Lighthouse performance score of 60 across home/product/collection pages — a reasonable target even outside the Theme Store. Minimize JS, lean on native browser features, and avoid parser-blocking scripts.
3. **Editor experience**: click through adding/removing/reordering every section and block you built in the theme editor itself, not just the storefront preview — this is the easiest thing to skip and the easiest thing to accidentally break (missing `shopify_attributes`, JS that doesn't re-run after an editor AJAX swap).
4. **Translations**: if the theme supports more than one storefront language, make sure `locales/` has a complete default file and that new hardcoded strings you added actually go through `{{ 'key' | t }}` rather than being left as raw English.
5. **Test on a development theme first**: use `theme push --unpublished` (or the `theme dev` preview link) to get real client/stakeholder sign-off before `theme publish` makes it live — don't push straight to the live theme for anything beyond trivial fixes.
6. **Version control**: for anything beyond a one-off tweak, keep the theme's source in Git rather than treating the Shopify theme library as the source of truth — `theme push`/`theme pull` sync between the local Git-tracked folder and Shopify, and this makes rollbacks and collaboration far less painful than editing directly on the store.

## Useful follow-up commands

| Command | Purpose |
|---|---|
| `shopify theme info` | Which store/theme you're currently connected to |
| `shopify theme check` | Lint the theme for errors, deprecations, perf issues |
| `shopify theme pull` | Download a theme's current code from Shopify to local |
| `shopify theme list` | List themes on the connected store |
| `shopify theme delete` | Remove a theme from the store |

If the user hits a `theme check` error or CLI error message you're not certain about, it's better to say so and suggest checking the exact message against Shopify's docs than to guess at a fix — theme-check rules and CLI flags change between CLI versions.
