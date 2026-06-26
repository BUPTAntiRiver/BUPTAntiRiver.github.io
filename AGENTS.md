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
- Ignored by `.gitignore`: `public/`, `.quartz-cache/`, `node_modules/`, `.obsidian/`, `private/`, `tsconfig.tsbuildinfo`

## Content

- All notes live under `content/` as Markdown, organized by topic (e.g. `Machine Learning/LLM/`).
- `content/.obsidian/` is the Obsidian vault config (sync target for Obsidian publish pipeline).
- Files/dirs matching `["private", "templates", ".obsidian"]` are excluded from the build.
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

`content/` is configured as an Obsidian vault using `git`-based sync. Vault plugins used: `obsidian-pangu`, `obsidian-latex-suite`. `alwaysUpdateLinks` is enabled — renaming files in Obsidian auto-updates internal links.
