# brendanmurrell.com

Source for [brendanmurrell.com](https://brendanmurrell.com). Built with [Hugo](https://gohugo.io) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed to GitHub Pages.

## Preview locally

Requires Hugo 0.132 or newer. CI builds with 0.147.6 extended.

```bash
git clone --recurse-submodules https://github.com/brmurrell3/brendanmurrell.com.git
cd brendanmurrell.com
hugo server
```

The site is served at http://localhost:1313.

## Layout

- `content/projects/`: one page per project. A project with images or other files is a folder with an `index.md` (see `macro-placer/`).
- `layouts/shortcodes/math.html`: renders LaTeX to MathML at build time. Invalid LaTeX fails the build.
- `static/`: files copied as is, including `CNAME` for the custom domain.
- `hugo.yaml`: site config, menu, and home page text.
- `themes/PaperMod/`: the theme, as a git submodule.

## Add a project

Create `content/projects/<slug>.md`:

```yaml
---
title: "Project name"
weight: 4
tags: ["tag"]
summary: "One sentence shown on the projects list."
---
```

`weight` sets the order on the projects page, lowest first.

## Deploy

Pushing to `main` runs `.github/workflows/hugo.yml`, which builds the site and publishes it to GitHub Pages.
