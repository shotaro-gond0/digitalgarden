<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="img/digital-garden-dark.svg">
    <img src="img/digital-garden.svg" alt="Digital Garden logo" width="128" height="128">
  </picture>
</p>

# Digital Obsidian Garden

## Changes in this fork

This repository is a fork of the official Public Template ([oleeskild/digitalgarden](https://github.com/oleeskild/digitalgarden)) with a few CSS/build customizations layered on top for a specific personal vault setup.

### Fonts

- Embedded the monospace font **PlemolJPConsole NF** (SIL OFL 1.1) via `@font-face` in [`src/site/styles/custom-style.scss`](src/site/styles/custom-style.scss) and applied it to code blocks (`pre`, `code[class*="language-"]`). Its half-width:full-width = 1:2 monospace ratio keeps box-drawing characters and ASCII art aligned in code blocks that mix Japanese text with ASCII.
- Also disabled line wrapping via `white-space: pre` / `word-wrap: normal`, letting overflow scroll horizontally instead, to prevent ASCII diagrams from breaking their layout.
- The font file itself lives at `src/site/styles/fonts/PlemolJPConsoleNF-Light.woff2` (plus its license) and is served to the browser via `eleventyConfig.addPassthroughCopy("src/site/styles/fonts");` in [`.eleventy.js`](.eleventy.js) — see the caution below, this exact line has a history of getting silently dropped by template updates.

### Colors

- Added [`src/site/styles/user/claude-diagrams.css`](src/site/styles/user/claude-diagrams.css), which reproduces the color system needed to correctly render (light/dark aware) raw SVGs produced by Claude's "Imagine" diagramming tool (`visualize:show_widget`) in Obsidian's Reading view. The exported SVGs depend on CSS classes/variables that only exist on `claude.ai` (`c-*`, `node`, `box`, `--color-*`, etc.), which are undefined here and render everything as solid black by default. Because double-indirected `var()` references don't resolve inside embedded SVGs, this file redefines text, borders, arrows, and the color ramps (purple/teal/coral/pink/gray/blue/green/amber/red) using literal hex colors instead.

### ⚠️ Caution when applying a template update (the "Update to X.X.X" button in Site Template)

The `Update to X.X.X` button in the Digital Garden plugin's settings screen, under the "Site Template" section, is **not a 3-way merge** — it auto-generates a branch/PR that overwrites target files wholesale with the new template version's contents. A customized file caught in that overwrite doesn't show up as a git merge conflict; it just silently loses the customization, invisible until you look at the diff. **Before merging such a PR:**

1. Check the PR's changed-file list:

   ```bash
   gh pr view <PR#> --repo shotaro-gond0/digitalgarden --json files --jq '.files[].path'
   ```

   and look for any of:
   - `.eleventy.js`
   - `src/site/styles/custom-style.scss`
   - `src/site/styles/user/claude-diagrams.css`
   - `README.md` — **this section itself is not exempt.** It was wiped out wholesale by the v1.91.0 template update (upstream's own README.md just overwrites this file) and had to be rewritten from git history afterward.
2. For `.eleventy.js` / `custom-style.scss` / `claude-diagrams.css`: if none of these three appear, the customizations are unaffected — merge as usual. If any do appear, check out the update branch locally and diff each flagged file against `main` (`git diff main -- <file>`), and confirm the pieces described in "Fonts" and "Colors" above are still present.
3. For `README.md`: a full diff isn't useful here, since upstream legitimately rewrites large parts of this file on every update. Instead, just confirm the `## Changes in this fork` heading (this whole section, from here through the end of this caution block) is still present near the top of the branch's `README.md`. If it's gone, don't try to patch in the missing lines — re-add the entire section verbatim (copy it from `main` before the update, e.g. `git show main:README.md`, or from this file's own git history) after the merge overwrites it.
4. If something's missing, restore it directly in the branch, commit (e.g. `fix: restore <what> removed by template update`), push, then merge.
5. After merging, a green Netlify build does **not** prove nothing regressed — a missing font or broken SVG colors doesn't fail the build, it just silently renders wrong (default font instead of the monospace one, or solid-black diagrams). Always open the deployed site afterward and visually confirm code-block box-drawing/ASCII alignment and Claude Imagine diagram colors in both light and dark mode.

This exact regression — the `eleventyConfig.addPassthroughCopy("src/site/styles/fonts");` line in `.eleventy.js` silently dropped by the template overwrite — has already recurred **three times** (v1.81.2, v1.83.4, v1.91.0), each requiring a manual restore commit after the fact.

---

This is the template to be used together with the [Digital Garden Obsidian Plugin](https://github.com/oleeskild/Obsidian-Digital-Garden).
See the README in the plugin repo for information on how to set it up.

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/oleeskild/digitalgarden)

---
## Docs
Docs are available at [docs.forestry.md](https://docs.forestry.md/)

---
## Plugins

The garden is extensible through plugins: directories under `src/plugins/`
that add markup to layout slots, site-wide styles and scripts, and
build-time Eleventy/markdown-it hooks. Core features like search
(`dg-search`), link previews (`dg-link-preview`), timestamps
(`dg-timestamps`), and math (`dg-math`) are themselves plugins built on
this API — `dg-link-preview` is the smallest one to read first.

- Docs (installing plugins, writing your own): [docs.forestry.md](https://docs.forestry.md/)
- Reference code: the first-party plugins under [`src/plugins/`](src/plugins/)
- Building plugins with an AI agent: this repo ships a
  [`garden-plugin-author` skill](skills/garden-plugin-author/SKILL.md) in
  the open [Agent Skills](https://skills.sh) format, teaching agents how
  to create, test, and publish garden plugins. Install it into any
  harness (Claude Code, Cursor, Codex, …) with:

  ```sh
  npx skills add oleeskild/digitalgarden
  ```

To try a third-party plugin manually, drop its directory into
`src/plugins/` — a valid `garden-plugin.json` is all it takes. Disable any
plugin via `src/plugins/plugins.json` (`{"plugins": {"dg-search": {"enabled": false}}}`).
Only install plugins from authors you trust: plugin code runs in your site
build and in your visitors' browsers.

---
## CSS Variables

The digital garden is fully customizable through CSS variables. Override these in `src/site/styles/custom-style.scss` to customize your garden's appearance.

### How to Customize

Add your overrides to `custom-style.scss`:

```scss
body {
    --dg-content-max-width: 800px;
    --dg-content-font-size: 16px;
    --dg-sidebar-max-width: 400px;
}
```

### Responsive Layout Notes

- Content will never overlap the filetree, regardless of `--dg-content-max-width` value
- The right sidebar (TOC/graph/backlinks) automatically hides when there isn't enough viewport space
- To make the sidebar appear at smaller viewports, reduce `--dg-sidebar-max-width`

### Available Variables

#### Color Variables
You can override the base Obsidian theme color variables directly:

| Variable | Description |
|----------|-------------|
| `--text-normal` | Normal text color |
| `--text-muted` | Muted/secondary text |
| `--text-faint` | Faint text |
| `--text-accent` | Accent color |
| `--text-accent-hover` | Accent hover color |
| `--link-color` | Link color |
| `--link-color-hover` | Link color hover |
| `--link-unresolved-color` | Link color unresolved |
| `--link-unresolved-opacity` | Link color unresolved opacity |
| `--background-primary` | Primary background |
| `--background-primary-alt` | Alt primary background |
| `--background-secondary` | Secondary background |
| `--background-secondary-alt` | Alt secondary background |
| `--interactive-normal` | Interactive element color |
| `--interactive-hover` | Interactive hover color |
| `--interactive-accent` | Interactive accent |
| `--interactive-accent-hover` | Interactive accent hover |

#### Layout Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-content-max-width` | `700px` | Maximum width of content area |
| `--dg-content-margin-top` | `90px` | Top margin for content |
| `--dg-content-margin-top-mobile` | `75px` | Top margin on mobile |
| `--dg-content-font-size` | `18px` | Base font size for content |
| `--dg-content-line-height` | `1.5` | Line height for content |

#### Sidebar (Right) Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-sidebar-top` | `75px` | Sidebar top offset |
| `--dg-sidebar-gap` | `80px` | Gap between content and sidebar |
| `--dg-sidebar-min-width` | `25px` | Minimum sidebar width |
| `--dg-sidebar-max-width` | `350px` | Maximum sidebar width |
| `--dg-sidebar-container-padding` | `20px` | Sidebar container padding |
| `--dg-sidebar-container-height` | `87%` | Sidebar container height |

#### Graph Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-graph-width` | `250px` | Local graph width |
| `--dg-graph-height` | `250px` | Local graph height |
| `--dg-graph-border-radius` | `10px` | Graph border radius |
| `--dg-graph-margin-bottom` | `20px` | Graph bottom margin |
| `--dg-graph-fullscreen-width` | `90vw` | Expanded/global graph width |
| `--dg-graph-fullscreen-height` | `85vh` | Expanded/global graph height |
| `--dg-graph-node-color` | `var(--text-accent)` | Active/current node color |
| `--dg-graph-node-color-muted` | `var(--text-faint)` | Neighbor node color |
| `--dg-graph-label-color` | `var(--text-normal)` | Node label text color |
| `--dg-graph-bg` | `var(--background-primary)` | Graph background color |
| `--dg-graph-border-color` | `var(--background-secondary)` | Graph border color |

#### Filetree (Left Sidebar) Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-filetree-width` | `250px` | Filetree sidebar width |
| `--dg-filetree-min-width` | `250px` | Minimum filetree width |
| `--dg-filetree-padding` | `10px 20px` | Filetree padding |
| `--dg-filetree-gap` | `80px` | Gap from content |
| `--dg-filetree-title-size` | `32px` | Filetree title font size |

#### TOC (Table of Contents) Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-toc-padding` | `5px` | TOC container padding |
| `--dg-toc-font-size` | `0.9rem` | TOC font size |
| `--dg-toc-max-height` | `220px` | TOC max height |
| `--dg-toc-title-size` | `1.2rem` | TOC title font size |
| `--dg-toc-item-padding` | `2px 0 2px 8px` | TOC item padding |
| `--dg-toc-indent` | `1em` | TOC nested list indent |

#### Backlinks Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-backlinks-margin-top` | `10px` | Backlinks section top margin |
| `--dg-backlinks-max-height` | `250px` | Backlinks list max height |
| `--dg-backlinks-title-size` | `0.9rem` | Backlinks title font size |
| `--dg-backlinks-card-size` | `0.85rem` | Backlink card font size |
| `--dg-backlinks-card-padding` | `6px 0` | Backlink card padding |
| `--dg-backlinks-icon-size` | `14px` | Backlink icon size |

#### Search Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-search-box-width` | `900px` | Search box width |
| `--dg-search-box-max-width` | `80%` | Search box max width |
| `--dg-search-box-radius` | `15px` | Search box border radius |
| `--dg-search-box-padding` | `10px` | Search box padding |
| `--dg-search-input-size` | `2rem` | Search input font size |
| `--dg-search-input-padding` | `10px` | Search input padding |
| `--dg-search-input-radius` | `5px` | Search input border radius |
| `--dg-search-results-max-height` | `50vh` | Search results max height |
| `--dg-search-result-size` | `1.2rem` | Search result font size |
| `--dg-search-result-radius` | `10px` | Search result border radius |
| `--dg-search-link-size` | `1.4rem` | Search link font size |

#### Search Button Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-search-btn-radius` | `8px` | Search button border radius |
| `--dg-search-btn-height` | `32px` | Search button height |
| `--dg-search-btn-padding` | `0 10px` | Search button padding |
| `--dg-search-btn-gap` | `8px` | Search button icon/text gap |
| `--dg-search-btn-font-size` | `0.85rem` | Search button font size |
| `--dg-search-btn-icon-size` | `14px` | Search button icon size |

#### Navbar Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-navbar-title-size-mobile` | `18px` | Navbar title size on mobile |
| `--dg-navbar-search-margin` | `20px` | Navbar search button margin |
| `--dg-navbar-search-min-width` | `36px` | Navbar search min width |
| `--dg-logo-height` | `40px` | Site logo height on desktop |
| `--dg-logo-height-mobile` | `32px` | Site logo height on mobile |
| `--dg-logo-margin` | `10px 15px` | Site logo margin |
| `--dg-filetree-logo-height` | `70px` | Site logo height in filetree sidebar |

#### Note Link / Filetree Item Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-notelink-padding` | `4px 8px 4px 12px` | Note link padding |
| `--dg-notelink-size` | `0.85rem` | Note link font size |
| `--dg-notelink-border-width` | `2px` | Note link left border width |
| `--dg-notelink-hover-bg` | `rgba(255, 255, 255, 0.05)` | Note link hover background |
| `--dg-folder-margin` | `4px 0 4px 2px` | Folder name margin |
| `--dg-folder-icon-size` | `14px` | Folder icon size |
| `--dg-inner-folder-padding` | `3px 0 3px 0` | Inner folder padding |
| `--dg-inner-folder-margin` | `12px` | Inner folder left margin |
| `--dg-filelist-margin` | `8px` | File list left margin |

#### Graph Controls Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-graph-ctrl-padding` | `6px 10px` | Graph controls padding |
| `--dg-graph-ctrl-radius` | `6px` | Graph controls border radius |
| `--dg-graph-ctrl-margin` | `10px` | Graph controls margin |
| `--dg-graph-ctrl-size` | `0.7rem` | Graph controls font size |
| `--dg-graph-ctrl-icon-size` | `14px` | Graph control icon size |
| `--dg-graph-ctrl-gap` | `10px` | Graph controls gap |

#### Timestamps Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-timestamps-size` | `0.8em` | Timestamps font size |
| `--dg-timestamps-gap` | `10px` | Timestamps gap |
| `--dg-timestamps-margin-top` | `20px` | Timestamps top margin |

#### Misc Component Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `--dg-overlay-bg` | `rgba(0, 0, 0, 0.5)` | Overlay background color |
| `--dg-mermaid-radius` | `25px` | Mermaid diagram border radius |
| `--dg-mermaid-padding` | `10px` | Mermaid diagram padding |
| `--dg-transclusion-padding` | `8px` | Transclusion container padding |
| `--dg-external-link-icon-size` | `13px` | External link icon size |
| `--dg-external-link-padding` | `16px` | External link right padding |
