# Babylon.js Documentation Site — Copilot Instructions

## Repository Overview

This is the **Babylon.js documentation website**, a statically-generated **Next.js** site using MDX for content. Documentation pages are authored as Markdown files in `content/` and mapped to routes via `configuration/structure.json`.

## Build & Dev Commands

```bash
npm install                # Install dependencies
npm run dev                # Local dev server at http://localhost:3000
npm run build              # Production static export
```

## Architecture

### Content → Route Pipeline

1. **`configuration/structure.json`** defines the site's page hierarchy as a nested JSON tree. Each node has `friendlyName`, `content` (path to `.md` file without extension), and optional `children`.
2. **`content/`** contains all Markdown documentation files, organized by topic.
3. **`pages/[...id].tsx`** is the catch-all dynamic route. It uses `getStaticPaths()` to pre-generate all valid URLs from `structure.json`, and `getStaticProps()` to load and serialize the markdown.
4. **`lib/buildUtils/`** handles content loading (`tools.ts`), URL generation (`content.utils.ts`), and redirect resolution (`redirects.ts`).

**To add a new page**: create a `.md` file in `content/` and add a corresponding entry in `configuration/structure.json`.

### Markdown Frontmatter

```yaml
---
title: Page Title              # Required
description: Meta description
keywords: comma, separated
videoOverview: youtube-id      # Optional video
furtherReading:                # Optional related links
  - title: Link Text
    url: https://example.com
---
```

### Custom Markdown Components

Use these in `.md` files (allowed by markdownlint config):

| Component | Usage |
|-----------|-------|
| `<Playground id="#ABC123" title="Demo" />` | Embed Babylon.js Playground |
| `<Youtube id="video-id" />` | Embed YouTube video |
| `<Alert>Important note</Alert>` | Callout/alert box |
| `<NME id="..." />` | Node Material Editor embed |
| `<NGE id="..." />` | Node Geometry Editor embed |
| `<CodePen id="..." />` | CodePen embed |
| `<CodeSandbox id="..." />` | CodeSandbox embed |
| `<Media url="..." />` | Media file embed |

### Styling

- **Material-UI** with a custom dark theme defined in `styles/theme.ts`
- **SCSS modules** for component-scoped styles
- Global styles in `styles/globals.scss`

### Redirects

`redirects.json` maps old URLs to new destinations. Redirects are resolved during static generation with a 5-second client-side auto-redirect.

## Key Conventions

- Markdown linting allows inline HTML only for the custom components listed above (`Playground`, `Youtube`, `kbd`, `Alert`)
- Line length is not enforced in markdown
- Content images should be committed to `public/` and referenced with relative paths
