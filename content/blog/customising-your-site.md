+++
title = "Customising Your Site"
date = 2026-03-04
description = "A practical guide to customising templates, adding pages, styling with Tailwind CSS, and configuring Zola."

[taxonomies]
tags = ["zola", "tailwind", "tutorial"]
+++

This post walks through the main ways to make this boilerplate your own.

## 1. Configuration

Open `zola.toml` and update the three most important settings:

| Setting       | Purpose                                                   |
| ------------- | --------------------------------------------------------- |
| `base_url`    | The production URL of your site                           |
| `title`       | Site name, available in templates as `{{ config.title }}` |
| `description` | Short description used in meta tags                       |

Any key under `[extra]` is accessible in templates via `{{ config.extra.key_name }}`.

## 2. Adding Pages

Create a Markdown file anywhere under `content/`. Zola maps the directory structure to URLs:

- `content/contact.md` → `/contact/`
- `content/projects/index.md` → `/projects/`

Every page needs a front matter block at the top, delimited by `+++`:

```toml
+++
title = "Contact"
description = "Get in touch."
+++
```

Content below the closing `+++` is your Markdown.

## 3. Adding Blog Posts

Drop a `.md` file into `content/blog/`. Give it a `date` field and it will appear in the blog listing, sorted newest first.

```toml
+++
title = "My Post"
date = 2026-01-15
description = "A one-sentence summary."

[taxonomies]
tags = ["tutorial"]
+++
```

Set `draft = true` in front matter to hide a post from production builds while still being able to preview it locally with `--drafts`.

## 4. Editing Templates

Templates live in `templates/` and use [Tera](https://keats.github.io/tera/docs/) syntax.

| Template       | Renders                             |
| -------------- | ----------------------------------- |
| `base.html`    | Shared layout (navbar, footer)      |
| `index.html`   | Homepage (`content/_index.md`)      |
| `page.html`    | All individual pages and blog posts |
| `section.html` | Section index pages (e.g. `/blog/`) |

All content templates extend `base.html` and override the `{% block content %}` block.

### Section-specific templates

To give a section its own template, create a subdirectory under `templates/` matching the section name. For example, `templates/blog/page.html` will only apply to posts under `content/blog/`.

## 5. Styling with Tailwind CSS

Edit `css/tailwind.css`. The `tailwind` Docker service watches this file and recompiles `static/tailwind.css` automatically on each save.

Add custom component styles using `@layer components`:

```css
@layer components {
  .btn {
    padding: 0.5rem 1rem;
    border-radius: 0.5rem;
    font-weight: 600;
    transition: background-color 0.15s;
  }
  .btn-primary {
    background-color: #2563eb;
    color: #fff;
  }
  .btn-primary:hover {
    background-color: #1d4ed8;
  }
}
```

Because Tailwind v4 scans your templates automatically, any utility class used in HTML will be included in the compiled output — no configuration needed.

## 6. Adding Tailwind Plugins

The Tailwind CLI is installed via Bun in `docker/tailwind/Dockerfile`. To add a plugin (e.g. `@tailwindcss/typography`):

1. Open `docker/tailwind/Dockerfile` and add:
   ```dockerfile
   RUN bun add @tailwindcss/typography
   ```
2. Import it in `css/tailwind.css`:
   ```css
   @plugin "@tailwindcss/typography";
   ```
3. Rebuild the container:
   ```bash
   docker compose build tailwind
   docker compose up
   ```

---

Found an issue or want to contribute? Open a pull request or file an issue on [GitHub](https://github.com/SpiZeak/zola-tailwind).
