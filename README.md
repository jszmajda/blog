# Mens Sana

My personal blog, available at [loki.ws](http://loki.ws)

## Setup

Built with Jekyll and Ruby 2.7.8.

**Note:** Ruby 3.x is not currently supported due to Jekyll 4.x's dependency on eventmachine, which doesn't compile on newer Ruby versions (3.4+). Stick with Ruby 2.7.8 until Jekyll 5.0 is released with better Ruby 3.x support.

### Requirements
- Ruby 2.7.8 (managed via RVM) - **Do not use Ruby 3.x**
- Bundler

### Installation
```bash
bundle install
```

### Development
```bash
bundle exec jekyll serve
```

### Build & Deploy
```bash
rake build    # Build the site
rake deploy   # Deploy to loki.ws
rake          # Build and deploy
```
