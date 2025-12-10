# chabotc.co.uk chabot.dev

A personal blog masquerading as a terminal emulator, built by someone who has spent far too many hours staring at monospace fonts and has the opinions to prove it.

## What Is This?

This is my blog. It pretends to be a bash shell because apparently two decades of software engineering leaves you with an aesthetic sensibility best described as "what if everything looked like `htop`?"

The site features:
- **Blog posts** rendered from Markdown, because I refuse to learn a CMS
- **An About page** that chronicles my wandering path through the tech industry
- **A Projects page** showcasing open source work of varying degrees of completion and ambition

## Tech Stack

Because every README needs a section that sounds like a grocery list:

- **[Astro](https://astro.build)** - For when you want React but also want your pages to actually load
- **[React](https://react.dev)** - The framework we all pretend to have strong opinions about
- **[Tailwind CSS](https://tailwindcss.com)** - Inline styles, but make it respectable
- **[shadcn/ui](https://ui.shadcn.com)** - Copy-paste component development, elevated to an art form
- **[Cloudflare Pages](https://pages.cloudflare.com)** - Hosting that costs approximately nothing

## Running Locally

```bash
# Install dependencies (pray to the npm gods)
npm install

# Start the dev server
npm run dev

# Build for production
npm run build

# Preview the production build
npm run preview
```

The site will materialize at `http://localhost:4321`, a port number that exists solely to remind you that Astro does things differently.

## Project Structure

```
├── src/
│   ├── components/     # Astro and React components
│   ├── content/        # Blog posts (Markdown)
│   ├── layouts/        # Page layouts with CRT effects
│   ├── lib/            # Utilities (mostly cn())
│   ├── pages/          # Routes
│   └── styles/         # Global CSS and terminal theming
├── public/             # Static assets
├── CLAUDE.md           # Documentation for AI assistants
└── README.md           # You are here
```

## Writing Blog Posts

Create a new `.md` file in `src/content/blog/` with frontmatter:

```markdown
---
title: "Your Extremely Clever Title"
description: "A description that will inevitably be truncated"
date: 2024-12-10
tags: ["existential-dread", "javascript"]
---

Your content here. Try not to start with "In this post, I will..."
```

**Important:** Do not include an H1 (`# Title`) in the content body. The template already renders the title from frontmatter, and you'll end up with the kind of duplication that makes designers weep.

## Design Philosophy

The terminal aesthetic isn't just nostalgia—it's a deliberate choice to strip away the visual noise that plagues modern web design. No hero images. No parallax scrolling. No cookie consent modals the size of a philosophy dissertation.

Just text. Monospace text. The way Ritchie and Thompson intended.*

<sub>*They probably did not intend this.</sub>

## Contributing

This is a personal blog, so "contributing" mostly means finding typos and judging my opinions silently. But if you spot something egregiously broken, issues are welcome.

## License

The code is MIT licensed. The opinions are my own and not for redistribution without significant editorial oversight.

---

*Built with mass quantities of coffee and mass quantities of mass quantified AI assistance.*
