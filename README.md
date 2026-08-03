# nikolauspschuetz.dev

Personal site + blog. Astro, deployed on Cloudflare Pages.

## Develop

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # -> dist/
```

## Write a post

Add a Markdown file under `src/content/blog/`:

```markdown
---
title: "Post title"
description: "One-line summary for previews and SEO."
date: 2026-08-03
tags: ["tag"]
draft: true   # flip to false (or remove) to publish
---

Body…
```

Drafts (`draft: true`) are hidden from the index and the homepage.

## Deploy (Cloudflare Pages)

- Framework preset: **Astro**
- Build command: `npm run build`
- Output directory: `dist`
- Then add the custom domain `nikolauspschuetz.dev` in the Pages project.
