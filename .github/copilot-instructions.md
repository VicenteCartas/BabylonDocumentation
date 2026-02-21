# Babylon.js 2D Documentation — Copilot Instructions

This session is focused on the **2D game engine documentation** for `@babylonjs/2d`. Doc pages live in `content/features/featuresDeepDive/2d/`. The parent Babylon.js repo contains the engine source at `Babylon.js/packages/dev/2d/`.

## Build & Dev Commands

```bash
npm install                # Install dependencies
npm run dev                # Local dev server at http://localhost:3000
npm run build              # Production static export
```

## Adding or Editing a 2D Doc Page

1. Create/edit the `.md` file in `content/features/featuresDeepDive/2d/`
2. Add/update the entry in `configuration/structure.json` under the `"2d"` node (each entry needs `friendlyName` and `content` path without extension)
3. Use YAML frontmatter at the top of the file:

```yaml
---
title: Feature Name
description: Brief description for SEO
keywords: 2d, feature, babylon.js
---
```

Optional frontmatter: `videoOverview` (YouTube ID), `tocLevels` (1-3, default 3), `furtherReading` (array of `{title, url}`).

## Current 2D Doc Pages (19 pages)

intro, sprites-and-animation, camera2d, tilemaps, collision, physics, input-mapping, pathfinding, grid-system, lighting, particles, tween-easing, state-machine, text-and-ui, nine-slice, scene-transitions, audio, isometric, turn-based

## Documentation Style

- **TypeScript code blocks** for all examples — show imports, setup, and usage
- **Property/method tables** for API reference (markdown tables with name, type, description)
- **"Tips" sections** for best practices and gotchas
- Cross-link other 2D pages with relative paths: `/features/featuresDeepDive/2d/<page>`
- Mention the coordinate system (Y-down, top-left origin) when relevant to avoid confusion with 3D

## Custom Markdown Components

Use these in `.md` files:

| Component | Usage |
|-----------|-------|
| `<Playground id="#ABC123" title="Demo" />` | Embed Babylon.js Playground |
| `<Youtube id="video-id" />` | Embed YouTube video |
| `<Alert severity="info">Note</Alert>` | Callout box (info/warning/error) |
| `<NME id="..." />` | Node Material Editor embed |
| `<NGE id="..." />` | Node Geometry Editor embed |
| `<SFE id="..." />` | Shader Former Editor embed |
| `<NRGE id="..." />` | NRE Graph Editor embed |
| `<CodePen id="..." />` | CodePen embed |
| `<CodeSandbox id="..." />` | CodeSandbox embed |
| `<Media url="..." />` | Media file embed |

## Content → Route Pipeline

1. **`configuration/structure.json`** — nested JSON tree defining page hierarchy. Each node: `friendlyName`, `content` (path to `.md`), optional `children`
2. **`content/`** — all Markdown files
3. **`pages/[...id].tsx`** — catch-all dynamic route using `getStaticPaths()` / `getStaticProps()`
4. **`lib/buildUtils/`** — content loading, URL generation, redirect resolution

## Key Conventions

- Line length is not enforced in markdown
- Inline HTML allowed only for the custom components above (plus `<kbd>`)
- Content images go in `public/` and are referenced with relative paths
- Math/LaTeX supported via `remark-math` and `rehype-katex`
