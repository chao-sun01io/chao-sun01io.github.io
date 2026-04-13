# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo-based personal blog ("Chao's garden") using the **LoveIt** theme, deployed to GitHub Pages at https://chao-sun01io.github.io.

## Build & Development Commands

```bash
# Local development server (with drafts)
hugo server -D

# Build for production (matches CI)
hugo --minify

# Create a new post
hugo new posts/my-post-title.md
```

Hugo version used in CI: **0.122.0 extended edition** (see `.github/workflows/hugo-gh-pages.yml`).

## Architecture

- **Config**: `hugo.toml` — site settings, menu, author info
- **Content**: `content/posts/` — Markdown files with YAML frontmatter
- **Theme**: `themes/LoveIt/` — git submodule from `chao-sun01io/LoveIt`
- **Layouts/Assets**: Currently empty; all rendering comes from the theme
- **Deployment**: GitHub Actions builds on push to `main`, publishes to `gh-pages` branch

## Content Frontmatter Format

```yaml
---
publish: true
title: "Post Title"
aliases: []
date: 2024-01-01T00:00:00+00:00
lastmod: 2024-01-01T00:00:00+00:00
tags: []
category: posts
summary: "Brief description"
---
```

## Theme (LoveIt)

The theme is a git submodule. To clone with the theme:
```bash
git clone --recurse-submodules <repo-url>
# Or if already cloned:
git submodule update --init --recursive
```

Available shortcodes from the theme: `image`, `link`, `typeit`, `admonition`, `mermaid`, `echarts`, `music`, `bilibili`, `mapbox`, `style`, `raw`, `person`, `script`, `version`.

Key theme constraint: `markup.highlight.noClasses` must be `false` in `hugo.toml` (see [LoveIt#158](https://github.com/dillonzq/LoveIt/issues/158)).
