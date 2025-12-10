# Chris Chabot's Blog

Personal blog built with Astro, React, Tailwind CSS, and shadcn/ui. Deployed on Cloudflare Pages.

## Tech Stack

- **Framework**: Astro 5.x with React integration
- **Styling**: Tailwind CSS 4.x with `@tailwindcss/typography` plugin
- **UI Components**: shadcn/ui (button, card, badge)
- **Font**: JetBrains Mono (self-hosted via `@fontsource/jetbrains-mono`)
- **Deployment**: Cloudflare Pages (`@astrojs/cloudflare` adapter)

## Commands

```bash
npm run dev      # Start dev server at localhost:4321
npm run build    # Build for production
npm run preview  # Preview production build locally
```

## Project Structure

```
src/
├── components/
│   ├── Header.astro         # Terminal-style navigation
│   ├── Footer.astro         # Command-line themed footer
│   ├── TerminalCard.astro   # Terminal window component
│   ├── BlogPostCard.astro   # Blog post preview cards
│   ├── ProjectCard.astro    # GitHub project cards
│   └── ui/                  # shadcn/ui components
├── content/
│   ├── config.ts            # Content collection schema
│   └── blog/                # Markdown blog posts
├── layouts/
│   └── BaseLayout.astro     # Main layout with CRT effects
├── pages/
│   ├── index.astro          # Homepage
│   ├── about.astro          # About page
│   ├── projects.astro       # GitHub projects showcase
│   └── blog/
│       ├── index.astro      # Blog listing
│       └── [slug].astro     # Individual post pages
└── styles/
    └── global.css           # Terminal-themed Tailwind config
```

## Writing Blog Posts

Blog posts live in `src/content/blog/` as Markdown files.

### Frontmatter Schema

```yaml
---
title: "Post Title"           # Required - displayed in page header
description: "Short summary"  # Required - shown in cards and meta
date: 2024-12-10              # Required - for sorting
tags: ["tag1", "tag2"]        # Optional - displayed as --flags
draft: false                  # Optional - set true to hide
---
```

### IMPORTANT: Avoid Duplicate Titles

The blog post template (`src/pages/blog/[slug].astro`) automatically renders the `title` from frontmatter as an H1. **Do NOT include an H1 (`#`) at the start of your markdown content** or the title will appear twice.

**Wrong:**
```markdown
---
title: "My Post Title"
---

# My Post Title

Content starts here...
```

**Correct:**
```markdown
---
title: "My Post Title"
---

Content starts here...

## First Section
```

## Design System

### Terminal Aesthetic

The blog uses a dark terminal-inspired theme with:
- GitHub dark color palette
- JetBrains Mono monospace font throughout
- Command-line UI patterns (`$` prompts, `--flags` as tags)
- Subtle CRT scanlines and noise overlay effects
- ASCII art header on homepage

### Theme Colors (defined in `global.css`)

```css
--color-background: #0d1117      /* Dark background */
--color-foreground: #c9d1d9      /* Light text */
--color-terminal-green: #3fb950  /* Primary accent */
--color-terminal-blue: #58a6ff   /* Links */
--color-terminal-yellow: #d29922 /* Warnings/highlights */
--color-terminal-cyan: #56d4dd   /* Tags */
--color-terminal-purple: #a371f7 /* Code keywords */
```

## Content Collections

Blog posts use Astro's content collections. The schema is defined in `src/content/config.ts`:

```typescript
const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    date: z.coerce.date(),
    tags: z.array(z.string()).optional().default([]),
    draft: z.boolean().optional().default(false),
  }),
});
```

## Deployment

The site is configured for Cloudflare Pages:

1. Push to `main` branch on GitHub
2. Cloudflare Pages auto-deploys
3. Build command: `npm run build`
4. Output directory: `dist`

## Adding New Pages

Follow the existing patterns:
1. Create `.astro` file in `src/pages/`
2. Import `BaseLayout`, `Header`, `Footer`
3. Use terminal-style headings with `$` prompt prefix
4. Wrap content in `max-w-6xl mx-auto px-4 sm:px-6`
