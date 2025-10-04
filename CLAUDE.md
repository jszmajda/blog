# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based personal blog called "Mens Sana" hosted at loki.ws. The blog contains two main content types: technical articles (code/) and personal essays (essays/), both using Markdown with Jekyll frontmatter.

## Ruby Version

**Current**: Ruby 2.7.8 (required for Jekyll 4.x compatibility)
**Issue**: Jekyll 4.x uses eventmachine which doesn't compile on Ruby 3.4.4 due to C++ compilation issues on macOS
**Future**: Wait for Jekyll 5.0 which will have better Ruby 3.x support, or use Ruby 3.1.x as an intermediate step

### RVM Commands
- `rvm use 2.7.8` - Switch to Ruby 2.7.8
- `rvm list` - List available Ruby versions

## Development Commands

### Build and Deploy
- `bundle exec jekyll build` - Build the site to _site/ directory
- `bundle exec jekyll serve` - Serve locally for development (uses eventmachine on Ruby 2.7.x)
- `rake build` - Build using Rake task
- `rake deploy` - Deploy to loki.ws via rsync (requires SSH access)
- `rake` - Build and deploy (default task)

### Dependencies
- `bundle install` - Install Ruby gems (ensure using Ruby 2.7.8)
- `bundle update` - Update gems

## Content Structure

### Posts Organization
- **Technical posts**: `code/_posts/YYYY-MM-DD-title.md`
- **Essays**: `essays/_posts/YYYY-MM-DD-title.md`
- All posts use Jekyll frontmatter with `layout: post` and `title:`

### Layouts
- `_layouts/default.html` - Main site template with navigation, social meta tags, and Google Analytics
- `_layouts/post.html` - Post template with Disqus comments
- `_layouts/none.html` - Minimal layout

### Static Assets
- `css/` - Sass files (main.sass, main_old.sass) and syntax highlighting CSS
- `img/` - Images and media files
- Generated CSS and HTML output goes to `_site/`

## Site Configuration

The site uses Jekyll with:
- Rouge syntax highlighter
- LSI (Latent Semantic Indexing) enabled
- Date-based permalinks
- UTF-8 encoding
- Disqus comments on posts
- Social media meta tags (Twitter Cards, Open Graph)
- Google Analytics tracking

## Content Guidelines

Posts should follow the existing frontmatter pattern:
```yaml
---
layout: post
title: Your Title Here
---
```

The site automatically generates an archive of all posts in the sidebar navigation.