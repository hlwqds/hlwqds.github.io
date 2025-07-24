# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a Jekyll-based blog using the **Chirpy theme** (v7.3), deployed to GitHub Pages. The site is configured for Chinese language (`lang: zh`) with timezone set to Asia/Shanghai. The blog focuses on technical content, particularly Linux kernel and system programming topics.

## Development Environment

### Prerequisites
- Ruby (version 3.1 recommended)
- Bundler gem
- Git

### Setup
```bash
# Install dependencies
bundle install

# For Windows users, additional gems are installed automatically:
# - tzinfo, tzinfo-data (timezone support)
# - wdm (directory monitoring)
```

### DevContainer
A `.devcontainer` configuration is available for VS Code Remote Containers with Jekyll pre-configured. The container includes:
- Jekyll development environment
- Zsh with Oh My Zsh
- Extensions for Liquid, Shell, Markdown, and Git

## Common Commands

### Local Development
```bash
# Serve the site locally with live reload
bundle exec jekyll serve

# Serve with drafts visible
bundle exec jekyll serve --drafts

# Serve with future-dated posts visible
bundle exec jekyll serve --future

# Build the site (output to _site/)
bundle exec jekyll build

# Build with production environment
JEKYLL_ENV=production bundle exec jekyll build
```

### Testing
```bash
# Run HTML validation tests
bundle exec htmlproofer ./_site
```

### Content Management
```bash
# Create a new post (manually)
# Posts go in _posts/ with naming format: YYYY-MM-DD-title.md

# Check for uncommitted changes
git status

# View commit history for a post (used by lastmod plugin)
git log --follow _posts/YYYY-MM-DD-post-name.md
```

## Architecture

### Directory Structure

**Content Directories:**
- `_posts/` - Blog posts with filename format `YYYY-MM-DD-title.md`
- `_tabs/` - Top-level navigation pages (About, Archives, Categories, Tags)
- `_data/` - Configuration data files:
  - `contact.yml` - Social contact links in sidebar
  - `share.yml` - Social sharing options for posts
- `assets/` - Static assets (images, CSS, JS)

**Configuration:**
- `_config.yml` - Main Jekyll configuration and Chirpy theme settings
- `Gemfile` - Ruby dependencies
- `_plugins/` - Custom Jekyll plugins

**Build Output:**
- `_site/` - Generated static site (excluded from git)

### Key Components

**Custom Plugins:**
- `_plugins/posts-lastmod-hook.rb` - Automatically sets `last_modified_at` frontmatter based on git commit history

**Post Frontmatter:**
Posts require YAML frontmatter with:
- `title` - Post title
- `date` - Publication date
- `categories` - Array of categories (e.g., `[kernel]`)
- `tag` or `tags` - Array of tags
- `pin` - (optional) Pin post to top of home page

Example:
```yaml
---
title: 进程管理
date: 2024-07-25 00:11:00
pin: true
categories: [kernel]
tag: [kernel]
---
```

**Tab Pages:**
Tabs in `_tabs/` use:
- `icon` - FontAwesome icon class
- `order` - Sort order in navigation

### Theme Architecture

This site uses the **gem-based Chirpy theme**. Key behaviors:
- Theme files are installed as a gem, not committed to the repository
- Only customized/overridden files appear in the repo
- Locate theme files: `bundle info --path jekyll-theme-chirpy`
- Theme layouts, includes, and assets are inherited from the gem

### Configuration Highlights

**Language & Localization:**
- Primary language: Chinese (`lang: zh`)
- Timezone: `Asia/Shanghai`
- Localization strings in `_data/locales/` (from theme gem)

**Collections:**
- `tabs` collection with custom output and ordering

**Defaults:**
- Posts use `post` layout with comments and TOC enabled by default
- Permalinks: `/posts/:title/`
- Drafts have comments disabled

**Kramdown Settings:**
- Rouge syntax highlighter with line numbers enabled for code blocks
- Custom footnote backlink symbol

### Deployment

**GitHub Actions Workflow** (`.github/workflows/jekyll.yml`):
1. Triggers on push to `main` branch
2. Sets up Ruby 3.1 and runs `bundle install`
3. Builds site with `bundle exec jekyll build` in production mode
4. Uploads artifact to GitHub Pages
5. Deploys to github-pages environment

**Important:** The site builds automatically on push to main. No manual deployment needed.

### Working with Posts

**lastmod Behavior:**
The `posts-lastmod-hook.rb` plugin automatically adds `last_modified_at` to posts that have multiple git commits. This happens during Jekyll build, so:
- First commit: No lastmod field
- Subsequent commits: lastmod field shows last commit date

**Images:**
Images can be referenced relatively in posts. Root-level images (like `OIP-C.jpg`, `process_state.png`, `thread_info_in_stack.png`) are used directly. For post-specific images, use standard markdown syntax with paths relative to site root.

**Future Posts:**
`future: true` is enabled in config, so future-dated posts will be published immediately.

### Theme Customization

To override theme defaults:
1. Locate the file: `bundle info --path jekyll-theme-chirpy`
2. Copy the file to your repo in the same relative path
3. Make modifications

Common override locations:
- `_layouts/` - Page layouts
- `_includes/` - Reusable components
- `_sass/` - Styles
- `assets/` - Scripts and media

### Important Notes

- Do not modify files in `_site/` - they are auto-generated
- The theme uses Sass with compressed output in production
- HTML compression is enabled in production (disabled in development)
- The `tools/` directory is excluded from Jekyll build
