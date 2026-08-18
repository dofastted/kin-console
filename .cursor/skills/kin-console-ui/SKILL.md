---
name: kin-console-ui
description: KIN Console UI adapter for the vendored UI/UX Pro Max skill. Use when designing, reviewing, restyling, or implementing UI in this repo — especially index.html, the admin workbench, tables, dialogs, login, navigation, tokens, or density/contrast work. Constrains recommendations to the existing single-file vanilla HTML/CSS/JS console.
paths:
  - "index.html"
  - ".cursor/skills/kin-console-ui/**"
  - ".cursor/skills/ui-ux-pro-max/**"
---

# KIN Console UI

This repo is **种核 KIN 管理台**: one static file (`index.html`), no bundler, no React, no Tailwind, no shadcn.

For general design intelligence, read and follow `.cursor/skills/ui-ux-pro-max/SKILL.md`. Then apply the constraints below. **Existing tokens in `index.html` win** over any generated palette or font pairing.

## Stack (do not change)

- Vanilla HTML + CSS custom properties + one `<script>`
- Chinese UI copy (`lang="zh-CN"`)
- Dark-only OLED admin (`color-scheme: dark`)
- Search stack: `html-tailwind` for UX/layout ideas only — implement as existing classes/tokens, never add Tailwind or a build step

Ignore sibling skills (`ui-styling`, `design`, `design-system`, `brand`, `slides`, `banner-design`) unless the user names them. They assume shadcn/Tailwind/marketing surfaces.

## Existing tokens (source of truth)

Read `:root` in `index.html` before changing color, type, or spacing.

| Token | Value | Role |
|-------|-------|------|
| `--bg` | `#020617` | Page |
| `--panel` / `--panel-2` | `#0E1223` / `#14192C` | Cards |
| `--sidebar` | `#070B16` | Nav |
| `--fg` / `--muted` / `--faint` | `#F8FAFC` / `#94A3B8` / `#64748B` | Text |
| `--ok` / `--accent` | `#22C55E` | Success / accent |
| `--warn` | `#F59E0B` | Warning |
| `--bad` / `--destructive` | `#EF4444` | Danger |
| `--primary` / `--primary-fg` | `#E8ECF2` / `#0A0E18` | Primary button |
| `--font` | Fira Sans | UI |
| `--mono` | Fira Code | IDs, keys, logs |
| Density | 13px body, 32px buttons, dense tables | Dashboard, not marketing |

Do not switch to JetBrains Mono, IBM Plex, or a light theme unless the user asks.

## Views

`data-view` routes: `overview`, `cluster`, `vm`, `import`, `usage`, `proxies`, `protocol`, `settings`. Login is `#loginGate`. Keep the sidebar + topbar + content shell.

## Workflow

1. Extract the product intent (admin / VM / quota / session / proxy).
2. Run UI/UX Pro Max with **dense + dark + developer-tool** terms:

```bash
python3 .cursor/skills/ui-ux-pro-max/scripts/search.py "developer tool admin console dashboard dark dense" --design-system --density 9 --variance 3 --motion 3 -p "KIN Console"
```

3. For a focused bug, one domain search:

```bash
python3 .cursor/skills/ui-ux-pro-max/scripts/search.py "<issue>" --domain ux
```

4. Implement only in `index.html` using current class names (`.btn`, `.card`, `.input`, `.badge`, `.tbl`, `.dialog`).
5. Do not persist a new `design-system/` unless the user asks. If one exists, page overrides still lose to tokens already in `index.html`.

## Must keep

- SVG icons (Heroicons-style stroke), never emoji-as-icon
- `cursor: pointer` on clickable controls
- Visible focus rings (do not use `outline: none` without a replacement)
- `prefers-reduced-motion` respected
- Compact labels: wrap or `+n` disclosure; no clipped chips
- Tables: horizontal scroll or card fallback at narrow widths
- Session/key strings may wrap; do not overflow the panel

## Must not

- Add `package.json`, Tailwind, React, Vue, or a bundler
- Convert the SPA into multiple HTML files
- Introduce a second type scale or rainbow accent palette
- Soften density into a marketing landing page
