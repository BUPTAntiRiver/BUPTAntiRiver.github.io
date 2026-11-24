# AGENTS.md

This is a **Quartz v4** static site (digital garden/blog) forked from `jackyzha0/quartz`. Content is Markdown in `content/`; build output goes to `public/` (GitHub Pages root).

## Commands

| Action | Command |
|--------|---------|
| Dev server | `npx quartz build --serve` |
| Production build | `npx quartz build` |
| Typecheck + format check | `npm run check` (runs `tsc --noEmit && prettier . --check`) |
| Format | `npm run format` (`prettier . --write`) |
| Test | `npm test` (`tsx --test`) |
| Sync git | `npx quartz sync` |

Always run `npm run check` before committing. CI runs `npm ci` → `npm run check` → `npm test` → `npx quartz build --bundleInfo -d docs`.

## Setup

- **Node:** `>=22` (`.node-version` = v22.16.0), **npm:** `>=10.9.2`. `.npmrc` sets `engine-strict=true`.
- Install: `npm ci` (respects lockfile).

## Code conventions

- **Prettier:** no semicolons, printWidth 100, trailingComma all, tabWidth 2
- **TypeScript:** strict mode, ESNext, JSX via Preact
- **No ESLint** — type checking + Prettier are the only linters
- Ignored by `.gitignore`: `public/`, `.quartz-cache/`, `node_modules/`, `private/`, `tsconfig.tsbuildinfo`, plus machine-local Obsidian state (`content/.obsidian/workspace*.json`, `cache`, `.trash/`)

## Content

- All notes live under `content/` as Markdown, organized by topic (e.g. `Machine Learning/LLM/`).
- `content/.obsidian/` is the Obsidian vault config and **is tracked** — see “Obsidian vault” below.
- Files/dirs matching `["private", "templates", ".obsidian", ".obsidian/**"]` are excluded from the build.
- Drafts use `draft: true` in frontmatter (filtered by `Plugin.RemoveDrafts()`).
- Images go in `content/assets/`.

## `quartz.config.ts` quirks

- `baseUrl` still reads `"quartz.jzhao.xyz"` — update when deploying with a custom domain.
- Footer links point to upstream Quartz repo/Discord, not user's own profiles.
- `Plugin.CustomOgImages()` is slow; comment out to speed up builds.

## Notable directories

| Path | Purpose |
|------|---------|
| `content/` | User's markdown notes (the actual site content) |
| `quartz/` | Quartz framework source (do not edit unless upstreaming) |
| `docs/` | Quartz documentation (not user content) |
| `public/` | Build output / GitHub Pages root (gitignored) |
| `.quartz-cache/` | Build cache (gitignored) |

## Obsidian vault

`content/` is an Obsidian vault synced through git (`npx quartz sync` commits and pushes). `alwaysUpdateLinks` is enabled — renaming files in Obsidian auto-updates internal links.

### What travels with the repo (`content/.obsidian/`)

| Path | Contents |
|------|----------|
| `app.json` | editor behaviour (`alwaysUpdateLinks`, `vimMode`, `spellcheck`, `strictLineBreaks`) |
| `appearance.json` | active theme (`cssTheme`), accent colour, UI / body / mono fonts |
| `core-plugins.json`, `community-plugins.json` | which core and community plugins are enabled |
| `hotkeys.json` | custom hotkeys |
| `themes/<name>/` | theme CSS + manifest (currently AnuPpuccin) |
| `plugins/<id>/` | `manifest.json`, `data.json` (the plugin's settings), `main.js`, `styles.css` |
| `snippets/` | CSS snippets, if any |

Machine-local files are gitignored: `workspace.json`, `workspace-mobile.json`, `cache`, `.trash/`. `main.js` and `theme.css` are vendored third-party bundles, marked `linguist-vendored -diff` in `.gitattributes` so they don't skew GitHub language stats or clutter `git diff`.

Vault plugins enabled: `obsidian-latex-suite`, `obsidian-linter`, `obsidian-minimal-settings`, `obsidian-hider`, `obsidian-style-settings`.

### Why the site ignores it

Quartz globs with `dot: false`, so `.obsidian/` is never scanned or emitted — a build reads 164 notes and `public/` contains no `.obsidian` path. The one place the directory name matters is `ignorePatterns` in `quartz.config.ts`, which lists both `.obsidian` and `.obsidian/**`. The live-preview watcher minimatches each changed path individually, so the bare name alone is not enough: with it alone, every settings write (and every theme removal, which the Assets emitter then tries to unlink from `public/`) triggers a rebuild or crashes `--serve`. Keep both entries.

### Restore on a new machine

1. `git clone <repo> && cd <repo>` (plus `npm ci` if you also want to build the site).
2. Open `content/` as a vault in Obsidian — the repo root is not the vault.
3. Settings, theme, plugins and their `data.json` arrive with the clone; turn off Restricted mode if Obsidian starts there.
4. The fonts named in `appearance.json` (`Maple Mono NF CN`) must be installed on the OS — they are not synced.
5. If Obsidian Sync is also active, disable its "Sync configuration" options; two writers on `.obsidian/` will conflict.

