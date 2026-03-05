# Zola + Tailwind CSS Boilerplate

A minimal, production-ready boilerplate for building static sites with [Zola](https://www.getzola.org/) and [Tailwind CSS v4](https://tailwindcss.com/). The entire development environment runs inside Docker — no local Rust or Node.js required.

## Features

- **[Zola](https://www.getzola.org/) v0.22.1** — single-binary static site generator written in Rust
- **[Tailwind CSS v4](https://tailwindcss.com/)** — utility-first CSS with automatic content scanning
- **Docker Compose** — zero local toolchain setup; one command to start
- **Blog section** — with date sorting, pagination, and tag taxonomies
- **Syntax highlighting** — via Zola's built-in highlighter (catppuccin-mocha theme)
- **Prose styles** — `@layer components` CSS for nicely formatted Markdown output
- **SEO-friendly** — `<title>` and `<meta name="description">` on every page

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with Compose v2

That's it. No Node.js, no Rust, no other local installs needed.

## Quick Start

```bash
git clone https://github.com/SpiZeak/zola-tailwind.git my-site
cd my-site
docker compose up
```

Open [http://localhost](http://localhost) in your browser. Zola and the Tailwind watcher both reload automatically when you edit files.

## Project Structure

```
.
├── compose.yml               # Docker Compose — Zola + Tailwind services
├── zola.toml                 # Zola site configuration
│
├── css/
│   └── tailwind.css          # Tailwind source (edit this)
│
├── static/
│   └── tailwind.css          # Compiled CSS (auto-generated — do not edit)
│
├── content/                  # Markdown content, mirrored to URLs
│   ├── _index.md             # Homepage
│   ├── about.md              # /about/
│   └── blog/
│       ├── _index.md         # /blog/ (section config)
│       └── *.md              # /blog/<slug>/
│
├── templates/                # Tera HTML templates
│   ├── base.html             # Shared layout (navbar, footer)
│   ├── index.html            # Homepage template
│   ├── page.html             # Single page / blog post template
│   └── section.html          # Section listing template (e.g. /blog/)
│
└── docker/
    └── tailwind/
        └── Dockerfile        # Tailwind CLI image (Bun-based)
```

## Configuration

Edit `zola.toml` to configure your site:

```toml
base_url   = "https://your-domain.com"  # Production URL (required)
title      = "My Site"                   # Available as {{ config.title }}
description = "My site description"      # Available as {{ config.description }}
```

Custom variables can be added under `[extra]` and accessed in any template as `{{ config.extra.variable_name }}`.

See the [Zola configuration reference](https://www.getzola.org/documentation/getting-started/configuration/) for all available options.

## Adding Content

### Page

Create a Markdown file anywhere under `content/`:

```markdown
+++
title = "My Page"
description = "A short description for SEO."
+++

Page content goes here.
```

### Blog Post

Create a Markdown file under `content/blog/`:

```markdown
+++
title = "My Post"
date = 2026-01-01
description = "A one-sentence summary shown in listings and meta tags."

[taxonomies]
tags = ["tutorial", "zola"]
+++

Post content goes here.
```

Set `draft = true` in front matter to hide a post from production while still previewing it locally (run `docker compose up` — Zola serves drafts automatically in dev mode).

### Front Matter Reference

| Field         | Type     | Description                                     |
| ------------- | -------- | ----------------------------------------------- |
| `title`       | string   | Page / post title                               |
| `date`        | date     | Publication date `YYYY-MM-DD`                   |
| `description` | string   | Short summary (listings + `<meta>` description) |
| `draft`       | bool     | Hide from production builds                     |
| `weight`      | integer  | Manual sort order within a section              |
| `tags`        | string[] | Under `[taxonomies]`, list of tag strings       |

## Templates

Templates use [Tera](https://keats.github.io/tera/docs/) syntax (similar to Jinja2).

| Template       | Renders                                       |
| -------------- | --------------------------------------------- |
| `base.html`    | Shared layout — extend this in all templates  |
| `index.html`   | Root `_index.md` (homepage)                   |
| `page.html`    | All individual pages and blog posts           |
| `section.html` | All section `_index.md` files (e.g. `/blog/`) |

### Key Template Variables

**Pages (`page.*`)**

```
{{ page.title }}              Page title
{{ page.content | safe }}     Rendered HTML body
{{ page.date }}               Publication date
{{ page.description }}        Front matter description
{{ page.permalink }}          Full canonical URL
{{ page.taxonomies.tags }}    List of tags
```

**Sections (`section.*`)**

```
{{ section.title }}           Section title
{{ section.pages }}           List of pages in the section
{{ section.content | safe }}  Rendered HTML body
```

**Global (`config.*`)**

```
{{ config.title }}            Site title
{{ config.description }}      Site description
{{ config.base_url }}         Base URL
{{ config.extra.* }}          Custom variables from [extra]
```

### Section-specific Templates

To override a template for a specific section, create a matching subdirectory under `templates/`. For example:

```
templates/
└── blog/
    ├── page.html     ← used only for posts under content/blog/
    └── section.html  ← used only for content/blog/_index.md
```

## Customising Tailwind CSS

Edit `css/tailwind.css`. The Tailwind watcher recompiles `static/tailwind.css` on every save.

Add custom component or utility styles using standard CSS layers:

```css
@layer components {
  .btn {
    padding: 0.5rem 1rem;
    border-radius: 0.5rem;
    font-weight: 600;
  }
  .btn-primary {
    background-color: #2563eb;
    color: #fff;
  }
}
```

### Adding Tailwind Plugins

1. Add the package in `docker/tailwind/Dockerfile`:
   ```dockerfile
   RUN bun add @tailwindcss/typography
   ```
2. Import it in `css/tailwind.css`:
   ```css
   @plugin "@tailwindcss/typography";
   ```
3. Rebuild the container:
   ```bash
   docker compose build tailwind && docker compose up
   ```

## Building for Production

**Step 1 — compile CSS (minified):**

```bash
docker compose run --rm tailwind \
  -i ./css/tailwind.css -o ./static/tailwind.css --minify
```

**Step 2 — build the site:**

```bash
docker compose run --rm zola build
```

The finished site is output to `public/`. Upload that directory to any static host.

## Deployment

After building, deploy the `public/` directory to any static hosting provider:

- [Cloudflare Pages](https://pages.cloudflare.com/)
- [Netlify](https://netlify.com)
- [Vercel](https://vercel.com)
- [GitHub Pages](https://pages.github.com)
- Any web server (nginx, Caddy, Apache, etc.)

## License

MIT
